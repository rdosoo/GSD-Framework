# Phase 2: Content Extraction

Content extracted from the Get Shit Done repository, organised by logical theme.

---

## 1. Core Philosophy & Motivation

### The Problem
- "Vibecoding" produces inconsistent results that fall apart at scale
- Claude Code is powerful but needs proper context engineering to be reliable
- Existing spec-driven tools (BMAD, Speckit) add enterprise overhead unsuitable for solo developers
- **Context rot**: quality degrades as Claude fills its context window

### The Solution
- GSD is a meta-prompting, context engineering, and spec-driven development system
- Complexity lives in the system, not in the user's workflow
- Behind the scenes: context engineering, XML prompt formatting, subagent orchestration, state management
- What the user sees: a few commands that just work
- Designed for solo developer + Claude Code workflow (no enterprise patterns)

### Design Principles
- Plans are prompts — executable instructions, not documents to interpret
- Imperative, brief, technical language — no filler, no sycophancy
- Context size as quality constraint — split aggressively when approaching limits
- Automation-first — Claude automates everything possible; humans verify results
- Atomic commits — every task gets its own surgical, traceable commit
- Goal-backward verification — verify deliverables match intent, not just that tasks ran

---

## 2. Context Engineering

### The Core Insight
Claude's quality follows a curve based on context window usage:
- **0-30%**: Peak quality
- **30-50%**: Good quality
- **50-70%**: Degrading quality
- **70%+**: Poor quality

### How GSD Manages Context
Each stage uses **fresh context windows** — subagents get a full 200k tokens purely for their task, with zero accumulated garbage from previous work.

### Structured Artifacts
| File | Purpose |
|------|---------|
| `PROJECT.md` | Project vision — always loaded |
| `REQUIREMENTS.md` | Scoped v1/v2 requirements with phase traceability |
| `ROADMAP.md` | Phase structure, progress tracking |
| `STATE.md` | Living memory — decisions, blockers, position across sessions |
| `CONTEXT.md` | User's implementation decisions for a phase |
| `RESEARCH.md` | Ecosystem knowledge (stack, features, architecture, pitfalls) |
| `PLAN.md` | Atomic task plans with XML structure and verification steps |
| `SUMMARY.md` | What happened, what changed, committed to history |
| `VERIFICATION.md` | Goal achievement report with gap analysis |

### State Management
- STATE.md is read FIRST in every workflow and updated after every significant action
- Kept under 100 lines — a digest, not an archive
- Tracks current position, velocity metrics, recent decisions, pending todos, and session continuity
- Enables instant session restoration across context resets

---

## 3. Multi-Agent Orchestration

### Architecture Pattern
Every stage uses: **thin orchestrator spawns specialised agents, collects results, routes to next step**.

The orchestrator never does heavy lifting — it coordinates.

### Agent Roster (11 Agents)

**Research Agents:**
- **gsd-project-researcher** — Investigates domain ecosystem (stack, features, architecture, pitfalls) via Context7 → official docs → web search
- **gsd-phase-researcher** — Phase-specific research with three depth levels (Quick/Standard/Deep)
- **gsd-research-synthesizer** — Combines outputs from parallel researchers into coherent summary
- **gsd-codebase-mapper** — Analyses existing codebases for brownfield projects

**Planning Agents:**
- **gsd-planner** — Creates 2-3 atomic task plans per phase with dependency analysis and goal-backward verification
- **gsd-plan-checker** — Validates plans achieve phase goals before execution begins
- **gsd-roadmapper** — Creates phased roadmaps from requirements with success criteria

**Execution Agents:**
- **gsd-executor** — Executes PLAN.md files with per-task commits, deviation handling, and checkpoint protocols

**Verification Agents:**
- **gsd-verifier** — Checks codebase delivers what phase promised using three-level artifact verification
- **gsd-integration-checker** — Verifies cross-phase integration and end-to-end flows
- **gsd-debugger** — Systematic debugging with persistent state across context resets

### Parallel Execution
Plans are organised into waves based on dependency analysis:
- Plans within a wave execute in parallel (independent work)
- Waves execute sequentially (dependent work)
- Each executor gets a fresh 200k context window
- Main context stays at 30-40% usage

---

## 4. XML Prompt Formatting

### Task Structure
```xml
<task type="auto">
  <name>Task N: Action-oriented name</name>
  <files>src/path/file.ts, src/other/file.ts</files>
  <action>What to do, what to avoid and WHY</action>
  <verify>Command or check to prove completion</verify>
  <done>Measurable acceptance criteria</done>
</task>
```

### Task Types
- `type="auto"` — Claude executes autonomously
- `type="checkpoint:human-verify"` — User must verify (90% of checkpoints)
- `type="checkpoint:decision"` — User must choose (9% of checkpoints)

### Semantic XML Containers
Tags serve semantic purposes (e.g., `<objective>`, `<action>`, `<verification>`).
Generic tags like `<section>`, `<item>`, `<content>` are banned.
Markdown headers handle hierarchy within XML containers.

---

## 5. Core Workflow (The Build Loop)

### Step 1: Initialise Project (`/gsd:new-project`)
1. **Questions** — Adaptive questioning until the system understands the idea completely
2. **Research** — Parallel agents investigate domain ecosystem (optional but recommended)
3. **Requirements** — Extracts what's v1, v2, and out of scope
4. **Roadmap** — Creates phases mapped to requirements with success criteria

**Produces:** PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md, .planning/research/

### Step 2: Discuss Phase (`/gsd:discuss-phase N`)
- Identifies gray areas in the phase based on what's being built
- Captures user preferences through adaptive questioning (4-question rhythm)
- Different question categories by feature type (visual, API, content, organisation)
- Output locked decisions are NON-NEGOTIABLE in subsequent planning

**Produces:** {phase}-CONTEXT.md

### Step 3: Plan Phase (`/gsd:plan-phase N`)
1. **Research** — Phase-specific investigation guided by CONTEXT.md
2. **Plan** — Creates 2-3 atomic task plans with XML structure
3. **Verify** — Plan checker validates against requirements; loops until pass

Plans are small enough to execute in a fresh context window — no degradation.

**Produces:** {phase}-RESEARCH.md, {phase}-{N}-PLAN.md

### Step 4: Execute Phase (`/gsd:execute-phase N`)
1. **Wave execution** — Groups plans by dependencies, parallel within waves
2. **Fresh context per plan** — 200k tokens purely for implementation
3. **Atomic commits** — Every task gets its own commit
4. **Verification** — Checks codebase delivers what phase promised

**Produces:** {phase}-{N}-SUMMARY.md, {phase}-VERIFICATION.md

### Step 5: Verify Work (`/gsd:verify-work N`)
1. Extracts testable deliverables from the phase
2. Walks user through manual acceptance testing
3. Diagnoses failures automatically with debug agents
4. Creates verified fix plans for re-execution

**Produces:** {phase}-UAT.md, fix plans if issues found

### Step 6: Repeat → Complete → Next Milestone
- Loop discuss → plan → execute → verify per phase
- `/gsd:complete-milestone` archives and tags release
- `/gsd:new-milestone` starts next version cycle

---

## 6. Verification Pipeline

### Three-Level Artifact Verification
1. **Exists** — File/component is present
2. **Substantive** — Real implementation, not stubs (TODO detection, placeholder detection)
3. **Wired** — Connected to the rest of the system (Component→API, API→Database, etc.)

### Goal-Backward Methodology
- Derive "must-haves" from phase goals: truths, artifacts, key links
- Task completion ≠ Goal achievement — never trust SUMMARY claims blindly
- VERIFICATION.md outputs structured gap analysis if issues found
- Gaps feed back into plan-phase for closure

### Stub Detection Patterns
- TODO/FIXME comments
- Placeholder text or hardcoded values
- Empty function returns
- Mock data in production code

---

## 7. Configuration & Model Profiles

### Core Settings
| Setting | Options | Default |
|---------|---------|---------|
| `mode` | `yolo`, `interactive` | `interactive` |
| `depth` | `quick`, `standard`, `comprehensive` | `standard` |

### Model Profiles
| Profile | Planning | Execution | Verification |
|---------|----------|-----------|--------------|
| `quality` | Opus | Opus | Sonnet |
| `balanced` | Opus | Sonnet | Sonnet |
| `budget` | Sonnet | Sonnet | Haiku |

Philosophy: Opus for planning/decisions, Sonnet for execution/research, Haiku for read-only tasks.

### Workflow Agent Toggles
| Setting | Default | Purpose |
|---------|---------|---------|
| `workflow.research` | true | Domain research before planning |
| `workflow.plan_check` | true | Plan validation before execution |
| `workflow.verifier` | true | Goal verification after execution |

### Git Branching
Strategies: none (default), phase (branch per phase), milestone (branch per milestone).

---

## 8. Quick Mode

For ad-hoc tasks that don't need full planning:
- Same planner + executor agents, same quality
- Skips research, plan checker, and verifier
- Tracked separately in `.planning/quick/`
- Use for: bug fixes, small features, config changes

---

## 9. Session Management

### Pause/Resume
- `/gsd:pause-work` creates a handoff document with current state
- `/gsd:resume-work` restores context from last session
- STATE.md tracks last session timestamp, completion status, and resume path

### Brownfield Support
- `/gsd:map-codebase` analyses existing code before starting a project
- Spawns parallel agents to analyse stack, architecture, conventions, and concerns
- Then `/gsd:new-project` knows the codebase — questions focus on what's being added

---

## 10. Tools & Technologies

### Runtime Support
- **Claude Code** (primary)
- **OpenCode** (open source, free models)
- **Gemini CLI**

### Installation
```bash
npx get-shit-done-cc
```
Interactive installer prompts for runtime and location (global vs local).

### Technical Stack
- Node.js (>=16.7.0) for installer
- Zero runtime dependencies
- esbuild for hook bundling (dev dependency only)
- Git for version control integration
- NPM for distribution

### File Format
All framework files are Markdown with embedded XML — no compiled code at runtime.
The system is pure meta-prompting: structured text files that teach Claude how to work.

---

## 11. Anti-Patterns (What GSD Explicitly Avoids)

### Enterprise Patterns (Banned)
- Story points, sprint ceremonies, RACI matrices
- Human dev time estimates
- Team coordination, knowledge transfer docs
- Change management processes

### Code/Doc Anti-Patterns
- Temporal language ("Previously", "We changed X to Y")
- Generic XML tags (`<section>`, `<item>`)
- Vague tasks ("Add authentication" with no specifics)
- Sycophantic language ("Great!", "I'd love to help")
- Filler words ("Just", "Simply", "Basically")

---

## 12. Community & Distribution

- **NPM:** get-shit-done-cc (v1.11.1)
- **GitHub:** glittercowboy/get-shit-done
- **Discord:** Active community server
- **Community Ports:** gsd-opencode (OpenCode), gsd-gemini (Gemini CLI) — now natively supported
- **Users:** Engineers at Amazon, Google, Shopify, Webflow
