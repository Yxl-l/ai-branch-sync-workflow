# 🤖 ai-branch-sync-workflow  
> AI-assisted, human-in-the-loop workflow for safely synchronizing business logic across divergent Git branches (e.g., `develop` → `fusion`), **without overwriting platform-specific code**.

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub Repo size](https://img.shields.io/github/repo-size/YxI-I/ai-branch-sync-workflow)](https://github.com/YxI-I/ai-branch-sync-workflow)
[![Last Commit](https://img.shields.io/github/last-commit/YxI-I/ai-branch-sync-workflow)](https://github.com/YxI-I/ai-branch-sync-workflow/commits/main)

---

## 🌟 Why This Workflow?
In complex projects with parallel branches (e.g., `main`, `develop`, `fusion`, `ios`, `android`), manual merges often:
- ❌ Break platform-specific logic  
- ❌ Introduce subtle regressions in business rules  
- ❌ Waste engineer time on repetitive diff reconciliation  

This workflow leverages **LLMs as intelligent diff assistants**, not auto-mergers — ensuring:
- ✅ Critical logic is preserved  
- ✅ Platform boundaries are respected  
- ✅ Every change is audited by a human  

> 🔑 **Core Principle**: *AI proposes, human approves.* No blind automation.

---

## 🧭 Workflow Overview

| Step | Phase | Key Action | Output |
|------|-------|------------|--------|
| 1 | 🔒 **Scope Lock** | Define time window & file scope (e.g., last 7 days, `/src/business/`) | `scope.json` |
| 2 | ⚖️ **Rule Setting** | Declare “do not touch” patterns (e.g., `*Platform*.java`, `config/`) | `rules.yaml` |
| 3 | 📄 **File-by-File Sync** | For each changed file: <br> • LLM compares `develop` vs `fusion` <br> • Generates patch + rationale | `patches/*.patch`, `rationales/*.md` |
| 4 | 🧪 **Test Fixup** | Run tests → AI suggests minimal fixes for breakages | `test-fixes/*.diff` |
| 5 | 🛠️ **Build & Test Loop** | Auto-retry build → AI refines patch until green | `build-log.md` |
| 6 | 🔍 **Logic Audit** | Engineer reviews critical methods (e.g., `calculatePrice()`) using side-by-side diff | `audit-report.md` |
| 7 | 📐 **Config Check** | Verify enums, constants, feature flags unchanged | `config-audit.txt` |

> 💡 All artifacts are versioned — you can replay or audit any sync run.

---

## 📥 Getting Started

### 1. Create your first README (you’re here!)
✅ You’ve just done this! 👏

### 2. Initialize core files
Run locally (or use GitHub’s web editor):
```bash
touch {RULES.md,GUIDE.md}.md .gitignore
echo "node_modules/" > .gitignore
