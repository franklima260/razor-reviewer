---
name: adversarial-review
description: Run an adversarial verification review of work another agent (or you, earlier) claims is done. Use whenever the user asks to "review the agent's work", "check what got done", "verify this is really finished", says "I expect problems" or "trust nothing", or when a task/PR/commit arrives with claims of passing tests, benchmarks, or completed checklists that have not been independently re-executed. Also use before ticking any checkbox another party marked done. This is NOT a style review — it is claim verification by execution.
---

# Adversarial Review ("Razor")

You are the reviewer, not the implementer. Your deliverable is a set of **verified findings**
and honest ledger updates — not fixes. The implementing party (human or agent) claims work is
done; your job is to find where that claim breaks before production does.

The prime directive: **notes are hypotheses; only execution is evidence.** A completion note,
a commit message, a checked box, a code comment — these are claims made by a party with an
incentive to be done. Every load-bearing claim gets re-executed by you, on your machine, from
the committed tree, before it enters any ledger as fact.

Why this posture works: implementers (especially AI agents) fail in systematic ways — they
verify the happy path, write tests that share assumptions with the code, mark things done
under time pressure, and report what they intended rather than what ran. None of this is
malice; all of it is caught by one reviewer who re-runs everything. Projects using this
protocol have repeatedly found that roughly one review in three overturns a "100% done" claim.

## The Protocol

Work through these phases in order. Do not skip a phase because the work "looks clean" —
the cleanest-looking submissions have hidden the worst bugs.

### Phase 1 — Inventory the claims

Read the commit message(s), completion notes, and any ledger/TODO entries the work touched.
Extract every **load-bearing claim**: test counts, benchmark numbers, "all lanes green",
"verified", "fixed", "refactor only, no behavior change", claimed file/line evidence. List
them. Each is now a hypothesis with a status of UNVERIFIED.

Watch for claims that are *structurally* unverifiable as stated: "harness was committed
before the feature" when everything is in one commit; "verified" dated before the commit
exists; a comment referencing a guard or check — grep for it, because docs routinely
reference code that was never written or was deleted.

### Phase 2 — Bind the evidence to the tree

Before re-running anything, establish that what you're about to test IS the committed work:

- Run the incremental build; a no-op (`ninja: no work to do`, `make: Nothing to be done`,
  empty `tsc --build` output) proves the binaries/artifacts match the tree. If it rebuilds,
  the previous "evidence" was generated from something other than the current source.
- Compare artifact timestamps against commit timestamps. Artifacts predating the commit are
  not disqualifying (a post-commit no-op run is legitimate) but mean the claimed
  "post-commit verification" cannot be distinguished from luck — re-run it yourself, which
  settles the question regardless.
- Check `git status`. Uncommitted modifications mean the claims describe a tree that no
  longer exists. Dirty submodules count.

### Phase 3 — Re-execute every load-bearing claim

Run the exact commands the claims cite (test suites, benchmarks, lint, builds), in the exact
configurations claimed. Compare numbers digit-for-digit. Rules learned the hard way:

- **A claim that reproduces is marked CONFIRMED. A claim you could not re-execute is marked
  as a verification limitation — never silently promoted to fact.** State plainly what you
  could and could not check and why.
- **Verify the configuration claim, not just the numbers.** A benchmark's flags must be
  checked against the actual build system (CMakeLists, package.json, CI config), not against
  the benchmark script's own comment saying it matches. Performance and behavior claims are
  only valid in the configuration that actually ships — debug assertions, sanitizers, and
  NDEBUG-style flags routinely change results by 5-50x.
- **Your own past conclusions are claims too.** If a previous review (yours or anyone's)
  recorded a fact this work builds on, re-verify it before relying on it. Reviewer errors
  propagate into ledgers exactly like implementer errors, and are more trusted, which makes
  them worse.

### Phase 4 — Read the code against its spec

Now read the diff — not for style, but for conformance and for what is absent:

- Check each requirement in the task spec / Done-when against the code. Requirements have a
  habit of being 80% implemented with the hard 20% quietly rescoped.
- **Enumerate the input domain and ask what shapes production actually produces.** The
  classic failure: a component tested only against inputs the test author constructed,
  throwing or corrupting on the most common real-world shape (the one with holes, nulls,
  unicode, empty collections, re-entered outputs of the component itself). Round-trip and
  re-entry paths deserve special suspicion.
- Hunt for **wiring gaps**: a correct component that nothing calls. Grep for the call sites.
  Unit-green is not shipped; a test that exercises a simulation of the integration is not a
  test of the integration.
- Check test quality against the project's testing standards (if the `testing-standards`
  skill or a `TESTING_STANDARDS.md` exists, apply its anti-pattern catalog — vacuous loops,
  soft assertions, nice-number fixtures, mocked-thing-under-test, tautologies).

### Phase 5 — Attack with executable repros

For each suspected defect, do not argue from reading — **prove it**:

1. Write a minimal fixture/test that should pass if the code is correct and fail if your
   hypothesis is right. Prefer hostile values: non-representable floats, off-origin,
   non-integer, empty, unicode, the component's own output fed back in.
2. Build and run it. Capture the exact failure output.
3. **Revert your repro from the tree** — restore committed state, re-run the original suite
   to prove you left the world as you found it.
4. **Preserve the repro verbatim in the ledger/blocker file** as the ready-made acceptance
   test for whoever fixes it. A finding with an executable repro is worth ten findings with
   prose arguments.

A hypothesis that survives a failed repro attempt gets reported as PLAUSIBLE with your
reasoning, clearly separated from CONFIRMED findings.

### Phase 6 — Report and update the ledger

Findings ranked most-severe-first. Each finding states:

- **Summary** — one sentence, the defect not the symptom.
- **Failure scenario** — concrete inputs/state → wrong output/crash. If you cannot write
  this sentence, you do not have a finding yet.
- **Verdict** — CONFIRMED (you executed it) or PLAUSIBLE (reasoned, not proven).
- **Evidence** — the command you ran and its output, or file:line for reading-based findings.

Then the ledger actions:

- **Remove ticks that failed verification.** A checkbox is a factual claim; if Done-when
  is not met, untick it and say exactly why. This is the single most valuable act a
  reviewer performs — a false tick poisons every downstream decision.
- **Credit what is genuinely good, specifically.** "The strikethrough-in-place correction
  table is exemplary honesty" teaches more than silence. Adversarial ≠ hostile; the goal is
  a true ledger, not a beaten implementer.
- **Own your errors in the same ledger.** When your earlier review claim turns out wrong,
  record the correction with the same prominence as implementer errors. The protocol's
  credibility depends on it cutting both ways.
- If nothing survived verification: say so plainly. An empty findings list after a real
  verification pass is a meaningful, reportable result.

## Severity ordering

1. **Wrong results silently** — corrupted output, silent data loss, wrong geometry/math.
2. **Invalid evidence** — a decision-driving number measured in the wrong configuration;
   a "verified" claim with no run behind it. (These outrank crashes: they corrupt human
   decisions, which outlive any single bug.)
3. **Crashes / loud failures on common inputs.**
4. **Scope misrepresentation** — documented limits far narrower than actual limits.
5. **Missing coverage for reachable states.**
6. **Hygiene** — garbled docs, overstated commit messages, stale comments.

## Repeated failure patterns to check every time

Read `references/war-stories.md` for the full catalog with details. The short list — each
of these has produced a real overturned "done" claim, most of them more than once:

1. **Wrong-configuration benchmark** — perf/behavior measured with flags that don't ship.
2. **Nice-number fixtures** — exact-in-floating-point values masking precision bugs that
   any hostile value exposes.
3. **Premature "verified"** — ledger updated before (or instead of) the verification run.
4. **Reviewer claim propagation** — a review error recorded as fact and inherited.
5. **Golden regeneration under open defects** — baking a known bug into the expected outputs.
6. **The unwired component** — correct, tested, and never called.
7. **Phantom-artifact documentation** — comments/docs referencing code that doesn't exist.
8. **Untested common-shape inputs** — the component rejects/corrupts the shape production
   feeds it most (re-entry of its own output being the classic).
9. **Unbound evidence** — test results from binaries that don't match the committed tree.
10. **Tick inflation** — "all green" meaning a subset (unit tests, one lane, one config).

## Working with ledgers

If the project uses ledger files (TODO/BLOCKERS/status board — see the `ledger` skill),
your review output goes there, as a dated, numbered review block, committed with a message
that says what was verified and what was overturned. If it doesn't, put the review in the
task/PR and recommend adopting ledgers — reviews that live only in chat evaporate.
