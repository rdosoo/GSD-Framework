# Phase 4: Presentation Outline

**Target:** 14 slides for a 15-20 minute presentation
**Audience:** Technical leadership interested in AI-augmented productivity
**Colour scheme:** Dark navy (#1B2A4A) primary, teal (#00B4D8) accent, orange (#FF6B35) highlights, white text
**Typography:** Sans-serif throughout (Calibri Light for titles, Calibri for body)

---

## Slide 1: Title Slide

**Title:** GET SHIT DONE
**Subtitle:** A Context Engineering Framework for Reliable AI-Augmented Development
**Footer:** github.com/glittercowboy/get-shit-done | MIT License

**Visual:** Clean, bold typography. GSD logo centred. Dark background.

**Speaker Notes:**
GSD is an open-source meta-prompting and context engineering system that makes AI coding assistants — primarily Claude Code — reliable for building production software. Created by solo developer TACHES, it's trusted by engineers at Amazon, Google, Shopify, and Webflow. Today I'll walk through what it does, how it works, and why it matters.

---

## Slide 2: The Problem

**Title:** The Context Rot Problem
**Key Points:**
- AI coding assistants degrade as context fills up
- Quality curve: peak at 0-30%, poor at 70%+ context usage
- "Vibecoding" produces inconsistent results that break at scale
- Existing tools add enterprise overhead without solving the core issue

**Visual:** Line chart showing quality degradation curve (0-30% green zone, 30-50% yellow, 50-70% orange, 70%+ red). Simple and impactful.

**Speaker Notes:**
Claude Code is powerful, but it has a fundamental limitation: as it accumulates context during a session, output quality degrades predictably. At 70%+ context usage, you get inconsistent, error-prone code. Existing spec-driven tools don't address this — they add sprint ceremonies and enterprise processes that make things worse for small teams. GSD was built specifically to solve this.

---

## Slide 3: What Is GSD?

**Title:** What Is GSD?
**Key Points:**
- Meta-prompting + context engineering system for Claude Code, OpenCode, Gemini CLI
- Complexity in the system, not the workflow
- One-command install: `npx get-shit-done-cc`
- Behind the scenes: subagent orchestration, XML prompt formatting, state management
- What you see: a few commands that just work

**Visual:** Two-column comparison. Left: "What You See" (simple command list). Right: "What's Happening" (agent orchestration diagram).

**Speaker Notes:**
GSD hides significant complexity behind simple slash commands. You interact with commands like /gsd:new-project and /gsd:execute-phase. Behind the scenes, the system manages 11 specialised agents, context windows, structured artifacts, and verification pipelines. It installs in one command and works across Mac, Windows, and Linux.

---

## Slide 4: Core Philosophy

**Title:** Design Principles
**Key Points:**
- Plans are prompts — executable XML, not documents
- Context is king — fresh 200k windows per execution unit
- Goal-backward verification — check outcomes, not task completion
- Automation-first — Claude does; humans verify
- No enterprise theatre — built for builders, not bureaucrats

**Visual:** Five icons in a row, each with principle name and one-line description. Minimal, clean.

**Speaker Notes:**
Five principles drive GSD's design. First, plans are literal prompts — XML-structured instructions Claude executes directly. Second, every execution unit gets a fresh context window, keeping quality in the peak zone. Third, verification checks that goals are met, not just that tasks ran. Fourth, Claude automates everything it can — humans make creative decisions. Fifth, no sprint ceremonies or story points — this is built for people who ship.

---

## Slide 5: The Build Loop

**Title:** The Core Workflow: Discuss → Plan → Execute → Verify
**Key Points:**
- `/gsd:new-project` — Questions, research, requirements, roadmap
- `/gsd:discuss-phase N` — Capture user preferences
- `/gsd:plan-phase N` — Research + atomic plans + validation
- `/gsd:execute-phase N` — Parallel wave execution
- `/gsd:verify-work N` — Manual acceptance testing

**Visual:** Circular workflow diagram with 4 stages (Discuss, Plan, Execute, Verify) connected by arrows. Centre shows "Per Phase". Side note: new-project feeds into the loop.

**Speaker Notes:**
The core workflow is a loop. You initialise your project once — GSD asks questions, researches the domain, extracts requirements, and builds a roadmap. Then for each phase you discuss your preferences, the system plans and validates, executors build in parallel fresh contexts, and you verify the results. If something's broken, debug agents diagnose and create fix plans — you just re-execute.

---

## Slide 6: Context Engineering

**Title:** How Context Engineering Works
**Key Points:**
- Structured artifacts serve as Claude's context and memory
- PROJECT.md (vision), ROADMAP.md (plan), STATE.md (memory)
- PLAN.md = XML-structured executable prompts
- Each artifact has size constraints mapped to quality zones
- STATE.md: living memory read first in every workflow

**Visual:** Layered stack diagram showing artifacts from PROJECT.md at top to PLAN.md at bottom. Arrows showing data flow between layers.

**Speaker Notes:**
GSD manages Claude's context through structured artifacts. PROJECT.md holds the project vision and is always loaded. ROADMAP.md tracks the phased plan. STATE.md is living memory — read first in every workflow, updated after every action. Plans are XML-structured with precise actions, file targets, and verification criteria. Each file has size constraints calibrated to Claude's quality curve.

---

## Slide 7: Multi-Agent Architecture

**Title:** 11 Agents, One Orchestrator Pattern
**Key Points:**
- Thin orchestrator spawns specialised agents, collects results
- Research: 4 parallel agents (domain, phase, synthesis, codebase mapping)
- Planning: Planner + Plan Checker (validation loop)
- Execution: Fresh 200k context per executor
- Verification: Verifier + Integration Checker + Debugger

**Visual:** Hub-and-spoke diagram. Orchestrator in centre, agent groups radiating out in four quadrants (Research, Planning, Execution, Verification).

**Speaker Notes:**
GSD uses 11 specialised agents coordinated by thin orchestrators. During research, four agents investigate the domain in parallel. The planner creates atomic task plans, and the plan checker validates them in a loop. Executors get fresh 200k-token contexts — your main session stays at 30-40% usage. After execution, the verifier confirms goals were achieved, not just that tasks completed. The orchestrator never does heavy lifting — it spawns, waits, and integrates.

---

## Slide 8: XML Prompt Formatting

**Title:** Plans as Executable Prompts
**Key Points:**
- Every task: name, files, action, verify, done
- Task types: auto, checkpoint:human-verify, checkpoint:decision
- 2-3 tasks per plan (stay in quality zone)
- Precise instructions eliminate guessing
- Verification built into every task

**Visual:** Code block showing an XML task example. Annotations pointing to key elements.

**Speaker Notes:**
Plans are structured XML optimised for Claude. Each task specifies exact files to touch, precise actions to take, a verification command, and acceptance criteria. Tasks stay small — 2-3 per plan — so each executor works in the peak quality zone. There's no ambiguity: Claude knows exactly what to build, how to verify it, and what "done" means. Checkpoints pause for human decisions where needed.

---

## Slide 9: Wave-Based Execution

**Title:** Parallel Execution with Fresh Contexts
**Key Points:**
- Plans grouped into dependency-ordered waves
- Parallel execution within waves, sequential across waves
- Each executor: fresh 200k-token context window
- Atomic git commit per task (surgical, traceable, revertable)
- Walk away, come back to completed work with clean git history

**Visual:** Wave diagram showing 3 waves. Wave 1: Plans A, B in parallel. Wave 2: Plan C (depends on A, B). Wave 3: Plan D. Each plan shows a "fresh context" icon.

**Speaker Notes:**
Execution is wave-based. The planner pre-computes dependency waves. Independent plans run in parallel — each in a fresh context with the full 200k-token budget. Every completed task gets its own atomic git commit with conventional commit format. This means you can walk away from your computer, come back, and find completed work with a clean, bisectable git history. Your main context stays light because the heavy lifting happened in subagent contexts.

---

## Slide 10: Verification Pipeline

**Title:** Goal-Backward Verification
**Key Points:**
- Three levels: Exists → Substantive → Wired
- Stub detection: TODO comments, placeholders, empty returns, hardcoded values
- Wiring checks: Component→API, API→Database, Form→Handler
- Task completion ≠ goal achievement
- Gaps feed back into plan-phase for closure

**Visual:** Pyramid/funnel with three levels. Level 1 "Exists" at base (widest). Level 2 "Substantive" in middle. Level 3 "Wired" at top (narrowest). Side arrow: "Gaps → Re-plan".

**Speaker Notes:**
Verification is goal-backward, not task-forward. Level 1 checks files exist. Level 2 confirms they contain real implementations — not stubs with TODOs or placeholder text. Level 3 verifies everything is wired: components connect to APIs, APIs connect to databases, forms connect to handlers. If gaps are found, they're structured as YAML and fed back into the planner for closure. The system never declares success just because tasks ran without errors.

---

## Slide 11: Configuration & Model Profiles

**Title:** Tunable Quality vs Cost
**Key Points:**
- Three model profiles: Quality (Opus everywhere), Balanced (smart allocation), Budget (minimal Opus)
- Execution mode: Interactive (confirm steps) or Yolo (auto-approve)
- Planning depth: Quick (1-3 plans), Standard (3-5), Comprehensive (5-10)
- Workflow agents toggleable: research, plan checker, verifier
- Git branching: none / per-phase / per-milestone

**Visual:** Table showing model profiles. Below: slider metaphor for depth and toggles for workflow agents.

**Speaker Notes:**
GSD is configurable. Model profiles let you balance quality against API cost — Quality uses Opus everywhere, Balanced puts Opus on planning and Sonnet on execution, Budget minimises Opus usage. You can control planning depth, toggle workflow agents on or off, choose execution mode, and set git branching strategy. This makes GSD practical for everything from side projects to production systems.

---

## Slide 12: Quick Mode & Session Management

**Title:** Flexibility Built In
**Key Points:**
- `/gsd:quick` — Ad-hoc tasks with GSD guarantees (atomic commits, state tracking)
- `/gsd:pause-work` / `/gsd:resume-work` — Session continuity across context resets
- `/gsd:map-codebase` — Brownfield support: analyse existing code first
- `/gsd:debug` — Systematic debugging with persistent state
- Phase management: add, insert, remove phases dynamically

**Visual:** Three panels: Quick Mode (lightning bolt icon), Session Management (pause/play icons), Brownfield (building icon). Brief description under each.

**Speaker Notes:**
Not every task needs full planning. Quick mode gives you GSD guarantees — atomic commits and state tracking — for bug fixes and small features. Session management lets you pause mid-phase and resume later with full context restoration. For existing codebases, map-codebase analyses your stack before new-project so the system already knows your patterns. You can also add, insert, or remove phases from the roadmap at any time.

---

## Slide 13: Impact & Adoption

**Title:** Why This Matters for Technical Leadership
**Key Points:**
- Predictable, verifiable output from AI coding assistants
- Clean git history with atomic commits (bisectable, revertable)
- Transparent process: every decision, research finding, and verification documented
- No vendor lock-in: Claude Code, OpenCode, Gemini CLI
- Open source (MIT), active community, fast evolution

**Visual:** Four metric cards: "Consistent Quality" (checkmark), "Full Traceability" (git icon), "Multi-Runtime" (three logos), "MIT Licensed" (open-source icon).

**Speaker Notes:**
For technical leadership, GSD provides three things. First, predictable quality — the verification pipeline ensures AI output meets goals, not just runs without errors. Second, full traceability — every decision, plan, and verification result is documented in structured files, and every code change has an atomic commit. Third, flexibility — it supports multiple AI runtimes, it's MIT licensed, and the community is growing fast. This is a practical tool for organisations exploring AI-augmented development.

---

## Slide 14: Getting Started

**Title:** Get Started in 60 Seconds
**Key Points:**
- Install: `npx get-shit-done-cc`
- Check: `/gsd:help`
- Start: `/gsd:new-project` (or `/gsd:map-codebase` first for existing code)
- Build: discuss → plan → execute → verify
- Resources: GitHub, Discord, NPM

**Visual:** Terminal mockup showing install command and first steps. QR code to GitHub repo.

**Speaker Notes:**
Getting started is one command. Install with npx, verify with /gsd:help, and start your first project. For existing codebases, run map-codebase first. Then follow the core loop: discuss, plan, execute, verify. The GitHub repo has full documentation, and the Discord community is active and helpful. GSD is MIT licensed and evolving fast — check the changelog for regular updates.
