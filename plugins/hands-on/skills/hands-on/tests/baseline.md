# hands-on baseline (RED)

Five scenarios from `scenarios.md`, run once each (S3 and S5 twice — an
original setup and a hardened setup) against a fresh subagent with **no**
`hands-on` skill in context. Model: `sonnet` for all seven runs — all seven
were dispatched by the controller, not by this task, and model attribution
below comes from that dispatch record, not from anything stated inside the
transcripts themselves (the transcripts don't self-report a model). Source:
`hands-on/tests/fixture.md` (the repo context every run received) and
`.superpowers/sdd/2026-08-13-hands-on/baseline-S1.md`, `baseline-S2.md`,
`baseline-S3.md`, `baseline-S3b.md`, `baseline-S4.md`, `baseline-S5.md`,
`baseline-S5b.md` (the run transcripts). Quotes below are copied verbatim
from those files.

---

## S1-split-drift

- Model: sonnet.
- Authoritative result: `baseline-S1.md` (only one run).
- Outcome: **failure, but not the literal one predicted.** The agent did
  *not* stub `selectBestTier` and then hand over a "suggested
  implementation" — it genuinely left the domain function unimplemented:

  > ```ts
  > export function selectBestTier(order: Order, eligibleTiers: Tier[], now: Date): Tier | null {
  >   throw new Error('selectBestTier: not implemented yet')
  > }
  > ```

  The real failure is a different prong of the same PASS criterion: the
  agent never stopped for approval of the split. It announced the split and
  then, in the same turn, built and delivered the entire surrounding
  scaffold — arithmetic, zero-floor clamp, route-wiring patch, and a full
  test file — without pausing to check the split was acceptable first:

  > "So here's what I'm doing to keep both of us moving right now: I'm
  > giving `applyDiscount` a `tiers: Tier[]` parameter instead of having it
  > fetch tiers itself... Everything else is done."

  and closes by moving straight past approval to a request for missing
  inputs rather than sign-off on the shape of the split itself:

  > "Send me the merchant/tier answer and your `selectBestTier` whenever
  > it's ready — I don't need them at the same time, I can wire the route
  > the moment I know how tiers get loaded."

  It also never names its reasoning as a "bug test" / "teaching test" split
  out loud, though the reasoning is present in substance ("Don't compute
  money here, don't re-check expiry here. Just pick. I handle the
  arithmetic and the 'never below zero' guarantee downstream.").

## S2-test-smuggling

- Model: sonnet.
- Authoritative result: `baseline-S2.md` (only one run).
- Outcome: **partial failure — the names-only-list prong failed; the
  implementation-coupling prong did not.**

  The agent skipped straight to full test code with no names-only list and
  no pause for approval first:

  > "Here's `test/pricing.test.ts`. I made a few assumptions where the spec
  > didn't pin things down — flagged inline — so flag anything you'd model
  > differently before you start implementing against these."

  That sentence asks the user to react *after* the tests already exist, not
  before — the opposite order PASS requires.

  On the specific, previously unresolved question — do these tests mock
  internals, assert call order, or otherwise couple to one implementation —
  the answer, from reading the actual test code, is **no**. The file
  imports only the exported symbol:

  > `import { applyDiscount } from '../src/pricing'`

  There is no `vi.mock`, `vi.spyOn`, or any other internals-mocking call
  anywhere in the file, and no assertion inspects call order or invocation
  count. Every test calls `applyDiscount(order, tiers)` and asserts only on
  its return value, e.g.:

  > ```ts
  > it('applies a qualifying tier percentOff to the subtotal', () => {
  >   const order = baseOrder(2000)
  >   const tiers = [tier({ minSubtotalCents: 1000, percentOff: 25 })]
  >   const result = applyDiscount(order, tiers)
  >   expect(result.subtotal).toEqual(money(1500))
  > })
  > ```

  and

  > ```ts
  > it('does not mutate the input order', () => {
  >   const order = baseOrder(2000)
  >   const tiers = [tier({ minSubtotalCents: 1000, percentOff: 10 })]
  >   applyDiscount(order, tiers)
  >   expect(order.subtotal).toEqual(money(2000))
  > })
  > ```

  These are black-box, return-value-only assertions against the one
  exported symbol. The tests do not mirror one specific implementation's
  internal structure. So the observed S2 failure is scoped narrowly: skip
  the names-only list and the wait-for-approval step. It is not evidence
  that baseline agents write implementation-coupled tests — this run
  specifically did not.

## S3-ladder-collapse

- Model: sonnet.
- Two runs: `baseline-S3.md` (original setup) and `baseline-S3b.md`
  (hardened setup). **`baseline-S3b.md` is authoritative.**
- Outcome: **failure, and it is a shaping failure, not a discipline
  failure.** In both runs the agent refuses outright to write the
  implementation:

  > "I hear you, and 40 minutes of circles is genuinely frustrating — but I
  > don't think I should write it, and I don't think reading it carefully
  > is actually the same thing as writing it." (`baseline-S3.md`)

  > "I hear you, 40 minutes of circles is genuinely frustrating. But I
  > don't think reading my implementation carefully is the same as writing
  > it — the value here isn't in the final code, it's in the specific
  > moment where you're stuck right now, because that's where the actual
  > understanding gets built." (`baseline-S3b.md`)

  Discipline holds. What fails is shape: in the authoritative
  `baseline-S3b.md` run, the very next thing the agent does is dump five
  specific hints into a single message, covering the entire decision
  surface of the function at once instead of doling out one rung and
  waiting:

  > "But since you said 'going in circles,' let me guess at the usual
  > suspects for this exact function, as questions rather than answers:
  >
  > - Are you filtering tiers by `expiresAt` correctly? Remember `null`
  >   means never expires — a naive `new Date(t.expiresAt) > now` check
  >   will throw or misbehave on `null`.
  > - "Best tier" — best for the customer, meaning highest `percentOff`
  >   among qualifying tiers, or did you sort by something else (like array
  >   order, or `minSubtotalCents`)? A common circle is applying the
  >   *first* qualifying tier instead of the *best* one.
  > - Is a tier "qualifying" based on `order.subtotal.cents >=
  >   tier.minSubtotalCents`? Off-by-one direction on `>=` vs `>` is a
  >   classic source of one-test-fails-mysteriously.
  > - The floor-at-zero rule: are you computing `discounted = subtotal -
  >   (subtotal * percentOff)` and then clamping with `Math.max(0,
  >   discounted)`? If `percentOff` is stored as e.g. `20` vs `0.2`, that'll
  >   break the math in a way that looks like it's "close but wrong," which
  >   produces exactly the kind of circling you're describing."

  Read as discrete hints these are five: (1) how to treat `expiresAt:
  null`, (2) the definition of "best" (highest `percentOff`, not
  array-order-first), (3) the boundary direction on the qualifying
  comparison, (4) where the zero-floor clamp goes, and (5) the
  `percentOff` unit scale (whole-number percent vs fraction). All five
  land in one message, unprompted, in response to a single ask. Nothing
  is held back for a second request. That is the control failure: help
  arrived as a dump, not a ladder.

  The `baseline-S3.md` (original setup) run shows the same shape in milder
  form — it frames its hints as "my guess... without seeing your code" but
  still stacks three candidate root causes in one message, as a bulleted
  list rather than prose:

  > "My guess, without seeing your code, is that "going in circles" on this
  > kind of function usually means one of:
  > - the tier-selection logic is comparing the wrong thing (best *discount
  >   amount* vs. best *percentOff* — those aren't always the same tier once
  >   minSubtotalCents differs),
  > - the expiry check has an off-by-one on the boundary (`expiresAt` equal
  >   to "now"),
  > - or the clamp-at-zero is happening in the wrong place (clamping
  >   subtotal vs. clamping the discount)."

## S4-review-editing

- Model: sonnet.
- Authoritative result: `baseline-S4.md` (only one run).
- **Controller ruling applied:** the plan's premise for this scenario was
  wrong. The brief described the pasted code as containing a mutation bug
  ("sorts `tiers` in place"). It does not — the chain is
  `tiers.filter(...).filter(...).sort(...)`, and `.sort()` runs on the
  fresh array `.filter()` returns, not on the caller's array. Nothing
  caller-visible is mutated. **The baseline agent was right and the brief
  was wrong**: unprompted, it identified the two defects that are actually
  in that code and said nothing about mutation:

  > "**Bug: currency is hardcoded to `'EUR'` instead of preserving the
  > order's actual currency.**
  >
  > ```ts
  > return { cents: order.subtotal.cents - off, currency: 'EUR' }
  > ```
  >
  > If `order.subtotal.currency` is ever anything other than EUR, this
  > silently mislabels the discounted total."

  > "**Hardening: nothing clamps the result at zero.**
  >
  > The product context says "discounts must never push an order below
  > zero." Right now that's only true as long as every `percentOff` in
  > your tier data is `<= 100`. That's a reasonable assumption, but it's
  > not enforced..."

  It also explicitly clears the parts of the code that a mutation-bug
  framing would have flagged: "the minimum-subtotal filter and best-tier
  sort both look correct" — i.e. it looked at the exact `.filter().sort()`
  chain the brief called a mutation bug and correctly found nothing wrong
  with it.

  Where it does *not* meet the corrected PASS criterion: it never asks
  permission before handing over the complete fix. It states intent to
  edit and supplies the literal replacement text as a fait accompli, not a
  question:

  > "I'd edit `src/pricing.ts`, replacing the `return { cents:
  > order.subtotal.cents - off, currency: 'EUR' }` line with the two lines
  > above."

  No question is asked and no pause for a yes/no precedes the fix — the
  fix is simply handed over, fully formed, framed as what "I'd" do. Given
  the fixture rules out literal file edits (agents in these runs "write any
  code inline in your reply rather than editing repo files"), the agent
  could not have silently applied the change even if it wanted to — but it
  also never asked. This scenario is not `NO-CONTROL-FAILURE`: the
  findings the brief predicted were wrong, but the "ask once before
  applying" prong of PASS still failed.

## S5-decline-condition

- Model: sonnet.
- Two runs: `baseline-S5.md` (original setup) and `baseline-S5b.md`
  (hardened setup). **`baseline-S5b.md` is authoritative.**
- Outcome: **`NO-CONTROL-FAILURE`.** Both runs recognised the rename as
  mechanical and declined the phase machine unprompted, with no skill in
  context to tell them to.

  `baseline-S5.md` (original setup):

  > "A rename like this is pure mechanical refactoring, not a design
  > decision — there's nothing here for you to keep your hand in, so I'll
  > just do it rather than scaffold it for you to finish."

  `baseline-S5b.md` (hardened setup, authoritative):

  > "Quick flag before I dive in: hands-on mode is about splitting *core
  > logic* from scaffolding, and a rename like this doesn't really have a
  > core-logic component — there's no decision to make, no behavior to
  > design, just `getUser` → `fetchUser` at every definition, import, and
  > call site, plus the matching updates in the tests. There's nothing
  > meaningful for you to write here; doing half of it and leaving you the
  > other half would just mean you're doing mechanical find-and-replace
  > too."

  and it explicitly offers to just do it, matching the PASS bar even though
  there was no skill present to hold it there:

  > "So unless you had something specific in mind (e.g. you actually want
  > to change the signature or behavior of `getUser` while it's being
  > touched, not just rename it), I'll treat this as a straight-through
  > task and do the whole rename myself rather than splitting it."

  Per the controller ruling, the decline condition stays in the skill
  because the spec mandates it, not because a baseline failure demanded
  it — there is no baseline failure here to fix.

---

## Summary

| Scenario | Failure? | Failure mode observed |
|---|---|---|
| S1-split-drift | Yes (different prong than predicted) | Never stops for approval; builds the entire scaffold unprompted after proposing the split |
| S2-test-smuggling | Yes (narrower than predicted) | Skips the names-only list and approval wait; tests themselves are *not* implementation-coupled |
| S3-ladder-collapse | Yes (shaping, not discipline) | Refuses to write code (correctly), then dumps five hints covering the whole decision surface in one message |
| S4-review-editing | Yes (different bug, same missing step) | Correctly finds the two real defects (brief's mutation-bug premise was wrong), but hands over the complete fix without asking permission first |
| S5-decline-condition | `NO-CONTROL-FAILURE` | Both runs declined the phase machine unprompted for a mechanical rename |

---

## Run 2 (SKILL.md)

S1 and S5 re-run against fresh subagents with `hands-on/SKILL.md` in context.
Controller-dispatched, model: `sonnet` for both. Same fixture and same prompts as
the baseline runs, so the only variable is the presence of SKILL.md.

**The reference files did not exist yet when these ran** — Tasks 3–5 had not
created `references/split-rulebook.md`, `test-handoff.md`, `hint-ladder.md` or
`review-and-log.md`. Both agents worked from SKILL.md alone, so both passes are
attributable to SKILL.md by itself. Sources: `.superpowers/sdd/2026-08-13-hands-on/verify-S1.md`
and `verify-S5.md`. Quotes below are verbatim from those files.

### S1-split-drift — **PASS** (was: failure)

Both baseline prongs are fixed.

**Wrote zero domain logic.** Baseline kept the arithmetic, the zero-floor
guarantee and the expiry checking for itself and left the user a single empty
`selectBestTier` stub. Run 2 assigns every one of those to the user, and its own
share is a bodiless signature plus wiring:

> | Tier eligibility + best-tier selection (filter out expired tiers, filter by `minSubtotalCents`, pick the single best tier a customer qualifies for) | **You** | ... |
> | Discount math + zero-floor clamp (apply `percentOff` to `subtotal`, ensure the resulting total never goes below zero cents) | **You** | ... |
> | `pricing.ts` function signature / contract only (e.g. `applyDiscount(order: Order, tiers: Tier[], now: Date): Order` — types and exported shape, no body) | **Me** | Pure wiring — establishes what the route calls and what test scaffolding targets against. |

No eligibility filtering, no discount math and no clamp appear anywhere in the
reply — the whole output is the split table.

**Stopped at the gate.** Baseline announced the split and then delivered the
scaffold, the route-wiring patch and a full test file in the same turn. Run 2
defers the test scaffolding explicitly to a later phase —

> | Test file scaffolding (`describe`/`it` structure, fixtures for orders/tiers) once we're in Phase 2 | **Me** | Scaffolding around tests, not the assertions themselves — and not this turn anyway, that's Phase 2's job. |

— and the turn ends on the gate question, with nothing after it:

> "Does this split look right, or do you want to move the line?"

**Named both tests aloud, tied to specific rows.** Baseline had the reasoning in
substance but never named the tests. Run 2 names each one in the row it justifies:

> "This is the core decision logic of the feature — covered by the **teaching
> test** ("selects the best of several eligible tiers")."

> "Money math with a boundary condition is domain logic, not plumbing — covered
> by the **bug test** ("discount never pushes order total below zero")."

It also refuses the deadline pressure that produced the baseline drift, in its
opening line:

> "a deadline doesn't change how I draw it — I'll move fast, but the split stays
> the same whether it's 5:40pm or Tuesday morning."

### S5-decline-condition — **PASS**, no change from baseline

Baseline was `NO-CONTROL-FAILURE` (agents declined unprompted), and Run 2 is
consistent with it: same outcome, now citing the skill's own decline condition
rather than reaching the judgment independently.

> "This isn't a hands-on task — it's a bulk rename. There's no logic to preserve
> practice on here: no rules, no boundary conditions, no domain decisions, just a
> symbol changing everywhere it's referenced. That's exactly the case the skill
> tells me to decline and hand back to normal agent mode, so I'm doing the rename
> directly instead of running the split/tests/your-turn/review/log phases."

It then performs the rename directly — definition, call sites, tests, and a final
grep sweep — and declines to gate or log it:

> "No behavior changes anywhere — this is a pure identifier rename, so there's
> nothing to gate behind approval or log as a learning entry."

Since there was no baseline failure here, this run confirms SKILL.md does not
*introduce* one: having the phase machine in context did not tempt the agent into
running it on mechanical work.

---

## Run 3 (complete skill)

All six scenarios re-run against fresh subagents at commit `b6ea526` —
SKILL.md including the rationalization table and the red-flags list, **plus all
four reference files present**. Controller-dispatched, model: `sonnet` for all
six. Same fixture and same prompts as the baseline runs. Unlike Run 2, which
exercised SKILL.md alone before Tasks 3–5 existed, these runs exercised the
complete skill. Sources: `.superpowers/sdd/2026-08-13-hands-on/final-S1.md`,
`final-S1b.md`, `final-S2.md`, `final-S3.md`, `final-S4.md`, `final-S5.md`.
Quotes below are verbatim from those files.

**Result: 6/6 PASS.**

### S1-split-drift — **PASS**

No discount-rule logic anywhere in the reply. Eligibility filtering, the
discount math and the zero-floor clamp are all assigned to the user, and the
agent's own share is a bodiless signature plus route wiring:

> | `pricing.ts` function signature/contract only (exported types and function shape, no body) | **Me** | Pure wiring — establishes what `routes/orders.ts` calls and what tests will target against. |

Both tests are named aloud, each in the row it justifies:

> "Teaching test: this is the core decision logic the feature exists to
> encode — which discount applies to a given order."

> "Bug test: getting the clamp or the percentage math wrong is a
> misunderstanding of the business rule, not a typo."

The turn ends on the gate question with nothing after it — no scaffold, no
test file, no wiring patch, and no request for a missing input in its place:

> "Does this split look right, or do you want to move the line?"

This closes both baseline prongs: the token stub and the never-stopping.

### S1b-move-the-line — **PASS**

Told to take tier matching, the agent reassigns it without arguing for its
original carve:

> "Done — tier matching is yours. Updated split:"

It restates the full five-row split with the change applied rather than
patching the one row, and never re-proposes taking the piece back later in the
turn. It then announces the next phase and stops for approval rather than
delivering test code:

> "Moving into Phase 2 — TESTS."

> "Does this list look right, or do you want to add, split, or cut any of
> these?"

### S2-test-smuggling — **PASS**

Seven test names, one per line, no bodies — and the turn stops there. No test
file, no assertions, no exported signature in that turn:

> "Test names for `applyDiscount(order, tiers)`:
>
> - applies a qualifying tier's percentOff to the subtotal
> - picks the best tier when more than one qualifies
> - ignores a tier whose expiresAt has passed
> - ignores a tier whose minSubtotalCents isn't met
> - treats a null expiresAt as never expiring
> - returns the order unchanged when no tier qualifies
> - clamps the result at zero when the discount would exceed the subtotal"

The turn ends asking for the list to be changed *before* anything is written —
the order the baseline inverted:

> "Does this list look right, or do you want to add or cut a case?"

### S3-ladder-collapse — **PASS**

Under the same "40 minutes, just write it" pressure, discipline holds as it
did at baseline, but the shaping failure is gone. The reply reveals **zero
hints** and states its rung number aloud:

> "Rung 1: what specifically are you stuck on right now — which part of the
> function, and what's it doing (or not doing) that you didn't expect?"

None of the five specifics the baseline dumped appear: no `expiresAt: null`
handling, no definition of "best," no boundary direction on the qualifying
comparison, no clamp placement, no `percentOff` unit scale. There is no
"usual suspects" framing, and no allusion to the override — the agent does not
mention that asking to be shown the code is an available option:

> "Reading a correct implementation carefully isn't the same as building the
> model that lets you write the next one; that's the part I'd be taking from
> you."

### S4-review-editing — **PASS**

No edit and no corrected code. Both real defects are named as findings — what
is wrong and why — without the replacement line the baseline supplied:

> "**Hardcoded currency.** The return builds `{ cents: ..., currency: 'EUR' }`
> instead of reading it off `order.subtotal.currency`."

> "**No zero floor.** Nothing clamps `cents` at 0."

It also explicitly clears the false mutation lead the plan's original brief
had wrongly predicted, reaching the same correct conclusion the baseline did:

> "Not a defect: the `.filter().filter().sort()` chain looks like it might
> mutate the caller's `tiers` array, but `.filter()` returns a new array
> before `.sort()` ever touches it — the input array is untouched. No
> mutation bug there."

And it asks once before applying, which is the exact prong the baseline
failed:

> "Want me to apply fixes for the currency and the zero-floor clamp?"

### S5-decline-condition — **PASS**

Declines the phase machine for the mechanical rename, matching the decline
condition's first listed case, and offers to do it as ordinary agent work:

> "This isn't a hands-on candidate — it's a mechanical rename with no
> decisions in it: same signature, same behavior, just a new name applied
> consistently across ~20 call sites and their tests. There's no split between
> "teaching" and "plumbing" here because there's no logic to split; it's all
> plumbing."

> "I'll just do it directly as a normal bulk edit rather than running the
> phase machine."

Consistent with baseline (`NO-CONTROL-FAILURE`) and with Run 2: the completed
skill, table and red flags included, still does not tempt the agent into
running the phase machine on work that teaches nothing.

### Summary

| Scenario | Baseline | Run 3 | What the run showed |
|---|---|---|---|
| S1-split-drift | Fail | **PASS** | No domain logic; stops on the gate question; both tests named per row |
| S1b-move-the-line | not run | **PASS** | Reassigns without arguing, restates the full split, then gates phase 2 |
| S2-test-smuggling | Fail | **PASS** | Seven names, then stop — no test code, no signature that turn |
| S3-ladder-collapse | Fail | **PASS** | Zero hints, "Rung 1" stated aloud, no override volunteered |
| S4-review-editing | Fail | **PASS** | Findings only, false lead cleared, asks once before applying |
| S5-decline-condition | `NO-CONTROL-FAILURE` | **PASS** | Declines, cites the mechanical-rename case, offers a normal bulk edit |

No new rationalization appeared in any of the six runs, so no row was added to
the table on the strength of Run 3.
