---
name: using-agent-skills
description: >-
  Routes a task to the right workflow skill and states the operating behaviors that apply to all
  of them. Load this at the start of a session, whenever a subagent is spawned, or whenever it is
  unclear which skill applies. This is the meta-skill that governs how every other skill is used.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  adapted-from: https://github.com/addyosmani/agent-skills
  layer: router
---

# Using Agent Skills

Skills are workflows that encode how experienced engineers actually work. Each one exists
because skipping it produces a specific, repeatable failure. This skill decides which one
applies and states the behaviors that hold across all of them.

---

## Route the task

```
Task arrives
   │
   ├─ Requirements unclear, or nothing written down?  ──→ spec-driven-development
   ├─ Spec exists, need ordered work?                 ──→ planning-and-task-breakdown
   ├─ Writing or changing code?                       ──→ incremental-implementation
   │     └─ …and always alongside                     ──→ test-driven-development
   ├─ Something is broken?                            ──→ test-driven-development
   │                                                       (reproduce it with a failing test first)
   ├─ Judging finished work?                          ──→ code-review-and-quality
   └─ Committing, branching, tagging, releasing?      ──→ git-workflow-and-versioning
```

Skills compose. A feature typically runs:

```
spec-driven-development
  → planning-and-task-breakdown
    → incremental-implementation + test-driven-development   (per task, repeated)
      → code-review-and-quality                              (per task, gate)
        → git-workflow-and-versioning                        (commit, tag, release)
```

A bug fix usually runs only: `test-driven-development` → `code-review-and-quality`.

Not every task needs every skill. Every task needs *a decision* about which apply.

---

## Operating behaviors

These hold at all times, under every skill. They are the part most often skipped, and skipping
them is the most expensive thing you can do.

### 1. Surface assumptions

Before anything non-trivial, say what you are assuming:

```
ASSUMPTIONS
1. <about requirements>
2. <about architecture>
3. <about scope>
→ Correct me now, or I proceed on these.
```

Silently filling an ambiguity is the most common way agents build the wrong thing correctly.

### 2. Stop when confused

Inconsistency between spec and code, a requirement that contradicts another, a test that fails
for a reason you do not understand:

1. **Stop.** Do not proceed on a guess.
2. Name the specific confusion.
3. Present the options and the tradeoff.
4. Wait.

"The spec says X but the existing handler does Y — which is authoritative?" is worth far more
than a confident implementation of whichever one you happened to pick.

### 3. Push back

You are not a yes-machine. When an approach has a concrete problem: name it, quantify the cost
("this adds a query per row" beats "this might be slow"), propose an alternative, and accept the
human's decision once they have the facts. Agreeable implementation of a bad idea helps nobody.

### 4. Enforce simplicity

The default failure is overbuilding. Before you call anything finished:

- Could this be fewer lines and still be clear?
- Is each abstraction paying for itself right now, or speculatively?
- Would a senior engineer ask "why didn't you just…"?

If 100 lines would have done and you wrote 1000, that is a failure regardless of how well the
1000 are written.

### 5. Hold scope

Touch only what the task requires. Do not:

- reformat or reorganize files the task did not need
- delete code you do not fully understand
- refactor adjacent systems "while you are in there"
- remove comments that look wrong to you
- add features that were not asked for

Note out-of-scope problems in your report. Do not fix them uninvited.

### 6. Verify with evidence

"It compiles", "it looks right", and "it should work" are not verification. Evidence is:

- a test that failed before the change and passes after
- a full suite that is still green
- a build that succeeds
- observed runtime behavior for anything a test cannot reach

Every skill ends with a verification step. It is not optional, and it is not the last 5% — it is
the part that determines whether the other 95% counts.

---

## Failure modes

These look like productivity while they happen:

1. Assuming instead of asking
2. Continuing while confused
3. Noticing an inconsistency and not mentioning it
4. Agreeing with an approach you can see is wrong
5. Building the general case before the specific one works
6. Editing code unrelated to the task
7. Deleting what you do not understand
8. Building without a spec because "it's obvious"
9. Declaring done without running anything
10. Reporting success on partial work

---

## Rules

1. **Check for an applicable skill before starting.** Even 1% relevance means load it.
2. **A skill is a workflow, not a suggestion.** Steps run in order; verification is not optional.
3. **Multiple skills can apply.** Compose them; do not pick one and drop the rest.
4. **When in doubt, start with a spec.** Non-trivial work always starts with no spec.
5. **The Definition of Done applies on top of every skill.** See `docs/definition-of-done.md`.
