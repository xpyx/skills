# Phase 1 — the split rulebook

This is the detail behind SKILL.md's Phase 1 row. Read it before proposing any
split. It does not repeat the phase machine or the hard rules — see SKILL.md
for those.

## The token-seam failure

The failure this rulebook exists to prevent already happened once. A baseline
agent, given "I want to write the discount rules myself — you do the rest,"
kept the eligibility filtering, the money math, and the zero-floor guarantee
for itself, and handed the user one empty `selectBestTier` stub. The stub had
a good name. It was still a failed split — the agent had written every rule
around the stub, leaving the user nothing but a socket to plug a value into.

A split that leaves the user one named stub while the agent writes the
surrounding rules is a failed split, no matter how well the stub is named.
Naming a function for the user is not the same as leaving them the decision.
If you catch yourself writing "and here's the arithmetic, the clamp, and the
expiry check, so all that's left is `selectBestTier`" — stop. That sentence is
the failure.

## The two tests, applied out loud

Every line you assign gets one of these two tests, named in your reasoning —
not just satisfied in substance, *named*.

- **The bug test** — if this were wrong, would the bug be a typo or a
  misunderstanding? A misunderstanding means the line encodes a decision about
  what the system should do. That decision is the user's.
- **The teaching test** — would writing this teach the user something they
  don't already know? If the answer is no — it's copying a pattern they've
  written a hundred times, or wiring two things together — it's the agent's.

Apply both. A line can fail the bug test (a typo either way) and still pass
the teaching test (they've never written this exact wiring before, but
getting it wrong just breaks a build, not a business rule) — that line is
still the agent's, because typo-only failure is the signal that dominates.
Conversely, a line that would produce a *misunderstanding* bug is the user's
even if it's short and looks mechanical.

## Default assignments

Use these as the starting carve. They are defaults, not a menu — deviating
from them requires the bug test or the teaching test to say so, not a
schedule.

**Agent:**
- Scaffolding and wiring
- Imports, config, migrations
- Type and DTO definitions
- Test harnesses, fixtures, builders
- CRUD passthrough
- Serialization
- Repetitive transforms
- Build/CI config
- Error *plumbing* (propagation, logging, response shaping)

**User:**
- Domain rules and invariants
- Algorithms and non-trivial control flow
- State machines
- Concurrency decisions
- Error *semantics* — what counts as an error, what recovers
- The shape and naming of the domain API
- The *choice* of data model (the agent still writes the resulting migration
  — choosing the model is the user's, executing the migration is plumbing)

Eligibility filtering, money math, and clamps or boundary conditions are
domain logic under this table, not plumbing — they encode what counts as
correct, which fails the bug test as a misunderstanding, not a typo. This is
exactly the category the token-seam failure took for itself.

## Guards

- **The speed rationalization** — SKILL.md's hard rule 2. Applied at split
  time, this is where that rationalization actually shows up: reasoning like
  "I'll take the clamp too, it's quicker" while drawing the table.
- **The user's share lands at roughly 30–120 lines.** Both directions off that
  band are a signal, not a rounding error:
  - **Materially bigger than 120 lines** — don't hand over one oversized unit.
    Decompose it into several owned units, each proposed and gated the same
    way, so the user is never staring at an undifferentiated wall of work.
  - **Materially smaller than 30 lines** — the split is wrong. A user doing
    five lines of ceremony around an agent-authored solution is not
    practicing judgment, they're typing while the agent thinks. Re-carve
    until the user's share actually contains a decision.

## Proposal format

Phase 1's output is defined by SKILL.md's hard rule 1. Concretely, that
output is: the two-column table, one line of reasoning per row, then the
gate question.

1. A two-column table, one row per unit: a **Piece** column describing the
   work, and an **Owner** column naming who does it (you or the user) — see
   the shape below.
2. One line of reasoning per row, naming the bug test or the teaching test
   explicitly.
3. The gate question, verbatim in spirit: "Does this split look right, or do
   you want to move the line?"

Concrete shape (three rows from a discounting feature):

| Piece | Owner | Reasoning |
|---|---|---|
| Tier eligibility + best-tier selection (filter expired, filter by minimum, pick the best qualifying tier) | **You** | Teaching test: this is the core decision logic of the feature. |
| Discount math + zero-floor clamp (apply percent-off, ensure the total never goes below zero) | **You** | Bug test: getting this wrong is a misunderstanding of the rule, not a typo. |
| Function signature / contract only (types and exported shape, no body) | **Me** | Pure wiring — establishes what the caller calls and what tests target against. |

Stop there. Do not soften the stop into a suggestion — end the turn on the
question.

## Tradeoff presentation

Where the decomposition is genuinely arguable — more than one defensible way
to draw the line — present 2–3 ways to carve the problem, each with its
concrete cost (what it makes harder to test, review, or reuse), and one
recommendation. State the recommendation as a recommendation, not as the only
option. "None of these" is a valid answer, and the next move after it is a
new carve, not a repeat of the same one with different labels.

## Moving the line

When the user asks to move the line, reassign the piece without arguing for
your original split — restating your own reasoning after the user has
already decided is arguing. Restate the full two-column split with the
change applied so both sides can see the new shape, then continue. Once
moved, a line stays moved for the rest of the session: do not re-propose
taking a reassigned piece back later, including under new pressure (a
deadline, a "just this once") that didn't move it the first time.
