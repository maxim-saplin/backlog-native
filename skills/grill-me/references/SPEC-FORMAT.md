# Spec format

Use this file only when starting a spec from nothing. If the repo already has `docs/INTENT-AND-DESIGN.md`, that is the product spec: plain-language intent. Patch it in place. Do not create `docs/REQUIREMENTS-AND-NOTES.md` beside it.

The greenfield default path is `docs/REQUIREMENTS-AND-NOTES.md`. Discover order: `docs/INTENT-AND-DESIGN.md`, then this default.

The product spec is the durable answer to *what are we building and why is it shaped this way*. One per project, patched as understanding changes — **not** one per feature, and never discarded when a feature ships.

`to-issues` reads it to break down scope. `tdd` reads it to respect architectural decisions. Keep it accurate or both degrade.

## Structure

```md
# {project} — requirements and notes

Living doc for product requirements and design decisions.

## Goal

What this exists to do, in a paragraph, in the project's own vocabulary
(see CONTEXT.md). Name the intended user and the thing they get.

## Scope

What is in. Then, explicitly, what is **out** — the out-list prevents more
argument than the in-list.

## Behaviour

What the system does that a user can observe. Group by capability, not by
layer. Precise enough that a slice can be cut from it; not so precise that
it becomes a design doc.

## Architecture

| Decision | Choice |
|----------|--------|
| {axis}   | **{choice}** — {why, and what it is not} |
| {rejected alternative} | **Rejected** for v1 — {why} |

## Open questions

Things genuinely undecided. Empty is a fine state. A question that has been
sitting here for months is either decided or not really a question.
```

## Rules

- **Prose and tables, not code.** File paths and snippets go stale fastest. The exception is a shape that prose cannot pin down precisely (a schema, a state machine, a type) — inline the decision-rich part only.
- **Record the rejected alternative.** A choice without its alternative reads as an accident and gets silently reversed later.
- **Patch, don't append.** When the truth changes, edit the sentence that is now wrong. A spec that only grows becomes an archaeology exercise.
- **No requirements ceremony.** No EARS syntax, no `SHALL`, no numbered requirement ids to trace. Issues trace to *behaviour*, and the acceptance criteria in each issue file carry the testable precision.
- **Distinct from the drift log.** The spec says what is true now. `docs/DECISIONS.md` says what changed during delivery and why. Do not merge them.

## On a blank slate

Do not scaffold every heading up front. Write `Goal` and `Scope` when the grill session settles them; add `Architecture` rows as trade-offs are actually made. A spec with five empty sections is worse than a spec with two full ones.
