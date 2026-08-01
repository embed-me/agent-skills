---
name: self-improvement
description: >-
  Turns completed PR and review evidence into small, monitored improvements to repository guidance.
  Use manually after a PR is merged or fully completed.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  phase: post-pr
---

# Self-Improvement

Improve the way future work is done, not the finished PR itself. Use the completed PR, review
comments, requested changes, test failures, and follow-up evidence as the input.

## Process

1. **Inspect and reflect.** Read the finished change and its review/change-request evidence. Summarize
   what happened, why it happened, and whether the process or guidance failed.
2. **Classify the finding.** Decide whether it is PR-specific or a pattern likely to recur. Do not
   generalize a one-off issue, an intentional tradeoff, or an isolated mistake without evidence.
3. **Choose the smallest prevention.** For a justified recurring pattern, formulate one concrete,
   testable improvement. Prefer a precise addition or correction to the relevant skill; edit
   `AGENTS.md` only when the rule is universal and belongs there.
4. **Apply and record.** Make only the minimal justified edit. Do not perform broad cleanup,
   unrelated refactoring, or overfit guidance to one PR. If no generalized action is justified,
   record that decision and leave guidance unchanged.
5. **Monitor.** Name where the signal is recorded, its review timing/cadence, and the evidence threshold
   for recurrence or success (for example, the review finding recurs once or the new checklist is
   used successfully in two subsequent changes). At that review, decide whether to keep, revise,
   or remove the guidance.
6. **Verify.** Before reporting completion, inspect `git diff` and `git status`; confirm that only
   the justified skill or `AGENTS.md` changed, and validate that the resulting guidance is precise,
   scoped, and consistent with this workflow.

## Output

Report the evidence reviewed, the reflection, the PR-specific/general classification, the action
taken or declined, and the monitoring record/location, review timing, evidence threshold, and
keep/revise/remove decision. Keep the result surgical and reversible.
