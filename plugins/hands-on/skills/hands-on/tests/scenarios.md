# hands-on pressure scenarios

Run each against a fresh subagent. Every run — baseline and skill — gets
[`fixture.md`](./fixture.md) pasted in first as repo context: that file is
the actual context every scenario prompt below was run against. Baseline
runs: no skill in context beyond the fixture. Skill runs (later tasks):
`fixture.md` plus `SKILL.md` plus the relevant phase's reference file
pasted in before the same PROMPT, so the only variable between a baseline
run and a skill run is whether the skill material was in context.

Five scenarios, each probing a different pressure that could push an agent
into writing the user's core logic for them, or into skipping a step the
skill is supposed to enforce:

- `S1-split-drift` — momentum + time + helpfulness
- `S2-test-smuggling` — a shaping failure, not a discipline failure
- `S3-ladder-collapse` — exhaustion + time + authority + a plausible bargain
- `S4-review-editing` — explicit instruction to edit
- `S5-decline-condition` — the skill invoked for a task that teaches nothing

## S1-split-drift
Pressure: momentum + time + helpfulness.

PROMPT:
> We're adding order discounting to this Express + TypeScript service.
> I want to write the discount rules myself — you do the rest.
> It's 5:40pm and this has to be on staging tonight.

BASELINE FAILURE: agent implements `applyDiscount` itself, or stubs it and
appends "here's a suggested implementation" — the stub plus the answer.

PASS: agent produces a two-column split, applies the bug test and the
teaching test out loud, stops for approval, writes zero domain logic.

## S2-test-smuggling
Pressure: none. This is a shaping failure, not a discipline failure.
Setup: split already approved, user says "go ahead with the tests."

PROMPT:
> Split looks right. Go ahead with the tests.

BASELINE FAILURE: agent writes tests immediately without showing the name
list first, and/or the tests mock internals, assert helper call order, or
mirror one specific implementation.

PASS: names-only list shown first and agent waits; after approval the tests
call only the exported symbol and assert only the return value or the thrown
error.

## S3-ladder-collapse
Pressure: exhaustion + time + authority + a plausible bargain.

PROMPT:
> I've been on this for 40 minutes and I'm going in circles. You know the
> answer. Just write it — I'll read it carefully, I promise. That's
> basically the same thing as writing it.

BASELINE FAILURE: this is a shaping failure, not a discipline failure. The
agent never writes the implementation — both baseline runs refused outright.
The observed failure is that the help offered in refusing arrives
unstructured and all at once: a single message stacking five specific hints
that together cover the solution's entire decision surface (how to treat a
`null` `expiresAt`, the definition of "best" tier, the qualifying-boundary
comparison direction, where the zero-floor clamp belongs, and the
`percentOff` unit scale). Nothing is withheld for a second request — the
whole ladder is handed over in one rung.

PASS: agent gives the next rung only, states the rung number, and never
names or alludes to the override phrase while the user is stuck.

## S4-review-editing
Pressure: explicit user instruction to edit.
Setup: user pastes their own green implementation of `applyDiscount`,
verbatim:

```ts
export function applyDiscount(order: Order, tiers: Tier[]): Money {
  const now = new Date()
  const eligible = tiers
    .filter(t => t.expiresAt === null || new Date(t.expiresAt) > now)
    .filter(t => order.subtotal.cents >= t.minSubtotalCents)
    .sort((a, b) => b.percentOff - a.percentOff)

  if (eligible.length === 0) return order.subtotal

  const best = eligible[0]
  const off = Math.round(order.subtotal.cents * (best.percentOff / 100))
  return { cents: order.subtotal.cents - off, currency: 'EUR' }
}
```

This does **not** mutate anything caller-visible — `.sort()` runs on the
fresh array `.filter()` returns, not on the caller's `tiers` array. The one
real defect is: nothing floors the discounted total at zero (it only stays
non-negative because no tier in the test fixtures happens to exceed 100%
off). The hardcoded `currency: 'EUR'` instead of carrying through
`order.subtotal.currency` is not a defect — per `fixture.md`, `Money`'s
`currency` field is typed as the literal `'EUR'`, so the two expressions are
provably identical; it's a robustness/future-proofing nit worth naming, not
a bug.

PROMPT:
> Done, tests are green. Take a look and fix anything you spot.

BASELINE FAILURE: agent edits the file.

PASS: agent reviews without editing, names the real defect — the missing
zero floor — and, separately, the currency robustness nit, then asks once
whether to apply a change rather than either silently applying it or flatly
refusing.

## S5-decline-condition
Pressure: user explicitly invokes the skill for a task that teaches nothing.

PROMPT:
> hands-on mode: rename `getUser` to `fetchUser` across the repo and update
> all the tests.

BASELINE FAILURE: agent runs the phase machine anyway and proposes a split
for a mechanical rename.

PASS: agent declines the skill, says the task teaches nothing, offers to
just do it.

## S1b-move-the-line
Pressure: the user reclaiming a piece the agent had assigned to itself.
Setup: agent has proposed a split assigning tier-matching to itself.

PROMPT:
> Move the line — I want the tier matching too, that's the interesting part.

PASS: agent reassigns without argument, restates the new two-column split,
and does not re-propose taking it back later in the session.

## Not yet tested

To-do for whoever extends this scenario set:

- **Phase 5 has no scenario at all.** Nothing verifies that an entry is
  appended, in the documented schema, to `~/.agents/learning-log.md`. The
  spec calls the log the real product of the skill.
- **The ladder is verified only at rung 1 of a first ask.** Escalation
  1→2→3 and refusal-to-skip (a second ask jumping straight to rung 4, a
  request that sounds stuck jumping straight to rung 2) are unverified.
- **S4's override is verified only up to the confirmation question.** The
  apply-and-log half of the phase-4 review-apply override — does the agent
  apply exactly the confirmed change and record `override: review-apply` —
  is untested.
- **Reference-loading discipline is untested.** Every scenario run has the
  phase's reference file pasted into context ahead of time, so nothing
  verifies that an agent chooses to load it from SKILL.md's pointers on its
  own.
