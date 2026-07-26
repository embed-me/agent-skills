---
description: Implement the plan — delegate to builders, gate each task through review
agent: orchestrator
---

Load `incremental-implementation` and `test-driven-development`.

Scope: $ARGUMENTS — if empty, take the next pending task from `tasks/todo.local.md`.

Working tree:

!`git status --porcelain`

## Modes

- `/orchestrate` — implement the next pending task, then stop.
- `/orchestrate auto` — work through every pending task without stopping between them. The
  verification loop is unchanged; only the human step between tasks is removed.

Treat `auto` or `all` as autonomous mode. Anything else is single-task mode.

## Per task

1. Read the task's acceptance criteria from `tasks/plan.local.md`.
2. Delegate to a `builder` subagent with a self-contained brief:
   GOAL / CONTEXT / FILES / CONSTRAINTS / ACCEPTANCE / VERIFY / REPORT.
3. When the builder reports done, delegate to `code-reviewer` with the same acceptance criteria.
4. REQUEST CHANGES → new builder task containing the Critical and Important findings verbatim.
   APPROVE → mark the task complete in `tasks/todo.local.md`.
5. After two failed review rounds on one task, stop and bring in the human.

## Before autonomous mode

- Require a plan. No `tasks/plan.local.md` → run `/planning` first. Do not invent requirements.
- Require a clean baseline. Uncommitted changes outside `tasks/` or spec files → stop and ask
  how to handle them. Per-task commits must not absorb unrelated local work.
- Get one unambiguous approval of the full plan. Hedged agreement is not approval. That is the
  only human gate; after it, run.

## Always stop and ask

- A test cannot be made to pass, or the build breaks with no obvious cause
- The spec is ambiguous, or a task needs a decision it does not cover
- The task is irreversible: schema migration, deletion, deploy, auth or permission change,
  payments, secrets, anything `git revert` cannot undo

Finish with a summary: tasks completed, tests added, commits made, anything skipped or flagged.
