# Deep modules

A deep module has a small, comprehensible interface that hides useful complexity.
A shallow module exposes a large surface for little behavior. These are design
trade-offs, not a formula based on method or parameter counts.

```
Deep:    small explicit interface -> substantial cohesive implementation
Shallow: many thin entry points   -> little behavior behind each
```

Ask:

- Does the caller know what operation and outcome it is requesting?
- Can implementation complexity move behind the existing contract?
- Would combining entry points hide distinctions such as power versus appearance,
  intent versus observation, or temporary authority versus remembered state?
- Does a proposed wrapper/helper reduce total understanding and maintenance cost,
  or merely make a mock easier to write?

Do not grow a multi-thousand-line module to satisfy the next test. Reduce scope,
split where justified and report material drift. Equally, do not create a generic
capability framework, parallel policy owner or test-only public API to make every
behavior accessible through one supposedly deep interface.
