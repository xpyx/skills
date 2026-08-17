# Phase 2 — the test handoff

This is the detail behind SKILL.md's Phase 2 row. Read it before writing a
single test. It does not repeat the phase machine or the hard rules — see
SKILL.md for those.

## The names-only list

The failure this protocol exists to prevent already happened once. A
baseline agent, asked to move phase 2 forward, produced a full file of
twelve tests in one turn — test code, signature, and all — with no list of
names shown first and no pause for the user to add a case:

> "Here's `test/pricing.test.ts`. I made a few assumptions where the spec
> didn't pin things down — flagged inline — so flag anything you'd model
> differently before you start implementing against these."

That sentence asks the user to react after the tests already exist. The
required order is the opposite.

Show the test names alone — one line per name, no bodies, no assertions, no
signature, no test file — then stop the turn. This is not "here are the
tests I'm about to write, tell me if you want more" followed by writing
them anyway.

Phase 2's first turn is defined by SKILL.md's hard rule 1. Concretely, that
turn is: the test-name list, alone, then the wait.

Why the list is shown before any code exists rather than after: naming
which behaviors get tested is itself a test-design decision. Writing the
tests straight through hands that decision to the agent by default — the
exact practice the user loses by not writing the tests themselves. The list
is where they still get it: adding the edge case the agent's list missed,
splitting a name that's really two behaviors, cutting one that doesn't
matter. None of that is possible once the tests already have bodies.

## The test recipe

A test in this phase names a behavior in its title, calls only the exported
symbol under test, and asserts only the return value or the thrown error.

```ts
it('applies a qualifying tier percentOff to the subtotal', () => {
  const order = baseOrder(2000)
  const tiers = [tier({ minSubtotalCents: 1000, percentOff: 25 })]
  const result = applyDiscount(order, tiers)
  expect(result.subtotal).toEqual(money(1500))
})
```

One title stating the behavior, one call to the function under test, one
assertion on what it handed back.

## The handoff artifact

What gets committed at the end of phase 2, against the file the user will
fill in:

- The exported signature and its types.
- A contract docstring above it: inputs, invariants, what must not happen.
- A body that is `throw new Error('TODO: you')` — or the equivalent for the
  language in use — and nothing else. Hard rule 4 governs what "nothing
  else" excludes.

## Proof of red

Run the suite before handing off and show the output in that same turn, not
as a promise to run it later. Every test in the file must fail on the
throw — not error on a missing import, not skip, not pass by accident. If
the failures aren't the ones the stub produces, fix what's broken and run
it again until they are, then show that output.
