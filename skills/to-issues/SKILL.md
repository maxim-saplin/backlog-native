---
name: to-issues
description: Break a plan or spec into independently-grabbable issues as repo-local markdown files using tracer-bullet vertical slices, tracked in an issues/BACKLOG.md queue with no external issue tracker. Also the entry point for filing a single follow-up issue or a regression. Use when the user wants to convert a plan into issues, create implementation tickets, file a follow-up or bug, or break down work into issues. Second stage of the backlog-native pipeline — consumes what the `grill-with-docs` skill settled, produces the queue the `orchestrate-issues` skill delivers.
---

# To Issues

Break a plan into independently-grabbable issues using vertical slices (tracer bullets).

Issues live in **`issues/`** as markdown files. **`issues/BACKLOG.md`** is the ordered queue: dependencies, type (AFK/HITL), and status. There is no external issue tracker — `issues/` *is* the tracker, and `BACKLOG.md` is its entry point.

## Artifacts

Paths are **defaults** — if the repo already names them differently, its names win.

| Role | Default path |
|---|---|
| Live queue | `issues/BACKLOG.md` — `todo` and `in progress` only |
| Completed queue | `issues/ARCHIVE.md` — rows move here on acceptance |
| Issue files | `issues/NNN-kebab-case-title.md` |
| Domain glossary | `CONTEXT.md` |
| Product spec | `docs/REQUIREMENTS-AND-NOTES.md` |

Read the glossary for domain vocabulary before drafting.

**If `issues/BACKLOG.md` does not exist, create it first** from [backlog-template.md](references/backlog-template.md) — a fresh project has no tracker, and every later step assumes one. Do this before writing any issue file.

## Two modes

Pick the mode before starting — most requests are the second kind.

**Breakdown mode** — new scope from a plan or spec section that needs several dependent slices. Run the full process below, quiz included.

**Straggler mode** — one self-contained follow-up: a bug, a doc-sync gap, a small behavior change noticed while doing something else, or a **regression** in already-delivered behavior. In a maturing project most requests are this shape. Skip steps 3 and 4 entirely: write the single issue file, append one BACKLOG row with `Blocked by` → `—`, and report the id. Do **not** quiz the user about granularity or dependencies — there is one slice and nothing blocks it. Ask only if the "one issue" turns out to need slicing after all, or if it collides with an issue already `in progress`.

A regression in a `done` issue is a **new** issue, not a reopening. Never edit a completed issue file or move its row back out of the archive; cite it in the new issue's description instead.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes an issue reference (number, path, or filename), read that file under `issues/` and any related entries in `issues/BACKLOG.md`.

Also read:

- the domain glossary
- the product spec, when breaking down new scope
- `issues/BACKLOG.md` — existing queue; and `issues/ARCHIVE.md` for the highest issue number already used

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Issue titles and descriptions should use the domain glossary's vocabulary, and respect the spec's architecture decisions for the area you're touching.

### 3. Draft vertical slices

Break the plan into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be `HITL` or `AFK`. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories this addresses (if the source material has them)

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Write issues to the repo

For each approved slice, create a markdown file under `issues/` and add a row to `issues/BACKLOG.md`.

**Numbering:** Use the next three-digit id after the highest `#` found anywhere — check `issues/BACKLOG.md`, `issues/ARCHIVE.md`, *and* the `issues/` filenames. Ids are never reused, so the highest completed id usually lives in the archive, not the live queue. Start at `001` on a fresh tracker.

**Filename:** `NNN-kebab-case-title.md` (e.g. `015-foo-bar.md`).

**Write in dependency order** (blockers first) so `Blocked by` can reference real issue ids.

**Issue file** — use this template:

```md
---
id: "NNN"
title: Short descriptive name
type: AFK
---

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation.

Avoid specific file paths or code snippets — they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it here and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2 — proof: runtime
- [ ] Criterion 3

## Blocked by

- [NNN](./NNN-blocking-issue.md) — short label

Or "None — can start immediately" if no blockers.
```

**Proof classes.** A criterion that changes behavior cannot be closed by reading code or grepping. Tag those criteria with the proof they require, so the acceptance gate can enforce it:

| Proof | Means | Use for |
|---|---|---|
| *(untagged)* | A passing test is enough | Pure logic, formatting, internal refactors |
| `proof: runtime` | The running app was driven and the result observed | Routing, auth, UI, integrations, anything user-visible |
| `proof: migration` | Verified against real migrated data, not a fixture | Schema and data-shape changes |

Tag sparingly — an issue where every criterion demands runtime proof is usually too thick a slice. Untagged is the default and the common case.

**BACKLOG row** — append to the Order table in `issues/BACKLOG.md`:

```md
| NNN | [Title](./NNN-kebab-case-title.md) | AFK | — | todo |
```

Set `Blocked by` to issue numbers (e.g. `011`, `011, 012`) or `—`. New issues start with `Status` → `todo`.

Add dependency notes below the table when parallel or sequential groups matter — a one-line prose note naming the chain, e.g. "**Store migration (023 → 024 → 026):** schema → verification → stats parity."

### Queue hygiene

`BACKLOG.md` holds only `todo` and `in progress` rows. The `orchestrate-issues` skill moves each row to `issues/ARCHIVE.md` when it accepts the issue, so the live queue stays short no matter how old the project gets — an agent reads the raw markdown every session, so a long table of completed rows is pure context cost. Collapsing them behind `<details>` does not help; only moving them out does.

If you are filing into a project whose `BACKLOG.md` still carries `done` rows, leave them alone unless the user asks for the migration — do not silently restructure their tracker.

Do NOT modify completed issue files unless the user explicitly asks for a rewrite.
