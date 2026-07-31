---
name: grill-me
description: Grilling session that interrogates your plan one question at a time, challenges it against the existing domain model and codebase, sharpens terminology, and updates the CONTEXT.md glossary inline as decisions crystallise. Use when the user says "grill me", asks to stress-test or poke holes in a plan, wants to settle requirements, or wants vocabulary pinned down before building. First stage of the backlog-native pipeline — when the plan is settled, hand off to the `to-issues` skill.
---

# grill-me — interrogate the plan before building it

<what-to-do>

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing.

If a question can be answered by exploring the codebase, explore the codebase instead.

</what-to-do>

<supporting-info>

## Domain awareness

During codebase exploration, also look for existing documentation:

### File structure

A project has **one context** and three distinct documents. Know which one a decision belongs in. Paths are **defaults** — if the repo already names them differently, its names win.

```
/
├── CONTEXT.md                        ← glossary only: terms, _Avoid_, _See also_
├── docs/
│   ├── REQUIREMENTS-AND-NOTES.md     ← product spec + architecture decisions
│   └── DECISIONS.md                  ← execution drift log (orchestrator-owned)
└── <source>/
```

Architectural choices go in the spec's Architecture table, naming the rejected alternative alongside the choice. Delivery-time drift goes in the drift log. Keep decisions in these two files rather than starting a third home for them.

**Create files lazily** — only when you have something to write. If `CONTEXT.md` exists, update it in place and never recreate it; if it does not, create it when the first term resolves, using [CONTEXT-FORMAT.md](references/CONTEXT-FORMAT.md). Same for the spec: patch it if present, start it when the session settles its first decision, using [SPEC-FORMAT.md](references/SPEC-FORMAT.md).

### On a blank slate

If the repo has no code and no `CONTEXT.md` yet, exactly two of the techniques below have nothing to work with — *challenge against the glossary* and *cross-reference with code*. Say so once, then lean on *sharpen fuzzy language* and *discuss concrete scenarios* instead, and keep writing the glossary as terms resolve. Do not repeatedly search a codebase that isn't there; cross-referencing resumes as soon as there is code.

## During the session

### Challenge against the glossary

When the user uses a term that conflicts with the existing language in `CONTEXT.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y — which is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account' — do you mean the Customer or the User? Those are different things."

### Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### Cross-reference with code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible — which is right?"

### Update CONTEXT.md inline

When a term is resolved, update `CONTEXT.md` right there. Don't batch these up — capture them as they happen. Use the format in [CONTEXT-FORMAT.md](references/CONTEXT-FORMAT.md).

`CONTEXT.md` should be totally devoid of implementation details. Do not treat `CONTEXT.md` as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

### Record architectural trade-offs in the spec

When a decision meets all three of these, it needs a written home:

1. **Hard to reverse** — the cost of changing your mind later is meaningful
2. **Surprising without context** — a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked one for specific reasons

Add it as a row in the spec's Architecture table (see [SPEC-FORMAT.md](references/SPEC-FORMAT.md)), naming the rejected alternative alongside the choice:

```md
| Decision | Choice |
|----------|--------|
| HTTP server | **Framework X** — not a hand-rolled gateway |
| Library-only, no server | **Rejected** for v1 |
```

If any of the three is missing, write nothing.

### Where the grill session ends

A grill session produces a sharpened glossary and a patched spec — it does **not** produce a PRD. There is no separate requirements document to write: the spec *is* the artifact, and it is durable rather than per-feature.

When the plan is settled, hand off to the `to-issues` skill, which reads the spec and glossary directly and breaks the work into `issues/`.

</supporting-info>
