# Phase 3: Framework Summary

---

## 1. Executive Summary

**Get Shit Done (GSD)** is a meta-prompting and context engineering system that makes AI coding assistants (Claude Code, OpenCode, Gemini CLI) reliable for building real software. It solves "context rot" — the quality degradation that occurs as an AI fills its context window — through structured artifacts, multi-agent orchestration, and fresh-context execution. The result: solo developers describe what they want and get consistent, production-quality code without enterprise overhead.

---

## 2. Core Philosophy

- **Complexity in the system, not the workflow.** Users interact with simple commands; behind the scenes, GSD manages context engineering, XML prompt formatting, subagent orchestration, and state management.
- **Plans are prompts.** Every PLAN.md is an executable instruction set, not a document to interpret. XML-structured tasks give Claude precise actions, file targets, and verification criteria.
- **Context is king.** Claude's output quality degrades predictably with context usage (peak at 0-30%, poor at 70%+). GSD keeps each execution unit in the sweet spot by spawning fresh 200k-token subagent contexts.
- **Goal-backward verification.** Task completion does not equal goal achievement. Verification checks that the codebase delivers what was promised, not just that commands ran.
- **No enterprise theatre.** No story points, sprint ceremonies, RACI matrices, or release committees. The system is built for one person plus AI, shipping real products.
- **Automation-first.** Claude automates everything it can. Humans verify results and make creative decisions — they don't run CLI commands.

---

## 3. Key Components

### Structured Artifacts
A set of living documents that serve as Claude's context and memory:
- **PROJECT.md** — Project vision, constraints, and key decisions (always loaded)
- **REQUIREMENTS.md** — Scoped requirements with v1/v2/out-of-scope categorisation
- **ROADMAP.md** — Phased execution plan with success criteria
- **STATE.md** — Living memory across sessions: position, velocity, decisions, blockers
- **PLAN.md** — Atomic XML-structured task plans (2-3 tasks each)
- **CONTEXT.md** — User's implementation preferences for a phase
- **RESEARCH.md** — Domain investigation findings
- **VERIFICATION.md** — Goal achievement report with gap analysis

### Multi-Agent System
11 specialised agents coordinated by thin orchestrators:
- **Research:** Project Researcher, Phase Researcher, Research Synthesizer, Codebase Mapper
- **Planning:** Planner, Plan Checker, Roadmapper
- **Execution:** Executor
- **Verification:** Verifier, Integration Checker, Debugger

### Command Interface
27 slash commands covering the full lifecycle: project initialisation, phase management, execution, session management, and utilities. Commands are thin wrappers that delegate to detailed workflows.

### Configuration System
Tunable settings for execution mode (interactive/yolo), planning depth (quick/standard/comprehensive), model profiles (quality/balanced/budget), workflow agent toggles, parallelisation, and git branching strategy.

---

## 4. Workflow Overview

GSD follows a **discuss → plan → execute → verify** loop per phase:

```
/gsd:new-project
    │
    ▼
Questions → Research → Requirements → Roadmap
    │
    ▼ (for each phase)
    │
/gsd:discuss-phase N ──► CONTEXT.md (user decisions)
    │
/gsd:plan-phase N ──► RESEARCH.md + PLAN.md files
    │
/gsd:execute-phase N ──► Wave execution + VERIFICATION.md
    │
/gsd:verify-work N ──► UAT + fix plans if needed
    │
    ▼ (repeat for all phases)
    │
/gsd:complete-milestone ──► Archive + tag release
    │
/gsd:new-milestone ──► Next version cycle
```

**Key properties of execution:**
- Plans execute in dependency-ordered waves (parallel within waves, sequential across waves)
- Each executor gets a fresh 200k-token context window
- Every task produces an atomic git commit
- Verification checks goals, not just task completion

---

## 5. Tools & Integration

### Supported Runtimes
- **Claude Code** (primary target)
- **OpenCode** (open source, free models)
- **Gemini CLI**

### Installation
```bash
npx get-shit-done-cc          # Interactive installer
npx get-shit-done-cc@latest   # Update to latest
```
Works on Mac, Windows, and Linux. Global or local (per-project) installation.

### Technical Requirements
- Node.js >= 16.7.0
- Git (for atomic commit integration)
- Zero runtime dependencies — pure meta-prompting via structured Markdown/XML files

### Key Integrations
- **Git:** Atomic per-task commits with conventional commit format, branching strategies (none/phase/milestone)
- **NPM:** Distribution via npmjs.com
- **Claude API:** Model profiles control which Claude model each agent uses (Opus/Sonnet/Haiku)

---

## 6. Implementation Steps (How to Adopt GSD)

1. **Install:** Run `npx get-shit-done-cc` and choose runtime + location
2. **Map existing codebase (if brownfield):** Run `/gsd:map-codebase` to analyse current code
3. **Initialise project:** Run `/gsd:new-project` — answer questions, approve roadmap
4. **For each phase:**
   - `/gsd:discuss-phase N` — Share implementation preferences (optional but valuable)
   - `/gsd:plan-phase N` — System researches, plans, and validates
   - `/gsd:execute-phase N` — Walk away; come back to completed work
   - `/gsd:verify-work N` — Manual acceptance testing
5. **Complete milestone:** `/gsd:complete-milestone` to archive and tag
6. **Continue:** `/gsd:new-milestone` for the next version

### Quick Mode
For ad-hoc tasks: `/gsd:quick` provides GSD guarantees (atomic commits, state tracking) with a faster path — skips research, plan checker, and verifier.

---

## 7. Benefits & Outcomes

### For Solo Developers
- **Reliability:** Consistent, production-quality output from AI coding assistants
- **No context rot:** Fresh contexts per execution unit prevent quality degradation
- **Minimal overhead:** Simple commands replace enterprise processes
- **Session continuity:** Pause and resume work across sessions without losing context

### For Technical Leadership
- **Predictable quality:** Structured verification pipeline ensures goals are met
- **Clean git history:** Atomic commits make debugging and rollback straightforward
- **Configurable quality/cost tradeoff:** Model profiles balance Claude API spend
- **Transparency:** Every decision, research finding, and verification result is documented in structured artifacts

### For Teams Adopting AI-Augmented Development
- **Reproducible process:** Same commands, same workflow, consistent results
- **Scalable from solo to small team:** The system adapts without adding ceremony
- **No vendor lock-in:** Supports multiple AI runtimes (Claude Code, OpenCode, Gemini CLI)
- **Open source:** MIT licensed, active community, fast evolution
