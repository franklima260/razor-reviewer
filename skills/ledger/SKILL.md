---
name: ledger
description: Set up or maintain ledger-driven project discipline for multi-agent / multi-session development — a TODO ledger with mission + commandments + status board, a BLOCKERS deviation ledger, and evidence documents. Use when starting a substantial project or feature ("set up the project files", "make a plan we can execute across sessions"), when work will span multiple agents or sessions, when the user mentions ticking tasks / status boards / blockers, or when a project's tracking has drifted from reality and needs an honest reset. Also consult it before ticking, unticking, or restructuring any existing ledger file.
---

# Ledger-Driven Development

Long-running work executed by rotating agents (and humans) fails through **context loss**
and **claim drift**: each new session re-derives state from stale notes, and each "done"
that wasn't verified compounds. The ledger system fixes both with three plain-Markdown
files, kept in the repo, treated as the single source of truth, updated only with verified
facts, and committed with the work they describe.

The system's one law, from which everything else follows: **the ledger records what ran,
never what was intended to run.**

## The three files

### 1. `<PROJECT>_TODO.md` — the spec and the state

Structure, top to bottom (see `assets/TODO_template.md` for a ready skeleton):

- **MISSION** — a few sentences stating what the work must achieve AND what must not
  change. The invariants ("everything else keeps working exactly as before") matter more
  than the goal; agents optimize for the goal and sacrifice invariants unless told.
- **COMMANDMENTS** — numbered, hard rules whose violation means the work is wrong even if
  tests pass. Write them from real APIs and real hazards, not generalities: name the
  functions that do exist, the ones agents plausibly invent, the files never to touch, the
  comparisons never to use. Every commandment should trace to a real or confidently
  predicted failure.
- **STATUS BOARD** (added once a phase has >3 tasks) — a table at the top of the active
  phase: one row per task, current state, and a `← START HERE` marker. A new agent must
  orient in thirty seconds without reading history. Include the working-environment recipe
  (exact PATH, build commands, current green counts) — hard-won environment knowledge
  evaporates between sessions unless written here.
- **PHASES and TASKS** — each task carries: an ID (`T2.3`), the spec, explicit
  **Done-when** criteria (observable, executable), and traps ("do not fix pre-existing
  failures; record them"). Order matters; gates between phases are explicit.
- **REVIEW BLOCKS** — adversarial reviews append dated, numbered blocks under the tasks
  they reviewed (see the `adversarial-review` skill). History is append-mostly: correct by
  adding a correction, not by rewriting what was believed at the time.

### 2. `<PROJECT>_BLOCKERS.md` — the deviation ledger

Every deviation from plan, discovered limitation, unresolved question, and accepted
workaround gets an entry with a **STATUS header** (OPEN / RESOLVED / ACCEPTED-DEVIATION,
with dates). Rules:

- When the plan and observed reality disagree and neither the plan nor stock behavior
  answers it, the agent STOPS and files a blocker rather than guessing. A wrong guess
  costs more than a paused session — this rule has saved multi-week remediations.
- Repro fixtures for confirmed bugs are preserved **verbatim** in the blocker entry — they
  become the ready-made acceptance test for the fix.
- Status headers are maintained: a resolved blocker that still says OPEN poisons every
  future agent's risk assessment. Audit them at every review.

### 3. Evidence documents — decisions and measurements

Spike results, benchmark numbers, baseline snapshots (`<PROJECT>_BASELINE.txt`, green-test
counts), and **human rulings recorded verbatim with the numbers they were made against**.
When later evidence changes those numbers materially, the ruling's recorded basis makes the
invalidation visible and the decision goes back to the human. Superseded numbers are marked
superseded and kept — never deleted (strikethrough-in-place beats silent replacement).

## Tick discipline

A checkbox is a factual claim with a burden of proof:

- Tick only when every Done-when item has been **executed and observed** — by you, now,
  from the committed tree. Not "should pass", not "passed earlier".
- Each tick gets a *completion note*: what was done, the exact verification commands, the
  observed counts (and when counts change, reconcile old → new explicitly), plus anything
  the next agent needs (new hazards discovered, decisions made and why).
- Reviewers remove ticks that fail verification, stating exactly which Done-when failed.
  An unticked task with an honest note is progress; a false tick is damage.
- **One task, one commit.** The commit message cites the task ID. Ledger updates commit
  together with (or immediately after) the work they describe — a ledger that's ahead of
  the tree is lying.

## STOP gates

Decisions that belong to the human are marked in the TODO as explicit STOP gates: the task
says "STOP: a human must review X before Y begins", and no agent proceeds past it,
regardless of how obvious the answer seems. Record the human's ruling verbatim, dated, in
both the TODO (at the gate) and the evidence doc. Typical gate material: go/no-go on
architecture, accepting a missed performance target, regenerating golden files, adding any
user-visible surface, spending significant money.

## Bootstrap procedure (new project or honest reset)

1. Write MISSION with the user — push for the invariants, not just the goal.
2. Record the **green baseline** before changing anything: full test suite, counts saved to
   a committed baseline file. If the tree doesn't build or the suite is badly red, STOP —
   that's the first blocker, not something to quietly fix in passing.
3. Draft phases and tasks with Done-when criteria. Front-load de-risking: if a later phase
   depends on an unproven library/approach, make task 0 of that phase a spike with a STOP
   gate for the human ruling.
4. Write the first commandments from the codebase's real hazards; add more as reviews find
   violations (each overturned claim usually yields one).
5. Instantiate testing standards (see the `testing-standards` skill) — the ledger's tick
   discipline is only as strong as the tests behind the ticks.
6. Commit the ledger files; from then on they evolve only through the rules above.

## Maintenance smells (audit these on every review pass)

- Status board disagrees with task checkboxes below it.
- Blocker entries whose STATUS header is stale (says OPEN, task note says fixed).
- "Verified/green/passes" without an executable command next to it.
- Completion notes claiming evidence dated before the commit they describe.
- A ledger edit in a commit containing no corresponding work (or vice versa).
- Rulings referenced but not recorded verbatim ("the user approved this" — where, when,
  against what numbers?).
