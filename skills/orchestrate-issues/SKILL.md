---
name: orchestrate-issues
description: Orchestrate unattended serial delivery of an issue backlog with named-team-first dispatch (fresh-worker fallback), mandatory parent judgment, risk-directed QA, verified commit checkpoints, and a drift log. Use when the user asks to orchestrate, deliver a backlog AFK, or work through issues/BACKLOG.md unattended. Do not use for occupant-led physical or live HITL. Third stage of the backlog-native pipeline; workers follow the tdd skill.
---

# Issue orchestration

Parent agent is the **orchestrator**. It plans, dispatches sequentially, reviews the actual changes itself, selects QA, accepts or rejects work, handles drift, and verifies the integrated result. Worker completion and QA verdicts are evidence, not acceptance decisions. The parent owns product direction, proportionality, and maintainability; it cannot delegate that responsibility to another approval agent.

For substantial implementation, prefer an established project-defined team (for example, a `PI_TEAM.md` roster with named dev + QA) over anonymous fresh workers. Fresh agents remain the fallback when no established team exists, and small subagent work is bounded and authorized only. Dispatch briefs carry the absolute repository root, a verified reference commit, the initial dirty paths, the absolute issue and governing paths, and — when material — the actual session model (verified, not inferred from a role label).

The parent does **not** implement issue scope itself except orchestration artifacts (the drift log, queue status, issue wording updates, spec patches when drift is accepted). It may integrate a reviewed external patch as the sole repository writer; that does not make its own integration an independent review. `AGENTS.md` is operating orientation, not product intent.

## Artifacts

Paths are **defaults** — if the repo already names them differently, its names win.

| Role | Default path | Written by |
|---|---|---|
| Live queue | `issues/BACKLOG.md` | created by `to-issues`; orchestrator sets status and moves rows out |
| Completed queue | `issues/ARCHIVE.md` | created by `to-issues`; orchestrator appends accepted rows |
| Issue files | `issues/NNN-*.md` | worker marks criteria; orchestrator edits wording |
| Drift log | `docs/DECISIONS.md` | orchestrator — sole writer |
| Domain glossary | `CONTEXT.md` | authored by `grill-me`; orchestrator may **patch**, never author |
| Product spec | existing intent document, else `docs/REQUIREMENTS-AND-NOTES.md` | authored by `grill-me`; orchestrator may **patch**, never author |

Discover an existing `docs/INTENT-AND-DESIGN.md` before using the greenfield default. Do not create a competing product spec.

## Roles

| Role | Mode | Owns |
|------|------|------|
| **Orchestrator** | read/write | Direct code and goal review, drift decisions, dispatch, acceptance, queue state, issue/spec updates |
| **Issue worker** | write | One issue file's scope; marks acceptance criteria; reports drift and evidence |
| **QA adversarial** | read-only | Functional correctness, missing behavioral proof, failure paths, and regressions; no fixes |

Four properties matter, however your client provides them:

1. **Dispatch to the named team first; fresh worker otherwise.** For substantial work use the named team member from the project's roster (dev implements, QA reviews). When no established team exists, the worker is a fresh agent with only the issue file and the docs it names — not the orchestrator continuing in the same context. A worker that has watched the whole session inherits the orchestrator's assumptions and stops being a check on them.
2. **Dispatch follows dependency order.** Serial processing is the default; a dependent issue waits for its predecessor's handoff and acceptance. Completion may be async where no dependency exists; do not force a blocking tool where async completion is valid.
3. **When selected, the QA reviewer is independent and read-only, and is not the agent that implemented.** The parent chooses QA under the risk rules below. Worker self-attestation never replaces parent review.
4. **The parent sees for itself before acceptance.** Read the changes and form a judgment against the user's goal, not only the worker's interpretation of the issue. An independent reviewer does not remove the parent's responsibility.

When QA is needed, use a named `qa` agent when available; otherwise supply the QA prompt below to a read-only subagent. Agent availability is not a reason to waive a required independent check.

> **Claude Code:** worker = `Agent` with `subagent_type: general-purpose`, `run_in_background: false`; QA = `Agent` with `subagent_type: Explore`, which is read-only by construction.
>
> **No subagent mechanism?** Use the established named team member; otherwise run a fresh session seeded with the dispatch prompt below. Return its handoff to the orchestrating session. When independent QA is required, obtain a separate read-only review; do not replace it with worker self-review.

## Current task and permission boundary

Use one current brief with task/checkpoint ID, goal, reference, absolute output
paths, allowed writes/resources and stop condition. Resolve relative artifact
paths against that brief's root. Acknowledge the task, then begin; do not add a
second approval wait unless explicitly required.

A new phase supersedes old task-specific write/runtime permissions. Delayed
messages and recalled results do not reactivate them. If task identity, source
base or permission is uncertain, stop the affected work and ask the parent.
Repeated drift requires a fresh bounded context, reassignment or a pause, not
indefinite rebriefing. Do not invent fresh tool provenance from an archived hash.

Verify actual tool cwd/root, HEAD and allowed absolute targets at task start and
after a phase/root change. Naming a worktree does not move a peer's tools into it.
If write isolation is not enforced, use external candidates and one repository
writer; preserve unrelated changes. Fake-body probes are still execution: their
no-I/O claim needs independent transport controls before imports/invocation, not
an empty recorder. Actual runtime checks need the owning procedure's permission
and target isolation. Do not duplicate that procedure here.

## Bootstrap

Read before dispatching the first `todo` issue: the live queue, the domain glossary, the product spec, and this skill.

Handle each missing artifact explicitly — do not improvise:

| Missing | Do |
|---|---|
| `issues/BACKLOG.md` | **Stop.** Tell the user to break the work down first (the `to-issues` skill owns tracker creation). Never invent a queue. |
| `issues/ARCHIVE.md`, and `BACKLOG.md` has no `done` rows | **Stop** likewise — `to-issues` creates both queue files together, so a tracker missing one is half-scaffolded. |
| `issues/ARCHIVE.md`, but `BACKLOG.md` *has* `done` rows | Not an error — this is the legacy flat tracker (see the row below). Creating `ARCHIVE.md` is part of the migration you offer, not a reason to stop. |
| Drift log | Create it when the first drift is logged |
| Glossary or spec | Note the absence once and proceed without them, telling each worker which exist. Look for an existing intent document first. Do not author them — that is the `grill-me` skill's job. |
| `BACKLOG.md` has `done` rows | A legacy flat tracker. Treat those rows as archived, dispatch none of them, and offer the migration to `ARCHIVE.md` once. |

Build the ordered queue from `BACKLOG.md`; anything in `ARCHIVE.md` is already done.

### Preflight — GO/STOP

Before the **first** dispatch, prove the environment can actually verify work:

| Check | Passes when | If it fails |
|---|---|---|
| Tests run | The relevant test command executes for executable changes. Skip executable tests for docs-only work or physical HITL with no harness change. | See the greenfield exception |
| App starts | The next issue needs a runnable surface and it can be launched. Do not start the app as a docs or live-hardware prerequisite. | See the greenfield exception |
| Queue is coherent | Every `Blocked by` id resolves; no cycles; no id in both queue files | **Always STOP** — this is a broken backlog, not a young project |

**Greenfield exception for executable work.** On a project with no test harness yet these checks *cannot* pass, and blocking on them is circular — the issue that would build the harness could never be dispatched. So: **if there is no working test runner**, that is **GO** for the next executable issue only, provided it is a **scaffold slice** (`to-issues` defines this: skeleton, dependencies, working test runner with one trivial passing test, a way to launch the thing). This holds whether or not the app already starts — a prototype that runs but has no tests still needs the harness before executable features ship. Say so explicitly in your report, deliver it, then re-run the relevant preflight before the next issue. Docs-only work does not need this exception or a harness.

If no test runner exists and the next issue is *not* a scaffold slice, STOP — but say plainly what is missing and that `to-issues` should file a scaffold slice as a new blocking issue that the remaining slices depend on. Do not imply the backlog is malformed; a breakdown that omitted it followed every other rule correctly, and do not name a specific id — ids are never reused.

Once a harness exists, a failure means regression, not youth — **STOP and report.** Never dispatch into an environment where "tests green" is unverifiable; a broken harness makes every downstream acceptance gate theatre. Repairing it is its own issue, not something smuggled into a feature.

After a rejected handoff, recheck the failed or unverified boundary and relevant regressions. Do not repeat a broad run merely because a source-only proposal or optional report formatter was rejected. Preserve authoritative results; distinguish reporting errors from runtime failures.

## Progressive exploration and intra-issue checkpoints

Do not dispatch a broad issue as one implementation task when its first job is
to discover what should change. Audits, inventories, pruning, migrations with
uncertain inputs, and inbox harvests can need exploration before a useful
implementation breakdown exists. Do not add this stage to already bounded work.

1. Dispatch a bounded, read-only discovery checkpoint first. Request findings,
   proposed dispositions, uncertainties, and the smallest useful next batches,
   not implementation or a promise to preserve every existing artefact. Use
   inspection and permitted focused probes; do not start shared runtimes or
   mutate them merely to complete an inventory.
2. Inspect the findings yourself and select QA for the named risks under the
   existing rules. Honour any explicit user review gate. Resolve material
   coverage losses, changed intent, or scope expansion with the user before
   implementation; QA approval does not authorize those decisions.
3. Write the next bounded assignment in the owning issue: outcome, allowed
   changes, non-goals, proof, and stop condition. Plan the next coherent batch
   from evidence; do not invent a detailed schedule for unexplored work. Keep
   checkpoints in the existing issue, not a parallel task tracker. File a
   separate issue through `to-issues` when work has an independent acceptance
   boundary or exceeds the authorized scope; do not split every edit into a ticket.
4. Dispatch the named team member or a fresh worker for the assigned checkpoint
   only, then repeat parent review, risk-directed QA, and authorized verified
   commits before dependent work. Pass forward accepted evidence and
   unresolved findings, not the entire conversation. A rejected handoff gets
   a scoped correction or reassessment, not another request to finish the
   whole issue.
5. Adapt the remaining breakdown after each handoff. Keep the issue `in progress`
   until the integrated outcome and all required criteria are proved. A discovery
   report, checkpoint approval, or partial green result is not issue acceptance.

Read-only discovery does not use TDD. Executable checkpoint changes do. Apply
preflight and proof requirements to the current assignment; a discovery
checkpoint cannot close implementation or runtime criteria.

## Parent review before acceptance

After each worker handoff, inspect the actual diff and form a provisional judgment before deciding whether to dispatch QA. Start from the latest user decisions and product intent, then read the changed implementation and relevant call sites. Do not derive the intended outcome from the worker's completion story or from spec conformance alone. After any QA, use its findings to update your own decision.

Check the boundaries that decide whether this is useful work:

- Does the implemented path deliver the user's operation, or only an artifact, example, or convenient demo?
- Do claims match observed evidence? Separate demo completion, shared-contract completion, and physical or production completion.
- Can the next dependent issue use the claimed foundation without copying special cases or adding a competing owner?
- If behavior is called shared or reusable, try the cheapest counterexample through another relevant client or a second configuration. This does not authorize enabling more hardware or building future features.
- Do ordinary requests, recovery, disable, and return-to-automatic paths preserve the intended contract? Check the actual decision path, not only helper state or generated structure.
- Is complexity staying within the owning abstraction? Examine monolith growth, duplicated decisions, and hardcoded pilot paths. Do not let "small pilot" become "pilot-only architecture."

Use a small scenario table in the issue's existing acceptance or evidence section: user action, expected outcome from intent, and observed result. Include ordinary exit/re-entry, explicit changes, background or safety actions, and restart when relevant. Distinguish request origins instead of treating every service write as user intent. If a spec-compliant result conflicts with product intent, stop and resolve the spec before dependent work; neither parent nor QA may invent a new preference as "common sense."

For mechanism-changing slices, report the baseline and delta of the few measures that expose growth: maintained behavioral implementations, generated size, helper count, largest routine, and bounded state payloads as applicable. Before scaling a costly mechanism, project growth across the remaining authorized scope and check relevant startup/config-check cost and payload limits in the permitted proof environment. Compare against a parent-set budget with a reason, not an arbitrary universal size limit. Generated volume alone is not failure. Reject independently maintained copies of the same policy semantics; parity tests do not justify a second implementation. Mechanical generation of bindings/data and independent expected test outcomes are not behavioral copies. Unexplained growth or pilot exceptions require a decision before expansion.

Keep review proportional to the changed claim. Use a focused disconfirming check, not repeated broad sweeps. Reuse sound evidence, but do not accept untested boundaries as proven. A docs-only change needs structural review, not an invented runtime test.

If direction or architecture is wrong, reject acceptance even when tests or QA pass, keep the issue `in progress`, and apply the drift protocol before further dispatch. Do not rewrite product intent to legitimize delivered code. Preserve the user's scope and deployment sequence; earlier live use is not a substitute for requested pre-deployment proof. Ask the user only when intent or authorized scope needs a decision.

Record a brief parent decision in the existing issue evidence: accept/reject, concrete behavior and boundary checked, remaining material risks, and QA selected/waived with the reason. If QA ran, include the disposition of its findings. Do not create another review tracker or approval ceremony.

## Risk-directed functional QA

The parent selects QA for a named uncertainty, not automatically after every worker or fix-up. Use an independent review for first shared-runtime behavior, ownership/migration or safety boundaries, material cross-client contracts, unresolved recovery/concurrency risks, and checks explicitly required by the user or repository. A bounded change with direct parent inspection and adequate focused evidence may omit a separate reviewer; record why. For a local fix-up, the parent can verify the reproduction and relevant regressions without restarting a broad review. If the fix changes a high-risk boundary or leaves an independent check unresolved, select targeted independent QA.

QA owns evidence-based bug hunting, not priorities or scope. Give it the parent judgment, specific risks, changed code, issue, reusable evidence, relevant intent stories, and prior findings. Do not request a desired verdict or invite open-ended polishing.

```text
Perform focused adversarial functional review for one candidate issue.

Issue: <issue path and id>
Worker handoff: <complete handoff>
Changed files: <paths and diff>
Existing checks: <commands and observed results>
Runtime or migration evidence: <exact observations, or none>
Governing docs: <user decisions, intent, glossary, issue, and relevant decisions>
Parent judgment and review target: <provisional decision, named risk, public workflow>
Intent stories: <expected outcomes independent of implementation and spec wording>
Prior findings: <concrete findings and reproductions, or none>

Read the implementation yourself. Exercise the narrowest relevant public
workflow within the review target. Hunt for functional bugs, regressions,
missing required proof, and failure or recovery cases. Compare intent stories
as well as spec conformance. A green test, generated shape, or checked criterion
does not prove runtime behavior. Reuse evidence for unchanged boundaries.
Each finding must name the required behavior at risk, concrete harm or missing
required proof, and the smallest reproduction/check. Separate blockers from
nonblocking suggestions and unresolved hypotheses. Do not turn stylistic
preferences or speculative future needs into blockers. Report intent/spec
conflicts to the parent; do not change priorities, scope, or expected behavior.
Do not edit files or implement fixes.

Return exactly:
VERDICT: PASS | FAIL
FINDINGS
- <blocking/nonblocking/hypothesis, severity, behavior at risk, evidence, or None>
EVIDENCE
- <what was run or inspected and the observed result>
PROPOSED FIX-UPS
- <smallest fix/check for a blocker, or None>
ORCHESTRATOR ACTION
<risk or intent conflict for parent disposition, or None>
```

The parent triages each finding: fix a confirmed blocker, perform a focused check of a hypothesis, defer a nonblocking suggestion, or reject it with evidence. A FAIL does not automatically dispatch repairs. Confirmed regressions, safety faults, or missing required behavior/proof cannot be waived for speed; keep the issue `in progress`. A missing verdict is not review evidence. Redispatch only parent-approved fixes, then recheck the failed path and relevant regressions; select more QA only under the risk rules above. If repeated rounds produce churn rather than new material evidence, stop the loop and reassess the issue and mechanism. Do not accept an unresolved blocker or polish a rejected architecture.

## Physical HITL

Do not dispatch unattended workers for operator-led live or physical work. The current session works with the user under the repository's operating procedures. Unattended workers cannot walk, click hardware, or supply physical observations.

## Delivery loop

Dispatch only rows whose `Status` is `todo`. **Any other status is not yours to start** — a `done` row in a live queue is a legacy tracker that predates the archive split, not work waiting to be done. Report those once, offer the migration, and never re-implement them.

For each `todo` row in order (respect blockers):

1. Set issue `Status` → `in progress` in `BACKLOG.md`
2. Dispatch issue worker — named team member, else fresh agent (see Roles); order by dependency, not forced blocking. For discovery-first work, use the bounded checkpoint loop above
   Before redispatching a failed or timed-out worker call, inspect the
   worktree for partial writes first; never retry or fall back automatically
   until the partial diff is reviewed.
3. Parent forms its own judgment against intent, scenario outcomes, mechanism growth, and the next dependency
4. Select targeted independent QA when risk or an explicit requirement calls for it; otherwise record the waiver and direct evidence
5. Parent triages findings and decides acceptance; unresolved blockers or product/architecture drift prevent acceptance regardless of a verdict
6. **Accept** only when required evidence and parent review support the goal → move the row into `ARCHIVE.md` with `Status` → `done` → make an authorized verified commit → next issue
7. **Reject** → leave `in progress` (or revert to `todo` if abandoning) → resolve drift or re-dispatch only approved gaps

After all `todo` rows are accepted:

- Parent verifies the integrated outcome against user intent and required evidence
- Select final independent QA under the same risk rules; record the decision
- Parent acceptance with no unresolved blockers → done | material defect or drift → scoped fixes and verification

Default: **serial on main** — one issue at a time.

Moving the row on acceptance is what keeps `BACKLOG.md` short. Do not batch the moves for later; an accepted issue that still sits in the live queue reads as unfinished work.

## Verified commit checkpoints

Confirm the user's commit/push authorization and repository policy at bootstrap. This skill does not grant permission to commit, branch, push, merge, or sync live. With standing commit authorization, commit each accepted issue before dispatching the next; follow repository push policy. Do not wait for a whole backlog to finish.

For a large issue, request worker handoffs at coherent, runnable checkpoints and commit them after focused validation and parent review. Apply the same QA risk rules to each checkpoint. Keep the issue `in progress` until all criteria pass; a checkpoint is not issue acceptance. Commit substantive verified fixes promptly rather than accumulating repeated review rounds in one dirty tree.

Inspect the staged diff and stage only the reviewed scope and its evidence. Preserve unrelated user/WIP changes. Never commit known broken syntax, failed relevant checks, unvalidated generated output, or an incomplete cross-file contract as a verified checkpoint. If no safe checkpoint exists, reduce the worker batch instead of hiding broken work in a commit. Do not change branches or revert WIP without authorization. Report checkpoint hashes and remaining work in the existing handoff/issue, not a new log.

## Issue worker dispatch

Attach skills **by name** — never by install path (`.claude/skills/…` vs `.agents/skills/…` varies by client).

The block below is copied verbatim into the worker's context (named team member or fresh fallback), so it has to stand alone — resolve every placeholder before sending, and tell the worker which docs actually exist.

```
Work on issue: issues/NNN-….md
Assignment: <whole issue, or named discovery/implementation checkpoint>
Checkpoint boundary: <outcome, allowed changes, non-goals, proof, stop condition;
write "whole issue" when no checkpoint is needed>

Read first: the issue file, plus whichever of these the project has —
I have told you which: the domain glossary, the product spec.

Development method: use the `tdd` skill for executable changes and follow it
exactly. For read-only discovery, docs-only work, or physical HITL with no
executable harness change, do not use TDD.

You are unattended: do not ask the user directly. The approved issue settles
its stated scope, not contradictory or missing product intent. Escalate a
material or hard-to-reverse choice to the parent before acting; continue only
independent work. Make ordinary reversible decisions within the approved
contract and report meaningful drift. Do not invent approval or treat existing
code as the product answer.

Rules:
- Scope limited to the current assignment above, not the whole issue or an expired task; old write/runtime permissions do not carry forward
- Verify actual tool cwd/root, HEAD and allowed absolute targets before the first write or after a phase/root change; a named worktree is not enforced isolation
- Claim fresh actions only from current observed tool results; label reused evidence and unavailable provenance
- For discovery-only assignments, do not implement; return findings, proposed dispositions, uncertainties, and recommended next batches
- Do not commit or push; return a verified checkpoint handoff when requested
- Report intent/spec conflicts and pilot exceptions; do not resolve them by inventing behavior
- For mechanism changes, report baseline/delta and bounded-state risks with the evidence
- Do not invent a second tracker, control surface, or proof-runner
- Do not patch product decisions into AGENTS.md
- Mark only acceptance criteria actually proved [x]; leave later-checkpoint criteria open
- Criteria tagged `proof: runtime` or `proof: migration` need observed
  evidence — drive the real thing and record what you saw. A passing unit
  test does not discharge them.
- Report drift — do NOT edit the drift log, the queue, or other issue files
- Return exactly this handoff:

  ## Handoff — issue NNN
  ### Assignment result — checkpoint outcome, or whole-issue result
  ### Acceptance criteria — mirror the issue file; distinguish proved, failed, and outside this checkpoint; name the proof used for each tagged one
  ### Tests — command + pass/fail
  ### Evidence — for each `proof:` criterion: what was run, what was observed
  ### Drift report (or "none")
  ### Notes for orchestrator
```

**If the `tdd` skill is not installed**, replace the *Development method* paragraph for executable work with: *Implement agreed behavior in vertical red–green–refactor slices. Inspect existing coverage before adding a case. Use the cheapest sufficient maintained boundary and independent expected outcomes. Verify why RED occurred; correct a stale SUT or invalid premise with evidence rather than restoring wrong behavior. Retain required protection when folding or retiring tests.* For docs-only work, require whitespace and structural review instead; do not apply the test-first fallback. Leave the unattended paragraph and the handoff exactly as they are, and skip the TDD audit in the gate. `orchestrate-issues` must never dispatch a prompt naming a skill the worker cannot load.

## Required acceptance evidence

These are whole-issue acceptance gates. Review a checkpoint against its assigned
boundary and proof; leave the issue open for criteria outside that checkpoint.

Reject whole-issue acceptance if any fail:

- The relevant checks pass. Use focused behavior checks for executable changes; docs-only work needs whitespace and structural review, not the test suite.
- All acceptance criteria `[x]`
- **Every `proof:`-tagged criterion has matching evidence** — an observed result, not a restatement of intent or a test name
- **The change was exercised, not just tested** when a runnable surface is in scope. Docs and physical HITL do not require a local app or sandbox. Tests-green-only is not acceptance for behavior a user can see.

  *How* to drive it: use whatever run/verify skill the repo provides. If it provides none, start the thing the way its README says to (or its smoke script) and record the exact command plus what was observed. If the project genuinely has no runnable surface yet, say that once in the handoff and move on — do not fabricate an observation, and do not treat the absence as a pass for later issues.
- TDD audit for executable work — apply the `tdd` skill's per-slice check: worthwhile agreed behavior or technical risk, maintained SUT, independent discriminating oracle, appropriate boundary and justified overlap/retirement. Read actual bodies and fixtures, not names or the worker's KEEP verdicts. Public visibility, green tests, unchanged coverage or a faster run do not establish value. Skip for docs-only work and when `tdd` was not attached.
- Domain language matches the glossary
- Parent directly reviewed goal and architecture alignment; unresolved drift → drift protocol

Evidence that only cites a file or a search result does not close a behavioral criterion.

## Drift protocol (orchestrator-only)

Log material drift to the drift log, update issue wording, patch spec/glossary if truth changed, then re-dispatch or accept. Workers never silently diverge.

Current user decisions outrank older issue wording. Do not change intent merely to make delivered work look aligned. A new product decision or scope expansion needs user approval; QA PASS cannot provide it.

When patching those two documents, two rules matter enough to restate here (a per-skill install cannot reach the `grill-me` skill's format references):

- **Patch, don't append.** Edit the sentence that is now wrong. A spec that only ever grows becomes archaeology.
- **An Architecture row names its rejected alternative.** A choice recorded without the option it beat reads as an accident and gets silently reversed later.
- **The glossary stays a glossary.** Terms only — no implementation detail, no drift narrative.

```md
## YYYY-MM-DD — {title}
**Issue:** NNN | **Drift:** … | **Resolution:** … | **Docs updated:** …
```

## Final integration review

The parent directly checks the integrated delivered workflow, public operations, recovery, and regressions against the latest user decisions. Use the risk-directed QA rules above to decide whether an independent reviewer is needed and name its target. Explicit user or repository requirements remain mandatory; a waiver never substitutes for required runtime/migration evidence. Triage any findings before dispatching fixes and record the final parent decision.

When selected, the reviewer must be a **different agent** than any that implemented and read-only by construction. Self-attestation is not review.

## Regressions

A `done` issue whose behavior later breaks is a **new** issue, filed via `to-issues` straggler mode. Do not reopen archived rows, edit completed issue files, or move rows back into the live queue — cite the original id in the new issue instead.

### Out of scope

- New CLI/automation wrappers
- `ORCHESTRATION-STATE.md` or other parallel state files (`BACKLOG.md` + `ARCHIVE.md` is the one allowed tracker)
- Parallel git branches (unless user requests)
- External issue trackers — `issues/` plus its queue files is the whole tracker
- Building or repairing the verification harness **yourself** — dispatch it as an issue (see the greenfield exception); never smuggle it into a feature issue
- Authoring the glossary or the spec — patch them when drift is accepted, but they are `grill-me` output
