# Tasks

> Living document tracking active, pending, and completed tasks.

## Active Tasks

- [ ] DOC-2: Add an example "Plan then Implement" flow to `README.md` showing Orchestrator → Planner → Workhorse delegation

## Pending Tasks

- [ ] TST-1: Verify `@01_architect` answers as `moonshotai/Kimi-K2.6-2026-04-20` in chat — Priority: high
- [ ] TST-2: Verify `@02_workhorse` answers as `openai/gpt-5.4-mini` in chat — Priority: high
- [ ] TST-3: Run a real Plan → Implement cycle: ask Planner to design a new rule file, then ask Workhorse to write it — Priority: medium
- [ ] DOC-3: Add "How to override models" section to `AGENTS.md` — Priority: low

## Completed Tasks

- [x] CFG-1: Pin models in `kilo.jsonc` (Orchestrator + Planner = Kimi K2.6, Workhorse = GPT-5.4-mini) — 2026-05-28
- [x] CFG-2: Change `01_architect` mode from `all` to `subagent` — 2026-05-28
- [x] CFG-3: Delete `00_orchestrator.md` and centralize orchestrator config in `kilo.jsonc` — 2026-05-28
- [x] CFG-4: Remove `AGENTS.md` from `instructions` array to avoid duplicate loading — 2026-05-28
- [x] CFG-5: Delete `.kilo/setup-prompts/` folder, remove from `kilo.jsonc` — 2026-05-28
- [x] CFG-6: Clean up `.kilo/` dependencies (node_modules, package.json) — 2026-05-28
- [x] DOC-1: Consolidate all getting-started docs into `README.md` (merged QUICKSTART + archived kilo-framework-readme) — 2026-05-28

## Blocked Tasks

- [ ] TST-4: Record a screen capture of subagent model switching — Blocked by: need OBS / screen-recording tool setup
