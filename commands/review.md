---
description: Five-axis review of the current change with a merge verdict
agent: code-reviewer
subtask: true
---

Load the `code-review-and-quality` skill and follow it.

Scope: $ARGUMENTS — if empty, review the staged changes, or the working tree if nothing is
staged, or the most recent commit if the tree is clean.

Changed files:

!`git status --porcelain`

Diff summary:

!`git diff HEAD --stat`

Recent commits:

!`git log --oneline -5`

Read the acceptance criteria (`tasks/plan.local.md` or the spec) before the diff. Read the tests before
the implementation. Run the project's test, build, and lint commands from `AGENTS.md` yourself
rather than trusting a claim that they pass.

Review across all five axes and then check the change against `docs/definition-of-done.md`.

Categorize every finding as Critical, Important, or Suggestion, with `file:line` and a specific
recommended fix. Report findings; do not fix them. End with an explicit APPROVE or
REQUEST CHANGES.
