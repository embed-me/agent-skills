---
name: incremental-implementation
description: >-
  Delivers a change as a sequence of small, verified, individually committable increments that
  each keep the build green. Use when implementing any feature or change that touches more than
  one file, or whenever a task feels too big to land in one step. Language agnostic.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  adapted-from: https://github.com/addyosmani/agent-skills
  phase: build
---

# Incremental Implementation

Land the change in slices that each work, rather than one slice that mostly works.

The reason is not tidiness. A 40-file diff that fails its test suite gives you no information
about *which* part is wrong. Ten commits that each passed give you a bisectable history and a
guaranteed rollback point. The increment is a debugging tool you build in advance.

Always run this alongside `test-driven-development`. This skill decides *what* to build next;
that one decides *how you know* it works.

---

## Before the first line

1. **Read the task's acceptance criteria.** If you cannot restate them, stop and ask.
2. **Read the code you will change**, plus its tests and its callers.
3. **Find the existing pattern.** Someone has solved something like this here already. Match it,
   even where you would have chosen differently. Consistency is worth more than your preference.
4. **Confirm a green baseline.** Run the suite before touching anything. If it is already red,
   report that and stop — do not build on a broken foundation, and do not inherit blame for it.
5. **State your assumptions** if anything in the brief is open.

---

## The increment cycle

```
1. IMPLEMENT  The smallest next behavior that is independently meaningful.
2. TEST       Write the test. Run it. Watch it fail for the right reason.
4. VERIFY     Full suite. Build. Lint and typecheck if configured.
5. COMMIT     One logical change. Imperative subject. Body says why.
6. REPEAT     Until the acceptance criteria are met.
```

Never carry two broken things at once. If the suite is red for two unrelated reasons, you have
lost the thread — revert to the last green commit and take a smaller bite.

---

## Slicing strategies

**Vertical (default).** A thin path through every layer. "Create a record with only a title,
end to end." Demonstrable, verifiable, and it exercises the integration seams early.

**Contract-first.** When several slices depend on a shared interface, define and land that
interface with its tests first, then build the implementations behind it. Use this when the
contract is genuinely shared, not to justify building all the plumbing up front.

**Risk-first.** When one part is uncertain — an unfamiliar library, an ambiguous requirement, a
performance question — do that part first, in the smallest form that answers the question. Learn
the expensive thing while the sunk cost is small.

---

## Rules

**0. Simplicity first.** The smallest change that satisfies the criteria. No speculative
generality, no configuration nobody asked for, no abstraction with one caller. You can always
generalize later; you cannot easily un-generalize.

**1. Scope discipline.** Touch only what the task requires. No drive-by reformatting, no
renaming to taste, no deleting code you do not understand, no "while I was in there". Real
problems outside scope go in your report, not in your diff.

**2. One thing at a time.** A refactor and a behavior change do not share a commit. Do the
refactor, verify green, commit. Then change the behavior.

**3. Keep it compilable.** Every commit builds and passes. Not "at the end" — every commit.

**4. Flag what is incomplete.** If a feature must land across several commits, guard the
unfinished path behind a flag defaulting to the current behavior. Shipping half-wired code with
no switch is how a partial feature becomes a production incident.

**5. Safe defaults.** New config, flags, and parameters default to the existing behavior. A
change should not alter anything for anyone who did not opt in.

**6. Rollback-friendly.** Additive before subtractive: add the new path, migrate, *then* remove
the old one — as separate commits. Expand and contract, never both at once.

**7. Stop after three failures.** Same failure three times means you are guessing. Report what
you tried, what happened, and your best theory. Guessing faster does not help.

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "It's faster to write it all and test at the end." | It is faster to *type*. It is much slower to debug 40 files at once. |
| "I'll commit once it's all working." | Then you have no rollback point and no bisectable history for the whole span. |
| "This refactor is related, so same commit." | The next person reading `git log` cannot tell your bug fix from your rename. |
| "I'll add the tests after." | You will not, and if you do they will be written to pass. |
| "The abstraction will be needed eventually." | Add it when the second caller exists. |

---

## Red flags

- More than roughly 400 changed lines with no commit
- The suite has been red for several steps
- You cannot describe the last commit in one sentence
- A commit mixes a rename, a refactor, and a behavior change
- Files in the diff that have nothing to do with the task
- New behavior with no accompanying test

---

## Verification

Per increment:

- [ ] Test written first and observed failing
- [ ] Full suite green
- [ ] Build succeeds
- [ ] Lint and typecheck pass (or are not configured)
- [ ] Diff contains only files this increment needed
- [ ] Committed with a message that explains why