---
description: Turn completed PR evidence into a small, monitored guidance improvement
agent: builder
---

Load the `self-improvement` skill and follow it.

Scope and context: $ARGUMENTS — provide the completed PR or merge context, review comments,
requested changes, test failures, or follow-up observations; if empty, use the most recent
completed change and available local history.

Current state:

!`git status --porcelain`

Recent history:

!`git log --oneline -5`

Work only after the PR/change is fully complete. Inspect the evidence before editing. Separate
one-off findings from recurring process problems, make at most the smallest justified change to a
skill or `AGENTS.md`, and define a meaningful signal for monitoring subsequent work. Do not clean
up unrelated guidance or invent a general rule from a single PR.
