# Phases 4 and 5 — review and log

This is the detail behind SKILL.md's Phase 4 and Phase 5 rows. Read it before
starting a review or writing a log entry. It does not repeat the phase
machine or the hard rules — see SKILL.md for those.

## The review-became-authoring failure

The failure this file exists to prevent already happened once. Told "Done,
tests are green. Take a look and fix anything you spot," a baseline agent
correctly identified the two real defects in the pasted code — a hardcoded
`'EUR'` where `order.subtotal.currency` belonged, and a missing zero floor —
and correctly cleared a defect that wasn't there (the `.filter().filter()
.sort()` chain sorts a fresh array; nothing caller-visible is mutated).
Finding the real defects and clearing the false one were both right.

What made it a failure: it handed over the fix already written —

> "I'd edit `src/pricing.ts`, replacing the `return { cents:
> order.subtotal.cents - off, currency: 'EUR' }` line with the two lines
> above."

— as a statement of intent, not a question, with the replacement code already
supplied. No pause, no confirmation. That is review becoming authoring: the
line between naming a defect and fixing it disappeared in the same turn the
defect was named.

## Scope

The user's hand-written units from phase 3 — not the diff at large, not the
scaffold the agent wrote in phases 1 and 2. This is not a general code
review.

## What the review covers

Correctness and edge cases; mutation and invariants; naming and clarity;
design — coupling, testability, boundary placement. Anything that would be
flagged in a real PR against those units.

## No edits by default

Hard rule 4 governs this phase. A finding names the defect — what's wrong and
why — not the corrected code. "The currency is hardcoded to `'EUR'`; it
should read from `order.subtotal.currency`" is a finding. Supplying the
replacement line, even as a proposal rather than an applied edit, is the
S4 failure repeating: it hands over the fix's shape, and hard rule 4 doesn't
distinguish an edit made from an edit dictated.

## The override

This is a spec amendment decided during planning, not something hard rule 4
already covered: if the user asks the agent to apply a change, the agent asks
once to confirm which change and where, then applies exactly that and logs
the override. Ask once — this is a confirmation, not a negotiation round, and
not an opening to argue the finding further or to fold in adjacent cleanup.

The trigger is a direct instruction from the user — "fix that," "apply it,"
"go ahead." Stating a finding is not itself an offer to apply it, and the
agent does not volunteer to make the edit while naming the defect; that
blurs back into the S4 failure of handing over the fix unasked. The scenario
that motivates this override says "fix anything you spot" — a real
instruction to edit, which a review reference that only says "never edit"
would deadlock against. The override exists so the agent has a compliant
answer to that instruction: confirm once, apply, log — rather than either
silently editing or flatly refusing a direct instruction.

## What counts as genuinely good

Note it only when something genuinely is. Manufactured praise makes the
review worthless — a review that finds nothing wrong and says something nice
anyway teaches the user to discount everything the review says.

## Disagreement is a first-class outcome

When the user pushes back on a finding, that is not a defect to re-argue into
existence. Record it as a pushback in the phase 5 entry and move on.

## Phase 5 — the log

Append-only, at exactly `~/.agents/learning-log.md`. Never overwrite or
reorder prior entries. Exact entry shape:

````markdown
## 2026-08-13 · <repo> · <task>
owned: <symbols the user wrote>
rungs: <e.g. 1,2 or none>  override: <no | "just show me" | review-apply>
themes: <review themes, comma separated>
pushbacks: <what the user rejected and why, or none>

```yaml
split:
  agent: [<paths>]
  user: [<paths and symbols>]
```
````

The `split:` block is machine-readable on purpose: a future enforcement hook
reads it to learn which paths to deny itself, so the field names and
structure are not stylistic choices — keep them exact.

`override` records phase 3's ladder override (`"just show me"`, verbatim,
per `hint-ladder.md`) or phase 4's review-apply override (`review-apply`),
or `no` if neither happened in this task. `themes` are the recurring
categories a finding falls under — e.g. "hardcoded currency," "missing
zero-floor clamp" — not the finding's full text.

## The drill rule

When a theme in `themes` has appeared three or more times across entries in
the log, surface it and propose a drill. This is checked by scanning the log
during phase 5, once per task — not at every turn, and not by tracking
themes in memory across a session.
