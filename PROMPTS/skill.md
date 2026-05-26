---
name: branch-sync
description: AI-assisted cross-branch code synchronization. Trigger whenever the user wants to sync code changes between two branches (e.g. develop-xx to fusion-xx), merge specific commits across branches, or replicate business logic changes from one platform branch to another. Also use when the user mentions "branch sync", "cross-branch", "sync to fusion/develop", "merge from branch", or needs to port changes between platform variants while preserving branch-specific differences.
---

# Cross-Branch Code Synchronization

Sync business code changes from a source branch to a target branch that has platform-specific differences. The target branch must receive all business changes while preserving its own unique code.

## When to Use

- Two branches share the same codebase but have platform/environment differences
- Source branch has new commits that need to be replicated to target
- Target branch has code that must NOT be overwritten during sync
- User wants controlled, file-by-file synchronization, not a blind git merge

## Prerequisites

Before starting, confirm with the user:

1. **Source branch name** (e.g. `develop-xx`) — where the changes originated
2. **Target branch name** (e.g. `fusion-xx`) — where changes need to go
3. **Commit time range** on source branch — what's in scope for this sync
4. **Known branch differences** — what the target branch has that's unique and must be preserved

## Workflow

Execute these steps in order. Do not skip steps or combine them.

### Step 1: Lock the commit range on the source branch

Switch to the source branch and extract the diff for the specified time range.

```bash
git checkout <source-branch>
```

Identify commits in the time range:

```bash
git log --oneline --after="<start-time>" --before="<end-time>"
```

Get the unique list of changed files:

```bash
git diff --name-only <first-commit>^..<last-commit>
```

Present the user with a clean checklist:

```
Files to sync (N files):
1. path/to/FileA.java — brief description of what changed
2. path/to/FileB.java — brief description of what changed
...
```

**Why this matters**: Without a locked commit range, AI will confuse historical differences between branches with the changes that actually need syncing.

### Step 2: Switch to target branch and analyze differences

```bash
git checkout <target-branch>
```

For each file in the checklist, compare source vs target:

```bash
git diff <source-branch>..<target-branch> -- path/to/FileA.java
```

Build a **difference report** for each file, categorizing diffs as:
- **Business changes to sync** — new logic, bug fixes, feature additions from the commit range
- **Target-only code to preserve** — platform-specific imports, different API calls, unique conditional logic
- **Unrelated differences** — formatting, comments, pre-existing differences outside the commit range

Present the report to the user for confirmation before making any changes.

### Step 3: Sync files one by one

Process **one file at a time**. For each file:

1. Read the current target branch version
2. Read the source branch version
3. Identify the specific business changes from the locked commit range (not all differences)
4. Apply only those business changes to the target file
5. Verify target-only code is still intact

After each file, briefly report what was synced and what was preserved.

**Critical rules**:
- Never overwrite target-only code
- Never sync unrelated differences that existed before the commit range
- If unsure whether a diff is business or platform-specific, ask the user
- Do not sync all diffs blindly — understand the intent behind each change

### Step 4: Sync test files

Business code signature changes (method parameters, new dependencies, constructor changes) will break test files. After syncing all business code:

1. Identify test files that reference synced classes
2. Fix compilation errors:
   - Method parameter type changes (e.g. `List<Long>` to `Long`)
   - New dependencies requiring `@Mock` annotations
   - New method calls needing mock return values
   - Constructor parameter changes
3. Run compilation to verify

### Step 5: Compile and verify

```bash
mvn clean compile -DskipTests
```

If compilation fails, fix errors immediately — they indicate missed syncs or incompatible changes. Repeat until clean compile succeeds.

### Step 6: Method-level verification

For critical business logic, do a line-by-line comparison between branches:

```
Compare <source-branch> and <target-branch> for:
- ClassName.methodName — verify logic is identical
```

Read the method from both branches and diff them. Report any discrepancies. This catches cases where AI thinks it synced correctly but missed something.

Focus on methods that:
- Were explicitly modified in the commit range
- Contain complex logic (conditionals, loops, calculations)
- Have platform-adjacent code (where it's easy to accidentally overwrite target-specific code)

### Step 7: Configuration consistency check

Verify that synced changes to configuration classes, constants, enums, and properties are consistent:

1. Check `application.yml` / `application.properties` for new/changed config keys
2. Check constant/enum classes for new values
3. Check `TableName`, `ColumnName` type classes for new entries
4. Verify config values make sense for the target platform environment

## Important Notes

| Concern | Guidance |
|---------|----------|
| Lock commit range first | Always determine the exact commit range before doing anything else. Without this, historical differences get treated as sync targets. |
| One file at a time | Never try to sync all files at once. File-by-file processing catches issues early and makes rollback easy. |
| Define diff rules upfront | Know which code is target-only before starting. If the user hasn't told you, ask. |
| Understand, don't copy | AI should understand what the change does and replicate the intent, not mechanically copy-paste diffs. |
| Don't forget tests | If business signatures changed, tests must follow. Compilation will catch most issues. |
| Verify at method level | AI can miss things it thinks it synced. Method-by-method comparison is the safety net. |
| Environment vs code | If an API returns empty data after sync, check whether it's an environment/config issue (different database, different service URLs) before touching code again. |

## Error Handling

- **Merge conflict detected**: Do not force-resolve. Show the user the conflict and ask which version to keep.
- **Target file doesn't exist**: The file is new in source. Copy it over, but check if it needs platform-specific adjustments (imports, API calls, config references).
- **Source file was deleted**: Confirm with the user before deleting on target — the target may still need it for platform-specific reasons.
- **Compilation fails after sync**: Read the error, identify the root cause, fix only the sync-related issue. Do not refactor surrounding code.
