# Phase 1: Repository Structure Analysis

## Repository: glittercowboy/get-shit-done
**Version:** 1.11.1
**License:** MIT
**Author:** TACHES
**NPM Package:** get-shit-done-cc

---

## Directory Tree Structure

```
get-shit-done/
├── .github/
│   ├── FUNDING.yml
│   └── pull_request_template.md
├── .gitignore
├── .work/
│   └── 001-map-gsd-deps/
│       └── 001-PROMPT.md
├── agents/                              # Subagent definitions
│   ├── gsd-codebase-mapper.md
│   ├── gsd-debugger.md
│   ├── gsd-executor.md
│   ├── gsd-integration-checker.md
│   ├── gsd-phase-researcher.md
│   ├── gsd-plan-checker.md
│   ├── gsd-planner.md
│   ├── gsd-project-researcher.md
│   ├── gsd-research-synthesizer.md
│   ├── gsd-roadmapper.md
│   └── gsd-verifier.md
├── assets/
│   ├── gsd-logo-2000.png
│   ├── gsd-logo-2000.svg
│   └── terminal.svg
├── bin/
│   └── install.js                       # NPX installer entry point
├── commands/gsd/                        # Slash command definitions
│   ├── add-phase.md
│   ├── add-todo.md
│   ├── audit-milestone.md
│   ├── check-todos.md
│   ├── complete-milestone.md
│   ├── debug.md
│   ├── discuss-phase.md
│   ├── execute-phase.md
│   ├── help.md
│   ├── insert-phase.md
│   ├── join-discord.md
│   ├── list-phase-assumptions.md
│   ├── map-codebase.md
│   ├── new-milestone.md
│   ├── new-project.md
│   ├── pause-work.md
│   ├── plan-milestone-gaps.md
│   ├── plan-phase.md
│   ├── progress.md
│   ├── quick.md
│   ├── remove-phase.md
│   ├── research-phase.md
│   ├── resume-work.md
│   ├── set-profile.md
│   ├── settings.md
│   ├── update.md
│   └── verify-work.md
├── get-shit-done/                       # Core framework
│   ├── references/                      # Deep-dive reference docs
│   │   ├── checkpoints.md
│   │   ├── continuation-format.md
│   │   ├── git-integration.md
│   │   ├── model-profiles.md
│   │   ├── planning-config.md
│   │   ├── questioning.md
│   │   ├── tdd.md
│   │   ├── ui-brand.md
│   │   └── verification-patterns.md
│   ├── templates/                       # Output templates
│   │   ├── codebase/
│   │   │   ├── architecture.md
│   │   │   ├── concerns.md
│   │   │   ├── conventions.md
│   │   │   ├── integrations.md
│   │   │   ├── stack.md
│   │   │   ├── structure.md
│   │   │   └── testing.md
│   │   ├── research-project/
│   │   │   ├── ARCHITECTURE.md
│   │   │   ├── FEATURES.md
│   │   │   ├── PITFALLS.md
│   │   │   ├── STACK.md
│   │   │   └── SUMMARY.md
│   │   ├── config.json
│   │   ├── context.md
│   │   ├── continue-here.md
│   │   ├── debug-subagent-prompt.md
│   │   ├── DEBUG.md
│   │   ├── discovery.md
│   │   ├── milestone-archive.md
│   │   ├── milestone.md
│   │   ├── phase-prompt.md
│   │   ├── planner-subagent-prompt.md
│   │   ├── project.md
│   │   ├── requirements.md
│   │   ├── research.md
│   │   ├── roadmap.md
│   │   ├── state.md
│   │   ├── summary.md
│   │   ├── UAT.md
│   │   ├── user-setup.md
│   │   └── verification-report.md
│   └── workflows/                       # Detailed process logic
│       ├── complete-milestone.md
│       ├── diagnose-issues.md
│       ├── discovery-phase.md
│       ├── discuss-phase.md
│       ├── execute-phase.md
│       ├── execute-plan.md
│       ├── list-phase-assumptions.md
│       ├── map-codebase.md
│       ├── resume-project.md
│       ├── transition.md
│       ├── verify-phase.md
│       └── verify-work.md
├── hooks/
│   ├── gsd-check-update.js
│   └── gsd-statusline.js
├── scripts/
│   └── build-hooks.js
├── BUG_REPORT.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── FIXES_APPLIED.md
├── GSD-STYLE.md
├── LICENSE
├── MAINTAINERS.md
├── package-lock.json
├── package.json
└── README.md
```

---

## File Descriptions

### Root-Level Files
| File | Description |
|------|-------------|
| `README.md` | Comprehensive project documentation: philosophy, installation, workflow, commands, configuration |
| `GSD-STYLE.md` | Style guide for contributors — XML conventions, naming, language tone, anti-patterns |
| `CONTRIBUTING.md` | Contribution guidelines — branch strategy, commits, releases, PR format |
| `CHANGELOG.md` | Version history in Keep a Changelog format |
| `FIXES_APPLIED.md` | Log of applied fixes |
| `BUG_REPORT.md` | Bug report template |
| `MAINTAINERS.md` | Maintainer information |
| `LICENSE` | MIT License |
| `package.json` | NPM package configuration (v1.11.1, zero runtime dependencies) |

### Agents (`agents/`)
Subagent definitions — each file is a prompt that defines a specialized Claude agent:
| Agent | Role |
|-------|------|
| `gsd-executor.md` | Executes PLAN.md files with per-task commits and deviation handling |
| `gsd-planner.md` | Creates atomic task plans with dependency analysis |
| `gsd-verifier.md` | Verifies phase goals achieved (not just tasks completed) |
| `gsd-project-researcher.md` | Domain ecosystem research before roadmap creation |
| `gsd-phase-researcher.md` | Phase-specific research before planning |
| `gsd-plan-checker.md` | Validates plans achieve phase goals before execution |
| `gsd-codebase-mapper.md` | Analyses existing codebases for brownfield projects |
| `gsd-debugger.md` | Systematic debugging with persistent state |
| `gsd-roadmapper.md` | Creates phased roadmaps from requirements |
| `gsd-research-synthesizer.md` | Synthesises parallel research outputs |
| `gsd-integration-checker.md` | Verifies cross-phase integration and E2E flows |

### Commands (`commands/gsd/`)
Slash command definitions — thin wrappers that delegate to workflows. 27 commands covering project init, phase management, execution, session management, and utilities.

### Core Framework (`get-shit-done/`)
- **References:** Deep-dive docs on checkpoints, model profiles, verification, TDD, git integration
- **Templates:** Output format templates for all generated artifacts (PROJECT.md, ROADMAP.md, PLAN.md, etc.)
- **Workflows:** Detailed process logic for each stage of the framework

---

## Key Documentation Files
1. `README.md` — Primary entry point; full framework overview
2. `GSD-STYLE.md` — Internal style and architecture guide
3. `CONTRIBUTING.md` — Contribution and release process
4. `get-shit-done/references/checkpoints.md` — Checkpoint/automation patterns
5. `get-shit-done/references/model-profiles.md` — Model selection strategy
6. `get-shit-done/references/verification-patterns.md` — Verification methodology

---

## Main Components Assessment

1. **Meta-Prompting System** — Commands, workflows, templates, and references form a layered prompt architecture
2. **Multi-Agent Orchestration** — 11 specialised agents coordinated by thin orchestrators
3. **Context Engineering** — Deliberate management of Claude's context window via structured artifacts
4. **State Management** — STATE.md as living memory; structured file conventions for session continuity
5. **Verification Pipeline** — Goal-backward verification ensuring deliverables match intent, not just task completion
6. **Git Integration** — Atomic per-task commits with conventional commit format
