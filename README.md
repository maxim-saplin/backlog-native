Four skills that take an idea to delivered code, with the backlog living in your repo as markdown instead of a tracker service. No API tokens, no MCP server, no project-management account — `issues/` **is** the tracker.

```
  grill-me     →  to-issues    →  orchestrate-issues  →  tdd
  │               │               │                      │
  CONTEXT.md      issues/*.md     row status             the code
  the spec        BACKLOG.md      DECISIONS.md           the tests
                  ARCHIVE.md
```

| Skill | Does |
|---|---|
| **grill-me** | Interviews you one question at a time, challenges your claims against the code, sharpens vocabulary. Writes the glossary and spec. |
| **to-issues** | Breaks the settled plan into thin vertical slices — numbered markdown files plus an ordered queue. |
| **orchestrate-issues** | Dispatches unattended workers one issue at a time, gates each handoff, logs drift, runs an adversarial QA pass. |
| **tdd** | Red/green/refactor with tracer bullets. Has an unattended mode for delegated work. |

## Install

```bash
npx skills add maxim-saplin/backlog-native --all
```

Add `-g` for a global install. `--all` takes every skill into every agent directory it detects, without prompting. To look first, or pick a subset:

```bash
npx skills add maxim-saplin/backlog-native --list
npx skills add maxim-saplin/backlog-native --skill to-issues --skill orchestrate-issues --skill tdd
```

**Manual alternative:** copy the directories under [`skills/`](skills/) (each `SKILL.md` plus its `references/`) into your agent's skills folder — `.agents/skills/` works across many harnesses, `.claude/skills/` for Claude Code.

## Use

Ask in plain language:

- "Grill me on this feature until we both understand it, then update the glossary."
- "Break that into issues."
- "Work through the backlog."
- "File a follow-up: the cost column shows stale values after a migration."

On a project with no `issues/` directory, `to-issues` scaffolds the queue files before writing the first issue. On one with no test harness, the orchestrator lets exactly one issue through — the scaffold slice — then requires a working harness before anything else ships.

## Why it's shaped this way

Getting one good pull request out of an agent is easy. Getting work you'd ship, repeatedly, over months, is not — and the thing that breaks first isn't the code.

> Specs rot silently. An agent hits something the spec didn't anticipate, makes a reasonable call, and moves on. Nothing records that the spec is now wrong. Fifty issues later the document describes a system that no longer exists, and nobody trusts it.

Hence three rules the skills enforce on each other:

- **Documents have assigned roles.** Terminology lives in `CONTEXT.md`, architectural trade-offs in the spec with the rejected alternative named, delivery surprises in a drift log. Nothing bleeds between them.
- **Drift is written down.** An unattended worker can't ask you a question, so every question becomes a drift report the orchestrator resolves and records.
- **Humans gate design and acceptance, not implementation.** Work is typed `AFK` or `HITL` so agents know what they may finish alone.

## Notes

**Dependencies.** `grill-me`, `to-issues`, and `tdd` each stand alone. `orchestrate-issues` needs a queue from `to-issues`, and dispatches workers that use `tdd` — without `tdd` it falls back to an inline test-first rule, but the two belong together.

**Brownfield works, arguably better.** `grill-me` is at its best on existing code, since its method is cross-referencing what you claim against what the code does. On a blank slate there's nothing to grill against. You can also adopt the back half only and skip the docs entirely — the orchestrator notes their absence once and carries on.

**Harness assumption.** Tested with Claude Code. No harness-specific install paths appear anywhere, and `orchestrate-issues` states the three properties its dispatch needs so you can map them onto another client — but it does assume you can run a fresh sub-agent, or will run workers as separate sessions.

**What's proven and what isn't.** The skills, the repo-local tracker, the drift log, and gated unattended workers have 53 serially-delivered issues behind them on a real project. Three mechanisms here do not: the `BACKLOG.md`/`ARCHIVE.md` split, the `proof:` criterion tags, and the orchestrator's preflight are new. They exist because reviewing that project's history showed where the gate was thin — not because they have miles on them.

**Name collision.** `tdd` is a generic name — if your project or another bundle already claims it, installing this overwrites it; use `--skill` to choose. `grill-me` and `tdd` are adapted from [`mattpocock/skills`](https://github.com/mattpocock/skills) (MIT — see [NOTICE](NOTICE)), where they are named `grill-with-docs` and `tdd`; only the latter still collides.

> **Biggest risk:** an acceptance gate that cannot see. If your agent can't run the tests *and* exercise the app, "tests green" is a claim you cannot check. Fix the feedback loop first — everything here compounds on it.

## Repository layout

```
skills/
  grill-me/
    SKILL.md
    references/
      CONTEXT-FORMAT.md             # glossary entry format
      SPEC-FORMAT.md                # product spec structure
  to-issues/
    SKILL.md
    references/
      backlog-template.md           # scaffold for a fresh tracker
  orchestrate-issues/
    SKILL.md
  tdd/
    SKILL.md
    references/
      tests.md                      # what a good test looks like
      mocking.md                    # when mocking is legitimate
      deep-modules.md               # small interface, deep implementation
      interface-design.md           # designing for testability
      refactoring.md                # refactor candidates
```
