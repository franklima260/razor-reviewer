# Attack briefs

The attack pass is run by a **fresh agent that has never seen the interview**. This is the
whole point: a context that helped write the spec fills its gaps unconsciously from things
it remembers discussing, and grades work it is invested in. An agent handed nothing but the
file experiences exactly what the implementing agent will experience later — every gap it
falls into is a gap that would have cost real rework.

## Dispatch rules

- Pass the **spec file path only**. Never the transcript, never a summary of the
  conversation, never "here's what we already covered" — that context is precisely what
  disables the attack.
- The agent may read the repository freely. Reading code is how it distinguishes a genuine
  gap from something the codebase already answers.
- The agent returns **questions for the user**, never fixes, and never edits the spec.
- Give it the cycle number. Later cycles should be told which findings were already answered
  in the spec, so it does not re-file them — that information lives in the spec itself, not
  in the conversation.

## The standing brief (every cycle)

> You are reviewing a specification you did not write, for work that has not started. You
> have not seen the conversation that produced it and you must not ask for it.
>
> Read `<spec path>`. Read whatever code it refers to.
>
> First, **steelman it**: state in three sentences what this plan is trying to do and why it
> is a reasonable approach. Attack that version — not a weak reading of it.
>
> Then report, as questions the author must answer:
>
> 1. **Missing decisions** — every place an implementer would have to invent something. Be
>    specific: name the line and what is undecided.
> 2. **Contradictions** — two statements in the spec that cannot both be true, or a decision
>    that violates a stated invariant.
> 3. **Unsupported assumptions** — anything presented as settled with nothing behind it,
>    especially claims about how existing code behaves. Check those against the code.
> 4. **Unaddressed failure modes** — how this breaks in production where the spec is silent.
> 5. **Load-bearing unknowns** — places where the plan rests on a figure or a claim nobody
>    has verified, such that being wrong invalidates a decision rather than costing an
>    afternoon. Check the Figures of record table: any row still marked `stated` or
>    `estimated` that a decision depends on is a finding. So is a number the spec relies on
>    that never made it into that table at all.
> 6. **Composed contradictions** — pairs of facts that are each fine alone and cannot both
>    be true together. Run the composition pass below; it is a required step, not an
>    optional lens.
>
> **On unknowns, report the test, never the verdict.** You are not entitled to conclude that
> something cannot be done — you are working from a document, not from measurements, and
> "this exceeds the budget" derived from two conversational estimates is conjecture wearing
> a number. Write "D3 assumes the laptop holds the full library; the only figure behind that
> is a stated estimate — measure actual free space and library size before building on it,"
> never "D3 is impossible." Finding the how is the author's job; your job is to make sure
> they are not finding it on top of something unmeasured.
>
> Also re-read the non-goals and invariants against everything the spec now says. A non-goal
> written in round one is frequently made false by a fact learned in round three, and nobody
> goes back to check it.
>
> Rules: every finding must be something that would cost real rework if left. Do not stack
> minor objections. Do not report style. If a section is genuinely solid, say so — a short
> honest report beats a padded one. If you find nothing, say that plainly; it is a
> meaningful result.

## The composition pass

The contradictions that survive every other check are the ones where **no single sentence is
wrong**. Each fact is true, was written at a different time, in a different section, by
someone thinking about a different problem — and they cannot all hold at once. Nobody
notices, because noticing requires holding two statements together that the document never
puts near each other.

Real examples, both of which survived multiple attack passes before this section existed:

- A spec said advances are funded "from our own cash," and the legal argument for the whole
  product rested on that phrase. Four rounds later the author said the float could be topped
  up from a line of credit. Both statements were recorded. Neither was questioned. Borrowed
  funding quietly dissolved the premise the legal theory stood on.
- A novel's protagonist was established as travelling west across a desert; her stated goal
  was reaching her brother, who was established — in a different file — as being in the
  capital, on the eastern rim. Two passes read a plot whose entire journey ran away from its
  own objective.

Scanning for "contradictions" does not catch these, because there is nothing locally wrong to
see. Use a procedure instead.

**Step 1 — Extract.** Go through the spec and list every declarative fact it asserts: what
exists, where things are, who decides, what something costs, where it comes from, how long it
takes, how much there is. Include facts embedded in the reasoning of decisions, not just the
ones in tables. This list is your working material; you do not need to report it.

**Step 2 — Group by referent.** Bucket the facts by what they are *about* — this resource,
this actor, this place, this quantity, this moment in time, this artifact. Checking every
pair is intractable and produces noise; checking within a bucket is small and precise. The
composition failures above would each have landed two facts in one bucket: *funding source*,
and *where the brother is versus where she goes*.

**Step 3 — Run every bucket against all six tests below.** Not "is this one plausible" —
*can these coexist*, asked six separate times.

**The six are tests, not bucket names.** This is the step that fails in practice. A bucket
gets named after whatever relation you noticed first — "the funding bucket," "the timing
bucket" — and then only that one question gets asked of it, and the bucket feels finished. In
a real run, a reviewer grouped a stated cash float together with a line-of-credit backstop,
correctly, then tested the pair only for **capacity** (is there enough money?), found that
question already tracked elsewhere, and moved on. The **provenance** question about the very
same two facts — borrowed money versus the "our own cash" premise holding up the legal
argument — was never asked, because the bucket had already been labelled as being about
capacity. Two facts, right bucket, wrong question, finding missed.

So: take each bucket and walk all six tests, including the ones that feel irrelevant. Most
combinations produce nothing in two seconds, which is the point — the cost of asking is
trivial and the cost of not asking is the finding nobody makes.

**An item already open elsewhere discharges that question only, never the bucket.** Finding
that the spec already tracks a capacity concern tells you nothing about whether the same
facts also conflict on provenance, position, or authority. Do not let a tracked question
retire untested ones about the same material.

The six tests:

- **Resource** — two things drawing on one budget, where each was sized alone. Disk, cash,
  time, headcount, rate limit, attention.
- **Provenance** — a thing's origin stated one way in one place and sourced another way
  elsewhere. Own cash versus borrowed. Measured versus assumed. Ours versus a vendor's.
- **Position** — where things are against how things move. Who is where, what travels which
  direction, what is reachable from what.
- **Time** — durations, ages, sequences, and deadlines that cannot coexist. Someone who
  witnessed an event three hundred years ago and has a grandchild of twenty-six.
- **Identity** — one entity named two ways, or two entities quietly treated as one. Name
  drift, an id that means different things in different systems, a role conflated with a
  person.
- **Authority** — someone said to decide or approve something that another rule places
  outside their power, or a gate that a second path routes around.

**Step 4 — Report the pair, not a verdict.** State both facts, quote where each came from,
and ask which gives. Then apply the distinction that governs everything here:

- Two **stated facts** that conflict is a genuine finding — the contradiction is demonstrable
  from the document, and you should report it as one.
- A conflict that only appears after doing **arithmetic on estimates** is not a finding, it is
  a load-bearing unknown. Report the measurement that would settle it. You are not entitled
  to conclude the plan fails from two numbers nobody measured.

A composition finding that turns out to be a wording problem rather than a real conflict is
still worth having: it means two sections of the spec described the same thing differently
enough to read as a contradiction, and the next reader will hit the same wall.

## Escalated briefs

Use these on later cycles, when the standing brief has gone quiet but the work is
high-stakes enough to be worth one more angle. Run them as **separate agents** — a single
agent asked to do all three does none of them well.

**Pre-mortem.** "It is six months from now and this shipped and was a mistake. Write the
post-incident summary: what went wrong, in what order, and what in this spec allowed it. Be
concrete about the trigger; 'it didn't scale' is not a finding, 'the nightly job overlaps
itself once the table passes ~2M rows' is."

**The uninvited.** "List what this spec does not mention at all. Not gaps in what it covers
— whole subjects that never come up. Operations, cost, permissions, data already in the
system, the people who have to change how they work, what happens on rollback. For each,
say whether the omission is fine or dangerous."

**Evidence audit.** "Extract every factual claim this spec makes about the existing system —
how something currently behaves, what is currently tested, what a component does. Verify
each against the actual code and report which ones are true, false, or unverifiable. A spec
built on a wrong belief about current behavior fails no matter how good the plan is."

## Reading the results

The attacker is not always right. It lacks context by design, so some findings will be
things the interview settled and the spec recorded badly — which is itself a finding: the
spec is unclear where a reader stumbled.

Do not silently discard findings you disagree with. Put them to the user with your view
attached: "the attacker flagged X; I think the spec already covers it in D3 but that section
reads ambiguously — should we tighten it, or is there something we actually missed?"
