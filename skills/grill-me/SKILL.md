---
name: grill-me
description: Interrogate the user about a plan, feature, or design until the spec is sound — alternating interview rounds with independent attack passes, and writing a dated spec file as it goes. Invoke when the user says "grill me", "interview me", "stress-test this plan", "make the spec less dumb", "poke holes in this before I build it", or asks to be questioned before implementation. Do NOT start this on your own; when a user opens substantial or vaguely-specified work with no spec, mention once that grilling would pay here, then drop it if they move on.
---

# Grill Me

Step 1 of every engineering project is making the spec less dumb. The expensive failure in
agent-driven development is not a bug — it is a half-built feature thrown away because the
spec was wrong, and nobody noticed until the code existed. This skill spends that time
deliberately, up front, where it is cheap.

The law this skill runs on: **facts are yours to find, decisions are the user's to make.**
Anything discoverable from the filesystem, the git history, the dependencies, or a tool is
never a question — look it up. Every question you ask must be one only the user can answer.
Asking a user something you could have grepped is the fastest way to make a grilling session
feel like an interrogation for its own sake.

The second law: **stop when it goes quiet, not when it gets tiring.** The session ends when
an independent attack on the spec produces nothing new — twice.

## The cycle

```
orient (facts) → interview round → update spec → independent attack → interview round → …
                                       ↑                                    │
                                       └────────── new questions ───────────┘
                                   until two consecutive attacks come back empty
```

The spec file is written early and updated every round. It is not a closing deliverable —
it is the working state, the thing the attacker reads, and the resume point if the session
is interrupted.

### Phase 0 — Orient before asking anything

Explore first. Read the repo, the recent commits, the ledger (`LEDGER.md`), any
related spec in `docs/specs/`, the actual code the change touches. For
a broad sweep, dispatch a read-only sub-agent rather than burning the main context on it.

Then state, in a few lines, what you established as fact. This is not a summary for
politeness — it is a correction opportunity. Wrong facts silently steer every question that
follows, and the user can only correct what they can see.

Ask what the user will regret not deciding. Order the questions by blast radius: what
is expensive to change later comes first, cosmetics never. Write your own questions from
the topic in front of you; a stock list asks about a project you are not building.

### Phase 1 — Interview rounds

Ask only the questions that are **ready** — the ones whose answers do not depend on an
answer you have not heard yet. A question that hinges on another question still open in this
round belongs to a later round.

- **Up to four per round**, and only when they are genuinely independent of each other. A
  decision that is architectural or hard to reverse gets asked **alone**, so it gets the
  user's whole attention.
- **Every question carries your recommended answer** and one line of why. A question with no
  recommendation stalls the session and pushes work back onto the user. This holds even when
  the answer is a fact you cannot reach — a vendor's capability, another team's system, a
  number only the user can look up. There the recommendation is the *route*: the cheapest way
  to find out and who can do it. "No recommendation, only you can answer this" is never an
  acceptable question.
- **Order by regret** — cost if we get this wrong, times the chance we get it wrong. The
  highest-regret question goes first in every round. If the user bails after three rounds,
  they will have answered the things that actually matter.
- **Never ask what you can look up.** If a question needs a fact you do not have, go get the
  fact — and do not block the whole round on it. Ask the rest of the round now; the
  questions downstream of that fact wait.

**Numbers the user says out loud are claims, not measurements.** "Around 900GB", "it's
already pretty full", "maybe 30GB a month" — these arrive in the middle of a conversation,
from memory, and they are frequently wrong by a factor that matters. Record every one in the
spec's **Figures of record** table with its source (`stated` / `estimated` / `measured`) and
your confidence. Never promote a stated figure to a budget. This project holds a ticked
checkbox to the standard of "executed and observed"; a number that a decision rests on does
not get a lower bar than a checkbox.

When a decision leans on a figure that is not yet measured, that figure goes in **Retire
first** — the short list of unknowns to settle before the bulk of the work starts, each with
the cheapest test that would settle it.

**The reopening rule.** Answers travel backwards as well as forwards. When something the
user just said contradicts or undermines a decision already settled, **reopen that decision
explicitly** — name the earlier decision, quote the answer that destabilized it, and ask it
again. Do not silently keep both.

Then **trace upward**. The decision a new fact visibly dents is rarely the only one it
touches; a fact arrives attached to whatever question happened to be on the table, not to
the decision it actually undermines. Walk the chain: what does this decision rest on, and
does that premise still hold? Keep walking until you reach a decision the new fact does not
disturb. Then **re-read the non-goals and invariants** — a non-goal written in round one is
routinely made false by a fact learned in round three, and nothing else in the process goes
back to check.

A spec containing two decisions that cannot both be true is worse than a spec with a hole in
it, because the hole is visible. Reopening will feel like going backwards; going backwards
on purpose is the feature.

**Scope events, and saying when you are going in circles.** Some answers settle a question.
Others change what is being built. These look identical in a chat window and are completely
different events, and the second kind is why a grilling session can run forever.

When an answer expands or redirects the work rather than resolving a question about it, say
so plainly in that round: *"That changes what we're building, not how — the instant-card
rail is a different integration from the one we were scoping."* Record it as a **scope
event** in the attack log with what changed. The quiet counter resets, and the reason is
written down as scope, not as an unresolved spec.

Then watch the trend across cycles. If two consecutive attack passes fail to produce fewer
findings than the one before **and** a scope event occurred in that stretch, stop and name
it:

> "Three cycles, findings going 6 → 7 → 8. That isn't the spec failing to converge — the
> requirements moved twice: the payment rail in round 3, the eligibility scope in round 4.
> This session will not go quiet while the target keeps moving. Do you want to freeze scope
> at what's written and finish, or keep expanding knowing we're not near the end?"

Put the choice to the user and wait. Do not decide it, and do not quietly keep cycling.
Rising finding counts are ambiguous on their own — they read as a spec that will not
converge when the real story is a requirement that will not hold still. Making that
distinction visible is the difference between a session the user can steer and one they
abandon.

A grilling that ends because the user knowingly froze scope is a success. One that runs
eleven cycles because nobody named the churn is a failure, however good the findings were.

**When the user cannot answer.** Some questions need a benchmark, another person, or a
decision not yet made. Do not halt the session and do not guess. Record it in the spec as an
**assumption with its blast radius**: what you are assuming, and specifically what has to be
thrown away or redone if the assumption is wrong. "Assuming the import runs nightly, not
continuously — if it's continuous, the dedupe strategy and the whole retry design change."
A named assumption with a price tag is a risk the user can price; an unnamed one is a
landmine.

### Phase 2 — Write the spec (early, then every round)

After the **first** interview round, write `docs/specs/YYYY-MM-DD--<slug>.md` from
`assets/SPEC_template.md`. Update it at the end of every round thereafter.

Early, because the attacker in Phase 3 reads the spec and nothing else — there must be
something to attack. Continuously, because an interrupted session resumes from this file,
and because a decision that has not been written down has not really been made.

Keep it honest as it grows: decisions with the reasoning that produced them, rejected
alternatives with why they lost, assumptions with blast radius, figures with their
provenance, and open questions still outstanding. A spec that records only the winning
answers teaches the next reader nothing about which constraints are load-bearing.

Two sections carry the epistemics and are easy to let rot. **Figures of record** must gain a
row every time a number enters the conversation — including numbers you derived yourself,
with the arithmetic shown. **Retire first** must gain a row every time a decision turns out
to rest on a figure nobody has measured. If either table is empty by round three, you are
not writing down what the session is actually standing on.

**Check each new fact against what the spec already says about the same thing.** Before
writing a fact down, ask what else in the document is about that subject — the same
resource, the same actor, the same place, the same budget, the same moment — and whether
they can all be true together. This is a one-question habit at write time and it is the
cheapest possible catch: the alternative is an attacker finding it three cycles later, or
nobody finding it. Facts that arrive late in a session are the dangerous ones, because
everything they contradict was settled while the room believed something else.

The failure this prevents has a shape worth recognizing: no sentence is wrong, the
contradiction lives *between* two sentences written at different times, and neither author
had the other in view. "Funded from our own cash," recorded in round one, and "we can draw
on the line of credit," recorded in round four, are each unremarkable — together they
dissolve the argument the first one was holding up. Check each new answer against the
ones already written down, one fact at a time, as you write.

### Phase 3 — Independent attack

Dispatch a **fresh agent that reads only the spec file** — never this conversation, never
the transcript, never your notes. Brief it in three lines: find the assumption that, if
false, wastes the most work; find two statements that cannot both hold; name what the spec
does not say and must. It reports findings only, and proposes nothing.

Why the isolation matters: a context that helped write the plan cannot attack it honestly.
It fills gaps unconsciously from things it remembers being discussed but never actually
written down, and it grades work it is invested in. An agent that only ever saw the spec
experiences exactly what the implementing agent will experience in three weeks — and every
gap it stumbles into is a gap that would have cost real rework.

The attacker's output is **questions, not fixes**. It returns:

1. **Missing decisions** — choices the implementer would have to invent.
2. **Contradictions** — two statements in the spec that cannot both hold.
3. **Unsupported assumptions** — claims presented as settled with nothing behind them.
4. **Unaddressed failure modes** — what happens when this breaks, and the spec is silent.

It must steelman before it strikes: state the plan in its strongest form first, then attack
that. An attacker that beats up a weak reading of the spec produces findings you will
correctly ignore, which trains everyone to ignore the next pass too.

### Phase 4 — Feed the findings back

The attacker's questions enter the next interview round, regret-ordered alongside anything
still open. Some of them will reopen settled decisions. That is the mechanism working, not
the session failing.

Report each cycle plainly: cycle number, how many new questions the attack produced, and
what it reopened. The trend is the useful signal — an attack that produces twelve questions
on cycle one and two on cycle four is converging.

### Stop rule

Loop until **an attack pass produces zero new questions, then run one more pass to
confirm.** Two consecutive quiet passes and the grilling is over.

A scope event resets the counter — but resets caused by scope are reported as such, never
absorbed silently. Keep the header and the attack log current **in the same edit** as the
findings they describe: the stop rule is counted from that bookkeeping, and a log written a
round late makes the count unverifiable.

Never stop because it has taken a while. If the user calls it early, that is their call —
but the spec must then state, in its own section, exactly what remains unresolved and what
the risk of proceeding is. A spec that quietly omits the questions nobody got around to is
the precise failure this skill exists to prevent.

## Never

- **Never write implementation code during a grilling.** Not a scaffold, not a "quick
  sketch to show what I mean". Throwaway prototypes are legitimate only when the user cannot
  answer a question without seeing something — say so, keep it disposable, and delete it.
- **Never accept "whatever you think" on a high-regret decision.** Restate the stakes in one
  sentence and ask once more. If the user still declines, record your choice as an
  assumption with blast radius and move on — never bury it as a settled decision.
- **Never bundle dependent questions into one round.** The second answer will be conditioned
  on a guess about the first.
- **Never let the spec lag the conversation.** A decision made in round three and written in
  round six is a decision that spent three rounds able to drift.
- **Never let the attacker strawman the plan** — and never stack minor objections to
  manufacture the impression of a weak spec. Findings must be things that would genuinely
  cost rework.
- **Never declare the work impossible.** Two conversational estimates and some arithmetic do
  not entitle you — or the attacker — to rule something out. When the plan appears to exceed
  a stated limit, you have found an **unknown to test first**, not a verdict: name the figure,
  name the decision resting on it, and propose the cheapest measurement that settles it.
  Finding the *how* is the job. "This doesn't fit" is a conclusion available only after
  someone measures, and it is the user's conclusion to draw, not yours.
- **Never treat the user's domain knowledge as an obstacle.** When they overrule a
  recommendation with a reason, record the reason in the spec — it is usually a constraint
  the questions failed to surface.

## After the spec

When the loop goes quiet, present the spec and **offer once** to turn it into a
`LEDGER.md` via the `ledger` skill — mission and invariants from the goals and non-goals,
rules from the failure modes the attacks surfaced, task rows from the decisions, and STOP
gates where the user's ruling is required. Offer it; do not insist, and do not do it
unasked. The spec is the deliverable the user chose.

Once the ledger exists, the spec is history. Do not keep updating both.

If the work proceeds, the assumptions section is the handoff that matters most: every one of
them is a thing the implementing agent must not silently resolve on its own.
