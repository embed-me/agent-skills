---
name: test-driven-development
description: >-
  Drives implementation and bug fixes with tests written first and observed failing, then makes
  them pass. Use when implementing any logic, fixing any bug, or changing any behavior. Includes
  stack discovery so it works with any language, framework, or test runner.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  adapted-from: https://github.com/addyosmani/agent-skills
  phase: verify
---

# Test-Driven Development

Write the test first, watch it fail, then make it pass.

The order matters more than it looks. A test written after the implementation is shaped by the
implementation — it tests what the code does, not what it should do, and it will happily pass
against a bug. A test you have watched fail for the right reason is the only kind you know
actually checks something.

---

## Discover the stack first

Never assume a test runner. Before writing a single test:

1. **Find the manifest** — `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`,
   `build.gradle*`, `*.csproj`, `Gemfile`, `composer.json`, `mix.exs`, `Makefile`, `justfile`.
2. **Read `AGENTS.md`** for the project's declared test commands. That table wins over inference.
3. **Read an existing test.** Copy its structure, naming, imports, fixture style, and assertion
   library. A test that looks foreign to the suite is a test nobody maintains.
4. **Find where tests live** — `test/`, `tests/`, `__tests__/`, `spec/`, `*_test.go`,
   `*.test.ts`, `*_spec.rb`, alongside the source or in a mirror tree. Follow the convention.
5. **Learn how to run one test**, not just all of them. The single-test loop is what makes TDD
   fast enough to actually do.

If the project has no tests at all, say so, and propose the smallest viable setup using the
ecosystem's default runner. Do not introduce a second test framework alongside an existing one.

---

## The cycle

### RED — write a failing test

```
1. Name the behavior in the test name: "rejects a withdrawal that exceeds the balance"
2. Write the smallest test that captures it
3. Run it
4. Confirm it fails — and read the failure
```

**Confirm it fails for the right reason.** A test that fails with `ImportError` or a typo is not
a red test; it is a broken test. It must fail on the assertion, because that proves the assertion
is real. This is the single most skipped step in TDD and the one that gives it all its value.

### GREEN — make it pass

Write the minimum code that satisfies the test. Not the general case. Not the abstraction you
can already see. Just enough. The next test will push the design further, and it will push it
somewhere better than your guess would have.

Run the test. Then run the full suite — you want to know immediately if you broke something.

### REFACTOR — clean up

With tests green, improve names, remove duplication, flatten nesting. Run the suite after each
change. Refactoring means changing structure without changing behavior; if a test breaks, you
changed behavior.

---

## Bug fixes: prove it first

Never fix a bug you have not reproduced in a test.

```
1. REPRODUCE  Write a test that demonstrates the bug. Run it. It MUST fail.
              If it passes, you have not understood the bug — keep investigating.
2. UNDERSTAND Find the root cause. Not the symptom, and not the first plausible line.
3. FIX        Change the code. The test passes.
4. REGRESS    Run the full suite. The fix must not break anything else.
5. GUARD      Consider whether nearby inputs have the same problem. Add those cases too.
```

The failing test is the proof that you fixed what was actually reported, and the guarantee that
it does not come back silently. A bug fix without a regression test is an invitation.

---

## What to test

**Test observable behavior, not implementation.** Assert on what the code returns, stores, or
emits — not on which internal methods were called. Interaction-heavy tests break on every
refactor while catching nothing, which trains people to distrust the suite.

```
Good:  after transfer(a, b, 50), a.balance == 50 and b.balance == 150
Bad:   ledger.record() was called once with ('transfer', 50)
```

**Prefer real implementations over mocks.** Use the real object, the real function, an in-memory
database, a temp directory. Mock at the genuinely awkward boundaries: network, clock, randomness,
paid third-party APIs, anything nondeterministic. Every mock is an assumption about someone
else's behavior that can silently drift out of date.

**Be repetitive in tests.** Tests are documentation. A little duplication that makes each test
readable standalone beats a clever shared helper that forces the reader to jump around to
understand what is being asked.

**Arrange, Act, Assert.** Set up, do the one thing, check the result. Visible sections.

**One concept per test.** Several assertions about one behavior is fine. Several unrelated
behaviors in one test means a failure tells you almost nothing about what broke.

**Name tests by behavior.** `test_rejects_expired_token`, not `test_auth_2`.

**Cover the edges.** Empty, null, zero, negative, boundary, maximum, unicode, concurrent,
duplicate, out of order. The happy path is the part that was going to work anyway.

---

## Test balance

| Level | Speed | Scope | Share |
|---|---|---|---|
| Unit | ms | One function or class, no I/O | Most |
| Integration | 100s of ms | Real components together, real DB, real files | Fewer |
| End to end | seconds | Full system through its real interface | Fewest |

Push each test to the lowest level that can actually catch the bug. High-level tests are slower,
flakier, and vaguer about what broke — but some bugs only exist between components, and those
are exactly the ones unit tests will never see.

---

## Anti-patterns

- Tests that pass whether or not the code works
- Tests asserting on log output or private internals
- Shared mutable state that makes order matter
- `sleep()` instead of waiting on a condition
- One test that exercises the entire feature
- Skipped or commented-out tests left in the suite
- Tests changed to match a bug rather than reveal it
- Snapshot tests blindly regenerated when they fail

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "I'll write tests after; it's faster." | It is faster right up until you are debugging without them. |
| "This is too simple to test." | Simple code with an untested edge case is where most production bugs live. |
| "It's hard to test." | That is a design signal. Hard-to-test usually means too coupled. |
| "The test would just mirror the code." | Then you are testing implementation. Test the behavior instead. |
| "Manual testing confirmed it." | Once. On your machine. Never again automatically. |

---

## Red flags

- New behavior with no test
- A test that has never been observed failing
- Coverage rising while confidence is not
- Fixing a bug without a reproducing test
- Deleting or skipping a test to make the suite green
- Tests that only pass when run in a specific order

---

## Verification

- [ ] Every new behavior has a test that was seen failing first
- [ ] Every bug fix has a reproducing test that failed before the fix
- [ ] Tests assert on behavior, not internals
- [ ] Full suite green
- [ ] Suite runs clean twice in a row (no order dependence, no flake)
- [ ] No skipped or commented-out tests added
