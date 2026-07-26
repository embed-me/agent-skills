---
name: planning-and-task-breakdown
description: >-
  Decomposes a spec into small, ordered, independently verifiable tasks with acceptance criteria,
  dependencies, and file scope — the form a subagent can execute without further context. Use
  after a spec exists, or whenever work is too large to start. Language and framework agnostic.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  adapted-from: https://github.com/addyosmani/agent-skills
  phase: plan
---

# Planning and Task Breakdown

Convert a spec into a queue of tasks small enough to finish, verify, and commit one at a time.

A good plan is one where a fresh agent with no memory of this conversation can pick up any
unblocked task and do it correctly. That constraint is what makes plans useful for delegation —
and it is a much higher bar than a plan that only makes sense to its author.

---

## When to use

- A spec exists and work needs to start
- A task feels too big to begin
- Multiple people or agents will work in parallel
- You are about to write code touching more than two or three files

---

## Process

### 1. Read only

Plan mode is read-only. Do not edit source while planning. The temptation to "just start on the
obvious part" is how plans end up describing work that already happened.

### 2. Map dependencies

List the components the spec touches, then draw the edges:
Anything with no inbound edge can start immediately. Anything with two or more inbound edges is
an integration point — plan a checkpoint there.

### 3. Slice vertically

**Vertical** — one thin, complete, working path through the whole stack:

A user can create a draft with a title only. Schema field + endpoint + minimal UI +
one test. Nothing else works yet, but this path works end to end.

**Horizontal** — one layer for everything at once:

Prefer vertical. Every vertical slice is demonstrable and independently verifiable. Horizontal
slices are individually unverifiable and only reveal integration problems at the very end, when
they are most expensive.

Slice horizontally only where a shared contract genuinely must exist first — a schema, an
interface, a protocol. When you do, that layer is its own task and it lands with tests.

### 4. Write tasks

```markdown
## Task N: <imperative title>

**Depends on:** <task numbers, or "none">
**Files:** <expected paths — a scope hint, not a contract>
**Estimated size:** S | M | L

### Goal
One sentence: what is true when this is done.

### Acceptance criteria
- [ ] <objectively checkable>
- [ ] Test exists that fails without this change

### Verification
Exact commands to run and expected result. Pull them from AGENTS.md — do not invent them.

### Notes
Patterns to follow, gotchas, things explicitly not to touch.
```

### 5. Size honestly

| Size | Shape | Action |
|---|---|---|
| **S** | One file, one behavior, one test | Good |
| **M** | A few files, one coherent slice | Good — the target |
| **L** | Many files, several behaviors | Split it |
| **XL** | "Implement the feature" | Not a task. Go back to step 3. |

If a task cannot be described without the word "and", it is probably two tasks.

If you cannot write its acceptance criteria, it is not ready — the spec is unclear, and that is
a finding to report, not a gap to paper over.

### 6. Order and checkpoint

Order by dependency, then by risk: **do the risky, uncertain, or foundational work first**.
Discovering a wrong assumption in task 2 costs one task; discovering it in task 12 costs twelve.

Insert a checkpoint after each coherent phase:

```markdown
### Checkpoint: <phase name>
- [ ] Tasks N–M complete and reviewed
- [ ] Full suite green, build clean
- [ ] Demonstrable: <what a human can now see working>
- [ ] Human sign-off before the next phase
```

### 7. Note parallelism

Explicitly mark which tasks can run concurrently. Two tasks are safe to parallelize only if they
have **no dependency edge and no overlapping files**. Concurrent edits to the same file produce
merge conflicts that cost more than the time saved.

### 8. Persist and present

Write `tasks/plan.local.md` (the full plan) and `tasks/todo.local.md` (the live checklist). Files, not chat
history — so the work survives compaction, a crash, or tomorrow.

Present the plan for approval before any implementation begins. This is the cheapest moment in
the whole project to change direction.

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "I'll figure out the order as I go." | You will discover the foundational task after building on top of a wrong guess. |
| "Splitting this is overhead." | The overhead is a heading. The alternative is an unreviewable 40-file diff. |
| "It's all one feature, so it's one task." | Features ship in slices. Tasks are slices. |
| "I'll write acceptance criteria later." | Later is when you have already decided, and criteria become a description of what you built. |

---

## Red flags

- A task without acceptance criteria
- A task described as "implement X" with no further detail
- No checkpoints in a plan with more than five tasks
- Every task depends on every other task (the slicing is horizontal)
- The plan lives only in the conversation
- Parallel tasks that share files

---

## Verification

- [ ] Every task has checkable acceptance criteria
- [ ] Every task names its dependencies
- [ ] Tasks are S or M; no L or XL remain
- [ ] Verification commands come from AGENTS.md
- [ ] Checkpoints exist between phases
- [ ] `tasks/plan.local.md` and `tasks/todo.local.md` are written
- [ ] The human approved the plan
