---
description: Turn a spec into ordered, verifiable tasks with acceptance criteria
agent: orchestrator
---

Load the `planning-and-task-breakdown` skill and follow it.

Scope: $ARGUMENTS

Read the spec first — `SPEC.md` or whatever the repo uses. If no spec exists and
the work is more than a bug fix, stop and run `/spec` thinking first: load
`spec-driven-development` and produce one before planning.

Current state:

!`git status --porcelain`

Recent history:

!`git log --oneline -10`

Then:

1. Stay read-only. No source edits while planning.
2. Map the dependency graph between the components this touches.
3. Slice vertically — thin end-to-end paths, not horizontal layers.
4. Write each task with acceptance criteria, dependencies, expected files, and the exact
   verification commands from `AGENTS.md`.
5. Order by dependency, then risk first.
6. Add checkpoints between phases.
7. Mark which tasks can run in parallel (no shared files, no dependency edge).
8. Write `tasks/plan.local.md` and `tasks/todo.local.md`.

Present the plan and wait for approval before delegating anything.
