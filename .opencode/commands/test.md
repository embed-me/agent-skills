---
description: Write tests first, make them pass, verify no regressions
agent: builder
---

Load the `test-driven-development` skill and follow it.

Scope: $ARGUMENTS — if empty, cover the behavior in the current uncommitted changes.

!`git status --porcelain`

Discover the stack before writing anything: read `AGENTS.md` for the test commands, then read an
existing test and match its structure, naming, and assertion style. Do not introduce a second
test framework alongside one that already exists.

Report: tests added, what each proves, suite status, build status.
