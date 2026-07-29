# How this setup fits together

Five layers. Each has one job, and they are deliberately not allowed to overlap.

| Layer | Lives in | Answers | Loaded |
|---|---|---|---|
| **Rules** | `AGENTS.md`, `docs/definition-of-done.md` | What is always true here | Always, every session |
| **Agents** | `.opencode/agents/*.md` | *Who* does the work | On selection or spawn |
| **Skills** | `.opencode/skills/*/SKILL.md` | *How* the work is done | On demand, via the `skill` tool |
| **Commands** | `.opencode/commands/*.md` | *When* work starts | On `/name` |
| **Harness** | `opencode.json` | What is allowed, and what is enforced | Startup |

The rule that keeps this coherent: **the human and the commands orchestrate; agents delegate
downward only; skills never invoke agents.** No layer reaches sideways or upward.

---

## Request flow

```
                    You
                     │
      ┌──────────────┴──────────────┐
      │                             │
  /planning  /build  /review     free-form
  /test  /code-simplify          message
  /hwtest
      │                             │
      └──────────────┬──────────────┘
                     ▼
        ┌────────────────────────┐
        │      ORCHESTRATOR      │
        │  plans · delegates ·   │  cannot edit source
        │  sequences · integrates│
        └───────────┬────────────┘
          ┌─────────┴─────────┬
          ▼                   ▼
    ┌───────────┐      ┌─────────────┐
    │  BUILDER  │  ──▶ │CODE-REVIEWER│
    │  edits    │  ◀── │  read-only  │
    └───────────┘ REQUEST CHANGES
          │              │
          └──────┬───────┘
                 ▼
        skill tool → .opencode/skills/*/SKILL.md
                 │
                 ▼
        permission harness (opencode.json)
```

---

## Why the roles are separated this way

**The orchestrator cannot edit source.** Not a suggestion — a `permission.edit` deny with an
allowlist for specs, plans, and ADRs. A coordinator that can also implement will, under any time
pressure, stop coordinating and start typing. Then nothing is planned, nothing is reviewed, and
the whole structure collapses into one agent doing everything in one context.

**The reviewer cannot edit anything.** A reviewer with write access fixes what it finds instead
of reporting it. The fix looks the same, but the finding vanishes — nobody learns the pattern,
the author never sees the mistake, and there is no record. Read-only is what makes it a gate
rather than a second author.

**The hardware tester is its own read-only role.** A green suite proves the logic, not the
device — clock skew, brownouts, a flash sector that only fails warm, and every timing assumption
the host mocked away are invisible to it. And a tester that can edit will make the bench pass
instead of reporting that it does not, which is the one failure mode that leaves no trace.

**Builders cannot spawn subagents.** `permission.task: deny`, plus `subagent_depth: 1` globally.
Recursive delegation produces work nobody is tracking and cost nobody predicted.

**Builders declare nothing else.** They inherit the global harness wholesale. This is not
laziness — agent permission rules are appended after the global ones and the last match wins, so
writing `bash: allow` in a builder would erase every global gate above it, including the denies
on `rm -rf /` and `git push --force`.

**Model split by job shape.** Coordination is the reasoning-heavy, low-volume, expensive-to-get-
wrong part — that gets `deepseek-v4-pro` at maximum reasoning effort. Implementation and review
are high-volume and well-specified by the time they start — those get `deepseek-v4-flash`. The
plan is the leverage point; that is where the capable model belongs.

---

## Lifecycle mapping

| Phase | Command | Agent | Skill | Artifact |
|---|---|---|---|---|
| DEFINE | — | orchestrator | `spec-driven-development` | `SPEC.md` |
| PLAN | `/planning` | orchestrator | `planning-and-task-breakdown` | `tasks/plan.local.md`, `tasks/todo.local.md` |
| BUILD | `/build` | builder | `incremental-implementation` | code + commits |
| VERIFY | `/test` | builder | `test-driven-development` | tests |
| REVIEW | `/review` | code-reviewer | `code-review-and-quality` | verdict |
| HARDWARE | `/hwtest` | hardware-tester | `hardware-test-execution` | findings + verdict |
| SIMPLIFY | `/code-simplify` | builder | — | smaller diff |
| SHIP | — | `/ship` | `git-workflow-and-versioning` | tag, changelog |

Commands are shortcuts, not the only path. Plain English works too — the routing table in
`AGENTS.md` and the `using-agent-skills` router cover the same ground without a slash.