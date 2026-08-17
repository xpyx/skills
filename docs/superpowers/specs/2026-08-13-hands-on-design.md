# hands-on — design

**Date:** 2026-08-13
**Status:** approved, ready for implementation plan

## Purpose

A skill that lets the user keep writing code by hand — for the sake of judgment
that only comes from authoring — while still using an agent for everything that
teaches them nothing.

The agent scaffolds. The user writes the core business logic. The agent never
crosses that line, guides with tradeoff choices rather than answers, reviews
what the user wrote without editing it, and records what happened so that
recurring weaknesses become visible over months.

Context: the user has worked almost entirely by commanding agents for eight
months. The risk being countered is becoming a reviewer who can no longer
evaluate what they review.

## Design decisions

Four decisions were settled during brainstorming and are not open questions:

1. **The split is proposed by the agent and approved by the user.** Not a fixed
   rulebook, not user-declared. The proposal itself is instructional — the user
   sees how the problem decomposes before agreeing to it.
2. **The handoff state is failing tests plus an empty signature.** The agent
   writes the signature, types, and a full red test suite; the user makes it
   green.
3. **Help is an escalating hint ladder**, four rungs, no skipping, with an
   explicit override phrase.
4. **After green: senior-style review with zero edits, plus a persistent log.**

Two follow-on calls, confirmed:

- The log is **central**, at `~/.agents/learning-log.md`, not per-repo — themes
  are only visible across projects and months.
- During phase 3 the agent **may** run tests and show raw failure output. That
  is tool output, not a hint. It may not diagnose the failure.

## Non-goals

- Not a general TDD skill. It borrows red-green but exists for the split.
- Not a code-review skill. Phase 4 review is scoped to the user's own hand-written
  units, not the diff at large.
- No mechanical enforcement in v1. See "Future: enforcement hook".

## Naming and trigger

Skill name: `hands-on`.

Triggers on explicit invocation (`/hands-on`) and on natural phrasing — "I want
to write this one myself", "hands on", "let me write the logic", "scaffold this
but leave the core to me".

### Decline condition (required)

The skill must refuse itself when the task teaches nothing: bulk renames,
dependency bumps, type regeneration, mechanical migrations, config plumbing with
no decisions in it. In that case it says so plainly and hands the work back to
normal agent mode.

Rationale: "use the agent for anything that doesn't teach you" is half the
premise. A learning skill that insists on being used for chores gets abandoned.

## The phase machine

Five phases, strictly ordered. The agent announces the phase it is entering and
may not advance without the user's approval.

| # | Phase | Agent produces | Gate |
|---|-------|----------------|------|
| 1 | SPLIT | two-column proposal + reasoning | user approves or moves the line |
| 2 | TESTS | test-name list, then red tests + signature + contract | user approves the list first |
| 3 | YOUR TURN | silence; hints only on request | user says the work is done |
| 4 | REVIEW | critique, no edits | user applies or pushes back |
| 5 | LOG | appended entry | — |

The failure this ordering prevents is drift: the agent proposes a split, hears
"ok", and carries straight on through writing the user's function, because
momentum is an agent's default state.

## Phase 1 — SPLIT

Output is a two-column table (agent / user) plus the reasoning behind each
assignment.

### The two tests, applied out loud

- **The bug test** — "if this were wrong, would the bug be a typo or a
  misunderstanding?" Misunderstanding → user's.
- **The teaching test** — "would writing this teach the user something they don't
  already know?" No → agent's.

### Default assignments

**Agent:** scaffolding and wiring, imports, config, migrations, type/DTO
definitions, test harnesses and fixtures and builders, CRUD passthrough,
serialization, repetitive transforms, build/CI config, error *plumbing*.

**User:** domain rules and invariants, algorithms and non-trivial control flow,
state machines, concurrency decisions, error *semantics* (what counts as an
error, what recovers), the shape and naming of the domain API, and the *choice*
of data model (the agent writes the resulting migration).

### Guards

- The agent may **never** claim a piece on the grounds that it is faster or more
  consistent at it. That rationalization is precisely what eats the user's
  practice.
- The user's share should land at roughly **30–120 lines**. Materially bigger:
  decompose into several owned units. Materially smaller: the split is wrong —
  the user is doing ceremony while the agent does the thinking.

### Tradeoffs at this phase

Where the decomposition is genuinely arguable, the agent presents 2–3 ways to
carve the problem with their costs and a recommendation, and accepts "none of
these".

## Phase 2 — TESTS

1. Agent proposes the **test-name list only** and stops. The user adds the edge
   cases it missed — this preserves the test-design skill that would otherwise
   be lost by not writing tests.
2. On approval, the agent writes the tests, the signature, and a contract
   docstring (inputs, invariants, what must not happen).
3. Agent runs them and shows they are red.

### Test constraints (required)

Behavioral only: no mocking of internals, no asserting call order, no snapshot
tests for logic, names state behavior rather than implementation.

Rationale: agent-written tests can smuggle in the agent's implementation and
walk the user down one path. These constraints are what keep the tests a
specification instead of a disguised solution.

## Phase 3 — YOUR TURN

The agent goes quiet. It does not volunteer anything.

### The hint ladder

One rung per request, no skipping, and the agent states which rung it is on:

1. A clarifying question back.
2. Name the concept or technique to look up.
3. A worked analogous example, in a **different** domain — never the user's
   actual function.
4. Pseudocode, no syntax.

**Override:** the phrase `"just show me"` produces real code and is recorded in
the log without judgment. The ladder's purpose is to make caving a deliberate
act rather than a slide.

### `options?`

At any point the user may ask for design-level alternatives — data structure,
where state lives, boundary placement — with tradeoffs. Never code. This does
**not** consume a ladder rung; it is a design conversation, not an answer.

### Debugging protocol

The agent **may** run the tests and show raw failure output. It **may not**
diagnose the failure. When a test fails it asks what the user expected versus
what they observed.

Rationale: debugging one's own mistakes is the densest learning in the loop, and
the user identified it as the thing they most want to stop delegating.

## Phase 4 — REVIEW

A senior engineer's PR review of the user's hand-written units. **The agent
never edits the code.**

Covers: correctness and edge cases; mutation and invariants; naming and clarity;
design — coupling, testability, boundary placement; anything that would be
flagged in a real PR.

It notes what is genuinely good **only when something genuinely is**.
Manufactured praise makes the whole review worthless.

The user applies the changes. Disagreeing is a first-class outcome, recorded
as a pushback rather than a defect.

## Phase 5 — LOG

Append to `~/.agents/learning-log.md`. Entry fields: date, repo, task, what the
user owned, rungs used, override yes/no, review themes, pushbacks.

The split proposal from phase 1 is written into the entry as a **structured
block** listing the paths and symbols each side owned.

When a theme recurs three or more times the agent surfaces it and proposes a
drill.

This cross-repo file is the real product of the skill; every upstream phase
feeds it.

## File layout

```
~/code/skills/hands-on/
  SKILL.md                      phase machine, hard rules, red-flags table
  references/split-rulebook.md  the two tests, defaults, guards
  references/hint-ladder.md     rungs, override, debugging protocol
  references/review-and-log.md  review checklist, log schema, drills
```

Symlinked into `~/.claude/skills/hands-on`, matching the existing setup where
`~/.claude/skills/*` are symlinks into source directories.

SKILL.md carries a red-flags table in the superpowers style, mapping the
rationalizations that break this skill to their reality — e.g. "it's faster if I
write this part" → that is the teaching test failing, hand it back.

## Future: enforcement hook

Not in v1. A `PreToolUse` hook that hard-denies `Edit`/`Write` on the paths the
user claimed in phase 1, reading the structured block written in phase 5.

Deferred deliberately: the boundary needs several real sessions before it is
worth locking, and enforcing an unproven line only creates a fight with one's own
tooling. SKILL.md carries a `FUTURE: enforcement hook` note pointing at the
structured block so this is an addition rather than a rewrite.

## Verification

The skill is verified per the `superpowers:writing-skills` discipline, including
a dry run against a real task where the agent must produce a split proposal, be
told to move the line, and respect the move.
