---
description: Reduce complexity in recent changes without changing behavior
agent: builder
---

Reduce complexity while preserving behavior exactly. Load `incremental-implementation` and
`test-driven-development`; this is a refactor, and the tests are what make it safe.

Scope: $ARGUMENTS — if empty, the code changed in the working tree or the last commit.

!`git diff HEAD --stat`

1. Read the projects `AGENTS.md`.
2. Confirm a green baseline before touching anything. No baseline, no refactor.
3. Understand the target code — purpose, callers, edge cases, existing coverage. If a behavior
   has no test, add one **before** you change the code, so you can prove the behavior survived.
4. Look for:
   - deep nesting → guard clauses, early returns
   - long functions → split by responsibility
   - nested ternaries and boolean tangles → if/else or a named predicate
   - vague names → descriptive ones
   - duplicated logic → one implementation
   - abstractions with a single caller → inline them
   - dead code → remove, after confirming it is truly unreferenced
5. Apply **one** simplification at a time. Run the tests after each. Commit each separately.
6. If a test fails, revert that step and reconsider — a failing test during a refactor means you
   changed behavior, which is the one thing this task must not do.

Do not add features, change interfaces, or expand scope. The diff should be smaller than it
started, and the test results identical.
