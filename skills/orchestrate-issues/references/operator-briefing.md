# Temporary goal brief

`STATUS.md` is a short, gitignored brief for **one approved long-running assignment**,
which can be the whole approved sprint or delivery batch. It is not a permanent
project dashboard, second tracker or session log. Borrow the useful Scrum pattern:
one goal, a small selected breakdown, and a clear definition of done. No sprint
ceremonies, metrics or new machinery are required.

## When it exists

- **Create** when an approved backlog task or explicit assignment spans sessions,
  has several meaningful steps, or continues while the user is away. Use judgment;
  do not create a ticket, team or timer merely to justify the brief.
- **Do not create** for discussion, a quick command, a small fix or unassigned work.
  A short task becoming a sustained assignment may warrant a brief, not more scope.
- **Keep** the brief at its current path until a responsible human has reviewed
  and explicitly acknowledged the disposition of the **whole assignment**. This
  applies to delivery, cancellation and long-term deferral. While waiting, report
  actual paused, blocked or review-pending work; do not imply that anyone is active.
- **Do not infer closure** from a timer or budget ending, a lane finishing or
  blocking, a handoff, an agent review, a sent report, or silence. Update the brief
  and continue any unblocked authorized work; protect stopped or unfunded work.
- **Retire only after acknowledgment.** Record the human decision in the owning
  issue or normal handoff before removing or moving the publisher-owned file.
  A missing brief is normal when none was warranted or it was retired after review.

Preserve other owners' edits and follow local retention when retiring. The review
can happen in the ordinary conversation; no new approval tracker or maintained
archive of status files is required. This is not permission to delete notes,
editor locks, or execution evidence.

## What the user reads

Aim for **100–150 words or fewer** for the summary; never pad to a minimum or hide giant paragraphs behind a line limit.

**Human attention comes first.** If anything needs the user, put a **Needs you**
section directly under the goal heading, before everything else. A reader who
opens the file must see every open question without scrolling or rereading chat.

- One numbered line per item, so the user can answer “1 yes, 2 no”: the question,
  the recommendation, and what waits on it (or “nothing waits”).
- Include every decision, approval, credential or action only a human can take,
  and every question already asked in chat that is still unanswered. Chat scrolls
  away; the brief is where open questions stay until they are answered.
- Keep the oldest open item first and give its age when it has waited more than
  a day. Add a draft or details path only when the user needs it to decide.
- Remove an item once answered, and record the answer in its owning issue.
  With nothing open, write `Needs you: nothing.` so silence is not ambiguous.
- Ordinary engineering choices stay with the team; do not pad this section.

Then:

1. Lead with the **Goal** of the full approved assignment, not the issue, patch or
   blocker receiving attention today. Change the goal only when the approved scope changes.
2. Define **Done when** as an observable result for that goal, not “tests and process complete.”
3. Group the whole assignment into **3–5 outcome-based steps**, with exactly one
   obvious **NOW**. Keep completed outcomes, independent work in flight and queued
   outcomes visible even when NOW is a narrow blocker. Use real counts where meaningful.
4. If the user needs issue-by-issue status, add a compact **Issue | Name | Status**
   table **after the bullets**, never instead of the goal, done condition, or summary.
   List only issues in the active assignment. Start each status cell with an
   explicit state such as **Ready**, **Active**, **Running**, **Waiting**,
   **Blocked**, or **Done**; add a few words for the dependency or next action
   only if useful. A progress note such as “patch staged” does not replace the
   state. Use Active only while someone is actually doing the work and Running
   only for a verified running process. Distinguish ready work that can be picked
   up from work waiting for a dependency. Open does not mean active.
5. Name an actionable blocker, if any: which outcome it blocks and why. Do not
   imply the whole assignment is stalled when authorized independent work can continue.
   A blocker that needs the user also goes in **Needs you**.
6. End with one **Updated** timestamp and at most one details pointer when useful.

No hashes, PID lists, policy/permission recitals, acceptance arguments, cost ledgers
or accumulated heartbeat/history diary. A live job may have one short actionable
health/budget line, not an operations dump. If wake-ups matter to the user, name
only the notifications or timers actually armed; a wake is not a restart guarantee.
Put evidence, detailed plans, durable
questions/answers and history in their existing homes. Deliver a requested
retrospective separately; do not expand the active brief into an overnight log.

## Update without creating noise

Update when an outcome completes, the plan changes, a real blocker or a new
question for the user appears, an open question is answered, or the user asks for
a checkpoint. Before replacing a brief, check that it still answers
“How is the whole approved assignment doing?” A blocker changes the affected step,
not the goal. Preserve actual meaning: “no change; waiting for X” is not new progress. A quiet verification may update the timestamp without moving
steps forward, appending history or sending another chat message. Do not copy every
commit, test count, gate amendment or worker message into the brief. Keep sprint
updates in the brief, not the backlog: the queue records order, issue references,
dependencies and durable workflow state, not sprint summaries or live telemetry.

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
# Goal: Deliver catalogue search, filtering and imports

**Needs you**
1. Grant the search service read access to the product index? Recommended: yes,
   read-only. Search release waits on it.

Done when: users can find products and new catalogue entries show up after import.

- Done — Category filters work in the released app.
- NOW — Search release is blocked by an index permission (Needs you 1).
- In parallel — Check the import against representative catalogue files.
- Next — Publish the complete catalogue workflow and usage notes.

| Issue | Name | Status |
|---|---|---|
| 021 | Category filters | Done |
| 022 | Catalogue search | Blocked: index permission |
| 023 | Catalogue imports | Active |
| 024 | Usage notes | Queued |
Updated: 14:30 UTC. Details: the owning issue.
```
