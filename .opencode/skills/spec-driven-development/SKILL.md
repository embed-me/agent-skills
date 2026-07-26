---
name: spec-driven-development
description: >-
  Turns a vague request into a written specification with explicit scope, acceptance criteria,
  and open questions before any code is written. Use when starting a new project, feature, or
  significant change, or whenever requirements are ambiguous. Language and framework agnostic.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  adapted-from: https://github.com/addyosmani/agent-skills
  phase: define
---

# Spec-Driven Development

Write down what you are building before you build it. Not a document for its own sake — a short
artifact that makes disagreement visible while it is still cheap to resolve.

The cost curve is brutal and well known: a misunderstanding caught in a spec costs a sentence.
Caught in review, it costs a rewrite. Caught in production, it costs an incident.

---

## When to use

- New project, feature, or subsystem
- A change whose blast radius you cannot describe in one sentence
- Requirements that arrived as a vague idea
- Anything where two reasonable engineers could build different things from the same request

**Skip it** for: typo fixes, dependency bumps, a bug with an obvious cause and an obvious fix.
Do not skip it because you are in a hurry — that is exactly when it pays.

---

## Process

### 1. Extract intent

Ask the questions whose answers change the design. Batch them; do not interrogate one at a time.

- What problem does this solve, for whom? What breaks today without it?
- What does success look like — specifically, in terms someone could check?
- What is explicitly **out** of scope?
- What existing behavior must not change?
- Any constraints: deadline, compatibility, performance, compliance, team convention?

If the human cannot answer something yet, record it under **Open questions** rather than
inventing an answer. An honest gap is a plan; a silent guess is a defect with a delay fuse.

### 2. Survey the ground

Before proposing anything, read enough of the codebase to know what already exists. Specs
written against an imagined architecture waste everyone's time.

- What is the existing pattern for this kind of thing?
- What can be extended rather than added?
- What will this touch that you did not expect?

### 3. Write the spec

Keep it to one page unless the work genuinely warrants more. Save it to `SPEC.md` at the repo
root.

```markdown
# <Feature name>

## Problem
What is wrong or missing today, and who feels it.

## Goal
One or two sentences. The state of the world when this is done.

## Scope
**In:** <bulleted, concrete>
**Out:** <bulleted, concrete — this section prevents more rework than any other>

## Behavior
The observable behavior, including error and edge cases. Written so a tester who has
never seen the code could check it.

## Constraints
Compatibility, performance budgets, security requirements, conventions to follow.

## Acceptance criteria
- [ ] <checkable statement>
- [ ] <checkable statement>
Each one must be objectively true or false. "Works well" is not a criterion.
"Returns 400 with a field-level error when the email is malformed" is.

## Non-goals
Things a reader might reasonably expect and will not get, with a one-line reason.

## Open questions
- [ ] <question> — blocking / non-blocking, and who can answer it

## Risks
What could go wrong, and what you would do about it.
```

### 4. Get agreement

Show the spec. Wait for an unambiguous yes. Treat "looks reasonable" and "sure, I guess" as
**not** approved — hedged agreement usually means the reader has not engaged with the scope
section yet, which is where the disagreements live.

Resolve every blocking open question before implementation starts.

### 5. Keep it alive

The spec is a living document, not a monument.

- When implementation reveals the spec was wrong, **update the spec**, then continue.
- When scope changes, update the Scope section and say so out loud.
- A spec that quietly diverges from the code is worse than no spec, because people trust it.

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "The requirements are obvious." | Then the spec takes five minutes. Write it. |
| "I'll write the spec after, from the code." | That is documentation, not specification. It cannot catch a misunderstanding. |
| "It's a small change." | Small changes with unstated scope are how one-line fixes become three-day refactors. |
| "The human knows what they want." | They know the outcome. They have not decided the edge cases. You will decide them by accident. |

---

## Red flags

- You are writing code and cannot state the acceptance criteria
- Scope grew during implementation and no one wrote it down
- Two people describe "done" differently
- The spec has no **Out** section
- Acceptance criteria that cannot be checked without asking the author

---

## Verification

Before moving to planning:

- [ ] Spec is written and saved to a known path
- [ ] Every acceptance criterion is objectively checkable
- [ ] Scope has an explicit **Out** list
- [ ] Blocking open questions are resolved
- [ ] The human has explicitly approved
