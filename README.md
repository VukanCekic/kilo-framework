# Kilo Custom Framework

A structured, agent-driven system for AI-assisted software development that preserves project context across sessions and enforces systematic coding protocols.

---

## How Sessions Start — The Framework Auto-Loads

**You do not need to explain this framework to the AI every time.**

When you open a Kilo Code chat in this project, the following is automatically injected into the AI's context before you type a single message:

1. **`kilo.jsonc`** — Defines the three agents, their pinned models, and permissions.
2. **`AGENTS.md`** — Project conventions, tier topology, and delegation rules.
3. **`.kilo/rules/*.md`** — Behavioral protocols (memory bank rules, implementation steps, debug routine, formatting, security).

The Orchestrator already knows:
- It has two subagents: `@01_architect` (Planner) and `@02_workhorse` (Implementation)
- It should check `.kilo/rules/memory-bank/` before responding
- It should delegate planning to `@01_architect` and coding to `@02_workhorse`

This means you can start working immediately. Just tell it what you want.

---

## Your First Session — Exact Prompts

Pick **one** of these as your first message. Copy-paste exactly.

### Option A: Resume where you left off
```
Continue Project Task
```
The Orchestrator loads `session-summary.md`, `project-state.md`, and `tasks.md`, then tells you what was in progress last time.

### Option B: Start something brand new (ignores old context)
```
Start New Task
```
Use this for experiments or unrelated work. The Orchestrator will not load previous summaries.

### Option C: Verify subagents are pinned to correct models
```
@01_architect What model are you running as?
```
Expected: `moonshotai/Kimi-K2.6-2026-04-20`

```
@02_workhorse What model are you running as?
```
Expected: `openai/gpt-5.4-mini`

If either reports a different model, `kilo.jsonc` is not being read. Check that your workspace root contains `kilo.jsonc`.

---

## How to Use the Three Tiers

| Tier | How to invoke | Use for | Example prompt |
|------|--------------|---------|---------------|
| **Orchestrator** | Talk normally | Session management, status checks, quick Q&A | `What is the current project state?` |
| **Planner** (`@01_architect`) | `@01_architect` | Design, review, architecture, ambiguous problems | `Plan the migration from REST to GraphQL.` |
| **Workhorse** (`@02_workhorse`) | `@02_workhorse` | Code, tests, refactoring, debugging | `Implement the auth service with JWT.` |

> **Rule:** The Orchestrator handles delegation automatically, but if you want to force a specific tier, use `@mentions`.

---

## Common Prompts

### Check status
```
Read tasks.md and project-state.md and summarize current status.
```

### Plan a feature
```
I need to add a discount coupon system. Plan the architecture first — do not write code yet.
```
Orchestrator routes to `@01_architect`.

### Implement a planned feature
```
Implement the coupon system exactly as the architect planned. Start with the database schema.
```
Orchestrator routes to `@02_workhorse`.

### Run a specific tracked task
```
Go to tasks.md and execute task DOC-1.
```

### Debug
```
Debug and fix the double-discounting bug in the checkout flow.
```
Orchestrator routes to `@02_workhorse` with the debug protocol.

### Review code without editing
```
@01_architect Review src/auth/login.ts for security issues. Do not edit files.
```

---

## Session Lifecycle

| Phrase | What it does |
|--------|-------------|
| `Continue Project Task` | Loads previous session summary + memory bank. Resume work. |
| `Start New Task` | Ignores previous memory. Starts clean. |
| `End Session & Summarize` | Generates a 200-500 token summary, saves to `session-summary.md`. **Always do this before stepping away.** |

---

## Architecture

### Agents & Models

| Agent | Mode | Model | Role |
|-------|------|-------|------|
| `orchestrator` | primary | `moonshotai/Kimi-K2.6-2026-04-20` (default) | Delegates to Planner or Workhorse |
| `@01_architect` | subagent | `moonshotai/Kimi-K2.6-2026-04-20` | Deep analysis, architecture specs, reviews |
| `@02_workhorse` | subagent | `openai/gpt-5.4-mini` | Implementation, testing, debugging |

Override any model via the VS Code model selector (per-agent memory) or `kilo.jsonc` (permanent).

### Rules (loaded automatically)

- **`session-memory.md`** — How to save/load session summaries and memory bank files
- **`implement.md`** — 6-step protocol: Analyze → Plan → Change → Test → Loop → Optimize
- **`debug.md`** — Diagnostic routine for persistent errors
- **`formatting.md`** — 2-space indentation
- **`security_blocks.md`** — Restricted files (`.env`, credentials, etc.)

### Memory Bank

| File | Purpose | Updated when |
|------|---------|-------------|
| `.kilo/rules/memory-bank/session-summary.md` | Last session recap | `End Session & Summarize` |
| `.kilo/rules/memory-bank/project-state.md` | Tech stack, module status, tech debt | After milestones |
| `.kilo/rules/memory-bank/tasks.md` | Active, pending, completed, blocked tasks | After milestones |

**Best practice:** Seed `project-state.md` and `tasks.md` with your project details before the first real coding session.

---

## Example: Plan → Implement → Summarize

### Step 1: Start
```
Continue Project Task
```

### Step 2: Plan
```
I need to add a discount coupon system to the checkout flow. Plan the architecture first.
```

**AI (Orchestrator → Planner):**
- Reads `project-state.md` and `tasks.md`
- Analyzes `src/checkout/`
- Returns structured plan: new service, DB table, validation, edge cases

### Step 3: Approve
```
Looks good. Proceed.
```

### Step 4: Implement
```
@02_workhorse Implement the coupon system exactly as planned. Start with the database schema.
```

**AI (Workhorse):**
1. Analyzes affected files
2. Writes `src/checkout/coupon/CouponService.ts`
3. Adds schema changes
4. Writes tests, runs suite, fixes regressions
5. Updates `project-state.md` and `tasks.md`

### Step 5: End session
```
End Session & Summarize
```

**Result:** `session-summary.md` is overwritten with a concise recap of what was done and what comes next.

---

## Best Practices

1. **Always use `End Session & Summarize`** before long breaks. It costs virtually nothing and saves re-contextualization time.
2. **Use the right tier.** Planner for "should we?" / "how should we?", Workhorse for "do it" / "fix it".
3. **Seed the Memory Bank** before your first real session. 5 minutes filling in `project-state.md` and `tasks.md` dramatically improves context quality.
4. **Keep Memory Bank docs honest.** If you make a manual change, update `project-state.md` so the AI knows.
5. **One logical change per session.** The Workhorse is optimized for completing one feature end-to-end.
6. **Trust the systematic protocol.** The 6-step `implement.md` protocol prevents common failures: breaking tests, missing edge cases, architectural drift.

---

## Troubleshooting

### "The AI doesn't remember what we did yesterday"
- Did you run `End Session & Summarize`?
- Check if `.kilo/rules/memory-bank/session-summary.md` exists and has content.

### "The AI is loading old context"
- Summaries are overwritten, not appended. Run `End Session & Summarize` to refresh.
- For a completely fresh start, use `Start New Task`.

### "The AI tried to edit a restricted file"
- Check `.kilo/rules/security_blocks.md`. Add missing files or fix overly broad patterns.

### "Tests are failing after the AI said they passed"
- The Workhorse runs tests locally, but verify yourself (`npm test`, `pytest`, etc.).
- Common causes: environment differences, subdirectory-only test runs, state leakage.

### "I started New Task but wanted to continue"
- Tell the AI: `Load the previous session summary from .kilo/rules/memory-bank/session-summary.md and incorporate it into context.`

---

## Files Reference

| File | Human reads | AI loads | Purpose |
|------|------------|----------|---------|
| `README.md` | Yes | No | This file. Human guide only. |
| `AGENTS.md` | Optional | **Yes** (auto) | Project conventions, tier topology, rules summary. |
| `kilo.jsonc` | Optional | **Yes** | Agent definitions, models, permissions, instruction sources. |
| `PROMPTING.md` | Yes | No | Advanced prompting patterns and delegation logic details. |
| `.kilo/agents/*.md` | Optional | **Yes** | System prompts for Planner and Workhorse. |
| `.kilo/rules/*.md` | Optional | **Yes** | Behavioral protocols injected into every session. |
| `.kilo/rules/memory-bank/*.md` | Optional | **Yes** | Cross-session state (summary, project state, tasks). |

---

*Start with `Continue Project Task` or `Start New Task`. The framework handles the rest.*
