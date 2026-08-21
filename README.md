> This "spec-driven" approach has been proven by building from scratch https://github.com/maxim-saplin/model-blend - a LiteLLM proxy blending multiple LLM endpoints into a single one. Worked great with Cursor and Composer 2.5

Five skills for running a development backlog inside your repo. Issues are markdown files in `issues/`, ordered by a queue in `BACKLOG.md`, versioned with the code. No tracker service, no API tokens, no MCP server.

```
  grill-me     →  to-issues    →  orchestrate-issues              →  tdd
  │               │               │  or orchestrate-issues-watchdog    │
  CONTEXT.md      issues/*.md     row status                         the code
  the spec        BACKLOG.md      DECISIONS.md                       the tests
                  ARCHIVE.md
```

| Skill                           | Does                                                                                                                              |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **grill-me**                    | Asks one question at a time, checks your answers against the code, and writes the glossary and spec as terms settle.              |
| **to-issues**                   | Splits a plan into thin vertical slices — one numbered markdown file each, plus a row in the queue.                               |
| **orchestrate-issues**          | Dispatches one unattended worker per issue, gates each handoff, logs drift, runs a read-only QA pass at the end. Default flavor. |
| **orchestrate-issues-watchdog** | Same queue and workers, stricter gates: per-issue adversarial QA plus an independent watchdog that can `COURSE-CORRECT` or `STOP`. |
| **tdd**                         | Red/green/refactor with tracer bullets. Includes an unattended mode for dispatched workers.                                       |

## Install

```bash
npx skills add maxim-saplin/backlog-native --all
```

Add `-g` for a global install. `--all` takes every skill into every agent directory it detects, without prompting — including both orchestrators. Pick one flavor with `--skill` if you do not want both loaded. To look first, or pick a subset:

```bash
npx skills add maxim-saplin/backlog-native --list
npx skills add maxim-saplin/backlog-native --skill to-issues --skill orchestrate-issues --skill tdd
npx skills add maxim-saplin/backlog-native --skill orchestrate-issues-watchdog
```

**Manual alternative:** copy the directories under [`skills/`](skills/) (each `SKILL.md` plus its `references/`) into your agent's skills folder — `.agents/skills/` works across many harnesses, `.claude/skills/` for Claude Code.

## Use

Ask in plain language:

- "Grill me on this feature until we both understand it, then update the glossary."
- "Break that into issues."
- "Work through the backlog."
- "Deliver the queue unattended with the watchdog."
- "File a follow-up: the cost column shows stale values after a migration."

On a project with no `issues/` directory, `to-issues` scaffolds the queue files before writing the first issue. On one with no test harness, the orchestrator lets exactly one issue through — the scaffold slice — then requires a working harness before anything else ships.

## Why it's shaped this way

The problem it targets is spec rot. An agent hits something the spec didn't anticipate, makes a reasonable call, and moves on. Nothing records that the spec is now wrong, so after fifty issues it describes a system that doesn't exist.

Three rules follow from that:

- **Each document has one job.** Terminology in `CONTEXT.md`, architectural trade-offs in the spec with the rejected option named, delivery surprises in a drift log.
- **Drift gets written down.** An unattended worker can't ask you a question, so each question becomes a drift report the orchestrator resolves and records.
- **Humans gate design and acceptance, not implementation.** Issues are typed `AFK` or `HITL` so agents know which they may finish alone.

## Notes

**Dependencies.** `grill-me`, `to-issues`, and `tdd` each stand alone. Both orchestrators need a queue from `to-issues`, and dispatch workers that use `tdd` — without `tdd` they fall back to an inline test-first rule, but they belong together. Install one orchestrator, not both, unless you want the agent to choose by trigger wording.

**Brownfield.** `grill-me` checks your answers against existing code, so it has more to work with on an established codebase than on a blank slate. You can also skip the docs entirely and install only `to-issues`, an orchestrator, and `tdd` — the orchestrator notes the missing glossary and spec once and carries on. If the repo already has an intent document (`docs/INTENT-AND-DESIGN.md`), that is the spec; do not create `docs/REQUIREMENTS-AND-NOTES.md` beside it.

**Harness assumption.** Tested with Cursor. No harness-specific install paths appear anywhere, and each orchestrator states the properties its dispatch needs so you can map them onto another client — but they do assume you can run a fresh sub-agent, or will run workers as separate sessions. `orchestrate-issues-watchdog` will use named `qa` / `watchdog` agents when the client exposes them; otherwise it pastes the role files into a fresh read-only subagent.

**What's proven and what isn't.** The skills, the repo-local tracker, the drift log, and gated unattended workers have 53 serially-delivered issues behind them on a real project. Three mechanisms here do not: the `BACKLOG.md`/`ARCHIVE.md` split, the `proof:` criterion tags, and the orchestrator's preflight are new. They came out of reviewing that project's history for places the gate was thin, and have not themselves been run at length. The watchdog flavor's per-issue QA and `COURSE-CORRECT`/`STOP` trajectory gates were exercised on a separate brownfield project; packaging them as a generic skill is new.

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
  orchestrate-issues-watchdog/
    SKILL.md
    references/
      qa-role.md                    # per-issue QA standing instructions
      watchdog-role.md              # trajectory watchdog standing instructions
  tdd/
    SKILL.md
    references/
      tests.md                      # what a good test looks like
      mocking.md                    # when mocking is legitimate
      deep-modules.md               # small interface, deep implementation
      interface-design.md           # designing for testability
      refactoring.md                # refactor candidates
```

 ## P.S>

 This product includes work derived from [mattpocock/skills](https://github.com/mattpocock/skills)
