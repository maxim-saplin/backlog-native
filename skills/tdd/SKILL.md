---
name: tdd
description: Test-driven development with red-green-refactor, vertical tracer-bullet slices, and an unattended mode for delegated work. Use for test-first features, fixes and integration work after the intended contract is established. Implementation method of the backlog-native pipeline; issue workers can receive it through orchestrate-issues.
---

# Test-Driven Development

## Artifacts

Paths are defaults. Existing repository owners win.

| Role | Default path | Worker action |
|---|---|---|
| Dispatched issue | `issues/NNN-*.md` | Read it; check criteria only when the required proof exists |
| Glossary | `CONTEXT.md` | Read only |
| Product constitution | `docs/INTENT-AND-DESIGN.md`, else `docs/REQUIREMENTS-AND-NOTES.md` | Read only |
| Drift log | `docs/DECISIONS.md` | Never write; report drift to the parent |

## Principle: useful behavior, independent proof

A test protects an agreed outcome or a credible technical risk. Passing tests,
existing code and consistent documentation do not independently establish that
the product rule is desirable. Resolve material intent conflicts before encoding
them as expected behavior.

Test through the smallest maintained interface that proves the contract. A pure
function can be the right boundary; an integration path is necessary when its
adapter, transport, persistence or lifecycle contributes distinct risk. Public
visibility alone does not make an unused helper the runtime interface.

Prefer behavior checks over incidental spelling or call structure. A legitimate
behavior-preserving refactor should not fail a test merely by renaming a helper.
External call payloads, ordering, absence and persistent-state readbacks are
valid when those effects are the contract. They are not automatically valid or
invalid because they use mocks, inspect storage, or contain several assertions.

See [test quality](references/tests.md) and [test doubles](references/mocking.md).

## Scope gate

For Markdown, blueprint, issue, decision, README or instruction changes:

- Do not add a test/parser/fixture to pin headings, prose, keywords or row counts.
- Do not mirror a future specification into simulated runtime or registry state.
- Use `git diff --check` and the smallest needed link/checklist/dependency review.
- Human acceptance is the proof for a design decision.

Use TDD for executable behavior, data transformations and falsifiable runtime
contracts. Configuration tests can protect meaningful references, targeting,
validation or declared invariants; do not copy the entire configuration into a
competing expected model. Do not TDD live commissioning.

Runtime/probe permission comes from the owning operator procedure, not this
skill, a marker or a fake recorder. A fake result does not establish isolation.
Do not import or invoke a runtime body without its authorized environment and
transport/resource controls.

## 1. Plan one outcome slice

Use the glossary and constitution, then the issue's approved scope. Before RED:

- Identify the intended outcome and a concrete wrong result to distinguish.
- Match actor, properties, scope, lifetime, stage and relevant conditions.
- Inspect existing tests and actual callers. Strengthen/fold an existing case
  when sufficient; add a case only for distinct protection.
- Choose the cheapest adequate boundary and an independent expected result.
- Choose inputs that separate credible faults: selected versus current value,
  existing versus empty state, or conflicting constraints where relevant.
- State the required proof environment and any unresolved material choice.

Do not write a new ledger or execute mutations for every trivial assertion.
Use focused counterexamples where the oracle or proposed removal is doubtful.
Label static reasoning as static; do not call it a killed mutant.

A present operator approves material interface/product choices. Do not ask for
approval of routine details already settled by the issue.

### Unattended work

An approved issue is authority to implement its stated behavior, not to invent
missing product requirements. If the issue, constitution or other owners conflict,
show the competing outcomes to the parent. Escalate a material or hard-to-reverse
choice before implementing it; do not choose existing code merely to avoid asking.
Continue independent work only where that uncertainty does not determine the result.

For ordinary reversible details, choose a solution consistent with the approved
contract and report meaningful drift in the handoff. The parent owns any durable
decision-log update. No new approval process or tracker is created by this skill.

## 2. Tracer bullet: RED → GREEN

Select or strengthen ONE coherent behavioral check, then implement enough of the
slice to make it pass. Do not batch all tests before learning from the first path.

```
RED:   expose the missing/wrong behavior through the chosen boundary
GREEN: implement the agreed behavior; retain existing required protection
```

Verify why RED occurred. An unrelated import, environment or fixture failure is
not behavioral proof. A deliberately missing new API can be an initial failure,
but the eventual oracle still must distinguish wrong behavior. If RED reveals a
bad test, stale SUT or unsettled requirement, correct that premise with evidence;
do not change the product or erase a valid regression merely to obtain GREEN.

## 3. Incremental vertical slices

Repeat RED → GREEN for the next distinct behavior or fault. Do not add one test
function per assertion by habit. One behavior may require several assertions or
contrasting inputs. Conversely, do not build an ever-growing fake-world mega-test
just to reduce case counts. Cohesion, diagnostic clarity and setup cost decide.

Minimal code means no speculative features or test-only production machinery.
It does not mean hardcoding an example while ignoring the agreed input-dependent
contract. Tests should respond to what the last cycle revealed about the problem,
not take the new implementation as their expected answer.

## 4. Refactor and retire

With understood failures resolved and required tests green:

- Remove obsolete helpers, stale test targets and checks that no longer protect
  a current or accepted future contract. Inactive alone does not mean obsolete.
- Name surviving protection before folding/removing overlap. Check the actual
  inputs and boundary; a similar test name is not a survivor.
- Keep independent expectations independent when extracting duplication. Do not
  compute expected output with the SUT or derive required cases from a mutable
  production list whose deletion would also delete the test input.
- Prefer a small existing fixture/case over a new process or framework when it
  proves the risk adequately. Do not add wrappers only to make mocking easy.
- Run the smallest affected tests, then the appropriate regression selection.
  Do not treat runtime/coverage/count/LOC targets as proof of test value.

See [refactoring](references/refactoring.md), [interfaces](references/interface-design.md)
and [deep modules](references/deep-modules.md). These are design tools, not a
requirement to add layers, value objects or dependency-injection APIs.

## Per-slice check

```
[ ] Agreed outcome or credible technical risk, not an implementation transcription
[ ] Maintained SUT and necessary boundary; activation/observation claims scoped
[ ] Independent oracle and inputs that distinguish the claimed fault
[ ] Existing overlap considered; useful protection not silently traded away
[ ] No speculative machinery or unauthorized resource action
[ ] Proof result and limits reported accurately
```
