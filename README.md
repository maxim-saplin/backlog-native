**Spec-driven development with no SaaS issue tracker.** Four skills that take a vague idea to delivered code: a grilling session that pins down what you actually mean, a breakdown into vertical slices, and unattended agent workers held to an acceptance gate. The backlog lives in your repo as markdown, versioned with the code it describes.

No API tokens. No MCP server. No project-management account. `issues/` **is** the tracker.

Tested with Claude Code; the skills name no harness-specific paths and should port to any Agent Skills client.

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
   CONTEXT.md         issues/*.md      ARCHIVE.md       the code
   the spec           BACKLOG.md       DECISIONS.md     the tests
```

| Skill | Does | Owns |
|---|---|---|
| **grill-with-docs** | Interviews you one question at a time, challenges claims against the code, sharpens vocabulary | Glossary + spec |
| **to-issues** | Breaks the settled plan into thin vertical slices as numbered markdown files | Issue files + live queue |
| **orchestrate-issues** | Dispatches unattended workers serially, gates each handoff, logs drift, runs adversarial QA | Queue status + drift log |
| **tdd** | Red/green/refactor with tracer bullets; has an unattended mode for delegated work | Code + tests |

Each skill works alone. The value compounds when they share the artifacts.

## Greenfield and brownfield

Greenfield gets the structure. **Brownfield gets the archaeology** — and `grill-with-docs` is *better* on an existing codebase, because its method is cross-referencing what you claim against what the code does and surfacing the contradictions. On a true blank slate there is nothing to grill against. On a codebase someone else wrote, or one an agent generated while you weren't reading closely, that interrogation is the point.

You can also adopt the back half only: `to-issues` plus `orchestrate-issues` will drive delivery on a project whose docs you never intend to write.

## How it worked for us

Built on a real project — an LLM proxy with five orchestration modes, delivered issue-by-issue:

|                          |                              |
|--------------------------|------------------------------|
| Issues delivered serially | **54**                      |
| Test files               | 44                           |
| Docs kept in sync        | glossary, spec, drift log    |
| Tracker services used    | **0**                        |

The pipeline's own gaps showed up honestly. An independent black-box acceptance pass — reading the docs as a user and running the real software — caught problems the per-issue gate had waved through, for about half a cent of API spend. That gap is why the acceptance gate in this bundle now demands the app be *driven*, not just tested, and why criteria touching routing, auth, or schemas carry a proof class a unit test cannot discharge.

## Install

From your project root (or add `-g` for a global install):

```bash
npx skills add maxim-saplin/backlog-native --all
```

Pick a subset instead, or see what's there first:

```bash
npx skills add maxim-saplin/backlog-native --list
npx skills add maxim-saplin/backlog-native --skill to-issues --skill orchestrate-issues
```

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

The tracker bootstraps itself — on a project with no `issues/` directory, `to-issues` scaffolds `BACKLOG.md` before writing the first issue.

> **Biggest risk:** an acceptance gate that cannot see. If your agent can't run the tests *and* exercise the app, "tests green" is a claim you cannot check, and the orchestrator's preflight will stop rather than pretend. Fix the feedback loop first — everything here compounds on it.
