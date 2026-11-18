> This "spec-driven" approach has been proven by building from scratch https://github.com/maxim-saplin/model-blend - a LiteLLM proxy blending multiple LLM endpoints into a single one. Worked great with Cursor and Composer 2.5

Four skills for running a development backlog inside your repo. Issues are markdown files in `issues/`, ordered by a queue in `BACKLOG.md`, versioned with the code. No tracker service, no API tokens, no MCP server.

```
  grill-me     ->  to-issues    ->  orchestrate-issues    ->  tdd
  |               |               |                        |
  CONTEXT.md      issues/*.md     row status               the code
  the spec        BACKLOG.md      DECISIONS.md             the tests
                  ARCHIVE.md
```

| Skill                           | Does                                                                                                                              |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **grill-me**                    | Asks one question at a time, checks your answers against the code, and writes the glossary and spec as terms settle.              |
| **to-issues**                   | Splits a plan into thin vertical slices — one numbered markdown file each, plus a row in the queue.                               |
| **orchestrate-issues**          | Delivers authorized issues directly as one agent (or with an optional configured team), uses a temporary goal brief for sustained work, checks intent/proof and logs drift. |
| **tdd**                         | Outcome-driven red/green/refactor: independent oracles, the cheapest adequate boundary, and explicit test strengthening, folding and retirement. |

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
- "Deliver the queue unattended, with your own review, risk-directed QA, and verified commit checkpoints."
- "File a follow-up: the cost column shows stale values after a migration."

On a project with no `issues/` directory, `to-issues` scaffolds the queue files before writing the first issue. On one with no test harness, the orchestrator lets exactly one issue through — the scaffold slice — then requires a working harness before anything else ships.

## Why it's shaped this way

The problem it targets is spec rot. An agent hits something the spec didn't anticipate, makes a reasonable call, and moves on. Nothing records that the spec is now wrong, so after fifty issues it describes a system that doesn't exist.

Three rules follow from that:

- **Each document has one job.** Terminology in `CONTEXT.md`, architectural trade-offs in the spec with the rejected option named, delivery surprises in a drift log.
- **Material ambiguity is resolved before acting.** Unattended workers escalate product conflicts and hard-to-reverse choices to the parent. Routine reversible decisions stay within the approved contract; meaningful drift is reported and recorded.
- **Humans gate design and acceptance, not implementation.** Issues are typed `AFK` or `HITL` so agents know which they may finish alone.

## Notes

**Dependencies.** `grill-me`, `to-issues`, and `tdd` each stand alone. The orchestrator needs a queue from `to-issues` and dispatches workers that use `tdd`. Without `tdd` it falls back to an inline test-first rule, but they belong together.

**Brownfield.** `grill-me` checks your answers against existing code, so it has more to work with on an established codebase than on a blank slate. You can also skip the docs entirely and install only `to-issues`, an orchestrator, and `tdd` — the orchestrator notes the missing glossary and spec once and carries on. If the repo already has an intent document (`docs/INTENT-AND-DESIGN.md`), that is the spec; do not create `docs/REQUIREMENTS-AND-NOTES.md` beside it.

**Single-agent first.** The current agent can plan, implement, verify and deliver the authorized checkpoint without a roster, fresh worker, timer or messaging infrastructure. Project-configured teams are an option, not an implicit instruction to create agents. Independent review, when used, is separate/read-only; self-review is never relabeled independent QA. If none is available, say so and record direct verification; explicit external-review requirements still apply. A named worktree is not tool isolation: verify paths/permissions and coordinate shared writers.

**Temporary goal brief, not a permanent dashboard.** Create ignored root `STATUS.md` only for an active approved assignment spanning sessions, several meaningful outcomes or work while the user is away. Lead with the goal and observable done condition, then 3–5 outcome-based steps with one NOW; normally 100–150 words, one Updated timestamp and only an actionable blocker/user choice. If needed, add a compact issue/name/status table after the bullets; don't replace the goal or summary. Only actively worked or running work is active; staged and waiting work stay distinct. Keep sprint updates in the brief, not the queue. No hashes, process ledger, policy recital or heartbeat/history dump. One agent/publisher owns it. Record the conclusion in the owning issue/handoff and retire it when done, cancelled or deferred long term: **no active assignment, no file**. Short fixes/discussion need none. The installed [lifecycle/template](skills/orchestrate-issues/references/operator-briefing.md) ships with the orchestrator.

**Permission follows the request.** Discussion alone writes no files; an explicit bounded request performs its named action and stops; autonomous ticket delivery requires an approved assignment. Side context is not filesystem isolation. A human may talk directly to a configured member, who checks existing obligations rather than silently preempt work. No new lead approval is needed for an independent authorized task. Follow local repository publication policy; remote issue reservation is conditional, not a requirement for an offline project.

**What's proven and what isn't.** The original skills, repo-local tracker, drift log, and gated unattended workers originally had 53 serially-delivered issues behind them on a real project, and have since evolved through 160+ issues in complex brownfield production environments. The separate watchdog gate was retired after failing in the field (described below). The subsequent direct parent-review model was then battle-tested and refined with critical field experience: mandatory parent judgment with concrete scenario-outcome checks, risk-directed QA targeting specific uncertainties rather than an automatic per-issue tax, verified commit checkpoints to prevent review churn across large issues, explicit triage of QA findings (separating blockers from nonblocking suggestions and hypotheses), and mechanism growth tracking to stop unchecked helper/script sprawl.

## Why no watchdog

A separate watchdog was tried and removed on 2026-10-03. It approved a working
single-area demo as a reusable foundation while missing the bigger picture. Fresh context did not produce independent judgment: the checkpoint was
framed by worker handoffs, QA PASS, and the proposed next action. Its approvals
added token cost and false assurance instead of correcting direction.

The replacement is not another approval agent. The parent must inspect the
actual changes against user intent before acceptance and challenge shared or
reusable claims with a cheap cross-client or second-configuration check. QA
continues to hunt functional bugs and missing behavioral proof. Neither green
tests nor QA PASS relieves the parent of its acceptance responsibility.

Existing consumers should remove the retired watchdog skill and custom agent
definitions when updating; installing a skill does not remove custom agents.

**Name collision.** `tdd` is a generic name — if your project or another bundle already claims it, installing this overwrites it; use `--skill` to choose. `grill-me` and `tdd` are adapted from [`mattpocock/skills`](https://github.com/mattpocock/skills) (MIT — see [NOTICE](NOTICE)), where they are named `grill-with-docs` and `tdd`; only the latter still collides.

**Executable claims need executable proof.** A broken harness cannot verify executable delivery; repair or scaffold it before dependent feature work. Documentation-only changes need structural review, not an invented test suite or app launch. Green tests alone do not establish product value: inspect the maintained SUT, independent oracle and distinct fault protection. Strengthen or fold existing checks before adding cases; do not pursue count, coverage or runtime targets as substitutes for useful protection.

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
    references/
      operator-briefing.md          # ignored STATUS.md contract and template
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
