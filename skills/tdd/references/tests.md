# Test quality: contract, oracle and boundary

## Prefer observable outcomes, not incidental structure

```typescript
// A maintained interface and a caller-visible outcome.
test("checkout creates a retrievable order", async () => {
  const order = await checkout(cart, paymentMethod);
  expect((await getOrder(order.id)).status).toBe("confirmed");
});
```

Integration style is not a quality grade. Use a pure-core test for arithmetic or
arbitration when sufficient; use an adapter/integration test for the extra risk
of serialization, targeting, persistence, dispatch or lifecycle. A function being
public does not make it a real runtime boundary: verify its maintained callers.

## Choose a discriminating oracle

| Weak setup | Better contrast |
|---|---|
| Current effect and selected effect are both Solid | Select a different value; use more than one value if testing passthrough |
| Empty saved state stays empty | Also preserve a populated independent input, including unrelated state |
| Native and policy limits are equal | Make each source stricter in a relevant case |
| Slider display is correct | Also inspect the actual boundary request when request semantics matter |
| Negative context has no user identity | Include inherited identity when parent-origin precedence is the rule |

Do not delete the original input automatically when adding a contrast. Empty
and populated state can detect different faults. Several assertions or parameter
rows are justified when they distinguish required outcomes, not merely more values.

## Doubles have proof limits

A fake DOM that invents IDs cannot prove real HTML wiring. A payload assertion
cannot prove backend holds or physical output. A fake recorder with zero calls
cannot prove that another path made no real I/O. Keep those boundaries explicit.

Source/format checks are usually weak behavior proof: dead code or a comment can
satisfy them. They can still validate real asset bindings, schema or approved
configuration. Reject incidental spelling, not every static check categorically.

## Calls and storage are sometimes the contract

Internal collaborator calls usually overfit structure. External command payload,
sequence, absence, scope or no-duplicate dispatch can be the actual required effect.
Assert those when they are the contract; do not reject them as call-count tests.

For ordinary business retrieval, prefer the retrieval interface over inspecting
an internal table. For persistence, rollback and cleanup contracts, a full store
comparison plus real reload and public projections may be necessary. Do not invent
a new production API merely to avoid inspecting storage that the contract owns.

## Removing or folding tests

Identify the exact survivor and a concrete fault it still detects at the needed
boundary. Check the body and fixture, not the name. Similar-looking guards can
protect different controls. A weak old test can have a distinct catch until the
new oracle is strengthened.

If a claim is intentionally retired without a survivor, say so. No case, coverage
or line target justifies hiding that loss. A configured-future test is not evidence
that the feature is shipped or currently used; inactivity alone is not obsolescence.
