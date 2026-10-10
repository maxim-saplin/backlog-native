---
name: tdd
description: Test-driven development with red-green-refactor loop, vertical tracer-bullet slices, and an unattended mode for delegated work. Use when the user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development. Implementation method of the backlog-native pipeline — used directly, or attached to issue workers by `orchestrate-issues`.
---

# Test-Driven Development

## Artifacts

Paths are **defaults** — if the repo already names them differently, its names win.

| Role | Default path | You |
|---|---|---|
| Issue file (when dispatched) | `issues/NNN-*.md` | read it; mark its criteria `[x]` |
| Domain glossary | `CONTEXT.md` | read only |
| Product spec | `docs/INTENT-AND-DESIGN.md` if present, else `docs/REQUIREMENTS-AND-NOTES.md` | read only |
| Drift log | `docs/DECISIONS.md` | **never write** — report drift in your handoff instead |

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe _what_ the system does, not _how_ it does it. A good test reads like a specification - "user can checkout with valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means (like querying a database directly instead of using the interface). The warning sign: your test breaks when you refactor, but behavior hasn't changed. If you rename an internal function and tests fail, those tests were testing implementation, not behavior.

See [tests.md](references/tests.md) for examples and [mocking.md](references/mocking.md) for mocking guidelines.

## Scope Gate: Documentation Is Not Runtime Behavior

Before starting RED, classify the change.

For Markdown, issue, decision, README, or instruction changes:

- Do not write a test, parser, validator, or fixture just to assert headings, keywords, row counts, or prose structure.
- Use `git diff --check` and the smallest structural review needed for links, checklist state, or dependency references.
- Human acceptance is the proof for design and documentation decisions.

Use TDD only when the change has executable behavior, a data transformation, or
a runtime contract that can be falsified through a public interface.

Do not grow a multi-thousand-line module to make the current test pass; stop,
reduce scope, or split, and report that in the drift handoff. Deep modules:
small public interface, complexity hidden. See
[deep-modules.md](references/deep-modules.md).

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This is "horizontal slicing" - treating RED as "write all tests" and GREEN as "write all code."

This produces **crap tests**:

- Tests written in bulk test _imagined_ behavior, not _actual_ behavior
- You end up testing the _shape_ of things (data structures, function signatures) rather than user-facing behavior
- Tests become insensitive to real changes - they pass when behavior breaks, fail when behavior is fine
- You outrun your headlights, committing to test structure before understanding the implementation

**Correct approach**: Vertical slices via tracer bullets. One test → one implementation → repeat. Each test responds to what you learned from the previous cycle. Because you just wrote the code, you know exactly what behavior matters and how to verify it.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

## Workflow

### 1. Planning

When exploring the codebase, use the domain glossary so that test names and interface vocabulary match the project's language, and respect the product spec's architecture decisions for the area you're touching.

Before writing any code:

- [ ] Confirm what interface changes are needed
- [ ] Confirm which behaviors to test (prioritize)
- [ ] Identify opportunities for [deep modules](references/deep-modules.md) (small interface, deep implementation)
- [ ] Design interfaces for [testability](references/interface-design.md)
- [ ] List the behaviors to test (not implementation steps)
- [ ] Get approval on the plan

Ask: "What should the public interface look like? Which behaviors are most important to test?"

**You can't test everything.** Focus testing effort on critical paths and complex logic, not every possible edge case.

#### Running unattended

When this skill is attached to an issue worker dispatched by `orchestrate-issues`, **there is no user to ask.** Do not stall waiting for approval and do not invent an answer. Instead:

| Checklist item | Unattended substitute |
|---|---|
| Confirm interface changes | Derive from the issue's **What to build**; if the issue is silent on a choice that is hard to reverse, pick the option most consistent with existing code and **report it in your handoff's Drift report** |
| Confirm which behaviors to test | The issue's **Acceptance criteria** *is* the approved, prioritized behavior list |
| Get approval on the plan | The approved issue file is the approval — proceed straight to the tracer bullet |

Anything you would have asked the user goes in the **Drift report** section of your handoff, not into a question. The orchestrator resolves it and, when material, records it in the project's drift log (default `docs/DECISIONS.md`) — you never write there yourself.

When a real user *is* present (direct invocation, not a worker), follow the checklist as written and ask.

### 2. Tracer Bullet

Write ONE test that confirms ONE thing about the system:

```
RED:   Write test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

This is your tracer bullet - proves the path works end-to-end.

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
```

Rules:

- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior

### 4. Refactor

After all tests pass, look for [refactor candidates](references/refactoring.md):

- [ ] Extract duplication
- [ ] Deepen modules (move complexity behind simple interfaces)
- [ ] Apply SOLID principles where natural
- [ ] Consider what new code reveals about existing code
- [ ] Run tests after each refactor step

**Never refactor while RED.** Get to GREEN first.

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
```
