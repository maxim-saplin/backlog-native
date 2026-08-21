---
name: orchestrate-issues-watchdog
description: Orchestrate unattended serial delivery of an issue backlog using a parent orchestrator, issue-worker subagents, per-issue adversarial QA, an independent product-intent watchdog, a drift log, and a final integration QA pass. Use when the user wants watchdog-gated AFK delivery, named qa/watchdog gates, or COURSE-CORRECT/STOP trajectory checks. Alternative third stage of the backlog-native pipeline — same queue as `orchestrate-issues`, stricter gates.
---

# Issue orchestration (watchdog)

Parent agent is the **orchestrator**. It plans, dispatches sequentially, accepts/rejects work, handles drift, invokes the independent watchdog on the schedule below, and runs final QA. It does **not** implement issue scope itself except orchestration artifacts (the drift log, queue status, issue wording updates, constitution patches when drift is accepted). `AGENTS.md` is orientation; do not patch product decisions into it.

Stricter flavor of `orchestrate-issues`: per-issue QA before archive, plus a longitudinal watchdog that can block with `COURSE-CORRECT` or `STOP`. Same tracker, same workers, same `tdd` attachment.

## Artifacts

Paths are **defaults** — if the repo already names them differently, its names win.

| Role | Default path | Written by |
|---|---|---|
| Live queue | `issues/BACKLOG.md` | created by `to-issues`; orchestrator sets status and moves rows out |
| Completed queue | `issues/ARCHIVE.md` | created by `to-issues`; orchestrator appends accepted rows |
| Issue files | `issues/NNN-*.md` | worker marks criteria; orchestrator edits wording |
| Drift log | `docs/DECISIONS.md` | orchestrator — sole writer |
| Domain glossary | `CONTEXT.md` | authored by `grill-me`; orchestrator may **patch**, never author |
| Product constitution | `docs/INTENT-AND-DESIGN.md` if present, else `docs/REQUIREMENTS-AND-NOTES.md` | authored by `grill-me`; orchestrator may **patch**, never author |

The constitution is the durable spec. Discover an existing intent document before creating the greenfield default. Never create `docs/REQUIREMENTS-AND-NOTES.md` beside an intent doc that already exists.

## Roles

| Role | Mode | Owns |
|------|------|------|
| **Orchestrator** | read/write | Queue order, dispatch, acceptance gate, queue status, drift log, issue/spec updates |
| **Issue worker** | write | One issue file's scope; marks acceptance criteria; reports drift and evidence |
| **Watchdog** | read-only | Product-intent alignment across issues; can block dispatch with `COURSE-CORRECT` or `STOP` |
| **QA adversarial** | read-only | Focused acceptance review of one candidate issue, its evidence, runnable behavior, and regressions; no fixes |

Five properties matter, however your client provides them:

1. **The worker is a fresh agent** with only the issue file and the docs it names — not the orchestrator continuing in the same context. A worker that has watched the whole session inherits the orchestrator's assumptions and stops being a check on them.
2. **Dispatch is synchronous.** The acceptance gate needs the handoff before the next issue starts.
3. **The QA reviewer is fresh, read-only, and is not the agent that implemented.** QA is a required acceptance gate after every issue worker and after every fix-up. Self-attestation is not review.
4. **The watchdog is read-only and independent of the orchestrator, workers, and QA reviewer.** It audits the delivery trajectory against product intent, not only the current issue's acceptance criteria.
5. **Named-agent selection is opportunistic, not a prerequisite.** Use a dedicated `qa` or `watchdog` agent when the client exposes that name; otherwise use a fresh generic read-only subagent supplied by the harness with the same role prompt. Never skip a gate because the preferred name is unavailable.

Paste [qa-role.md](references/qa-role.md) or [watchdog-role.md](references/watchdog-role.md) into the subagent as standing instructions, then the checkpoint prompt below. Named custom agents are optional; consumers may copy those role files into `.cursor/agents/` or `.github/agents/`.

> **Claude Code:** worker = `Agent` with `subagent_type: general-purpose`, `run_in_background: false`; QA and watchdog use fresh read-only agents, preferring named custom agents when available and otherwise using `subagent_type: Explore`.
>
> **No subagent mechanism?** Run each worker as a fresh session seeded with the dispatch prompt below, and paste its handoff back to the orchestrating session. Slower, same guarantees. What you must not do is let one continuous context implement *and* accept its own work.

## Bootstrap

Read before dispatching the first `todo` issue: the live queue, the domain glossary, the product constitution, and this skill.

Handle each missing artifact explicitly — do not improvise:

| Missing | Do |
|---|---|
| `issues/BACKLOG.md` | **Stop.** Tell the user to break the work down first (the `to-issues` skill owns tracker creation). Never invent a queue. |
| `issues/ARCHIVE.md`, and `BACKLOG.md` has no `done` rows | **Stop** likewise — `to-issues` creates both queue files together, so a tracker missing one is half-scaffolded. |
| `issues/ARCHIVE.md`, but `BACKLOG.md` *has* `done` rows | Not an error — this is the legacy flat tracker (see the row below). Creating `ARCHIVE.md` is part of the migration you offer, not a reason to stop. |
| Drift log | Create it when the first drift is logged |
| Glossary or constitution | Note the absence once and proceed without them, telling each worker which exist. Discover `docs/INTENT-AND-DESIGN.md` then `docs/REQUIREMENTS-AND-NOTES.md`. Do not author them — that is the `grill-me` skill's job. |
| `BACKLOG.md` has `done` rows | A legacy flat tracker. Treat those rows as archived, dispatch none of them, and offer the migration to `ARCHIVE.md` once. |

Build the ordered queue from `BACKLOG.md`; anything in `ARCHIVE.md` is already done.

### Preflight — GO/STOP

Before the **first** dispatch, prove the environment can actually verify work:

| Check | Passes when | If it fails |
|---|---|---|
| Queue is coherent | Every `Blocked by` id resolves; no cycles; no id in both queue files | **Always STOP** |
| Tests run | For issues that change executable behavior: the project's test command executes. For docs, queue, constitution, or physical HITL issues: skip. Do not start the app as a docs or live-hardware preflight. | See the greenfield exception |
| App starts | Only when the next issue needs a runnable surface. Physical HITL and Markdown issues do not. | See the greenfield exception |

**Greenfield exception.** On a project with no test harness yet these checks *cannot* pass, and blocking on them is circular — the issue that would build the harness could never be dispatched. So: **if there is no working test runner**, that is **GO** for the next issue only, provided it is a **scaffold slice** (`to-issues` defines this: skeleton, dependencies, working test runner with one trivial passing test, a way to launch the thing). This holds whether or not the app already starts — a prototype that runs but has no tests still needs the harness before anything else ships. Say so explicitly in your report, deliver it, then re-run the full preflight before the next issue.

If no test runner exists and the next issue is *not* a scaffold slice, STOP — but say plainly what is missing and that `to-issues` should file a scaffold slice as a new blocking issue that the remaining slices depend on. Do not imply the backlog is malformed; a breakdown that omitted it followed every other rule correctly, and do not name a specific id — ids are never reused.

Once a harness exists, a failure means regression, not youth — **STOP and report.** Never dispatch into an environment where "tests green" is unverifiable; a broken harness makes every downstream acceptance gate theatre. Repairing it is its own issue, not something smuggled into a feature.

Re-run the tests check after any rejected handoff.

## Per-issue adversarial QA gate

Run a fresh QA review after every issue worker handoff, before the issue can be archived or the watchdog can approve the next step. QA is the focused acceptance reviewer, not the product-strategy watchdog.

Prefer the dedicated named agent `qa` when the client exposes it. If it is unavailable, invoke a fresh generic read-only subagent available to the harness with this same prompt and role. If the client has no subagent mechanism, run a fresh read-only session seeded with this prompt. The reviewer may execute tests and drive the runnable surface, but must not edit files, implement fixes, change queue state, or act as the watchdog.

Paste [qa-role.md](references/qa-role.md) as standing instructions, then this checkpoint prompt:

```text
Perform focused adversarial acceptance review for one candidate issue.

Issue: <issue path and id>
Worker handoff: <complete handoff>
Changed files and diff: <paths and summary>
Existing tests and commands: <commands and results>
Runtime or migration evidence: <exact commands and observed results, or none>
Governing docs: <intent, glossary, issue, queue, decisions, and relevant postmortems>
Prior QA findings: <summary or none>

Read the repository evidence yourself. Review the implementation and exercise
the narrowest relevant runnable workflow. Check the issue's acceptance criteria,
proof-tagged evidence, user-observable behavior, failure and recovery paths,
scope boundaries, terminology, and regressions in the touched behavior. Treat
tests or file citations without observed behavior as insufficient evidence.

This is issue acceptance, not product-strategy review. Report a possible
product-intent drift as an acceptance risk, but leave the trajectory verdict to
the watchdog. Do not edit files or apply fixes.

Return exactly the output format in the QA role.
```

QA returns `PASS` only when the candidate issue's criteria and evidence are actually discharged. On `FAIL`, leave the issue `in progress`, dispatch concrete implementation fix-ups to a fresh issue worker, and run QA again. Do not run the normal post-issue watchdog gate until QA passes, unless the QA finding itself reports material product drift that requires an immediate watchdog check.

## Watchdog gate

QA runs after every worker and every fix-up. Watchdog does **not** run after every QA PASS. It is a longitudinal product-drift check against the constitution, not issue acceptance.

Run the watchdog:

- after each HITL issue
- after any issue that changes the product constitution or skills (`AGENTS.md` orientation-only edits are not a trigger unless they change a hard stop)
- when a worker or QA reports product drift
- at the end of a sequential AFK group
- when the queue is empty, before final integration QA
- immediately when the orchestrator proposes changing constitution wording

Skip watchdog after a docs-only AFK that did not change constitution or skills. Archive on QA `PASS` in that case.

Prefer the dedicated named agent `watchdog` when the client exposes it. If it is unavailable, invoke a fresh generic read-only subagent with this same prompt and role. The watchdog must be independent of the orchestrator, workers, and QA reviewer.

Provide the watchdog with the checkpoint stage, the latest explicit user decisions, active epic and issue ids, accepted issues since its previous check, worker handoff and evidence, known drift, and the next proposed dispatch. Tell it which constitution, glossary, queue, and decision documents exist. Current explicit user decisions outrank older issue wording. Do not pass `AGENTS.md` as intent.

Paste [watchdog-role.md](references/watchdog-role.md) as standing instructions, then this checkpoint prompt:

```text
Audit this orchestration checkpoint for product drift.

Checkpoint: <candidate-acceptance | periodic | drift-reported | pre-final-QA>
Latest explicit user decisions: <concrete decisions from the current conversation, or none>
Active epic/issues: <ids and paths>
Accepted since previous watchdog: <ids or none>
Current handoff/evidence: <summary with exact paths and commands>
Candidate QA result: <PASS and summary, or not applicable for a drift checkpoint>
Known drift: <summary or none>
Next proposed action: <archive issue NNN, rework issue NNN, dispatch issue NNN, or final integration QA>
Available governing docs: <paths>

Read the repository evidence yourself. Judge whether the current trajectory still delivers the practical user outcome, not merely whether child criteria are checked. Return exactly the verdict format in the watchdog role.
```

If the `watchdog` custom agent is unavailable in the client, use a fresh read-only agent seeded with the same prompt and the watchdog role. Never skip a required checkpoint because the named agent cannot be loaded.

Verdict handling is mandatory:

- `CONTINUE` permits the proposed acceptance or dispatch.
- `COURSE-CORRECT` blocks archive and further dispatch until the orchestrator applies the required issue/order/constitution correction, logs material drift, sends implementation changes through QA again, and re-runs the watchdog.
- `STOP` blocks all dispatch until the user resolves the product or architecture decision and the orchestrator records it, updates the affected artifacts, and re-runs the watchdog.
- The orchestrator cannot silently downgrade or overrule a finding. An override requires an explicit user decision recorded in the drift log.

## Physical HITL

If the user is present and the next issue is physical HITL (hardware, walk-by, live environment), **do not dispatch unattended workers.** The current session implements the issue and takes user observation as `proof: runtime`. Unattended workers cannot walk or click.

## Delivery loop

Dispatch only rows whose `Status` is `todo`. **Any other status is not yours to start** — a `done` row in a live queue is a legacy tracker that predates the archive split, not work waiting to be done. Report those once, offer the migration, and never re-implement them.

For each `todo` row in order (respect blockers):

1. Set issue `Status` → `in progress` in `BACKLOG.md`
2. Dispatch issue worker — fresh agent, synchronous (see Roles)
3. Dispatch the per-issue adversarial QA reviewer on the worker handoff
4. If QA returns `FAIL`, leave the issue `in progress`, dispatch a concrete fix-up worker, and repeat QA until it returns `PASS`
5. After QA `PASS`, if this checkpoint is on the watchdog schedule, run the watchdog before changing queue/archive state. Otherwise archive on QA `PASS`.
6. If the watchdog returns `COURSE-CORRECT` or `STOP`, leave the issue `in progress`, apply the drift protocol or obtain the required user decision, send implementation changes through QA again, and repeat the watchdog gate
7. **Accept** after QA `PASS`, and after watchdog `CONTINUE` when watchdog ran; move the row out of `BACKLOG.md` into `ARCHIVE.md` with `Status` → `done` → next issue

After all `todo` rows are accepted:

- Watchdog final gate across the complete delivered workflow
- Final integration QA pass (read-only agent, never one that implemented); per-issue QA remains mandatory and is not replaced by this pass
- PASS → done | FAIL → fix-ups → re-run QA, then watchdog if the schedule requires it

Default: **serial on main** — one issue at a time.

Moving the row on acceptance is what keeps `BACKLOG.md` short. Do not batch the moves for later; an accepted issue that still sits in the live queue reads as unfinished work.

## Issue worker dispatch

Attach skills **by name** — never by install path.

The block below is copied verbatim into a fresh worker's context, so it has to stand alone — resolve every placeholder before sending, and tell the worker which docs actually exist.

```
Implement issue: issues/NNN-….md

Read first: the issue file, plus whichever of these the project has —
I have told you which: the domain glossary, the product constitution.
Do not treat AGENTS.md as the constitution.

Development method: if this issue is docs, queue, orientation, or physical
HITL with no executable harness change, do not use TDD. Otherwise use the
`tdd` skill and follow it exactly.

You are unattended. There is no user to ask, and nobody will answer a
question. Every question you would have asked becomes a line in your
Drift report instead. Do not stall; do not invent an approval.

Rules:
- Scope limited to this issue only
- Do not invent a second tracker, control surface, or proof-runner
- Do not patch product decisions into AGENTS.md
- Mark acceptance criteria [x] when done
- Criteria tagged `proof: runtime` or `proof: migration` need observed
  evidence — drive the real thing and record what you saw. A passing unit
  test does not discharge them.
- Report drift — do NOT edit the drift log, the queue, or other issue files
- Return exactly this handoff:

  ## Handoff — issue NNN
  ### Acceptance criteria — mirror the issue file; name the proof used for each tagged one
  ### Tests — command + pass/fail
  ### Evidence — for each `proof:` criterion: what was run, what was observed
  ### Drift report (or "none")
  ### Notes for orchestrator
```

**If the `tdd` skill is not installed**, replace only the *Development method* line with: *Implement test-first — one failing test, minimal code to pass, repeat. Tests assert on public behaviour, never on internals. Never refactor while red.* Leave the unattended paragraph and the handoff exactly as they are, and skip the TDD audit in the gate. `orchestrate-issues-watchdog` must never dispatch a prompt naming a skill the worker cannot load.

## Acceptance criteria used by per-issue QA

The QA reviewer must return `FAIL` if any of these fail. The orchestrator must not archive an issue based only on its own reading of the worker handoff:

- Tests green means the cheapest relevant gate for the files and behavior that changed, not an automatic full-suite rerun. If the worker changed only issue/docs/queue artifacts, validate their structure and links and reuse existing executable evidence; do not run the test suite solely for Markdown or queue edits. If the worker changed executable behavior, run its focused gate and add smoke, integration, or full-suite coverage only when the changed surface and acceptance proof require it.
- All acceptance criteria `[x]`
- **Every `proof:`-tagged criterion has matching evidence** — an observed result, not a restatement of intent or a test name
- **The change was exercised, not just tested** when the issue has a runnable surface in scope. Docs, queue, and live HITL that cannot be driven from the workstation do not require starting the app. How to drive it: use whatever run/verify skill the repo provides. If it provides none, start the thing the way its README says to (or its smoke script) and record the exact command plus what was observed. If the project genuinely has no runnable surface yet, say that once in the handoff and move on — do not fabricate an observation, and do not treat the absence as a pass for later issues.
- TDD audit — skip for docs-only issues and when `tdd` was not attached. Otherwise the worker's tests satisfy the `tdd` skill's per-cycle checklist.
- Domain language matches the glossary
- Constitution aligned; unresolved drift → drift protocol

Evidence that only cites a file or a search result does not close a behavioral criterion.

## Drift protocol (orchestrator-only)

Log material drift to the drift log, update issue wording, patch constitution/glossary if truth changed, then re-dispatch or accept. Workers never silently diverge.

Watchdog findings are drift inputs, not advisory review comments. For `COURSE-CORRECT` or `STOP`, record the finding and resolution before dispatch resumes. Do not rewrite intent merely to make delivered work appear aligned; changed intent requires an explicit user decision.

When patching those two documents, two rules matter enough to restate here (a per-skill install cannot reach the `grill-me` skill's format references):

- **Patch, don't append.** Edit the sentence that is now wrong. A constitution that only ever grows becomes archaeology.
- **An Architecture row names its rejected alternative.** A choice recorded without the option it beat reads as an accident and gets silently reversed later. If the constitution is prose, add a short prose decision rather than converting it into a table.
- **The glossary stays a glossary.** Terms only — no implementation detail, no drift narrative.

```md
## YYYY-MM-DD — {title}
**Issue:** NNN | **Drift:** … | **Resolution:** … | **Docs updated:** …
```

## Final QA gate

Run the same dedicated `qa` agent, or the same generic read-only fallback when the named agent is unavailable, as a fresh reviewer for the complete delivered workflow. This final integration pass is broader than any single issue but remains acceptance-focused: inspect the implementation, all issue files, constitution, drift log, evidence, and end-to-end behavior. Return `PASS | FAIL`, findings by severity, exact evidence, and suggested fix-ups. On `FAIL`, apply fix-ups through an issue worker, rerun QA, and then rerun the watchdog.

The reviewer must be a **different agent** than any that implemented, and read-only by construction. Self-attestation is not review.

## Regressions

A `done` issue whose behavior later breaks is a **new** issue, filed via `to-issues` straggler mode. Do not reopen archived rows, edit completed issue files, or move rows back into the live queue — cite the original id in the new issue instead.

### Out of scope

- New CLI/automation wrappers
- `ORCHESTRATION-STATE.md` or other parallel state files (`BACKLOG.md` + `ARCHIVE.md` is the one allowed tracker)
- Parallel git branches (unless user requests)
- External issue trackers — `issues/` plus its queue files is the whole tracker
- Building or repairing the verification harness **yourself** — dispatch it as an issue (see the greenfield exception); never smuggle it into a feature issue
- Authoring the glossary or the constitution — patch them when drift is accepted, but they are `grill-me` output
