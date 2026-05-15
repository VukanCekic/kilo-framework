# Kilo Framework Prompting Guide

How to talk to the Orchestrator so it routes your request correctly and uses the full three-tier system.

---

## The Golden Rule

**Always talk to the Orchestrator.** Never mention `@01_architect` or `@02_workhorse` in your prompt as if you are talking to them directly. The Orchestrator reads your intent and decides which agent to spawn.

---

## Load `context.md` First

Before every session or request, ensure `.kilo/setup-prompts/context.md` is loaded into the AI's context.

- **If using `kilo.jsonc`:** add `.kilo/setup-prompts/context.md` to the `instructions` array so it is injected automatically.
- **If starting manually:** paste or attach `context.md` before your first message.

`context.md` contains the Orchestrator's behavioral contract — tier topology, delegation rules, task-tool spawning instructions, and the sub-agent context bridge. Without it, the Orchestrator will attempt to do planning or implementation itself instead of routing to the correct agent.

---

## Session Starters

Use these exact phrases to control context at the beginning of a conversation.

| Phrase | What Happens |
|--------|-------------|
| `Continue Project Task` | Loads `session-summary.md` + `project-state.md` + `tasks.md` and resumes previous work. |
| `Start New Task` | Ignores all previous session memory. Starts clean. Use for unrelated work or experiments. |
| `End Session & Summarize` | Generates a checkpoint summary, saves it to `session-summary.md`, and updates project state. Use before long breaks. |

---

## Request Patterns

The Orchestrator classifies your intent and routes automatically. Use these patterns to get predictable behavior.

### 1. Plan before coding → routes to Planner (`@01_architect`)

Use when you need analysis, architecture, or a step-by-step spec before touching files.

```
I need to add a discount coupon system. Plan the architecture first.
```

```
How should we handle error states in the checkout flow?
```

```
Review the availability module for consolidation opportunities.
```

**What the Planner will do:**
- Inspect the relevant parts of the codebase.
- Identify affected files, edge cases, and tradeoffs.
- Return a structured plan (no code changes).

---

### 2. Implement or fix → routes to Workhorse (`@02_workhorse`)

Use when the plan is clear or the task is concrete.

```
Implement the coupon validation endpoint exactly as planned.
```

```
Fix the test:ci script in package.json.
```

```
Refactor the availability event processor to remove duplicated retry logic.
```

**What the Workhorse will do:**
- Read memory bank + relevant files.
- Follow the 6-step protocol (analyze → plan → change → test → loop → optimize).
- Write tests, run them, fix failures.
- Report completion.

---

### 3. Use Memory Bank as the source of truth

Reference tasks directly instead of repeating details.

```
Orchestrator, go to tasks.md and execute task #1.
```

```
Orchestrator, what is the current status of task #4?
```

```
Mark task #2 as completed and update project-state.md.
```

---

### 4. Debug a stubborn issue → routes to Workhorse with debug protocol

```
Debug and fix the double-discounting bug in the checkout flow.
```

```
The build is failing after the last merge. Diagnose and fix it.
```

**What the Workhorse will do:**
- Enter debug protocol: diagnose → observe → reason → fix.
- Run tests to verify.
- Update memory bank with any newly discovered issues.

---

## Two-Phase Workflow (Recommended for non-trivial work)

For anything larger than a one-line fix, always do **Plan → Implement**:

1. **Plan:** `How should we refactor the availability module?` → Orchestrator spawns Planner.
2. Review the plan yourself.
3. **Implement:** `Implement the availability refactor exactly as planned.` → Orchestrator spawns Workhorse.

This prevents the Workhorse from writing premature code and prevents the Planner from asking for approval on every file edit.

---

## What NOT to do

| Bad | Why it fails |
|-----|-------------|
| `@01_architect Plan this for me` | The Orchestrator should route; direct agent mentions bypass the delegation logic. |
| `Fix the auth bug AND redesign the landing page` | Mixing unrelated work dilutes session summaries and reduces context relevance. |
| `Just do it` (ambiguous) | The Orchestrator may misclassify planning work as implementation. Be explicit: "Plan first" or "Implement now." |

---

## Quick Reference

| I want to... | Say this |
|-------------|----------|
| Resume work | `Continue Project Task` |
| Start fresh | `Start New Task` |
| Checkpoint | `End Session & Summarize` |
| Analyze / design | `How should we...?` / `Plan...` |
| Build / fix | `Implement...` / `Fix...` / `Refactor...` |
| Run a tracked task | `Orchestrator, execute task #N from tasks.md` |
| Debug | `Debug and fix...` |
| Get status | `What is the current project state?` |

---

*Talk to the Orchestrator. It handles the rest.*
