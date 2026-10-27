# Choosing and designing test doubles

Prefer real pure logic. Double external effects, time or unavailable resources at
the narrowest useful boundary. An owned adapter can still represent an external
boundary; ownership alone does not forbid a double.

Avoid replacing the behavior under test with mocked answers. Do not mock an
internal pipeline so thoroughly that a fake's own state changes become the oracle.
Use a real adapter path where its serialization, identity, targeting or lifecycle
is the distinct risk; do not require real resources for every input variant.

## Reuse an adequate seam before adding one

Dependency injection can make an external effect explicit:

```typescript
function processPayment(order, gateway) {
  return gateway.charge(order.total);
}
```

Use it when it improves the real interface. Do not add production wrappers,
endpoint SDKs, parameters or a parallel runtime solely to make a test convenient.
A specific operation can be clearer than a generic fetch method, but a small
transport fake dispatching several paths can be appropriate. 'No conditionals in
a fake' is not a design goal.

## Keep the double honest

- Supply inputs independently of the SUT's expected calculation.
- Record real boundary effects and assert the values the contract requires.
- Distinguish a caller's declared operation from identity, origin and physical
  observations. A user ID does not encode a UI gesture.
- Keep mutable fixture state isolated; extracting a shared helper must not create
  a hidden shared ledger or replace an independent expected result.
- State what the double omits. Add a small independent wiring/integration check
  only where that omission creates a meaningful gap.

## Isolation is separate from mocking

Patching a consumer name does not patch an imported function's defining globals.
A zero-call recorder does not establish zero transport attempts. Follow the owning
runtime procedure for resource authorization, default-deny transport controls and
actual target verification before any executable fake/body probe. This document
does not authorize one or define a second runner/isolation framework.
