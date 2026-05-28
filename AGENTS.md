# Kilo Custom Framework

This is the entry point for the **Kilo Custom Framework** — a structured, agent-driven system for AI-assisted software development inside Kilo Code.

It enforces **systematic coding protocols**, **cross-session project memory**, and a **three-tier agent topology**: Orchestrator (Kimi K2.6) coordinates the session, delegates planning to the Planner (Kimi K2.6 via `01_architect`), and delegates execution to the Workhorse (GPT-5.4-mini via `02_workhorse`).

---

## Quick Navigation

| Document                                     | Purpose                                                                            |
| -------------------------------------------- | ---------------------------------------------------------------------------------- |
| `README.md`                                  | Full user guide, example flows, troubleshooting                                    |
| `kilo.jsonc`                                 | Primary session coordinator — delegates to Planner and Workhorse                   |
| `.kilo/agents/01_architect.md`               | Planning & review agent definition (Planner)                                       |
| `.kilo/agents/02_workhorse.md`               | Implementation & testing agent definition (Workhorse)                              |
| `.kilo/rules/memory-bank/session-summary.md` | Auto-generated: last session recap                                                 |
| `.kilo/rules/memory-bank/project-state.md`   | Living doc: module status, active workstreams, tech debt                           |
| `.kilo/rules/memory-bank/tasks.md`           | Living doc: active, pending, completed, and blocked tasks                          |

---

## Session Entry Points

Every session starts with one of three user actions. The AI must honor the semantics exactly:

| Action                      | Behavior                                                                                                       |
| --------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Continue Project Task**   | Load `.kilo/rules/memory-bank/session-summary.md` into context and resume where the previous session left off. |
| **Start New Task**          | Start a clean session. Do **not** load any previous session summary.                                           |
| **End Session & Summarize** | Generate a concise summary of the current session and overwrite `.kilo/rules/memory-bank/session-summary.md`.  |

> **Rule:** If the context begins with text explicitly marked as `PREVIOUS PROJECT SESSION SUMMARY`, you **must** use this information as the primary context for continuing work.

---

## Agents

The framework runs a **three-tier topology** — Orchestrator → Planner → Workhorse — with each tier mapped to a dedicated model.

### Orchestrator (Primary mode)

- **Model:** **Kimi K2.6** (default; override via model selector if you need a more powerful model)
- **Role:** Coordinate the session. Analyze user intent, manage context and lifecycle, and delegate work to the Planner or Workhorse.
- **When to use:** This is the default entry point. You interact with the Orchestrator for all requests.
- **Delegation decisions:**
  - Route to **Planner** for open-ended analysis, architecture design, code review, or ambiguous problems.
  - Route to **Workhorse** for concrete implementation, refactoring, testing, or debugging tasks.
  - Can execute quick memory-bank reads or lightweight edits directly instead of delegating.

### @01_architect — Planner (Kimi K2.6)

- **Model:** **Kimi K2.6**
- **Role:** Deep codebase analysis. Accepts high-level intent and produces detailed, step-by-step implementation specs, including file dependencies, expected outputs, and edge-case analysis.
- **When to use:** "Design the data model", "Review this PR", "Plan the migration", "How should we handle error states?"
- **Permissions:** Ask before editing files or running bash commands (only `git diff` is allowed without ask).
- **Session behavior:** Loads memory bank on start. Updates memory bank after architectural decisions.

### @02_workhorse — Execution Workhorse (GPT-5.4-mini)

- **Model:** **GPT-5.4-mini**
- **Role:** Receive specific technical requirements and implement them autonomously.
- **When to use:** "Implement the auth service", "Refactor this module", "Write tests", "Debug the build failure"
- **Permissions:** Full edit and bash access.
- **Session behavior:** Loads memory bank on start. Follows `.kilo/rules/implement.md` for all code changes. Follows `.kilo/rules/debug.md` for errors. Updates memory bank after completion.

---

## Rules

Rules are loaded into every session via `kilo.jsonc`. They define behavior constraints and protocols.

### `.kilo/rules/session-memory.md`

Cross-session project memory protocol. Defines:

- How summaries are saved and loaded.
- Token efficiency targets (200-500 tokens).
- Retroactive recovery from chat history.
- The three session lifecycle commands.

### `.kilo/rules/implement.md`

Systematic code implementation protocol. Defines:

- Programming principles (modularity, reuse, code preservation, design patterns).
- 6-step protocol: Analyze → Plan → Make Changes → Test → Loop → Optimize.
- Mandatory pre- and post-implementation memory-bank updates.
- Testing requirements: dependency-based testing, no-breakage assertion, separate test files.
- Relentless completion rule: do not stop until implementation is tested and verified.

### `.kilo/rules/debug.md`

Debugging protocol for persistent errors. Defines:

- **Diagnose** step: gather errors, logs, symptoms, and project context.
- Observation → Reasoning → Root-cause analysis flow.
- References `.kilo/rules/implement.md` for applying fixes.

### `.kilo/rules/formatting.md`

- 2-space indentation.

### `.kilo/rules/security_blocks.md`

Restricted files that must never be read, edited, or deleted:

- `.env`
- `credentials.json`
- `private_keys/`

---

## Memory Bank

The Memory Bank in `.kilo/rules/memory-bank/` preserves context across sessions.

| File                 | Purpose                                                              | Updated By                      |
| -------------------- | -------------------------------------------------------------------- | ------------------------------- |
| `session-summary.md` | Snapshot of last session: tasks, files changed, blockers, next steps | AI on "End Session & Summarize" |
| `project-state.md`   | Living dashboard: tech stack, module status, workstreams, tech debt  | AI after milestones             |
| `tasks.md`           | Active, pending, completed, and blocked tasks                        | AI after milestones             |

**Best practice:** Seed `project-state.md` and `tasks.md` before your first real session. It dramatically improves AI context quality.

---

## Project Conventions

### Code Style

- Use TypeScript for all new files.
- Follow functional programming patterns; avoid classes.
- Export components as default exports.

### Architecture

- Maintain strict separation of concerns.
- Ensure all AI outputs are logged and reviewed when running in assisted modes.
- Never call APIs directly from components — use a service layer.
- No business logic inside UI components; use hooks for logic separation.

### Security

- Never commit API keys or secrets.
- Always validate user input using Zod before processing.

---

## Configuration

The framework is wired together in `kilo.jsonc`:

```jsonc
{
  "$schema": "https://app.kilo.ai/config.json",
  "instructions": [
    ".kilo/rules/formatting.md",
    ".kilo/rules/security_blocks.md",
    ".kilo/rules/session-memory.md",
    ".kilo/rules/implement.md",
    ".kilo/rules/debug.md",
  ],
  "compaction": {
    "auto": true,
    "threshold_percent": 80,
    "prune": true,
    "tail_turns": 2,
  },
  "agent": {
    "orchestrator": {
      "mode": "primary",
      "permission": {
        "task": {
          "*": "deny",
          "01_architect": "allow",
          "02_workhorse": "allow",
        },
      },
    },
  },
}
```

- **Instructions:** All rules are loaded into every session context.
- **Compaction:** Context auto-prunes at 80% threshold, preserving the last 2 turns.
- **Orchestrator:** Denies all subagents by default; only `01_architect` and `02_workhorse` are allowed.
