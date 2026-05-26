# 🔄 AI Branch Sync Workflow

[English](./README.md) | [中文](./README.zh-CN.md)

> AI-assisted safe branch synchronization workflow for long-lived multi-platform codebases.

This project provides a practical workflow for synchronizing business logic changes between branches while preserving platform-specific implementations.

---

# 📌 Problem Statement

Traditional `git merge` and `cherry-pick` workflows often fail in:

* Multi-platform projects
* OEM customizations
* SaaS / private deployment forks
* Region-specific branches
* Long-lived maintenance branches

Typical problems include:

* Platform-specific logic being overwritten
* Historical diffs being incorrectly merged
* Public APIs accidentally modified
* AI-generated merges introducing regressions

This workflow solves those issues by introducing:

* Semantic diff analysis
* Platform-aware filtering
* Human-in-the-loop review
* Safe patch generation

---

# 🎯 Core Philosophy

```text
AI only proposes patches.
Human reviewers make the final merge decision.
```

This is NOT an auto-merge system.

It is a:

> Human-reviewed AI-assisted synchronization workflow.

---

# 🏗️ Architecture

```text
develop branch
    ↓
diff extraction
    ↓
AI semantic analysis
    ↓
platform filtering
    ↓
safe patch generation
    ↓
human review
    ↓
compile & test
    ↓
audit
    ↓
merge
```

---

# ✅ Use Cases

This workflow is especially useful for:

* Android / iOS shared business logic
* SaaS + private deployment branches
* OEM customer forks
* Regional product variants
* Multi-tenant systems
* Long-lived release branches
* Enterprise customization branches

---

# ❌ When NOT to Use This

Do NOT use this workflow for:

* Short-lived feature branches
* Standard GitFlow workflows
* Projects without platform differences
* Simple repositories where normal merge works

In those cases:

* `git merge`
* `rebase`
* `cherry-pick`

are usually enough.

---

# 📂 Recommended Repository Structure

```text
ai-branch-sync-workflow/
│
├── README.md
├── README.zh-CN.md
│
├── PROMPTS/
│   ├── safe-file-sync.md
│   ├── audit-method.md
│   └── fix-test-errors.md
│
├── PRINCIPLES/
│   └── AI_SYNC_PRINCIPLES.md
│
├── examples/
│   ├── before/
│   ├── after/
│   ├── prompts/
│   └── patches/
│
├── templates/
│   ├── sync-analysis.md
│   ├── patch-output.md
│   └── audit-output.md
│
└── docs/
    ├── architecture.md
    ├── failure-cases.md
    └── best-practices.md
```

---

# 🚀 Standard Workflow

---

# Step 1: Identify the Sync Scope

## Goal

Extract only the relevant business changes from the source branch.

Avoid syncing:

* Historical diffs
* Unrelated formatting changes
* Legacy branch inconsistencies

---

## Prompt Example

```text
Please summarize all commits in develop-xx between:
2026.4.22 15:00 and 2026.5.6 16:14.

List all modified files (deduplicated)
and provide a short summary for each file.
```

---

# Step 2: Establish Protection Rules

## Goal

Teach the AI what MUST NOT be modified.

This defines the synchronization boundary.

---

## Prompt Example

```text
This is the modification list from develop-xx.
I now need to synchronize these changes into fusion-xx.

Please analyze the overall differences first.
After that, I will ask you to sync files one by one.

Rules:
1. fusion-xx contains platform-specific logic that MUST be preserved.
2. Do not sync unrelated historical differences.
3. Process files one at a time.
```

---

# Step 3: Safe File Synchronization

## Goal

Synchronize business logic while preserving target branch customizations.

---

## Prompt Example

```text
Please synchronize file:
ReportOfPassRateServiceImpl.java

Requirements:
1. Compare the develop-xx and fusion-xx versions.
2. Identify business logic changes introduced in develop-xx.
3. Merge those changes into fusion-xx while preserving all fusion-specific logic.
```

---

# Step 4: Fix Test Failures

## Goal

Repair test code affected by API or signature changes.

---

## Common Issues

* Method signature changes
* Missing mocks
* Constructor parameter changes
* New dependency injection requirements
* Missing mock return values

---

## Prompt Example

```text
Please fix all test compilation errors caused by the synchronized changes.

Common fixes:
- Parameter type changes
- Missing @Mock dependencies
- Missing mock return values
- Constructor parameter updates
```

---

# Step 5: Compile & Test

## Goal

Create a fast feedback loop.

Detect:

* Missing imports
* API mismatches
* Bean injection failures
* Incomplete synchronization
* Mocking issues

---

## Recommended Commands

```bash
mvn test
```

or:

```bash
gradle test
```

---

# Step 6: Method-Level Audit

## Goal

Prevent AI hallucinations and hidden regressions.

This is the MOST IMPORTANT step.

---

## Recommended Targets

Focus on:

* ServiceImpl classes
* Validators
* Rule engines
* Strategy classes
* Complex conditional logic
* Core business aggregation methods

---

## Prompt Example

```text
Compare the method methodName in XXXServiceImpl
between develop-xx and fusion-xx.

Perform a line-by-line diff and confirm that
business logic is identical after excluding
platform-specific code.
```

---

# Step 7: Configuration Consistency Validation

## Goal

Ensure constants, enums, and configurations remain consistent.

Prevent:

* Enum mismatches
* Feature flag regressions
* Configuration drift
* Constant definition conflicts

---

## Prompt Example

```text
Please verify that all modified configuration classes,
constants, and enum definitions remain consistent
between both branches.
```

---

# 🛡️ Safe Sync Rules

These rules should ALWAYS be given to the AI.

```text
1. NEVER modify platform-specific code.
2. NEVER overwrite fusion-only logic.
3. NEVER sync unrelated formatting changes.
4. NEVER change public APIs unless explicitly required.
5. Only output patch + rationale.
6. If uncertain, output:
⚠️ MANUAL REVIEW REQUIRED
```

---

# 📑 Standard AI Output Format

Recommended structure:

```markdown
## 1. Change Analysis

## 2. Proposed Patch

## 3. Rationale & Confidence

## 4. Risk Flags
```

Benefits:

* Stable output structure
* Easier code review
* Better auditability
* Easier future automation

---

# ❌ Common AI Failure Cases

---

## Case 1: Overwriting Platform Logic

### Problem

AI replaces the entire target file.

### Risk

* Platform behavior loss
* Production regressions
* Compilation failures

### Prevention

* Mark platform-specific sections
* Only allow patch output
* Never allow whole-file replacement

---

## Case 2: Syncing Historical Diffs

### Problem

AI mistakes old differences for new business changes.

### Prevention

Always restrict synchronization to:

* Explicit commit ranges
* Explicit time windows

---

## Case 3: Modifying Public APIs

### Problem

AI changes:

* Method signatures
* DTO structures
* Return types

### Prevention

```text
NEVER change public APIs.
```

---

## Case 4: Incorrect Conditional Merge

### Problem

AI breaks:

* if/else branches
* feature flags
* fallback logic

### Prevention

Always perform:

* Method audits
* Line-by-line comparison
* Human review

---

# 🧪 Example Scenario

## develop branch

```java
public BigDecimal calculateTotal(Order order) {
    return order.getAmount().multiply(new BigDecimal("1.13"));
}
```

Business change:

* Added tax calculation

---

## fusion branch

```java
public BigDecimal calculateTotal(Order order) {
    // PLATFORM: ANDROID
    if (isAndroidChannel(order)) {
        return androidCalculator.calculate(order);
    }

    return order.getAmount();
}
```

Platform-specific logic exists.

---

## Expected AI Patch

```diff
 public BigDecimal calculateTotal(Order order) {
     // PLATFORM: ANDROID
     if (isAndroidChannel(order)) {
         return androidCalculator.calculate(order);
     }

-    return order.getAmount();
+    return order.getAmount().multiply(new BigDecimal("1.13"));
 }
```

---

# 📌 Key Principles

## 1. Sync Small Chunks

Recommended:

* 1~3 files at a time
* Under 300 lines of core logic per request

Large prompts dramatically increase AI mistakes.

---

## 2. Mark Platform Code Explicitly

Recommended markers:

```java
// PLATFORM: ANDROID
```

or:

```java
@PlatformSpecific
```

Without markers, AI cannot reliably distinguish platform boundaries.

---

## 3. AI Is an Assistant, Not an Authority

AI responsibilities:

* Extract semantic diffs
* Generate safe patches
* Identify risks
* Assist auditing

Human responsibilities:

* Understand business intent
* Validate platform boundaries
* Make merge decisions
* Control production risk

---

# 📄 License

MIT
