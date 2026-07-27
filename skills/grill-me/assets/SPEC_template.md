# <Feature or decision name>

**Date:** YYYY-MM-DD
**Status:** GRILLING IN PROGRESS | READY | SUPERSEDED BY <path>
**Cycles completed:** <n interview+attack cycles; quiet passes: n/2>

---

## Problem

What is actually wrong or missing today, in a few sentences. Written so someone who has
never seen this conversation understands why the work is worth doing. Symptoms and their
observed cost, not the solution.

## Goals

What this work must achieve. Each one observable — someone must be able to look at the
finished work and say yes or no.

- G1 —
- G2 —

## Non-goals and invariants

What this work must **not** change. This section matters more than Goals: implementers
optimize for the goal and sacrifice everything unstated. Name the things that keep working
exactly as they do now, the files nobody touches, the behavior users already depend on.

- N1 —
- N2 —

## Figures of record

Every number this spec leans on, with where it came from. **A figure stated in conversation
is a claim, not a measurement** — the same standard this project applies to a ticked
checkbox. Nothing here is a budget until the Verified column says so.

Source is one of: `stated` (the user said it, off the cuff), `estimated` (derived or
inferred, show the arithmetic), `measured` (someone ran something — say what).

Any row that is load-bearing for a decision and not yet `measured` belongs in
**Retire first** below.

| # | Figure | Value | Source | Confidence | How it gets verified | Verified |
|---|--------|-------|--------|------------|----------------------|----------|
| V1 |  |  | stated / estimated / measured | low / med / high |  | no |

Figures are re-checked as the work proceeds. When a measurement contradicts the number a
decision was made against, mark the old value superseded (keep it, strike it) and take the
affected decision back to the person who made it.

## Retire first

The unknowns that have to be settled before the bulk of the work starts — the ones where
being wrong invalidates a decision rather than costing an afternoon. Ordered by what they'd
cost us.

This is not a list of reasons the project might be impossible. It is the shortest path to
knowing which shape the work actually takes.

| # | Unknown | Which decision rests on it | How we find out (the cheapest test) | Result |
|---|---------|----------------------------|--------------------------------------|--------|
| R1 |  |  |  | pending |

## Decisions

One entry per decision the grilling settled. A decision with no reasoning is a decision the
next reader cannot safely revisit.

### D1 — <the decision, stated as a claim>

- **Chosen:**
- **Why:**
- **Rejected:** <alternative> — <why it lost>
- **Reopened:** <if this decision was reopened mid-session, what forced it, and what changed>

### D2 — …

## Assumptions and blast radius

Things assumed rather than established, each with what breaks if it turns out false. This is
the most important section for whoever implements: every row here is something they must
**not** quietly resolve on their own.

| # | Assumption | If wrong, this is what has to change | Cost if wrong |
|---|------------|--------------------------------------|---------------|
| A1 |  |  | low / medium / high |

## Failure modes

What goes wrong once this is live, and what the design does about it. Populated largely by
the attack passes. Anything here with no mitigation is a deliberate acceptance — say so.

| # | Failure | Trigger | Mitigation (or ACCEPTED) |
|---|---------|---------|--------------------------|
| F1 |  |  |  |

## Blast radius

What else this touches — callers, downstream consumers, data already written, other agents
or people working nearby, anything that has to be migrated or backfilled.

## Done-when

Executable, observable criteria. Not "works correctly" — the exact commands or observations
that settle it. These become the Done-when lines if this spec is later turned into a ledger.

- [ ]
- [ ]

## Open questions

Still unresolved at the time of writing. Empty is the goal; if the session ended early, this
section is what makes that honest.

| # | Question | Who or what can answer it | Blocks |
|---|----------|---------------------------|--------|
| Q1 |  |  |  |

## Attack log

One line per attack pass — the record that the stop rule was actually satisfied rather than
declared. Updated in the same edit as the findings it describes, never a round later.

**Driver** is what produced the findings: `spec gap` (the plan was incomplete) or
`scope change` (the work being planned changed). A rising finding count with `spec gap` as
the driver means the design is genuinely unsettled; with `scope change`, it means the target
moved and the count says nothing about how close the spec is.

| Cycle | Date | New findings | Driver | Decisions reopened |
|-------|------|--------------|--------|--------------------|
| 1 |  |  | spec gap / scope change |  |

## Scope events

Every answer that changed *what* is being built rather than settling *how*. Empty is normal
and good; entries here explain why the cycle count is what it is.

| Round | What changed | What it replaced | Cycle counter reset? |
|-------|--------------|------------------|----------------------|
|  |  |  |  |
