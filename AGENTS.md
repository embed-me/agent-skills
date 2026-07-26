# AGENTS.md

Operating rules for any AI coding agent working in this repository (opencode, Claude Code,
Cursor, Copilot, Codex, Gemini CLI, …). This file is loaded automatically into every session.

Keep it short. Detail belongs in skills, not here.

---

## 1. The skill contract

This repo ships **skills**: reusable workflows at `.opencode/skills/<name>/SKILL.md`.

**Before you act on any non-trivial request, load the matching skill and follow it.**

- In opencode: call the `skill` tool — `skill({ name: "spec-driven-development" })`.
- In any other agent: read the file directly — `.opencode/skills/<name>/SKILL.md`.
- Always load `using-agent-skills` first. It is the router.

Skills are workflows, not suggestions. Follow the steps in order. Do not partially apply them.

**Rationalizations that are always wrong:**

- "This is too small for a skill." — Load it; a small task follows a short path through it.
- "I'll just quickly implement this." — This is how unreviewed, untested code lands.
- "I'll gather context first, then check." — Checking *is* gathering context.

---

## 2. Agent roster

Can be discovered by reading `.opencode/agents/`.

---
## 3. Workflow

1. Look for a manifest: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`,
   `build.gradle*`, `*.csproj`, `Gemfile`, `composer.json`, `mix.exs`, `Makefile`, `justfile`.
2. Prefer scripts the repo already defines (`package.json` scripts, `Makefile` targets, `just`
   recipes, `pyproject` tool sections) over inventing a command line.
3. Check CI (`.github/workflows/*`, `.gitlab-ci.yml`) — CI is the ground truth for how this
   project is actually built and tested.
4. Run the test suite once to confirm a green baseline **before** changing anything.
5. Then update this table so the next session does not have to rediscover it.

Never introduce a new package manager, test runner, or framework because you are more familiar
with it. Match what is already here.

---

## 4. Working agreement

**Surface assumptions.** Before non-trivial work, state what you are assuming about
requirements, architecture, and scope. Invite correction, then proceed.

**Stop when confused.** Conflicting requirements, a spec that disagrees with the code, an
unexpected test failure — name the conflict and ask. Do not pick an interpretation and hope.

**Push back.** You are not a yes-machine. If an approach has a concrete downside, say so,
quantify it, propose an alternative, and accept the human's decision once they have the facts.

**Stay in scope.** Touch only what the task requires. Do not reformat untouched files, delete
code you do not understand, "clean up" adjacent modules, or add unrequested features.

**Prefer boring.** If 100 lines will do, do not write 1000. Abstractions must earn their keep.

**Verify, don't assume.** "It compiles" and "it looks right" are not evidence. Evidence is a
test that failed before your change and passes after, plus a green suite.

**Leave the repo runnable.** Every commit builds and passes tests.

---

## 5. Definition of Done

Every change clears the standing bar in `docs/definition-of-done.md` *in addition to* its own
acceptance criteria. That file is loaded into your context automatically — apply it, do not
re-read it from disk.

---

## 6. Approval boundaries

The permission harness in `opencode.json` runs most work without interrupting the human.
Some things always stop and ask. Do not try to route around a prompt:

- Do not rewrite a gated command to dodge a pattern (`git push` is gated; so is any wrapper
  around it). If you are blocked, say so and explain what you wanted to do and why.
- Do not write outside the project directory without asking.
- Do not read or print secrets: `.env`, keys, credentials, tokens.
- Do not push, publish, deploy, or mutate infrastructure without explicit approval in this
  session — approval for one action is not approval for the next.
