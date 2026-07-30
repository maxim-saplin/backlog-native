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
| Live queue | `issues/BACKLOG.md` | orchestrator |
| Completed queue | `issues/ARCHIVE.md` | orchestrator |
| Issue files | `issues/NNN-*.md` | worker marks criteria; orchestrator edits wording |
| Drift log | `docs/DECISIONS.md` | orchestrator only |
| Domain glossary | `CONTEXT.md` | orchestrator only |
| Product spec | `docs/REQUIREMENTS-AND-NOTES.md` | orchestrator only |

## Roles

| Role | Dispatch | Mode | Owns |
|------|----------|------|------|
| **Orchestrator** | Parent agent | read/write | Queue order, dispatch, acceptance gate, queue status, drift log, issue/spec updates |
| **Issue worker** | `Agent` → `subagent_type: general-purpose`, `run_in_background: false` | write | One issue file's scope; marks acceptance criteria; reports drift and evidence |
| **QA adversarial** | `Agent` → `subagent_type: Explore` (read-only by construction) | read-only | Holistic review vs product spec; no fixes |

Dispatch workers with `run_in_background: false` — the acceptance gate needs the handoff before the next issue starts.

## Bootstrap

Read before dispatching the first `todo` issue: the live queue, the domain glossary, the product spec and initial intent, and this skill.

Create the drift log if missing. Build the ordered queue from `BACKLOG.md`; anything in `ARCHIVE.md` is already done.

### Preflight — GO/STOP

Before the **first** dispatch, prove the environment can actually verify work:

| Check | Passes when |
|---|---|
| Tests run | The project's test command executes and reports (green, or a *known* red — not an import error or a missing runner) |
| App starts | The thing under development can be launched, or you can state plainly that it has no runnable surface |
| Queue is coherent | Every `Blocked by` id resolves; no cycles; no id in both `BACKLOG.md` and `ARCHIVE.md` |

**Any failure → STOP and report.** Never dispatch a worker into an environment where "tests green" is unverifiable — a broken harness makes every downstream acceptance gate theatre. Repairing the harness is its own issue, filed via `to-issues`, not smuggled into the first feature.

Re-run the tests check after any rejected handoff.

## Delivery loop

For each issue in order (respect blockers):

1. Set issue `Status` → `in progress` in `BACKLOG.md`
2. Dispatch issue worker (`Agent`)
3. Run acceptance gate on handoff
4. **Accept** → move the row out of `BACKLOG.md` into `ARCHIVE.md` with `Status` → `done` → next issue
5. **Reject** → leave `in progress` (or revert to `todo` if abandoning) → re-dispatch with concrete gaps

After all issues accepted:

- QA adversarial subagent
- PASS → done | FAIL → fix-ups → re-run QA

Default: **serial on main** — one issue at a time.

Moving the row on acceptance is what keeps `BACKLOG.md` short. Do not batch the moves for later; an accepted issue that still sits in the live queue reads as unfinished work.

## Issue worker dispatch

Attach skills **by name** — do not inline TDD rules, and do not hardcode an install path (`.claude/skills/…` vs `.agents/skills/…` varies by client).

```
Implement issue: issues/00N-….md

Read first: the domain glossary, the product spec, the issue file

Development method: use the `tdd` skill and follow it exactly, running
unattended (see its "Running unattended" section — there is no user to ask).

Rules:
- Scope limited to this issue only
- Mark acceptance criteria [x] when done
- Criteria tagged `proof: runtime` or `proof: migration` need observed
  evidence — drive the real thing and record what you saw. A passing unit
  test does not discharge them.
- Return structured handoff (below)
- Report drift — do NOT edit the drift log, the queue, or other issue files
```

Workers are unattended: they cannot ask the user anything. Every question a worker would have asked comes back as a drift report line for the orchestrator to resolve.

### Handoff schema

```md
## Handoff — issue 00N
### Acceptance criteria — mirror issue file, note the proof used for each tagged one
### Tests — command + pass/fail
### Evidence — for each `proof:` criterion: what was run, what was observed
### Drift report (or "none")
### Notes for orchestrator
```

## Acceptance gate (per issue)

Reject if any fail:

- Tests green
- All acceptance criteria `[x]`
- **Every `proof:`-tagged criterion has matching evidence** — an observed result, not a restatement of intent or a test name
- **The change was exercised, not just tested.** For anything with a runnable surface, the handoff must show the app was driven and what happened. *How* to drive it is the project's business — use whatever run/verify skill the repo provides. Tests-green-only is not acceptance for behavior a user can see.
- TDD audit (`tdd` skill checklist)
- Domain language matches the glossary
- Spec aligned; unresolved drift → drift protocol

Evidence that only cites a file or a search result does not close a behavioral criterion.

## Drift protocol (orchestrator-only)

Log material drift to the drift log, update issue wording, patch spec/glossary if truth changed, then re-dispatch or accept. Workers never silently diverge.

```md
## YYYY-MM-DD — {title}
**Issue:** 00N | **Drift:** … | **Resolution:** … | **Docs updated:** …
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
- Building the verification harness — preflight *reports* a broken one; repairing it is a separate issue
