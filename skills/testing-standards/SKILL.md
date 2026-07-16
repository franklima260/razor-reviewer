---
name: testing-standards
description: Instantiate, audit against, or enforce binding automated-testing standards in a codebase. Use when the user asks to "set up testing standards", "audit our tests", "are we testing the right things", "make sure the tests are real", mentions test quality / coverage discipline / flaky or vacuous tests, or whenever new tests are being written in a project that has (or should have) a TESTING_STANDARDS.md. In agent-written codebases the tests ARE the spec — treat any substantial feature work as a trigger to check this skill's enforcement section.
---

# Testing Standards — instantiate, audit, enforce

Premise: in AI-assisted development, code is only as good as its tests, because tests are
the only part of the system that gets independently re-executed. An agent's claim "it
works" is unverifiable; a strict, honest test suite converts that claim into something a
machine checks thousands of times. Weak tests are worse than none — they launder false
confidence. This skill maintains a binding standards document and enforces it.

The canonical template lives at `assets/Testing_Standards_Template.md` (~500 lines,
language-agnostic, with agent instruction blocks baked in). It covers: non-negotiable
rules, what "tested" means per function/data-path/UI-component, a forbidden anti-pattern
catalog (vacuous loops, defensive guards, soft assertions, tautologies, mock-the-thing-
under-test, snapshot abuse, nondeterministic timing, loose matchers, internal-state
assertions), cross-layer contract testing, the test pyramid, a PR checklist, agent process
rules, and a living incident log.

This skill has three modes. Pick by situation:

## Mode 1 — BOOTSTRAP (project has no standards doc)

Copy the template to `doc/TESTING_STANDARDS.md` (or the project's docs dir) and complete
its embedded agent instruction block — the 9 steps that turn the template into a binding,
project-specific document. Key judgment calls the template can't make for you:

- **Section 0 is the contract with reality.** Fill the harness table with commands you have
  RUN, not commands that should work. Record the current green counts next to each run
  command — future audits diff against them.
- **Scope to the surface being built.** If the project extends an existing codebase, scope
  the standards to the new/touched surface, not a rehash of the host project's suite —
  otherwise the doc becomes an unenforceable essay.
- **Determinism primitives (0.3) deserve real research**: find the project's actual async
  wait, fake clock, and seeded-random mechanisms. "No sleeps" without a named replacement
  just drives sleeps underground.
- **The incident log (§10) starts honest**: if the project is new, leave it explicitly
  empty and add entries as bugs happen. Never seed it with hypotheticals presented as
  history.
- Delete every scaffold section that doesn't apply, renumber, and remove the instruction
  block when done. A standards doc with leftover `<PLACEHOLDERS>` is unenforceable and
  reads as abandoned.
- **§3 is extensible, and extending it is the point.** The template's anti-pattern catalog
  (§3.1–3.10) is the floor, not the ceiling. When an incident or review catches a new way
  tests lie, add it as a numbered entry (§3.11, §3.12, …), dated and traced to the incident
  that caught it — project-specific learning stays in the project's doc, while the catalog
  mechanism transfers everywhere. Two near-universal candidates worth seeding at bootstrap
  in any project that computes values (money, percentages, coordinates, timestamps,
  quantities): **nice-number fixtures** (round, exactly-representable values mask precision
  and rounding bugs that any hostile value exposes — every numeric invariant needs a
  hostile twin) and **symmetric round-trip masking** (your writer and your reader agreeing
  proves consistency, not correctness — round-trip claims need at least one spec-derived,
  hand-authored fixture). Cite these by the project doc's section number once added; never
  cite a §3 number the doc doesn't contain.

## Mode 2 — AUDIT (standards exist; verify the suite against ground truth)

This is a pedantic, evidence-based sweep — the testing equivalent of an adversarial review.
Trust nothing; bring every claim back to an executed command.

1. **Run the suite(s) yourself, now.** Record counts. Compare against the doc's recorded
   counts and the project baseline. Discrepancies are findings, whoever caused them.
2. **Grep-sweep for the anti-pattern catalog** — the project doc's §3: the template's
   §3.1–3.10 plus any project-added entries (see Mode 1). Practical greps that pay (note
   the first is not in the base template's catalog — file it as a project §3 entry when it
   bites):
   - float `==`/`assertEquals` on floating-point values without tolerance
   - assertions inside loops without a preceding length assertion (vacuous loops)
   - `>=` / `<=` counts where the exact value is knowable (soft assertions)
   - `if (x) { assert... }` in test bodies (defensive guards)
   - sleeps / fixed delays in tests
   - snapshot files with no companion semantic assertions
   - mocks of symbols defined in the module under test
3. **Fixture honesty check**: sample the fixtures. Are they all nice numbers (integers,
   origin-centered, round sizes, ASCII)? Every invariant needs a hostile twin — the
   war-story record shows hostile fixtures catching bugs that nice fixtures structurally
   cannot (see the adversarial-review skill's references). Missing hostile twins on
   precision-sensitive code are findings, not suggestions.
4. **Mutation spot-checks on load-bearing tests**: pick the 3-5 tests protecting the most
   important invariants; break the production code deliberately; confirm each test fails;
   revert. A test never observed to fail is unproven. Record which tests were
   mutation-checked in the audit note.
5. **Wiring check**: for each recently added component, confirm something outside its own
   test file calls it, and at least one test enters through a real entry point.
6. **Contract check** (template §6): find hand-mirrored types/enums/params across layers;
   each is a finding with the generator/contract-test fix named.
7. Write the audit as a dated report (in the ledger if the project uses one): findings
   ranked by severity, each with file:line and the executed evidence, plus explicit
   credit for what is genuinely strong.

## Mode 3 — ENFORCE (ongoing, during every feature/review)

When new code lands (yours or another agent's):

- **Harness before feature**: fixtures for the new behavior get written FIRST and observed
  failing. A test born after the code it tests, passing on first run, is unproven until
  mutation-checked (template §9.3). Budget real time for this — writing thorough tests
  is not overhead on the work; on an agent-built project it IS the work.
- New tests are screened against the §3 anti-pattern catalog before commit.
- Every bug fix ships with the regression test that fails before / passes after — the
  repro from the blocker ledger is that test, verbatim, when one exists.
- Every new invariant test gets its hostile twin at birth (cheaper than retrofitting) —
  and if the project's standards doc has no §3 entry making that binding yet, add one in
  the same commit.
- The PR checklist (template §8) is answered literally, with commands run, not vibes.
- **Never reduce test strictness to make a failing test pass.** Loosening an assertion is
  a deliberate, explained act — silently widening a tolerance or deleting an assertion to
  go green is the single most corrosive move in agent-written code, because it converts
  the safety net into camouflage.
- When a real bug ships despite green tests, add it to the incident log (§10) with the
  anti-pattern that let it through — the log is the document's immune memory, and it is
  what makes the rules stick for future agents.
