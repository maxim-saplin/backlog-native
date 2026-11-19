---
name: orchestrate-issues
description: Deliver an explicitly authorized issue backlog directly as one agent, or with an optional project-configured team; use a temporary goal brief only for sustained assignments, meaningful checkpoints, risk-directed review and verified delivery. Do not use for unattended physical or live human-in-the-loop work.
---

# Issue delivery

The responsible agent is the **orchestrator**. By default it implements directly,
verifies the result against user intent, handles drift and owns delivery. A
single-agent project needs no roster, subagent, messaging service or self-dispatch.
Tests and completion stories are evidence, not product judgment.

When a project explicitly configures a team, honor its roles and writer policy.
The coordinator may delegate bounded checkpoints instead of implementing them;
it still owns product direction, proportionality, maintainability and acceptance.
An independent reviewer does not replace that responsibility. Do not provision
more agents merely because these instructions describe optional roles.

## Artifacts and permission

Existing repository names win over these defaults.

| Role | Default path | Authority |
|---|---|---|
| Live queue | `issues/BACKLOG.md` | Ordered work; responsible agent claims/updates under repository policy |
| Completed queue | `issues/ARCHIVE.md` | Accepted rows only |
| Issue | `issues/NNN-*.md` | Scope, questions, checkpoints, acceptance and evidence |
| Drift log | `docs/DECISIONS.md` | Responsible agent records material dispositions |
| Glossary | `CONTEXT.md` | Terms, not history; patch existing entries |
| Product spec | existing intent document, else `docs/REQUIREMENTS-AND-NOTES.md` | Intent and deliberate trade-offs |
| Temporary goal brief | ignored `STATUS.md`, for an approved long assignment | One publisher; goal/done condition/3–5 outcomes/one NOW; retain until a human reviews and acknowledges the whole assignment's disposition |

Use [the temporary goal-brief lifecycle/template](references/operator-briefing.md)
only for approved work that spans sessions, has several meaningful steps or
continues while the user is away. No brief for discussion, quick actions or an
unassigned backlog. Keep it roughly 100–150 words and at the level of the approved
assignment, including the whole sprint when that was assigned. Keep the explicit
goal, done condition and short outcome bullets even when adding a compact issue
status table. Distinguish active work from staged or waiting work. The backlog
is the durable queue, not a sprint report. A blocker or delegated checkpoint
changes the current step, not the goal. Discussion alone writes no files,
including the briefing. An explicit bounded
request performs its named action and stops; it does not start backlog delivery.
Only an approved assignment starts this loop. No backlog row, idle peer, tool or
delayed message grants additional permission.

For authorized delivery, keep assignment scope, root/reference, allowed resources
and required proof in their existing issue/task handoff—not in the user-facing
`STATUS.md`. The optional goal brief selects only the current outcomes. Verify
actual cwd/root, HEAD, dirty paths and targets before work and after phase/root
changes. A named worktree is not enforced tool isolation. Use isolated candidates
and one repository writer if shared writes cannot be safely coordinated.
A new phase supersedes old task-specific permission; uncertainty stops the
**affected** action, not unrelated permitted work. Repeated drift needs a bounded
fresh context, reassignment or simplification, not indefinite rebriefing.

## Bootstrap and preflight

Read current repository state, queue, owning issue, glossary and existing product
spec. Discover an existing intent document before assuming the greenfield name.
If the current long assignment warrants `STATUS.md`, reconcile it with the owning
issue and fresh observations, not the other way around. Absence is normal for
short work or after human-reviewed closeout, not merely because work is paused.
Do not create a competing spec or turn
`AGENTS.md` into one.

| Missing or inconsistent | Action |
|---|---|
| Live queue, or archive with no legacy completed rows | Stop and use `to-issues` to scaffold the tracker |
| Archive absent but completed rows live in the queue | Treat as legacy history; offer archive migration, never redispatch them |
| Glossary/spec | Note absence once and proceed from approved issue/user intent; do not author them as incidental delivery work |
| Drift log | Create only when a material drift decision needs recording |
| Briefing | Normal when no active long assignment; reconstruct only when that assignment actually needs one |
| Unresolved dependency IDs or cycles | Stop affected dispatch; repair the actual queue inconsistency |

Before executable work, verify the relevant test command runs and any runnable
surface needed for the checkpoint can start. Documentation needs structural and
whitespace review, not a harness or app launch. Physical work follows the owning
human procedure, never a local simulator invented for proof.

**Greenfield exception:** no test runner is GO only for an authorized scaffold
slice: skeleton, dependencies, a working runner with one passing test, and a
runnable surface or explicit absence. Recheck before subsequent executable work.
If the next issue is not that slice, use `to-issues` to add it as a new blocker;
do not invent an ID. A previously working harness that fails is a regression,
not the greenfield exception. Report and scope its repair rather than smuggle it
into a feature.

After rejection recheck the failed/unverified boundary and relevant regressions.
Do not repeat a broad run merely because a source proposal or optional formatter
was rejected. Preserve results and distinguish reporting errors from execution.

## Bounded discovery and implementation

Do not treat an uncertain audit, inventory or migration as one giant build task.
For discovery-first work:

1. Perform a bounded read-only checkpoint, directly or through configured peers.
   Return findings, proposed dispositions, uncertainties and smallest next batches;
   do not mutate shared resources merely to complete an inventory.
2. Review intent and risks. Honor user review gates; a reviewer's advice cannot
   authorize coverage loss, scope expansion or changed product intent.
3. Record the next checkpoint in the **owning issue**: outcome, allowed changes,
   non-goals, proof and stop condition. Adapt from evidence rather than invent a
   detailed schedule for unexplored work. Separate independent scope into issues,
   not a ticket for every edit or a second task tracker.
4. Implement directly by default; delegate only where configured. Use `tdd` for
   executable changes, not discovery or Markdown. Review and commit coherent,
   runnable checkpoints before dependent work under the repository's policy.
5. Keep the issue open until its integrated goal and required criteria are proved.
   A discovery report, checkpoint or partial green result is not issue acceptance.

## Review against intent

At each checkpoint inspect the actual diff and relevant call sites. Start from
current user decisions, not the implementation's completion story. In a team,
form your own judgment before selecting QA and triage its findings afterward.

- Does this deliver the user's operation or merely a demo, report or helper?
- Do claims distinguish requested, observed, tested and delivered behavior?
- Can the next dependent use the foundation without copying policy or pilot cases?
- For a reusable contract, use the cheapest relevant second configuration/client
  counterexample. This does not authorize new resources or future features.
- Check normal exit/re-entry, explicit changes, background/safety actions and
  restart where relevant. Identify request origins rather than treat every write
  as user intent. Keep a compact expected/observed scenario table in the issue.
- Does complexity remain in its owner? Reject independently maintained copies of
  policy semantics; parity tests do not justify duplicates. Mechanical generated
  bindings/data and independent expected outcomes are not behavioral copies.

For mechanism changes, report the baseline/delta of the few growth measures that
matter: implementations, helper count, largest routine, generated size or bounded
state as applicable. Before scaling an expensive mechanism, project the remaining
scope and check startup/payload cost in the permitted environment. Use a reasoned
checkpoint budget, not arbitrary universal size targets. If direction is wrong,
reject even when tests pass; do not rewrite intent to legitimize delivered code.
Use proportional disconfirming checks and reuse unchanged evidence.

Record the decision in the issue: accept/reject, actual boundary checked,
remaining risks and independent QA selected/absent with the reason. No extra
approval tracker is needed.

## Risk-directed review; single-agent use

Select independent review for a **named uncertainty**, not an automatic tax per
fix: first shared-runtime behavior, ownership/migration/safety, material
cross-client contract or unresolved recovery/concurrency risk. Explicit user and
repository review requirements remain binding.

With no independent reviewer available, perform direct intent/scenario checks
and record `QA: none (single-agent session; no independent reviewer available)`.
Do not label self-review independent, invent a QA verdict or silently create a
team. If external review is explicitly required, that acceptance gate remains
blocked until obtained or changed by its owner; independent work may continue.

When configured, the independent reviewer is read-only and did not implement.
Give it scope/reference/diff, relevant intent and expected outcomes, existing
proof with limits, prior findings and the named risks. Ask for concrete bugs,
regressions or missing required proof, not polishing or a desired verdict.

Optional review request:

```text
Review checkpoint <issue/reference> read-only.
Goal/scenarios: <user intent and independent expected outcomes>
Changed paths: <actual diff/callers>
Checks/proof: <observed results and unverified boundaries>
Review target/prior findings: <named risk and existing evidence>
Do not implement or change scope. Use the smallest permitted check.
Return findings classified blocker / suggestion / hypothesis, with behavior at
risk, evidence and smallest reproduction. Report intent/spec conflicts.
```

A reviewer does not set priorities. Triage: fix confirmed blockers, check a
material hypothesis narrowly, defer nonblocking suggestions or reject findings
with evidence. A failed review does not automatically authorize all proposed
repairs. Unresolved safety faults, required behavior or proof block acceptance;
missing review is not a PASS. Recheck approved fixes, not another broad sweep by
habit. If rounds create churn, simplify/reassess mechanism and scope.

## Delivery loop

Work only explicitly authorized `todo` rows or the current approved checkpoint.
Other in-progress work is not yours to take over; archived work is not a rerun.

1. Claim according to repository policy; create/refresh a brief only if the assignment warrants it.
2. Implement directly, or use the optional delegated handoff below. Before any
   retry of failed/timed-out delegated work, inspect its partial writes first.
3. Review the actual goal, scenarios, growth and next dependency yourself.
4. Select independent review for named risks if available/required; label absence.
5. Triage findings and drift before dependent work.
6. Accept an issue only with required evidence: move its row to archive, make the
   authorized verified delivery and record the conclusion. Keep `STATUS.md` until a
   human reviews and explicitly acknowledges the disposition of the entire approved
   sprint or assignment. An individual issue, work window or agent handoff cannot
   close it. Never batch accepted rows in the live queue indefinitely.
7. Reject: keep in progress (or return to todo if abandoning), resolve the actual
   gap or intent conflict and recheck approved scope only. If cancelled, deferred or
   waiting for review, show the actual state; do not remove or move the brief without
   the whole-assignment human acknowledgment above.

Sequence dependent work and protect shared writers or runtime resources. When an
outcome blocks, keep other already-authorized, noninterfering outcomes moving where
useful; do not create busywork or treat a local patch as delivery of the batch. Follow
the repository's branch policy. After authorized scope is complete, verify integrated
operations, recovery and local entry points/files. A remote
commit or request that the user pull is not local handoff. Integrate safely,
verify paths/contents or name the concrete obstacle and unfinished delivery.

## Verified commits and acceptance

This skill grants no commit, push, merge, runtime or spending permission. With
standing authorization, commit each accepted issue and coherent substantive
checkpoint promptly. Stage explicit reviewed paths, inspect the complete index,
preserve unrelated changes and honor one-writer/live-resource boundaries. Never
commit failed relevant checks, broken syntax or an incomplete cross-file contract
as verified work; shrink the batch when no coherent checkpoint exists.

Whole-issue acceptance requires:

- Relevant checks pass; Markdown uses structural/whitespace review, not tests for
  headings or an invented app launch.
- All required criteria are proved. Each `proof: runtime` / `proof: migration`
  has actual observation in the claimed environment, not a restated criterion,
  file pointer alone, test name or simulated proof of real behavior.
- Executable work follows the `tdd` per-slice audit: agreed outcome/credible risk,
  maintained SUT, independent oracle/discriminating inputs, cheapest sufficient
  boundary, justified overlap and retained protection when retiring tests. Read
  bodies and fixtures; test count or green status is not value.
- Vocabulary and architecture fit user intent; unresolved drift is not hidden.
- Parent/direct judgment supports acceptance and mandatory external review is
  satisfied. A lone agent's direct verification remains accurately labeled.

A checkpoint is judged against its assigned proof; later criteria stay open.
If no runnable surface exists, say so once—do not claim a later behavior passed.

## Optional delegated handoff

Skip in direct single-agent operation. In a configured team, supply absolute
root/reference/dirty paths, scope, allowed resources and stop condition. Attach
`tdd` by **name**, never an install path, only if available. Otherwise require its
portable red–green–refactor principles inline for executable work. Discovery and
Markdown need no TDD.

```text
Work on <issue>, checkpoint <scope>, root/reference <actual paths/revision>.
Read <existing governing documents>. Allowed changes/resources: <explicit>.
Proof and stop condition: <bounded outcome>; later phases are not authorized.
Do not ask the user directly on delegated work: escalate material product or
hard-to-reverse conflicts to the responsible owner; continue independent work.
Choose ordinary reversible details within the contract. Do not invent approval.
Do not commit/push unless explicitly assigned. Do not edit the queue, briefing
or decision log; report material drift. Check only criteria actually proved.
Return: assignment result; criteria/proof with limits; exact checks/results;
drift; blocker/next action. Distinguish current observation from reused evidence.
```

Actual handoff/readback, not message delivery alone, supports acceptance. A fresh
context can reduce shared assumptions but is not independent judgment by itself.

## Drift, regressions and non-goals

Current user decisions outrank stale issue text. Record material drift, patch
the now-wrong sentence in the owning spec/issue and redispatch only approved
scope. A trade-off records its rejected alternative; the glossary stays terms,
not implementation/history. Do not change product intent merely to match code.
A worker reports drift; its coordinator owns durable dispositions.

A later regression of accepted work is a **new** issue via `to-issues`, not a
reopened archived row. Do not rewrite accepted history to hide it.

No new CLI/orchestration wrapper, external tracker, scheduler or approval agent
is implied. `STATUS.md` is permitted only as a temporary goal-first brief, not a
permanent dashboard or requirements/acceptance ledger. No active goal means no file. Do not
author a new glossary/spec, preempt unrelated work or silently turn discussion
into implementation.
