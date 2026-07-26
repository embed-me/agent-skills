---
name: git-workflow-and-versioning
description: >-
  Structures commits, branches, history, semantic versioning, tags, and changelogs so history
  stays atomic, bisectable, and safe to roll back. Use when committing, branching, resolving
  conflicts, or cutting a release. Language and framework agnostic.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  adapted-from: https://github.com/addyosmani/agent-skills
  phase: ship
---

# Git Workflow and Versioning

History is a debugging tool. Every commit is a save point you can return to, bisect through, and
revert cleanly — but only if each one is a single coherent change that builds and passes.

A history of "wip", "fix", and "more changes" is not a history. It is a single large commit
wearing a costume.

---

## Commits

### Atomic

One logical change per commit. A commit should be describable in one sentence with no "and".

```
Good:  Validate email format on registration
Good:  Extract token parsing into its own function
Bad:   Add validation, fix login bug, rename helpers, update deps
```

If your subject line needs "and", split it. Stage selectively (`git add -p`) rather than
`git add -A`; blind staging is how unrelated work gets swept into a commit and breaks the clean
rollback guarantee.

### Frequent

Commit at each green point, not at the end of the day. Uncommitted work is unbacked work, and it
grows harder to review with every hour it stays unstaged.

### Green

Every commit builds and passes tests. Not "the branch passes at the end" — every commit. A red
commit in the middle makes `git bisect` useless exactly when you need it.

### Well described

```
<type>: <imperative summary, ~50 chars>

Why this change is needed. What problem it solves. What you
considered and rejected, if that is not obvious.

Refs: #123
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `build`, `ci`.

**The subject says what. The body says why.** The diff already shows what changed; it can never
show why. Six months from now, "why" is the only thing anyone needs from this message.

Write in the imperative — "Add retry logic", not "Added" or "Adds". It completes the sentence
"if applied, this commit will…".

Never mention the tooling that produced the change. Commit messages describe the code, not the
authoring process.

---

## Branching

Default to **trunk-based**: short-lived branches off the main line, merged within a day or two.
Long-lived branches accumulate conflicts at a rate proportional to their age.

```
<type>/<short-description>

feat/user-authentication
fix/session-timeout
refactor/extract-validator
```

Rules:

- Branch from an up-to-date main
- One branch per logical unit of work
- Keep it short-lived; rebase or merge from main frequently
- Delete after merge

For work that must land incrementally but cannot be exposed yet, use a feature flag defaulting
off rather than a long-lived branch. Trunk-based development with flags beats branch-based
development with merges, and it is a much shorter conversation with a conflict.

---

## Before you commit

Every time:

- [ ] Full test suite green
- [ ] Build succeeds
- [ ] Lint and formatter clean
- [ ] `git diff --staged` reviewed line by line — read your own change first
- [ ] No debug output, commented-out code, or stray TODOs left behind
- [ ] No secrets, keys, tokens, or credentials
- [ ] No unrelated files
- [ ] Generated artifacts, build output, and dependency directories are gitignored, not committed

That `git diff --staged` read catches more than any hook. Do it.

---

## History hygiene

**Safe, local, reversible** — do these freely: commit, branch, switch, stash, fetch, local
merge, `git revert`.

**Requires care** — these rewrite or discard: `reset --hard`, `clean -fd`, `rebase`, `push`,
`checkout --`, force operations. Confirm before running them.

**Never on shared history** — `push --force`, `filter-branch`, rewriting anything another person
may have pulled. If a bad commit is already public, `git revert` it. Reverting is honest;
rewriting shared history breaks everyone else's clone.

`git revert` over `git reset` on anything already pushed, always.

---

## Git as a debugging tool

- `git bisect` — find the commit that introduced a bug in log(n) steps. This only works if your
  commits are atomic and green, which is the whole argument for both.
- `git log -S "<string>"` — find when a string entered or left the codebase.
- `git blame -w` — who last touched this line, ignoring whitespace-only changes.
- `git log --follow <file>` — history across renames.

---

## Versioning

Semantic versioning, `MAJOR.MINOR.PATCH`:

- **MAJOR** — breaking change to a public interface. Anything that forces a consumer to act.
- **MINOR** — backward-compatible new functionality.
- **PATCH** — backward-compatible bug fix.

Pre-1.0, breaking changes go in MINOR by convention — but say so in the changelog anyway.

"It's only a small break" is still a break. If a consumer must change their code, it is MAJOR.

**The tag is the release.** Tag the exact commit, annotated, and let build tooling derive the
version from the tag rather than from a hand-edited constant that will eventually disagree.

```
git tag -a v1.4.0 -m "Release 1.4.0"
```

Tagging and pushing tags are gated actions in this setup. Ask before running them.

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "I'll clean up the history later." | You will not, and by then someone has pulled it. |
| "One big commit is simpler." | Until you need to revert one part of it. |
| "The diff explains itself." | The diff shows what. Nobody can reconstruct why. |
| "It's just a small force push." | It is a small force push onto someone else's afternoon. |
| "I'll bump the version by hand." | The constant and the tag will disagree, and you will find out during a release. |

---

## Red flags

- Commit messages like "wip", "fix", "updates", "asdf"
- A commit that does not build
- Generated files, `node_modules/`, `dist/`, or `.env` in the diff
- A branch older than a week
- Force push to a shared branch
- A release with no tag, or a tag with no changelog entry
- A breaking change released as a PATCH

---

## Verification

- [ ] Each commit is one logical change
- [ ] Each commit builds and passes tests
- [ ] Subjects are imperative; bodies explain why
- [ ] No secrets or generated artifacts committed
- [ ] Version bump matches the actual nature of the change
- [ ] Release is tagged and the changelog updated
