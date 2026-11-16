# Temporary goal brief

`STATUS.md` is a short, gitignored brief for **one active long-running assignment**,
not a permanent project dashboard, second tracker or session log. Borrow the useful
Scrum pattern: one goal, a small selected breakdown, and a clear definition of done.
No sprint ceremonies, metrics or new machinery are required.

## When it exists

- **Create** when an approved backlog task or explicit assignment spans sessions,
  has several meaningful steps, or continues while the user is away. Use judgment;
  do not create a ticket, team or timer merely to justify the brief.
- **Do not create** for discussion, a quick command, a small fix or unassigned work.
  A short task becoming a sustained assignment may warrant a brief, not more scope.
- **Keep** while that goal is actively worked or has a concrete near-term blocker.
  An active user decision can stay visible with its recommendation and consequence.
- **Retire** on delivery/acceptance or cancellation: record the conclusion in the
  owning issue or normal task handoff, then remove the publisher-owned local brief.
  No active assignment means no `STATUS.md`; an absent file is normal, not a fault.
- **Defer** long-term blocked work in its existing issue, notify the user once, and
  retire the brief rather than leave an abandoned task looking active.

Preserve other owners' edits and follow local retention when retiring. This rule
is not permission to delete old user notes, editor locks or execution evidence.
Do not create a maintained archive of status files by default.

## What the user reads

Aim for **100–150 words or fewer**; never pad to a minimum or hide giant paragraphs behind a line limit:

1. Lead with the user outcome: **Goal**, not the agent's activity or ticket number.
2. Define **Done when** as an observable result, not “tests and process complete.”
3. Show **3–5 outcome-based steps**, in order, with exactly one obvious **NOW**.
   Use real counts where meaningful; do not invent percentages for unequal tasks.
4. Name an actionable blocker, if any: what cannot happen and why. Include only a
   genuine user choice with a recommendation; ordinary engineering stays with us.
5. End with one **Updated** timestamp and at most one details pointer when useful.

No hashes, PID lists, policy/permission recitals, acceptance arguments, cost ledgers
or accumulated heartbeat/history diary. A live job may have one short actionable
health/budget line, not an operations dump. Put evidence, detailed plans, durable
questions/answers and history in their existing homes. Deliver a requested
retrospective separately; do not expand the active brief into an overnight log.

## Update without creating noise

Update when an outcome completes, the plan changes, a real blocker appears or the
user asks for a checkpoint. Preserve actual meaning: “no change; waiting for X”
is not new progress. A quiet verification may update the timestamp without moving
steps forward, appending history or sending another chat message. Do not copy every
commit, test count, gate amendment or worker message into the brief.

The sole agent owns it by default; a configured team nominates one publisher per
checkout. Others report meaningful changes, not concurrent edits. A direct side
request does not silently transfer that ownership. Discussion alone writes no
files, including the brief; existing request/permission boundaries still apply.

On resume, use a brief only if it matches the currently approved goal. Reconcile
with the issue/repository and known observations; never infer execution authority
or liveness from old text. Reconstruct a missing brief **only if** an active long
assignment still needs one. Do not revive completed work or provision a service.

## Example (illustrative, not a current assignment)

```md
# Goal: Deliver product search and filtering

Done when: users can find products by name and category in the released app.

- Done — Agree on search and filter behavior.
- Done — Implement search and category filtering.
- NOW — Verify ordering, empty results and keyboard use.
- Next — Publish the change and usage notes.

Needs you: nothing.
Updated: 14:30 UTC. Details: the owning issue.
```
