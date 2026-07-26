# Definition of Done

The standing bar every change clears before it counts as finished. This file is loaded into
every session automatically via `instructions` in `opencode.json`.

Acceptance criteria vary per task and answer *"did we build the right thing?"*
The Definition of Done is the same every time and answers *"is it actually finished?"*
A change is done only when **both** hold.

---

## Correctness

- [ ] All acceptance criteria for the task are met
- [ ] Behavior verified at runtime — compiling and typechecking are not verification
- [ ] New behavior has a test that fails without the change and passes with it
- [ ] Existing tests still pass; no regressions
- [ ] Edge and error paths handled, not just the happy path

## Quality

- [ ] Naming and structure reveal intent without a comment explaining *what* the code does
- [ ] No duplicated business logic
- [ ] No dead code, debug output, or commented-out blocks left behind
- [ ] Diff is scoped to the task — no unrelated refactors or reformatting
- [ ] Lint and formatter pass

## Integration

- [ ] Works with the rest of the system, not only in isolation
- [ ] Migrations, config changes, and feature flags accounted for
- [ ] Backward compatibility considered for any public interface change

## Documentation

- [ ] Public interfaces and user-facing behavior documented
- [ ] Decisions worth preserving recorded (ADR or spec update)
- [ ] Docs describe the current state, not the change history

## Ship-readiness

- [ ] Security reviewed for anything touching untrusted input, auth, or user data
- [ ] No secrets in code, config, tests, fixtures, or logs
- [ ] Rollback path exists for anything risky
- [ ] Human reviewed and approved before merge or deploy

---

## How to apply

- **Per task** — Correctness and Quality before checking the task off
- **Per feature** — Integration and Documentation before calling the feature complete
- **Per release** — the whole list is the floor

Tailor this once for the project, then reuse it unchanged. A bar that gets renegotiated under
deadline pressure is not a bar.

---

## Red flags

- "It's done, I just haven't run it yet" — unverified work is not done
- "Tests pass" standing in for done while docs, regressions, or runtime checks are skipped
- A different standard applied when the deadline is close
- Acceptance criteria treated as the whole bar, with no standing quality floor
- Done declared before human review on a change that needed it
