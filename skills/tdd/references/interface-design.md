# Interface design for testability

A good interface exposes the real contract clearly. Testability is a benefit,
not permission to redesign production solely around a convenient fake.

- **Separate meaningful intent.** If two operations have different authority or
  lifecycle, the interface must declare the distinction or document its ambiguity.
  Fewer methods/parameters is not better if it hides required information.
- **Keep pure decisions separate from effects where useful.** A result-returning
  calculation is often the cheapest test boundary. A side-effecting operation is
  not wrong when the effect is its purpose.
- **Accept dependencies when it clarifies a real boundary.** Reuse adequate existing
  seams; do not inject every internal collaborator or create an SDK per endpoint.
- **Minimize caller complexity.** A small explicit surface can hide substantial
  implementation. One generic method with an opaque request is not automatically
  simpler, nor does fewer methods mean fewer behaviors need protection.
- **Check maintained use.** A testable exported helper with no production callers
  can be stale. Do not preserve it merely because its unit test is green.
