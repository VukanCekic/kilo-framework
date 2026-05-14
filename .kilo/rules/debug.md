---
description: Debugging protocol for persistent errors or incomplete fixes. Always include when debugging.
globs:
alwaysApply: true
---

# DEBUGGING PROTOCOL

> This debugging routine is for persistent errors or incomplete fixes. Use this routine only when stuck.

---

## Diagnose

1. Gather all error messages, logs, and behavioral symptoms.
2. Add relevant context from files.
3. Retrieve relevant project architecture, plan, and current working task as specified in `.kilo/rules/memory-bank/`.

---

## Debugging Rules

- Whenever you fail with any test result, always add more context using **Diagnose** and debug the issue effectively first. Only when you have complete information should you move toward a fix.
- Explain your **observations** and then give your **reasonings** to explain why this is exactly the issue and not anything else.
- If you are not sure, first get more observations by adding more diagnose context to the issue so you exactly and specifically know what is wrong. Additionally, you can seek clarification if required.
- Understand architecture using **Analyze Code** (defined in `.kilo/rules/implement.md`) relevant to the issue.
- Use step-by-step reasoning to think of all possible causes like architectural misalignment or design flaws, rather than just surface-level bugs.
- Look for similar patterns already solved elsewhere in the codebase or in `.kilo/rules/memory-bank/`. Use web search if needed.
- Present your fix using **Reasoning Presentation** for validation.
- Start modifying code to update and fix things using **Systematic Code Protocol** and **Testing** (both defined in `.kilo/rules/implement.md`).

---

## Reference Flow

When a bug is encountered:
1. Execute **Diagnose** fully.
2. Apply **Analyze Code** from `.kilo/rules/implement.md`.
3. Apply **Plan Code** from `.kilo/rules/implement.md` with a focus on root-cause analysis.
4. Apply **Make Changes** from `.kilo/rules/implement.md`.
5. Apply **Testing** from `.kilo/rules/implement.md`.
6. Verify the fix resolves the issue without introducing regressions.
7. Update `.kilo/rules/memory-bank/` with any newly discovered issues or architectural insights.
