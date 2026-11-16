# Local operator briefing

`STATUS.md` is an optional-in-discussion, first-class delivery briefing, **not a
second tracker**. During an authorized delivery assignment, maintain it for the
user and the next session. Queue, requirements, questions, decisions and evidence
keep their existing durable homes. Track this convention/template, ignore the
populated root file with `/STATUS.md`, and never force-add it.

## Authority and ownership

- One publisher per checkout: the working agent by default, or an explicitly
  nominated publisher when a project configures a team. Others send meaningful
  updates through existing channels. Do not infer takeover from idle or stale
  status. Distinct worktrees may have explicitly scoped briefings.
- Discussion and brainstorming alone write **no files**, including this briefing.
  An explicit request to update a proposal authorizes that artifact, not delivery.
  A bounded request performs its named action and stops; do not make a status
  convention an excuse for follow-on edits or provisioning agents.
- An approved autonomous assignment covers its stated checkpoint and permitted
  delivery. A backlog row, side conversation, available tool or delayed message
  does not revive expired authority. Conversation isolation is not file/tool
  isolation; check the actual workspace and permissions.
- A human may ask any available member a question or assign a bounded side task.
  Check current obligations; do independent work, queue a conflict or explain it.
  Notify the current owner if shared writes, resources or priorities change. A
  non-publisher does not rewrite the shared briefing. No redundant lead approval
  is needed for an already authorized, non-interfering action.
- Never store credentials or unique evidence here. Even an ignored file may be
  loaded into a prompt. Reference private records without copying their contents.
  The briefing never authorizes spending, destructive work or publication.

## Read and refresh

At authorized task start, resume, meaningful checkpoint, blocker, execution
transition, acceptance or handoff, reconcile the briefing with actual repository
state, the owning issues and known live-record pointers. Use existing wakes if
configured; do not create a timer, team or service to use these skills.

Keep two times: **Verified UTC** for the last actual check and **Last material
change UTC** for real progress. A quiet heartbeat may change only the former.
A stale timestamp is a warning to verify, not proof of failure or permission to
repeat execution. Missing live facts stay unknown. If two expected checkpoints
show no progress, reassess scope/blockers rather than report motion indefinitely.

Rewrite current sections, normally within 60–80 lines; keep only a few recent
changes/heartbeats and links to history. An explicitly requested retrospective
may occupy a compact dated roll-up; do not let it become a permanent session log.
Keep operational phase separate from the queue's durable lifecycle status.

If the briefing is missing, reconstruct it from the durable sources and fresh
observations. If testing recovery, preserve the current file instead of deleting
it. Do not infer old authority, recreate secrets or manufacture evidence. A new
machine can recover durable decisions, not another machine's unrecorded live state.

## Attention and communication

Material owner questions have stable issue-scoped IDs and a durable home. Mirror
only the choice, options, recommendation, impact of waiting and independent work
that can continue. If none are waiting, say **None**. Ordinary reversible details
within approved scope are the responsible agent's job, not the user's inbox.

Record the answer/disposition in the issue or decision record, clear the open
item and retain a short resolution pointer. Do not leave an answered question in
memory as an outstanding task.

Batch pending reports when safe. A cycle can end with nothing to tell the user.
Interrupt for a material decision, concrete safety blocker, requested heartbeat or
useful delivered outcome; not every commit, test count, acknowledgment or correction.
No formal inbox/outbox implementation is required.

With configured peers, handoffs name the task/checkpoint, actual root/reference,
allowed changes/resources, result, evidence boundary, blockers and next action.
Distinguish static reasoning, process execution, integration and user-facing
observation. Delivery of a message is not completion or acceptance. A receiving
owner acknowledges the scoped handoff and returns its disposition; delegated
cross-team work closes through the responsible leads without worker fanout.
Direct human questions remain allowed. Freeze the reviewed paths during a gate;
later messages do not silently widen scope or unfreeze a candidate.

## Template

```md
# STATUS
Publisher: <agent or designated publisher> | Checkout/host: <scope>
Verified UTC: <actual check> | Last material change UTC: <meaningful progress>

## Needs your attention
None, or <issue/Qn — choice; options; recommendation; impact of waiting>

## Active work
Task | Agent / optional team | Phase | Proved / pending | Next checkpoint / blocker

## Execution and authority
Live work: <observed state or unknown; record pointer>
Authority: <current assignment/limits and source pointer, or none>
Next existing wake / expiry: <coverage and limitations, or none>

## Recent changes and heartbeats
<few dated outcomes/checks; durable evidence and older-history pointers>
```
