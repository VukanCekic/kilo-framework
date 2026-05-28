# Kilo Custom Framework

A structured, agent-driven framework for AI-assisted software development that preserves project context across sessions and enforces systematic coding, testing, and debugging protocols.

**New to this framework?** Start with [`QUICKSTART.md`](./QUICKSTART.md) — exact copy-paste prompts to verify subagent models and run your first Plan → Implement session.

---

## Table of Contents

1. [What Is This?](#what-is-this)
2. [Directory Structure](#directory-structure)
3. [Core Concepts](#core-concepts)
   - [Agents](#agents)
   - [Rules](#rules)
   - [Memory Bank](#memory-bank)
4. [Getting Started](#getting-started)
5. [Tier 1: Orchestrator (Coordination)](#tier-1-orchestrator)
6. [Tier 2: Planner / Architect (Planning & Review)](#tier-2-planner)
7. [Tier 3: Workhorse (Implementation)](#tier-3-workhorse)
8. [Example Flows](#example-flows)
9. [Session Lifecycle Commands](#session-lifecycle-commands)
10. [Best Practices](#best-practices)
11. [Troubleshooting](#troubleshooting)

---

## What Is This?

This framework adds ** cross-session project memory** and **systematic development protocols** on top of Kilo Code. It solves the classic problem of losing context when you close your editor, switch tasks, or return to a project after a break.

Instead of re-explaining your codebase to the AI every session, the framework:
- Saves a concise summary of each session to a **Memory Bank**.
- Loads that summary automatically when you resume work.
- Enforces a 6-step systematic protocol for all code changes.
- Provides a dedicated debugging routine for stubborn issues.

---

## Directory Structure

```
.kilo/
├── agents/
│   ├── 01_architect.md      # Planning & review agent (Planner)
│   └── 02_workhorse.md      # Implementation agent (Workhorse)
└── rules/
    ├── formatting.md          # Code style rules
    ├── security_blocks.md     # Restricted files list
    ├── session-memory.md      # Cross-session memory protocol
    ├── implement.md           # Systematic coding protocol
    └── debug.md               # Debugging protocol
    └── memory-bank/
        ├── session-summary.md # Auto-generated: last session recap
        ├── project-state.md   # Living doc: module status, tech debt
        └── tasks.md           # Living doc: active/pending tasks

AGENTS.md                     # Project core guidelines
kilo.jsonc                    # Framework configuration
```

---

## Core Concepts

### Agents

The framework runs a **three-tier topology**, with each tier mapped to a dedicated model:

| Tier | Agent | Purpose | Permissions | Model |
|------|-------|---------|-------------|-------|
| **Orchestrator** | Primary mode | Analyzes user intent, manages session lifecycle, delegates planning to Planner and execution to Workhorse | Full edit + bash | **Kimi K2.6** (default; override via UI) |
| **Planner** | `01_architect` | Deep codebase analysis, produces step-by-step implementation specs with file dependencies and expected outputs | Ask before edit/bash | **Kimi K2.6** |
| **Workhorse** | `02_workhorse` | Receives approved plans, writes code, runs tests, verifies behavior | Full edit + bash | **GPT-5.4-mini** |

The Orchestrator routes tasks in real time:
- By default, subagent tasks are **denied** unless explicitly delegated to an allowed agent.
- The Planner and Workhorse are the only allowed subagents.

### Rules

Rules are markdown files that define behavior. They are loaded in every session via `kilo.jsonc`:

| Rule | Purpose |
|------|---------|
| `formatting.md` | Indentation, naming conventions, JSDoc requirements |
| `security_blocks.md` | Files that must never be read/edited (`.env`, `credentials.json`, etc.) |
| `session-memory.md` | How to save, load, and respect session summaries |
| `implement.md` | The 6-step systematic code protocol (analyze → plan → change → test → loop → optimize) |
| `debug.md` | Diagnostic routine for persistent errors |

### Memory Bank

The Memory Bank (`memory-bank/`) is the heart of cross-session continuity.

| File | Who Updates | Purpose |
|------|-------------|---------|
| `session-summary.md` | AI (on user request or session end) | Snapshot of the last session: what was done, what is blocked, what comes next |
| `project-state.md` | AI (after milestones) | Living project dashboard: module status, active branches, known tech debt |
| `tasks.md` | AI (after milestones) | Task tracker: active, pending, completed, blocked |

---

## Getting Started

### 1. Place the framework in your project root

Copy the entire framework directory into your project workspace:

```bash
your-project/
├── .kilo/              <-- place here
├── src/
├── package.json
└── ...
```

### 2. Initialize the Memory Bank templates

Open the three files in `.kilo/rules/memory-bank/` and fill in your project basics:

- **`project-state.md`** — Add your tech stack, module list, and current phase.
- **`tasks.md`** — Seed it with your first few tasks.
- **`session-summary.md`** — Leave as-is; it will be overwritten automatically.

### 3. Test the framework

Verify that agents spawn with their pinned models and that the delegation chain works.

In the Kilo Code chat panel:

```
@01_architect What model are you running as?
```

Expected: `moonshotai/Kimi-K2.6-2026-04-20`

```
@02_workhorse What model are you running as?
```

Expected: `openai/gpt-5.4-mini`

If an agent reports the Orchestrator's model instead, check that `kilo.jsonc` is in the project root and that the provider slug is valid.

**Override models (optional):**

The defaults are cost-optimized: Kimi K2.6 for reasoning, GPT-5.4-mini for code generation. To use a different model for a specific agent:

- **Via the VS Code model selector:** Switch to the agent, pick a new model (e.g., `openai/gpt-5.4`), and the selector remembers it per agent.
- **Via `kilo.jsonc` (permanent):**

```jsonc
{
  "agent": {
    "orchestrator": {
      "model": "openai/gpt-5.4"
    }
  }
}
```

### 4. Start your first session

You have three entry points for every session:

| Command | What Happens |
|---------|--------------|
| **Continue Project Task** | Loads `session-summary.md` into context and resumes where you left off |
| **Start New Task** | Starts a clean session without loading previous context |
| **End Session & Summarize** | Generates a summary and writes it to `session-summary.md` |

> **Tip:** Always use "Continue Project Task" when returning to ongoing work. Use "Start New Task" for exploring unrelated ideas.

---

## Tier 1: Orchestrator

**Best for:** managing the session, deciding whether a task needs planning or execution, routing to the right agent.

### What It Does
- Loads Memory Bank context and previous session summaries.
- Parses user intent and determines which tier should handle the request.
- Delegates to **Planner** for open-ended analysis, architecture design, code review, or ambiguous problems.
- Delegates to **Workhorse** for concrete implementation, refactoring, testing, or debugging tasks.
- Can perform quick reads or lightweight edits directly instead of delegating.

### When to Use
- Start any conversation — the Orchestrator is the default entry point.
- "Continue where we left off" or "Start a new task."
- Route a request explicitly: "Plan this out before we code it."

---

## Tier 2: Planner / Architect

**Best for:** planning, architecture reviews, asking questions, designing APIs, reviewing code before it ships.

### What It Does
- Analyzes your codebase structure and dependencies.
- Produces detailed, step-by-step implementation specs.
- Identifies edge cases before a single line of code is written.
- Reviews existing code for maintainability and security.

### What It Won't Do (Without Permission)
- Edit files.
- Run bash commands (except `git diff`).
- Commit changes.

### When to Use
- "Design the data model for a new feature."
- "Review this PR for security issues."
- "How should we handle error states in the checkout flow?"
- "Plan the migration from REST to GraphQL."

---

## Tier 3: Workhorse

**Best for:** writing code, running tests, refactoring, debugging, and anything that requires touching files.

### What It Does
- Receives highly specific technical requirements (usually from a Planner spec).
- Follows the full 6-step implementation protocol (see `implement.md`).
- Writes tests alongside implementation code.
- Runs tests locally and fixes failures autonomously.
- Updates Memory Bank docs after completing work.

### When to Use
- "Implement the user authentication service with JWT and refresh tokens."
- "Refactor the `PaymentGateway` class into smaller functions."
- "Write unit tests for the `calculateTotal` utility."
- "Debug why the build is failing after the last merge."

---

## Example Flows

### Flow A: New Feature (Orchestrator → Planner → Workhorse)

A complete lifecycle from idea to shipped feature, using all three tiers.

#### Step 1: Start the session

```
User: [Click "Continue Project Task"]
AI (Orchestrator):  Loaded previous session summary. Project: e-commerce API.
                    Active tasks: checkout module 60% complete.
```

#### Step 2: Orchestrator delegates to Planner

```
User: I need to add a discount coupon system to the checkout flow.

Orchestrator:
  [Reasoning: open-ended architecture request → delegate to Planner]
  Routing to @01_architect for planning.

Planner:
  1. Reads .kilo/rules/memory-bank/project-state.md
  2. Analyzes src/checkout/ dependencies
  3. Produces implementation spec:
     - New CouponService in src/checkout/coupon/
     - Database table: coupons (code, discount_type, amount, expires_at)
     - Validation logic in checkout pipeline
     - Edge cases: expired codes, stacking rules, usage limits
  4. Files to change: src/checkout/validator.ts, src/db/schema.ts
  5. Expected outputs: spec doc + file dependency list
```

#### Step 3: Approve the plan

```
User: Looks good. Proceed.
```

#### Step 4: Orchestrator delegates to Workhorse for implementation

```
User: @02_workhorse Implement the coupon system exactly as
      the Planner spec. Start with the database schema.

Workhorse:
  [Step 1] Analyze Code
  - Reads memory-bank/ for current status
  - Reads src/checkout/validator.ts and src/db/schema.ts
  - Dependency analysis: affects checkout, database, and API routes

  [Step 2] Plan Code
  - Creates a structured proposal: files, functions, lines to change
  - Identifies side effects: existing checkout tests may break

  [Step 3] Make Changes
  - Documents current state in memory-bank/
  - Adds coupon table to schema.ts
  - Creates src/checkout/coupon/CouponService.ts
  - Adds validation hook in checkout/validator.ts
  - Simulates checkout flow with and without coupons

  [Step 4] Testing
  - Writes tests in src/checkout/coupon/CouponService.test.ts
  - Runs full test suite
  - Fixes a regression in existing checkout tests

  [Step 5] Loop 1-4
  - Implements API route /api/coupons/validate
  - Adds integration tests
  - Verifies all tests pass

  [Step 6] Optimize
  - Refactors duplicate validation logic into shared utility

  Post-Implementation:
  - Updates memory-bank/project-state.md: "CouponService: stable"
  - Updates memory-bank/tasks.md: marks coupon task as completed
```

#### Step 5: End the session

```
User: [Click "End Session & Summarize"]
AI:  Summary saved to .kilo/rules/memory-bank/session-summary.md
     Key points:
     - Implemented CouponService with validation
     - Added 14 unit tests, 3 integration tests
     - All tests passing
     - Next: wire coupon UI to the API endpoint
```

---

### Flow B: Bug Fix (Workhorse Only)

A debugging session using the systematic protocol.

#### Step 1: Start the session

```
User: [Click "Continue Project Task"]
AI:  Previous session: implementing CouponService. Next step
     was wiring UI to API. User is now reporting a bug.
```

#### Step 2: Describe the bug

```
User: The checkout total is wrong when a percentage coupon
      is applied. It's double-discounting.
```

#### Step 3: Workhorse enters debug mode

```
User: @02_workhorse Debug and fix the double-discounting bug.

Workhorse:
  <DIAGNOSE>
  - Reads error logs / user description
  - Reads src/checkout/coupon/CouponService.ts
  - Reads src/checkout/validator.ts
  - Retrieves project architecture from memory-bank/

  Observation:
  - calculateTotal() applies discount, then validator.ts
    applies the same discount again before final price display.

  Reasoning:
  - This is exactly a duplication bug, not a math error.
  - The discount should be applied in one place only.

  <ANALYZE CODE>
  - Flow: checkout API → validator.ts → calculateTotal() → response
  - Both validator.ts and calculateTotal() call applyCoupon()

  <PLAN CODE>
  - Remove applyCoupon() from validator.ts
  - Keep it only in calculateTotal()
  - Update tests that relied on validator doing the math

  <MAKE CHANGES>
  - Removes duplicate discount logic
  - Adjusts validator tests
  - Runs simulation: 100 USD + 10% coupon = 90 USD (correct)

  <TESTING>
  - Runs CouponService tests
  - Runs checkout integration tests
  - All passing

  Post-Implementation:
  - Updates memory-bank/project-state.md: notes fixed tech debt
  - Updates memory-bank/tasks.md
```

#### Step 4: End the session

```
User: [Click "End Session & Summarize"]
AI:  Summary saved. Fixed double-discounting bug by removing
     duplicate coupon application in validator.ts.
     Next: resume UI wiring.
```

---

### Flow C: Clean Slate Exploration

When you want to experiment without polluting project memory.

```
User: [Click "Start New Task"]
AI:  Starting clean session. No previous summary loaded.

User: What if we rewrote the entire checkout module in Rust?
      Don't touch any files, just explore the idea.

Orchestrator: [Decides this is strategic planning → routes to Planner]

Planner: [Analyzes codebase and gives a feasibility report
         without making any changes]

User: Interesting but not actionable right now. Discard.
AI (Orchestrator):  No files modified. memory-bank/ was not touched.
```

---

## Session Lifecycle Commands

These three commands drive every session. Think of them as your navigation controls.

### `Continue Project Task`

**When:** You are returning to ongoing work.

**What the AI does:**
1. Checks `.kilo/rules/memory-bank/session-summary.md`.
2. If found, reads it and injects it at the top of context:
   ```
   === PREVIOUS PROJECT SESSION SUMMARY ===
   [content]
   === END PREVIOUS PROJECT SESSION SUMMARY ===
   ```
3. Proceeds with your first message, now fully aware of prior progress.

**If the file is missing:** The AI informs you and starts clean.

### `Start New Task`

**When:** You want to explore something unrelated, or you want a fresh context window.

**What the AI does:**
- Explicitly **does not load** `session-summary.md`.
- Starts the session with only the current user input and general project context.

### `End Session & Summarize`

**When:** You are done for the day, switching projects, or want to checkpoint progress.

**What the AI does:**
1. Reviews the current session's conversation and changes.
2. Generates a concise summary (target: 200-500 tokens) covering:
   - Main tasks worked on
   - Important files modified or discussed
   - Unresolved issues or problems
   - Planned next steps
   - Key architecture or design decisions
3. Overwrites `.kilo/rules/memory-bank/session-summary.md` with this summary.
4. Confirms completion to the user.

**Retroactive use:** If a session crashed or you forgot to summarize, open that conversation history and click the button. The AI will generate the summary from the visible history.

---

## Best Practices

### 1. Seed your Memory Bank before the first real session

Spend 5 minutes filling in `project-state.md` and `tasks.md`. It dramatically improves the quality of AI context in early sessions.

### 2. Summarize before long breaks

Always click **End Session & Summarize** when you step away for more than an hour. It costs virtually nothing and saves significant re-contextualization time later.

### 3. Use the right agent for the right job

- **Orchestrator** for session management and deciding whether to plan, execute, or both
- **Planner** for "should we?" and "how should we?"
- **Workhorse** for "do it" and "fix it"

Passing implementation work to the Planner will slow you down with approval prompts. Passing open-ended design work to the Workhorse may result in premature code without proper analysis.

### 4. Keep Memory Bank docs honest

If you make a change outside of AI assistance (e.g., manual hotfix), update `project-state.md` and `tasks.md` so the AI knows the real state next session.

### 5. One logical change per session (when possible)

The workhorse is optimized for completing one feature or fix end-to-end. Mixing unrelated work (e.g., "fix auth bug AND redesign the landing page") dilutes the session summary and reduces context relevance.

### 6. Trust the systematic protocol

The 6-step protocol in `implement.md` may feel verbose, but it prevents the most common AI coding failures: breaking existing tests, missing edge cases, and architectural drift. Do not skip steps.

---

## Troubleshooting

### "The AI doesn't remember what we did yesterday"

- Did you click **End Session & Summarize**? If not, there is no summary to load.
- Check if `.kilo/rules/memory-bank/session-summary.md` exists and has content.
- If it is empty, run the summarize command retroactively from yesterday's conversation.

### "The AI is loading old context that no longer applies"

- Summaries are overwritten, not appended. Click **End Session & Summarize** to refresh.
- For a completely fresh start, use **Start New Task**.
- If the summary is stale mid-session, tell the AI: "Ignore the previous session summary. The state has changed because ..."

### "The AI tried to edit a restricted file"

- `security_blocks.md` defines the blacklist. If a file is missing from the list but should be protected, add it there.
- If the AI is incorrectly blocked from a file you want edited, check `security_blocks.md` for overly broad patterns.

### "Tests are failing after the AI said they passed"

- The workhorse has permission to run tests locally, but verify by running them yourself:
  ```bash
  npm test
  # or
  pytest
  # etc.
  ```
- If the AI claims tests pass but they do not on your machine, the most common causes are:
  1. Environment differences (Node version, OS-specific paths).
  2. The AI ran tests in a subdirectory, not the full suite.
  3. State leakage from previous commands.

### "The session summary is too long / too short"

- The AI targets 200-500 tokens. If your project is extremely complex, manually edit `session-summary.md` after generation to highlight only the most critical next steps.
- Conversely, if the summary is too vague, prompt the AI with: "Include specific file paths and function names in the summary."

### "I accidentally started a New Task but wanted to continue"

- Just tell the AI: "Load the previous session summary from `.kilo/rules/memory-bank/session-summary.md` and incorporate it into context."
- The AI will read the file and continue as if you had used the Continue button.

---

## Summary Cheat Sheet

| I want to... | Action |
|--------------|--------|
| Resume yesterday's work | **Continue Project Task** |
| Start something unrelated | **Start New Task** |
| Checkpoint progress | **End Session & Summarize** |
| Plan before coding | Delegate to **@01_architect** (Planner) |
| Write/fix code | Delegate to **@02_workhorse** |
| Debug a stubborn issue | Delegate to **@02_workhorse** with `debug.md` protocol |
| Update project status | Ask the Orchestrator to update `memory-bank/project-state.md` |
| Track tasks | Ask the Orchestrator to update `memory-bank/tasks.md` |

---

*This framework is designed to grow with your project. Start simple, stay consistent with session summaries, and let the systematic protocols handle the heavy lifting.*
