# Quickstart Guide

Get the Kilo Custom Framework running and see the three-tier agent topology in action within 5 minutes.

---

## Prerequisites

1. The framework files are in your project root (`kilo.jsonc`, `.kilo/`, `AGENTS.md`).
2. You are using the Kilo Code extension in VS Code (or the Kilo CLI chat).
3. Your project workspace is opened at the root where `kilo.jsonc` lives.

---

## Opening a Session

Every conversation with Kilo Code starts as a session. You have three entry points. Pick one.

### Entry A: Continue where you left off

Use this when returning to ongoing work. It loads the Memory Bank automatically.

**What to click/type:**
```
Continue Project Task
```

**What the Orchestrator does:**
1. Reads `.kilo/rules/memory-bank/session-summary.md`
2. Reads `.kilo/rules/memory-bank/project-state.md`
3. Reads `.kilo/rules/memory-bank/tasks.md`
4. Summarizes the current status and waits for your next instruction.

> **Tip:** If you are starting this framework for the first time, the Memory Bank is already seeded. The Orchestrator will tell you the active tasks and ask what to do next.

---

### Entry B: Start something brand new

Use this to ignore all previous session memory and start fresh.

**What to click/type:**
```
Start New Task
```

**What the Orchestrator does:**
- Does **not** load `session-summary.md`.
- Treats your first message as the only context.

> **When to use this:** Experimenting with unrelated ideas, testing the framework in isolation, or when the previous summary is stale.

---

## Scenario 1: Verify the Framework Is Wired Correctly

The fastest way to confirm everything works is to spawn each subagent and ask it to identify its model.

### Step 1 — Verify the Planner model

Copy-paste this into the chat exactly:

```
@01_architect What model are you running as? State your provider and model ID exactly.
```

**Expected response:**
> `moonshotai/Kimi-K2.6-2026-04-20`

If it says anything else (e.g., the same model as your primary chat), something is wrong with `kilo.jsonc`. Check that the file is at the **project root** and that the `model` string is valid.

### Step 2 — Verify the Workhorse model

```
@02_workhorse What model are you running as? State your provider and model ID exactly.
```

**Expected response:**
> `openai/gpt-5.4-mini`

### What just happened?

You used `@mentions` to invoke subagents. Each subagent spawned in an **isolated session** with its own system prompt, pinned model, and permissions. The result flowed back into your main chat.

---

## Scenario 2: Complete a Real Task (Plan → Implement)

This is the core workflow. Let's execute a real task from the Memory Bank: write the `QUICKSTART.md` file.

You can do this in **one message** or **two messages**. The two-message approach is recommended because you can review the plan before spending tokens on implementation.

### Two-Message Approach (Recommended)

**Message 1 — Planning:**

```
Go to tasks.md and look at task DOC-1. Plan the architecture for that task. Do not write any files yet.
```

**What happens:**
- Orchestrator reads `tasks.md`.
- Orchestrator sees DOC-1 is a planning task.
- Orchestrator routes to `@01_architect`.
- Planner analyzes the codebase, identifies where `QUICKSTART.md` should live, what sections it needs, and what files to reference.
- Planner returns a structured plan.

**Message 2 — Implementation:**

```
Implement the QUICKSTART.md exactly as planned by the architect. Follow the Systematic Code Protocol.
```

**What happens:**
- Orchestrator routes to `@02_workhorse`.
- Workhorse reads the plan, reads the Memory Bank, writes `QUICKSTART.md`.
- Workhorse updates `tasks.md` to mark DOC-1 as completed.
- Workhorse updates `project-state.md` to reflect the Documentation module is now stable.

---

### One-Message Approach (Faster)

If the task is small and you trust the delegation:

```
Go to tasks.md, find task DOC-1, and execute it. Plan first if needed, then implement.
```

The Orchestrator will decide whether to route to Planner first or send straight to Workhorse based on task ambiguity.

---

## Scenario 3: Ask a Codebase Question

Use this when you need analysis but don't want any file changes.

```
@01_architect Review our current Memory Bank files. Are project-state.md and tasks.md well-structured? Identify any gaps.
```

**What happens:**
- Planner reads the Memory Bank files.
- Planner gives a review with suggestions (e.g., "Add a Known Issues section to project-state.md").
- **No files are edited** because Planner has `edit: ask` permission. It will not touch files unless you explicitly tell it to.

---

## Scenario 4: Debug or Refactor Something

Use this when something is broken or you want code changed.

```
@02_workhorse There is a typo in kilo.jsonc — the orchestrator model is set to the wrong provider. Find and fix it.
```

**What happens:**
- Workhorse reads `kilo.jsonc`.
- Workhorse diagnoses the issue.
- Workhorse edits the file.
- Workhorse runs a quick validation (if applicable).
- Workhorse updates Memory Bank.

---

## Scenario 5: Check Status Without Doing Work

Sometimes you just want to know where things stand.

```
What is the current project state? Read project-state.md and tasks.md and summarize.
```

**What happens:**
- Orchestrator handles this directly (no subagent needed).
- It reads both Memory Bank files.
- It gives you a concise status report.

---

## Scenario 6: End Your Session Properly

When you are done for the day, always run this so the next session can resume seamlessly.

**What to click/type:**
```
End Session & Summarize
```

**What happens:**
1. The Orchestrator reviews everything discussed in this session.
2. It generates a concise summary (200–500 tokens).
3. **Overwrites** `.kilo/rules/memory-bank/session-summary.md`.
4. Updates `project-state.md` and `tasks.md` if anything changed.

> **Rule:** Do not skip this. If you close VS Code without summarizing, the next "Continue Project Task" will have stale or missing context.

---

## Prompt Cheat Sheet

| I want to... | Exact prompt |
|--------------|-------------|
| Resume yesterday's work | `Continue Project Task` |
| Start something unrelated | `Start New Task` |
| Save checkpoint | `End Session & Summarize` |
| Plan something | `Plan [feature] before we code it.` or `@01_architect Plan [feature]` |
| Implement something | `Implement [feature] exactly as planned.` or `@02_workhorse Implement [feature]` |
| Review without edits | `@01_architect Review [file/module] for [issue]. Do not edit.` |
| Debug / fix | `@02_workhorse Debug and fix [bug description].` |
| Check status | `What is the current project state?` |
| Run a tracked task | `Go to tasks.md and execute task [ID].` |

---

## Common Mistakes

| Mistake | Why it fails |
|---------|-------------|
| `@01_architect do this now` | The Planner is read-first. It will ask for approval on every edit. Use it for planning only. |
| `@02_workhorse how should we design X?` | The Workhorse does not do open-ended design. It will write premature code. Use Planner for "how should we." |
| No `@mention` on a concrete task | The Orchestrator may handle it directly instead of spawning a specialized subagent. Be explicit if you want the Planner or Workhorse. |
| Forgetting `End Session & Summarize` | Next session has no context. You will waste time re-explaining everything. |

---

## What to Try Right Now

If you just finished reading this, here are **3 exact prompts** to paste into your chat to see the framework work:

1. **Verify models:**
   ```
   @01_architect What model are you running as?
   ```

2. **Check status:**
   ```
   Read tasks.md and tell me which task has the highest priority.
   ```

3. **Do real work (Plan → Implement):**
   ```
   Go to tasks.md, pick task TST-3, and execute it. Plan first, then implement.
   ```

---

*The framework is ready. Start with Step 1 above and iterate.*
