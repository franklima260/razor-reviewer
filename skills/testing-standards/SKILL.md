---
name: testing-standards
description: Instantiate, audit against, or enforce binding automated-testing standards in a codebase. Use when the user asks to "set up testing standards", "audit our tests", "are we testing the right things", "make sure the tests are real", mentions test quality, coverage discipline, flaky or vacuous tests, or whenever new tests are written in a project that has (or should have) a TESTING_STANDARDS.md. In agent-written codebases the tests are the spec, so treat substantial feature work as a trigger to check the enforcement section.
---

# Testing Standards

In an agent-built codebase the tests are the spec. A test that cannot fail is a
lie that every later agent believes.

The template is `assets/Testing_Standards_Template.md`, 59 lines. Keep the
project's copy under 60 lines. A longer document is an essay, and nobody enforces
an essay.

## Mode 1 — BOOTSTRAP (no standards doc)

Copy the template to the project's docs dir and fill §0 from commands you have
**run**, with the counts you observed. Name the project's real async-wait, fake
clock and seeded-random APIs. Delete any row that does not apply. A doc left with
`<PLACEHOLDER>` text is unenforceable and reads as abandoned.

## Mode 2 — AUDIT (the doc exists; check the code against it)

Trust nothing. Bring every claim back to an executed command.

1. **Run the suites yourself, now.** Compare the counts with §0. A difference is a
   finding, whoever caused it.
2. **Grep for the forbidden patterns.** These pay for themselves:
   - float `==` or `assertEquals` on floats with no tolerance
   - assertions inside a loop with no length assertion before it
   - `>=` / `<=` where the exact count is knowable
   - `if (x) { assert ... }` inside a test body
   - sleeps or fixed delays
   - snapshot files with no companion assertion
   - mocks of symbols defined in the module under test
3. **Fixture honesty.** Are the fixtures all nice numbers, origin-centred, round
   sizes, ASCII? Every invariant needs a hostile twin. Hostile values catch bugs
   that nice values structurally cannot.
4. **Mutation spot-check.** Take the 3 to 5 tests that protect the most important
   invariants. Break the production code. Confirm each test fails. Revert. A test
   never observed to fail is unproven. Name the tests you checked.
5. **Wiring check.** For each new component, confirm something outside its own test
   file calls it, and that one test enters through a real entry point.
6. Put the findings in the ledger rows, with file:line and the executed evidence.

## Mode 3 — ENFORCE (during every feature and review)

- **Harness before feature.** Write the fixture first and watch it fail. A test born
  after the code, green on its first run, is unproven until it is mutation-checked.
- **Never reduce strictness to go green.** Widening a tolerance or deleting an
  assertion converts the safety net into camouflage. If it must be loosened, say so
  in the commit message and say why.
- Every bug fix ships the regression test that fails before and passes after. When a
  blocker row names a repro test, that is the test.
- A real bug that ships past green tests earns a new numbered row in the forbidden
  table, dated, naming the incident.
