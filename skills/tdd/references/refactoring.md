# Refactor candidates

Default: refactor with required behavior green and understood. If RED exposes an
invalid oracle or stale target, correct that premise with evidence first; do not
restore obsolete behavior merely to enable refactoring.

Consider these candidates, not mandatory transformations:

- **Obsolete/duplicate code or tests:** delete or fold before adding abstraction;
  name lost protection and its survivor, or explicitly retire the claim.
- **Repeated setup:** reuse a small fixture/helper when it lowers reader cost and
  preserves isolation. Avoid both process-per-assertion and fake-world mega-tests.
- **Incidental source checks:** replace with meaningful outcomes where adequate;
  retain independent asset/schema checks whose boundary a fake does not represent.
- **Shallow or long modules:** combine, split or deepen only when the real interface
  and maintenance cost improve. SOLID, private helpers and value objects are tools,
  not a requirement to manufacture layers.
- **Duplicated expectations:** do not deduplicate independent oracles by importing
  the implementation's answer or required-case list.

Use targeted tests during changes and appropriate regressions afterward. Measure
cost when relevant, but do not optimize test count, LOC, coverage or speed at the
expense of meaningful fault protection.
