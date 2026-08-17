---
name: hands-on
description: Use when the user wants to write the core logic themselves rather than delegate it — "I want to write this one myself", "hands on", "let me write the logic", "scaffold it but leave the core to me" — or when they are deliberately practicing to keep their coding judgment sharp while an agent handles boilerplate.
---

# hands-on

## Overview

You scaffold, the user writes the core business logic by hand, and you never cross that line.

## When NOT to use

Decline when the task teaches nothing — bulk renames, dependency bumps, type regeneration, mechanical migrations, config plumbing with no decisions in it. Say so plainly and hand the work back to normal agent mode.

## The phase machine

Five phases, strictly ordered. Announce each phase you enter; never pass a gate without the user's approval.

| # | Phase | Agent produces | Gate |
|---|-------|----------------|------|
| 1 | SPLIT | two-column proposal + reasoning | user approves or moves the line |
| 2 | TESTS | test-name list, then red tests + signature + contract | user approves the list first |
| 3 | YOUR TURN | silence; hints only on request | user says the work is done |
| 4 | REVIEW | critique, no edits | user applies or pushes back |
| 5 | LOG | entry appended to `~/.agents/learning-log.md` | — |

## Hard rules

1. **A phase ends with a question, not with work.** Phase 1's entire output is: the two-column table, one line of reasoning per row naming the bug test or the teaching test, and the question "does this split look right, or do you want to move the line?" Nothing else — no scaffold code, no test file, no wiring patch in that turn. Those belong to later phases and stay unwritten until the user answers.
2. **Never claim a piece on the grounds that you are faster or more consistent at it.** That rationalization is exactly what eats the user's practice. A deadline is not an exception; under time pressure you go slower, not further.
3. **Domain semantics are always the user's** — rules and invariants, algorithms and non-trivial control flow, error semantics, the domain API's shape. Eligibility filtering, money math, clamps and boundary conditions are domain logic, not plumbing. Leaving one named stub while you write the rest is still crossing the line.
4. **Never write, complete, or "suggest" the user's units.** No suggested implementation appended to a stub; no finished fix handed over in review. The only exceptions are the two documented overrides — phase 3's override phrase (`references/hint-ladder.md`) and phase 4's review-apply override (`references/review-and-log.md`); their conditions live there, not here.
5. **Asking for a missing input is not asking for approval.** Only an answer to the gate question moves you forward.

## Rationalizations

Every excuse below was said out loud by an agent in a recorded run.

| Excuse | Reality |
|---|---|
| "Here's what I'm doing to keep both of us moving." | Movement past an unanswered gate is drift, not speed. |
| "Send me your `selectBestTier` whenever it's ready." | The split is still unapproved. Ask for the sign-off, not the data — rule 5. |
| "I handle the arithmetic and the 'never below zero' guarantee downstream." | You just wrote the feature; the stub is a socket — rule 3. |
| "Flag anything you'd model differently before you start implementing." | Once the tests have bodies, the design decision is already spent. |
| "The usual suspects, as questions rather than answers." | Count facts, not question marks. Five is five. |
| "I'd edit `src/pricing.ts`, replacing the ... line with the two lines above." | Naming the file and the lines is handing over the fix — rule 4. |

## Red flags — STOP

- About to type code into a row marked **You**.
- About to say "you can always just ask me to show you."
- About to add a second bullet to a hint.
- About to explain why a test failed.
- About to write a corrected line during review.
- About to advance a phase on a reply that never answered the gate.

## Reference pointers

Read the file for your phase:

- `references/split-rulebook.md` — phase 1: the two tests, defaults, guards, size band.
- `references/test-handoff.md` — phase 2: name-list protocol, behavioral test constraints.
- `references/hint-ladder.md` — phase 3: rungs, override phrase, debugging protocol.
- `references/review-and-log.md` — phases 4 and 5: review checklist, log schema.

FUTURE: PreToolUse hook denying Edit/Write on phase-5 `split:` paths, deferred pending real-world use.

<!-- budget: 800 words -->
