---
description: Systematic implementation protocol for code mode. Always include when writing or modifying code.
globs:
alwaysApply: true
---

# IMPLEMENTATION (CODE MODE)

## Pre-Implementation Requirements

Before every code implementation or change, ALWAYS do these 2 things:
1. Read and understand the documentation, plans, and tasks in `.kilo/rules/memory-bank/`.
2. Get required code context from `src` and other code locations.

---

## Programming Principles

- **algorithm_efficiency:** Use the most efficient algorithms and data structures.
- **modularity:** Write modular code; break complex logic into smaller atomic parts. Whenever possible, break into classes, files, directories, modules, and functions.
- **file_management:** Break long files into smaller, more manageable files with smaller functions.
- **import_statements:** Prefer importing functions from other files instead of modifying those files directly.
- **file_organization:** Organize files into directories and folders.
- **reuse:** Prefer to reuse existing code instead of writing it from scratch.
- **code_preservation:** Preserve what works. Do not modify working components without necessity.
- **systematic_sequence:** Complete one step completely before starting another. Keep a systematic sequence of functionalities.
- **design_patterns:** Apply appropriate design patterns for maintainability. Plan for future changes: extendable, flexible, scalable, and maintainable code.
- **proactive_testing:** Any functionality code should be accompanied by proper test code as specified in the Testing section.

---

## Systematic Code Protocol

### [Step 1] Analyze Code

#### Dependency Analysis
- Which components will be affected?
- What dependencies exist?
- Is this local or does it affect core logic?
- Which functionalities will be affected and how?
- What cascading effects will this change have?

#### Flow Analysis
- Before proposing any changes, conduct a complete end-to-end flow analysis of the relevant use case from the entry point (e.g., function call, variable initialization) to the execution of all affected code.
- Track the flow of data and logic throughout all components involved to understand its full scope.
- Document these dependencies thoroughly, including the specific usage of functions or logic in files specified in `.kilo/rules/memory-bank/`.

### [Step 2] Plan Code

- If needed, initiate the clarification process.
- Use step-by-step reasoning to outline a detailed plan including component dependencies and architectural considerations before coding.
- Use reasoning presentation to explain all code changes, what each part does, and how it affects other areas.

#### Structured Proposals
Provide a proposal that specifies:
1. What files, functions, or lines of code are being changed.
2. Why the change is necessary (bug fix, improvement, or new feature).
3. All directly impacted modules or files.
4. Potential side effects.
5. A detailed explanation of any tradeoffs.

### [Step 3] Make Changes

1. **Document Current State** in files specified in `.kilo/rules/memory-bank/`.
   - What is currently working?
   - What is the current error or issue?
   - Which files will be affected?

2. **Plan Single Logical Change at a Time**
   - One logical feature at a time.
   - Fully resolve this one change by accommodating appropriate changes in other parts of the code.
   - Adjust all existing dependencies and issues created by this change.
   - Architecture preservation: Ensure that all new code integrates seamlessly with the existing project structure and architecture before finalizing changes. Do not make changes that disrupt existing code organization or files.

3. **Simulation Testing**
   - Simulate user interactions and behaviors by performing dry runs, trace calls, or other appropriate methods to rigorously analyze the impact of proposed changes on both expected and edge-case scenarios.
   - Generate feedback on all potential side effects.
   - Do not propose a change unless the simulation passes and verifies that all existing functionality is preserved. If a simulation breaks, provide fixes immediately before proceeding.
   - If simulation testing passes, do the actual implementation.

### [Step 4] Perform Testing

Follow the Testing section below for all code changes.

### [Step 5] Loop Steps 1-4

- Incorporate all changes systematically, one by one.
- Verify and test them one by one.

### [Step 6] Optimize

- Optimize the implemented code after all changes are tested and verified.

---

## Reference

- Reference relevant documentation and best practices.
- Use web search if needed to refer to documentation or best practices.

---

## Testing (Always Write Tests After Implementation)

### Dependency-Based Testing
Create unit tests for any new functionality. Run all tests from the Analyze Code step to confirm that existing behavior is still as expected.

### No Breakage Assertion
After you propose a change, run the tests yourself and verify that they pass. Do not rely on the user to do this. Be certain that existing code will not be broken.

1. Write test logic in separate files from the code implementation to keep the code clean and maintainable.

### Test Plan
- Think of sufficiently exhaustive test plans for the functionalities added or updated against the requirements and desired outcomes.
- Define comprehensive test scenarios covering edge cases.
- Specify appropriate validation methods for the project's stack.
- Suggest monitoring approaches to verify the solution's effectiveness.
- Consider potential regressions and how to prevent them.

2. Write test code for ANY added critical functionality ALWAYS. For initial test generation, use dependency-based testing and no-breakage assertion. Then use the test plan to write code for extensive testing.
3. Document testing as specified in `.kilo/rules/memory-bank/`.

---

## Post-Implementation Requirements

After every code implementation or change, ALWAYS do these 2 things:
1. Update other possibly affected code in `src` and other code locations.
2. Update the documentation, plans, and tasks in `.kilo/rules/memory-bank/`.

---

## Relentless Completion Rule

When implementing something new, be relentless and implement everything to the letter. Stop only when you are done until successfully testing, not before.
