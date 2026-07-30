# Backlog template

Scaffold for a fresh tracker. Copy the two blocks below into `issues/BACKLOG.md` and `issues/ARCHIVE.md`, replacing `<project>` and the doc links with the project's real paths. Drop the example row once a real issue exists.

Do not add columns. Five is the whole schema — anything richer belongs in the issue file.

---

## `issues/BACKLOG.md`

```md
# <project> backlog

Live delivery queue for AI agents. Completed issues move to [`ARCHIVE.md`](./ARCHIVE.md).
Source spec: [`REQUIREMENTS-AND-NOTES.md`](../docs/REQUIREMENTS-AND-NOTES.md). Domain glossary: [`CONTEXT.md`](../CONTEXT.md).

## Order

| #   | Issue                                    | Type | Blocked by | Status |
|-----|------------------------------------------|------|------------|--------|
| 001 | [First slice](./001-first-slice.md)      | AFK  | —          | todo   |

## Status values

| Status        | Meaning                                                  |
|---------------|----------------------------------------------------------|
| `todo`        | Not started; blockers (if any) are `done`                |
| `in progress` | Active worker or orchestration session                   |

Accepted issues are not listed here — their rows live in [`ARCHIVE.md`](./ARCHIVE.md) with status `done`.

## For agents

Pick the lowest `#` where `Status` is `todo` and every `Blocked by` issue appears in the archive as `done`. Implement using the `tdd` skill. On starting work set `Status` → `in progress`. Mark acceptance criteria `[x]` in the issue file as you complete them. Criteria tagged `proof: runtime` or `proof: migration` need observed evidence, not a passing unit test.

Do not add a second tracker. This file plus `ARCHIVE.md` and the issue files are the whole system.
```

## `issues/ARCHIVE.md`

```md
# <project> backlog archive

Accepted issues, most recent last. Rows move here from [`BACKLOG.md`](./BACKLOG.md) when the acceptance gate passes.
Issue numbers are never reused — when picking the next id, check the highest `#` here as well as in `BACKLOG.md`.

| #   | Issue | Type | Blocked by | Status |
|-----|-------|------|------------|--------|
```

---

## Notes

**Never link a skills install path.** A tracker that says `.agents/skills/tdd/` breaks under any harness that installs to `.claude/skills/` or elsewhere. Refer to skills by name.

**Dependency notes** go below the Order table as prose when a chain matters:

```md
**Store migration (023 → 024 → 026):** schema → verification → stats parity.
```
