---
name: code-review-and-quality
description: >-
  Reviews a change across correctness, readability, architecture, security, and performance, and
  returns categorized findings with a verdict. Use before merging any change, whether written by
  a human or an agent. Language and framework agnostic.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  adapted-from: https://github.com/addyosmani/agent-skills
  phase: review
---

# Code Review and Quality

Judge finished work against what it was supposed to do, and against the standard the codebase
holds. Produce findings someone can act on without asking you a follow-up question.

Review is not proofreading. It is the last point at which a wrong decision is cheap to reverse.

---

## Process

### 1. Get the context

Read the task's acceptance criteria and the relevant spec section **before** the diff. Without
them you can only assess whether the code is *nice*, not whether it is *right* — and those come
apart constantly.

### 2. Read the tests first

Tests reveal what the author believed the behavior should be. They are the fastest signal in the
whole review:

- Is there a test for the new behavior at all?
- Would it fail if the implementation were wrong? (Mentally break the code — does a test catch it?)
- Are edge cases covered, or only the happy path?
- Does any test assert on internals rather than behavior?

A change with weak tests is a change nobody can safely modify later. Treat that as a finding in
its own right, not a footnote.

### 3. Read the change

Read the diff, then read enough surrounding code to judge whether the change belongs there.
Reviewing a diff in isolation catches typos and misses architecture.

### 4. Run the verification

Where the project makes it cheap, run the tests, the build, and the linter yourself. A handoff
report claiming green is usually sincere and occasionally wrong; the cost of checking is one
command.

### 5. Apply the five axes

**Correctness.** Does it satisfy the acceptance criteria? Edge cases — empty, null, zero,
boundary, unicode, concurrent, duplicate? Are error paths handled or swallowed? Off-by-one?
Race conditions? Does any state transition leave the system inconsistent on failure?

**Readability and simplicity.** Could the next engineer follow this cold? Are names accurate and
consistent with the codebase? Is the control flow flat, or nested deeply enough to need a map?
Do comments explain *why* rather than restate *what*? Is anything clever where obvious would do?
Could this be meaningfully shorter without becoming dense?

**Architecture.** Does it follow existing patterns or invent a new one — and if new, is that
justified and recorded? Are module boundaries respected, and do dependencies flow the right way?
Is the abstraction level right — not premature, not duplicated? Does it add coupling that will
be hard to remove?

**Security.** Is untrusted input validated at the boundary, not deep inside? Are queries
parameterized and output encoded for its sink? Is authorization checked on every new path, not
just the one in the demo? Any secret in code, config, tests, fixtures, or logs? Any new
dependency, and is the trade worth it? Does an error message leak internals?

**Performance.** N+1 queries. Unbounded reads, loops, or allocations. Missing pagination on a
list. Blocking I/O on a hot path. A new query pattern with no supporting index. Judge against
realistic data volume — and do not invent optimization work the change does not need.

### 6. Check the standing bar

cceptance criteria answer "did we build the right thing"; the Definition of Done answers 
"is it finished to our standard". Both must hold.

### 7. Categorize and decide

- **Critical** — blocks merge. Security hole, data loss, broken functionality, missing test for
  new behavior.
- **Important** — should block. Wrong abstraction, unhandled error path, misleading public name,
  uncovered edge case, silent behavior change.
- **Suggestion** — optional. Style, naming nits, future simplification.

**Never approve with an open Critical.** Every Critical and Important finding carries a specific
recommended fix.

---

## Conduct

**Review the code, not the author.** "This function does two things" — not "you made this too
complicated."

**Cite file and line.** A finding without a location cannot be acted on.

**Say what is good, specifically.** Review that only finds fault stops being read, and vague
praise is worse than none.

**Do not pad.** Three real findings beat fifteen nitpicks. Volume buries the ones that matter.

**Be honest about uncertainty.** "I am not sure this handles concurrent writes — can you walk me
through it?" is a legitimate finding. Confident wrong findings cost trust fast.

**Separate pre-existing problems.** Issues in untouched code go in Suggestions, marked as
pre-existing. Do not block an author on a mess they inherited.

**Size matters.** A change over roughly 400 lines is reviewed materially worse than one under
it — attention drops and everything after the first screen gets skimmed. If a diff is too large
to review properly, say so and ask for it to be split. That is itself a valid finding.

**If you can edit, do not.** Reporting a finding preserves the record and lets the author learn.
Fixing it silently converts a review into a rewrite and destroys both.

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "It's a small change, skim it." | Small changes ship most incidents, precisely because they get skimmed. |
| "Tests pass, so it's fine." | Tests pass on code that solves the wrong problem. |
| "The author knows this area better." | Then ask them a question. Deferring silently is not review. |
| "I'll approve and file a follow-up." | Follow-ups filed at merge time have a low completion rate. |
| "Nitpicking will slow things down." | So will the outage. Distinguish nits from findings and report both honestly. |

---

## Red flags

- Approval with no findings at all on a non-trivial change
- Findings with no file or line
- No test for new behavior, unremarked
- Review that only comments on formatting
- A diff too large to hold in your head, approved anyway
- The reviewer also wrote the code

---

## Verification

- [ ] Acceptance criteria read before the diff
- [ ] Tests read before the implementation
- [ ] All five axes considered
- [ ] Verification commands actually run, or their absence stated
- [ ] Findings categorized, each with a location and a fix
- [ ] Definition of Done checked
- [ ] Explicit verdict given
