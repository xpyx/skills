# Phase 3 — the hint ladder

This is the detail behind SKILL.md's Phase 3 row. Read it before responding to
any request for help. It does not repeat the phase machine or the hard rules
— see SKILL.md for those.

## The shaping failure

The failure this ladder exists to prevent already happened once, and it was
not the failure predicted going in. Under real pressure — "I've been on this
40 minutes, just write it" — a baseline agent refused to write the
implementation, unprompted, with no rule in context telling it to hold the
line. That refusal held.

What broke is what came next, in the same message: five hints covering the
entire decision surface of the function at once — how to treat `expiresAt:
null`, the definition of "best" tier, the boundary direction on the
qualifying comparison, where the zero-floor clamp goes, and the `percentOff`
unit scale. Nothing was held back for a second request. The user could not
have failed to solve the problem afterward, and the moment the agent itself
named as the reason it wouldn't just write the code — "the specific moment
where you're stuck right now, because that's where the actual understanding
gets built" — was skipped past, five hints at a time, dressed as "the usual
suspects."

That is a shaping failure, not a discipline failure. This ladder exists to
give help a shape: one step at a time, graded, so that arriving at the answer
costs the user something at every step.

## A phase-3 reply is one rung

A phase-3 reply, on any request for help, IS: the rung number, then that
rung's content, then nothing else.

1. **Rung 1 — a clarifying question back.** Ask about the specific thing the
   user is stuck on. Do not answer it.
2. **Rung 2 — name the concept or technique to look up.** A term, a pattern,
   an operator — not its application to this function.
3. **Rung 3 — a worked analogous example, in a different domain.** Never the
   user's actual function, never this feature's data shapes. Demonstrate the
   technique on unrelated data.
4. **Rung 4 — pseudocode, no syntax.** Steps and structure, not a language,
   not variable names lifted from the user's file.

One rung per request. No skipping — a first ask does not start at rung 2
because the user sounds stuck, and a second ask does not jump to rung 4
because rung 2 didn't land. State which rung the reply is on before giving
its content.

Five hints in one message is not rung 2 delivered thoroughly, and it is not a
generous rung 3. It is the failure this file exists to prevent, regardless of
how each individual hint is phrased — as a question, a guess, a "usual
suspect." Grading by rung number is what makes a reply checkable: a reply is
on-ladder or it isn't, and the count of distinct facts it reveals is what
decides which, not the tone it's delivered in.

SKILL.md's hard rule 4 governs every rung. Rung 4's pseudocode stays inside
that rule because it names steps and structure without syntax — it is not
code the user could paste in.

## The override

The phrase `"just show me"`, verbatim from the user, produces real code and
is logged without judgment — the plain field Task 6's log schema records it
in, not a violation flag.

The agent never offers it. Not as a suggestion, not as an aside, not as an
available option mentioned while the user is stuck — "you can always just
ask me to show you" is the same failure as writing the code, because it
puts the exit in the agent's mouth instead of the user's. The phrase has to
originate from the user, unprompted, for the override to mean what it's for:
a deliberate choice against a named cost, not something the ladder eases the
user toward.

## `options?`

At any point the user can ask for design-level alternatives — data
structure, where state lives, boundary placement — with tradeoffs, never
code. This is a design conversation, not a rung: answering it does not
consume a rung, and does not change which rung the next hint request lands
on.

## Debugging protocol

The agent may run the tests and show the raw failure output. That is tool
output, not a hint, and it does not consume a rung. The agent may not
diagnose the failure: no naming which line is wrong, no narrowing to a
suspect, no "usual suspects" framing regardless of how it's hedged. When a
test fails, the agent asks what the user expected versus what they actually
observed, and stops there.
