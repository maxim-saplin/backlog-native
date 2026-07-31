Four skills for running a development backlog inside your repo. Issues are markdown files in `issues/`, ordered by a queue in `BACKLOG.md`, versioned with the code. No tracker service, no API tokens, no MCP server.

```
  grill-me     →  to-issues    →  orchestrate-issues  →  tdd
  │               │               │                      │
  CONTEXT.md      issues/*.md     row status             the code
  the spec        BACKLOG.md      DECISIONS.md           the tests
                  ARCHIVE.md
```

| Skill | Does |
|---|---|
| **grill-me** | Asks one question at a time, checks your answers against the code, and writes the glossary and spec as terms settle. |
| **to-issues** | Splits a plan into thin vertical slices — one numbered markdown file each, plus a row in the queue. |
| **orchestrate-issues** | Dispatches one unattended worker per issue, gates each handoff, logs drift, runs a read-only QA pass at the end. |
| **tdd** | Red/green/refactor with tracer bullets. Includes an unattended mode for dispatched workers. |

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

The problem it targets is spec rot. An agent hits something the spec didn't anticipate, makes a reasonable call, and moves on. Nothing records that the spec is now wrong, so after fifty issues it describes a system that doesn't exist.

Three rules follow from that:

- **Each document has one job.** Terminology in `CONTEXT.md`, architectural trade-offs in the spec with the rejected option named, delivery surprises in a drift log.
- **Drift gets written down.** An unattended worker can't ask you a question, so each question becomes a drift report the orchestrator resolves and records.
- **Humans gate design and acceptance, not implementation.** Issues are typed `AFK` or `HITL` so agents know which they may finish alone.

## Notes

**Dependencies.** `grill-me`, `to-issues`, and `tdd` each stand alone. `orchestrate-issues` needs a queue from `to-issues`, and dispatches workers that use `tdd` — without `tdd` it falls back to an inline test-first rule, but the two belong together.

**Brownfield.** `grill-me` checks your answers against existing code, so it has more to work with on an established codebase than on a blank slate. You can also skip the docs entirely and install only `to-issues`, `orchestrate-issues`, and `tdd` — the orchestrator notes the missing glossary and spec once and carries on.

**Harness assumption.** Tested with Claude Code. No harness-specific install paths appear anywhere, and `orchestrate-issues` states the three properties its dispatch needs so you can map them onto another client — but it does assume you can run a fresh sub-agent, or will run workers as separate sessions.

**What's proven and what isn't.** The skills, the repo-local tracker, the drift log, and gated unattended workers have 53 serially-delivered issues behind them on a real project. Three mechanisms here do not: the `BACKLOG.md`/`ARCHIVE.md` split, the `proof:` criterion tags, and the orchestrator's preflight are new. They came out of reviewing that project's history for places the gate was thin, and have not themselves been run at length.

**Name collision.** `tdd` is a generic name — if your project or another bundle already claims it, installing this overwrites it; use `--skill` to choose. `grill-me` and `tdd` are adapted from [`mattpocock/skills`](https://github.com/mattpocock/skills) (MIT — see [NOTICE](NOTICE)), where they are named `grill-with-docs` and `tdd`; only the latter still collides.

**If the agent can't run your tests, none of this works.** The acceptance gate exists to check a worker's claims by running things; without a working test command "tests green" is unverifiable, and the orchestrator's preflight stops instead of accepting it. Get the test command and a way to launch the app working before anything else.

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
