**Spec-driven development with no SaaS issue tracker.** Four skills that take a vague idea to delivered code: a grilling session that pins down what you actually mean, a breakdown into vertical slices, and unattended agent workers held to an acceptance gate. The backlog lives in your repo as markdown, versioned with the code it describes.

No API tokens. No MCP server. No project-management account. `issues/` **is** the tracker.

Tested with Claude Code. The skills name no harness-specific install paths, and `orchestrate-issues` states the three properties its worker dispatch needs so you can map them onto whatever your client offers — but it does assume you *have* a way to run a fresh sub-agent, or are willing to run workers as separate sessions.

## Why

Handing an agent a feature and getting a pull request back is easy. Getting work you'd actually ship, repeatedly, over months, is not — and the thing that breaks first isn't the code.

> Specs rot silently. An agent hits something the spec didn't anticipate, makes a reasonable call, and moves on. Nothing records that the spec is now wrong. Fifty issues later the document describes a system that no longer exists, and nobody trusts it.

So this bundle makes three things load-bearing:

**Documents have assigned roles, and skills respect them.** A glossary is not a spec is not a decision log. Terminology goes in `CONTEXT.md` and nothing else does. Architectural trade-offs go in the spec with the rejected alternative named. Delivery-time surprises go in a drift log. When an agent learns something that contradicts a document, updating the document is part of the job.

**Drift is an artifact, not an accident.** Unattended workers cannot ask you questions, so every question becomes a written drift report the orchestrator resolves and records. The spec stays true because divergence is visible.

**Humans gate design and acceptance, not implementation.** You get grilled hard up front and you judge the result. In between, work is typed `AFK` or `HITL` so agents know what they may finish alone.

## The pipeline

```
   grill-with-docs  →  to-issues  →  orchestrate-issues  →  tdd
        │                  │                │                │
   CONTEXT.md         issues/*.md      row status        the code
   the spec           BACKLOG.md       DECISIONS.md      the tests
                      ARCHIVE.md
```

| Skill | Does | Owns |
|---|---|---|
| **grill-with-docs** | Interviews you one question at a time, challenges claims against the code, sharpens vocabulary | Authors the glossary + spec |
| **to-issues** | Breaks the settled plan into thin vertical slices as numbered markdown files | Creates both queue files; writes issue files |
| **orchestrate-issues** | Dispatches unattended workers serially, gates each handoff, logs drift, runs adversarial QA | Row status and moves; drift log |
| **tdd** | Red/green/refactor with tracer bullets; has an unattended mode for delegated work | Code + tests |

`grill-with-docs`, `to-issues`, and `tdd` each stand alone. `orchestrate-issues` expects `to-issues` to have produced a queue, and dispatches workers that use `tdd` — install it without `tdd` and it falls back to an inline test-first rule, but the pair belongs together.

## Greenfield and brownfield

Greenfield gets the structure. **Brownfield gets the archaeology** — and `grill-with-docs` is *better* on an existing codebase, because its method is cross-referencing what you claim against what the code does and surfacing the contradictions. On a true blank slate there is nothing to grill against. On a codebase someone else wrote, or one an agent generated while you weren't reading closely, that interrogation is the point.

You can also adopt the back half only: `to-issues` plus `orchestrate-issues` (plus `tdd`) will drive delivery on a project whose docs you never intend to write — the orchestrator notes the missing glossary and spec once and carries on without them.

## How it worked for us

Built on a real project — an LLM proxy exposing three orchestration modes — delivered issue by issue:

|                           |                           |
|---------------------------|---------------------------|
| Issues delivered serially | **53** (ids up to 054)    |
| Test modules              | 40                        |
| Docs kept in sync         | glossary, spec, drift log |
| Tracker services used     | **0**                     |

The pipeline's own gaps showed up honestly. An independent black-box acceptance pass — reading the docs as a user and running the real software against a live API — caught problems the per-issue gate had waved through, for about half a cent of spend. That is why the gate here now demands the app be *driven*, not just tested.

**What has and hasn't been proven.** The four skills, the repo-local tracker, the drift log, and unattended workers under an acceptance gate all have those 53 issues behind them. Three mechanisms in this bundle do **not**: the `BACKLOG.md`/`ARCHIVE.md` split, the `proof:` criterion tags, and the orchestrator's preflight are new here — the reference project ran with a single flat backlog, no proof tags, and no preflight. They exist because reviewing that project's history showed where the gate was thin, not because they have miles on them.

## Install

From your project root (or add `-g` for a global install):

```bash
npx skills add maxim-saplin/backlog-native --all
```

`--all` installs every skill into every agent directory it detects, without prompting. To look first, or pick a subset:

```bash
npx skills add maxim-saplin/backlog-native --list
npx skills add maxim-saplin/backlog-native --skill to-issues --skill orchestrate-issues --skill tdd
```

> **Name collision:** `tdd` and `grill-with-docs` keep the names they have upstream in [`mattpocock/skills`](https://github.com/mattpocock/skills), which these two are adapted from. If you already have those installed, this will overwrite them — use `--skill` to choose. `tdd` is a generic enough name that your own project may have claimed it too.

**Manual alternative:** copy the directories under [`skills/`](skills/) (each `SKILL.md` plus its `references/`) into your agent's skills folder — e.g. `.agents/skills/` (works across many harnesses) or `.claude/skills/` (Claude Code).

## Repository layout

```
README.md
LICENSE
NOTICE                              # attribution for the two adapted skills
.gitattributes
.gitignore

skills/
  grill-with-docs/
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

## Use

Start a session and ask in plain language:

- "Grill me on this feature until we both understand it, then update the glossary."
- "Break that into issues."
- "Work through the backlog."
- "File a follow-up: the cost column shows stale values after a migration."

The tracker bootstraps itself — on a project with no `issues/` directory, `to-issues` scaffolds `BACKLOG.md` and `ARCHIVE.md` before writing the first issue. On a brand-new project with no test harness yet, the orchestrator's preflight allows exactly one issue through — the scaffold slice — then requires a working harness before anything else ships.

> **Biggest risk:** an acceptance gate that cannot see. If your agent can't run the tests *and* exercise the app, "tests green" is a claim you cannot check, and the orchestrator's preflight will stop rather than pretend. Fix the feedback loop first — everything here compounds on it.
