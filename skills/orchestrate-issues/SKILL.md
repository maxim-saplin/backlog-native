---
name: orchestrate-issues
description: Orchestrate serial end-to-end delivery of an issue backlog using a parent orchestrator, unattended issue-worker subagents, a per-issue acceptance gate, a drift log, and a final adversarial QA pass. Use when the user wants MVP or backlog delivery via issues, wants to work through issues/BACKLOG.md, or asks to deliver a queue of issues. Third stage of the backlog-native pipeline — delivers the queue the `to-issues` skill produced, dispatching workers that follow the `tdd` skill.
---

# Issue orchestration

Parent agent is the **orchestrator**. It plans, dispatches sequentially, accepts/rejects work, handles drift, and runs final QA. It does **not** implement issue scope itself except orchestration artifacts (the drift log, queue status, issue wording updates, spec patches when drift is accepted).

## Artifacts

Paths are **defaults** — if the repo already names them differently, its names win.

| Role | Default path | Written by |
|---|---|---|
| Live queue | `issues/BACKLOG.md` | created by `to-issues`; orchestrator sets status and moves rows out |
| Completed queue | `issues/ARCHIVE.md` | created by `to-issues`; orchestrator appends accepted rows |
| Issue files | `issues/NNN-*.md` | worker marks criteria; orchestrator edits wording |
| Drift log | `docs/DECISIONS.md` | orchestrator — sole writer |
| Domain glossary | `CONTEXT.md` | authored by `grill-me`; orchestrator may **patch**, never author |
| Product spec | `docs/REQUIREMENTS-AND-NOTES.md` | authored by `grill-me`; orchestrator may **patch**, never author |

## Roles

| Role | Mode | Owns |
|------|------|------|
| **Orchestrator** | read/write | Queue order, dispatch, acceptance gate, queue status, drift log, issue/spec updates |
| **Issue worker** | write | One issue file's scope; marks acceptance criteria; reports drift and evidence |
| **QA adversarial** | read-only | Holistic review vs product spec; no fixes |

Three properties matter, however your client provides them:

1. **The worker is a fresh agent** with only the issue file and the docs it names — not the orchestrator continuing in the same context. A worker that has watched the whole session inherits the orchestrator's assumptions and stops being a check on them.
2. **Dispatch is synchronous.** The acceptance gate needs the handoff before the next issue starts.
3. **The QA reviewer is read-only and is not the agent that implemented.** Self-attestation is not review.

> **Claude Code:** worker = `Agent` with `subagent_type: general-purpose`, `run_in_background: false`; QA = `Agent` with `subagent_type: Explore`, which is read-only by construction.
>
> **No subagent mechanism?** Run each worker as a fresh session seeded with the dispatch prompt below, and paste its handoff back to the orchestrating session. Slower, same guarantees. What you must not do is let one continuous context implement *and* accept its own work.

## Bootstrap

Read before dispatching the first `todo` issue: the live queue, the domain glossary, the product spec, and this skill.

Handle each missing artifact explicitly — do not improvise:

| Missing | Do |
|---|---|
| `issues/BACKLOG.md` | **Stop.** Tell the user to break the work down first (the `to-issues` skill owns tracker creation). Never invent a queue. |
| `issues/ARCHIVE.md`, and `BACKLOG.md` has no `done` rows | **Stop** likewise — `to-issues` creates both queue files together, so a tracker missing one is half-scaffolded. |
| `issues/ARCHIVE.md`, but `BACKLOG.md` *has* `done` rows | Not an error — this is the legacy flat tracker (see the row below). Creating `ARCHIVE.md` is part of the migration you offer, not a reason to stop. |
| Drift log | Create it when the first drift is logged |
| Glossary or spec | Note the absence once and proceed without them, telling each worker which exist. Do not author them — that is the `grill-me` skill's job. |
| `BACKLOG.md` has `done` rows | A legacy flat tracker. Treat those rows as archived, dispatch none of them, and offer the migration to `ARCHIVE.md` once. |

Build the ordered queue from `BACKLOG.md`; anything in `ARCHIVE.md` is already done.

### Preflight — GO/STOP

Before the **first** dispatch, prove the environment can actually verify work:

| Check | Passes when | If it fails |
|---|---|---|
| Tests run | The project's test command executes and reports — green, or a *known* red; not an import error or a missing runner | See the greenfield exception |
| App starts | The thing under development can be launched, or you can state plainly it has no runnable surface yet | See the greenfield exception |
| Queue is coherent | Every `Blocked by` id resolves; no cycles; no id in both queue files | **Always STOP** — this is a broken backlog, not a young project |

**Greenfield exception.** On a project with no test harness yet these checks *cannot* pass, and blocking on them is circular — the issue that would build the harness could never be dispatched. So: **if there is no working test runner**, that is **GO** for the next issue only, provided it is a **scaffold slice** (`to-issues` defines this: skeleton, dependencies, working test runner with one trivial passing test, a way to launch the thing). This holds whether or not the app already starts — a prototype that runs but has no tests still needs the harness before anything else ships. Say so explicitly in your report, deliver it, then re-run the full preflight before the next issue.

If no test runner exists and the next issue is *not* a scaffold slice, STOP — but say plainly what is missing and that `to-issues` should file a scaffold slice as a new blocking issue that the remaining slices depend on. Do not imply the backlog is malformed; a breakdown that omitted it followed every other rule correctly, and do not name a specific id — ids are never reused.

Once a harness exists, a failure means regression, not youth — **STOP and report.** Never dispatch into an environment where "tests green" is unverifiable; a broken harness makes every downstream acceptance gate theatre. Repairing it is its own issue, not something smuggled into a feature.

Re-run the tests check after any rejected handoff.

## Delivery loop

Dispatch only rows whose `Status` is `todo`. **Any other status is not yours to start** — a `done` row in a live queue is a legacy tracker that predates the archive split, not work waiting to be done. Report those once, offer the migration, and never re-implement them.

For each `todo` row in order (respect blockers):

1. Set issue `Status` → `in progress` in `BACKLOG.md`
2. Dispatch issue worker — fresh agent, synchronous (see Roles)
3. Run acceptance gate on handoff
4. **Accept** → move the row out of `BACKLOG.md` into `ARCHIVE.md` with `Status` → `done` → next issue
5. **Reject** → leave `in progress` (or revert to `todo` if abandoning) → re-dispatch with concrete gaps

After all `todo` rows are accepted:

- Adversarial QA pass (read-only agent, never one that implemented)
- PASS → done | FAIL → fix-ups → re-run QA

Default: **serial on main** — one issue at a time.

Moving the row on acceptance is what keeps `BACKLOG.md` short. Do not batch the moves for later; an accepted issue that still sits in the live queue reads as unfinished work.

## Issue worker dispatch

Attach skills **by name** — never by install path (`.claude/skills/…` vs `.agents/skills/…` varies by client).

The block below is copied verbatim into a fresh worker's context, so it has to stand alone — resolve every placeholder before sending, and tell the worker which docs actually exist.

```
Implement issue: issues/NNN-….md

Read first: the issue file, plus whichever of these the project has —
I have told you which: the domain glossary, the product spec.

Development method: use the `tdd` skill and follow it exactly.

You are unattended. There is no user to ask, and nobody will answer a
question. Every question you would have asked becomes a line in your
Drift report instead. Do not stall; do not invent an approval.

Rules:
- Scope limited to this issue only
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

**If the `tdd` skill is not installed**, replace only the *Development method* line with: *Implement test-first — one failing test, minimal code to pass, repeat. Tests assert on public behaviour, never on internals. Never refactor while red.* Leave the unattended paragraph and the handoff exactly as they are, and skip the TDD audit in the gate. `orchestrate-issues` must never dispatch a prompt naming a skill the worker cannot load.

## Acceptance gate (per issue)

Reject if any fail:

- Tests green
- All acceptance criteria `[x]`
- **Every `proof:`-tagged criterion has matching evidence** — an observed result, not a restatement of intent or a test name
- **The change was exercised, not just tested.** For anything with a runnable surface, the handoff must show the app was driven and what happened. Tests-green-only is not acceptance for behavior a user can see.

  *How* to drive it: use whatever run/verify skill the repo provides. If it provides none, start the thing the way its README says to (or its smoke script) and record the exact command plus what was observed. If the project genuinely has no runnable surface yet, say that once in the handoff and move on — do not fabricate an observation, and do not treat the absence as a pass for later issues.
- TDD audit — the worker's tests satisfy the `tdd` skill's per-cycle checklist: each test describes behavior not implementation, uses only the public interface, and would survive an internal refactor. Judge this from the diff, not from the worker's say-so. Skip if `tdd` is not installed.
- Domain language matches the glossary
- Spec aligned; unresolved drift → drift protocol

Evidence that only cites a file or a search result does not close a behavioral criterion.

## Drift protocol (orchestrator-only)

Log material drift to the drift log, update issue wording, patch spec/glossary if truth changed, then re-dispatch or accept. Workers never silently diverge.

When patching those two documents, two rules matter enough to restate here (a per-skill install cannot reach the `grill-me` skill's format references):

- **Patch, don't append.** Edit the sentence that is now wrong. A spec that only ever grows becomes archaeology.
- **An Architecture row names its rejected alternative.** A choice recorded without the option it beat reads as an accident and gets silently reversed later.
- **The glossary stays a glossary.** Terms only — no implementation detail, no drift narrative.

```md
## YYYY-MM-DD — {title}
**Issue:** NNN | **Drift:** … | **Resolution:** … | **Docs updated:** …
```

## Final QA gate

Read-only adversarial review vs spec + drift log + all issue files. Return PASS | FAIL, findings by severity, suggested fix-ups. On FAIL → fix-up loop → re-run QA.

The reviewer must be a **different agent** than any that implemented, and read-only by construction. Self-attestation is not review.

## Regressions

A `done` issue whose behavior later breaks is a **new** issue, filed via `to-issues` straggler mode. Do not reopen archived rows, edit completed issue files, or move rows back into the live queue — cite the original id in the new issue instead.

### Out of scope

- New CLI/automation wrappers
- `ORCHESTRATION-STATE.md` or other parallel state files (`BACKLOG.md` + `ARCHIVE.md` is the one allowed tracker)
- Parallel git branches (unless user requests)
- External issue trackers — `issues/` plus its queue files is the whole tracker
- Building or repairing the verification harness **yourself** — dispatch it as an issue (see the greenfield exception); never smuggle it into a feature issue
- Authoring the glossary or the spec — patch them when drift is accepted, but they are `grill-me` output
