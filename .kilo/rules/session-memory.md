---
description: Cross-Session Project Memory — Save and load session summaries for context continuity across interrupted sessions.
globs:
alwaysApply: true
---

# Cross-Session Project Memory Protocol

## Purpose
Ensure smooth transition and context preservation when working on a single project across multiple sessions, even if sessions are interrupted (e.g., closing the editor, switching tasks, long breaks).

## Memory Bank Location
All project memory documents live in `.kilo/rules/memory-bank/`:
- `session-summary.md` — Summary of the most recent session (auto-generated).
- `project-state.md` — Current project state and module status.
- `tasks.md` — Active, pending, completed, and blocked tasks.

## Session Lifecycle

### 1. Start of Session: "Continue Project Task"
When continuing work on an existing project:
1. **Check** for the existence of `.kilo/rules/memory-bank/session-summary.md`.
2. **If found:** Read its contents and prepend it to the system context, clearly marked as:
   ```
   === PREVIOUS PROJECT SESSION SUMMARY ===
   [content]
   === END PREVIOUS PROJECT SESSION SUMMARY ===
   ```
3. **If not found:** Proceed with a clean session and inform the user that no previous summary exists.
4. If the context begins with text explicitly marked as "PREVIOUS PROJECT SESSION SUMMARY", you **MUST** use this information as the primary context for continuing work. This text describes the project's state at the end of the previous session.

### 2. During Session
- Keep mental track of files modified, decisions made, and issues encountered.
- Update `.kilo/rules/memory-bank/project-state.md` and `.kilo/rules/memory-bank/tasks.md` after completing significant milestones.

### 3. End of Session: "End Session & Summarize"
When the user indicates the session should end or be summarized:
1. **Generate** a concise yet comprehensive summary covering:
   - Main tasks worked on.
   - Important files modified or discussed.
   - Unresolved issues or problems.
   - Planned next steps.
   - Key architecture or design decisions made.
2. **Write** this summary to `.kilo/rules/memory-bank/session-summary.md`, overwriting any previous content.
3. Ensure the summary is structured using the template in that file.
4. Confirm to the user that the summary has been saved.

### 4. Start New Task (Clean Slate)
When the user explicitly wants to start a new task from scratch:
- Do **NOT** load `.kilo/rules/memory-bank/session-summary.md`.
- Proceed with only the current user input and general project context.

## Token Efficiency Rules
- Keep the session summary concise (target: 200-500 tokens).
- Focus on actionable context: what was done, what is blocked, and what comes next.
- Avoid duplicating full file contents or large code blocks in the summary.
- Use file paths and brief descriptions rather than inline code where possible.

## Retroactive Recovery
If a previous session ended without a summary (crash, accidental closure, forgotten):
1. Allow the user to request a retroactive summary from chat history.
2. Generate the summary based on the visible conversation history.
3. Save it to `.kilo/rules/memory-bank/session-summary.md` as usual.
