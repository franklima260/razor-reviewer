# Testing Standards — <PROJECT>

Binding. A change that breaks a rule here is wrong, even if the suite is green.
Keep this file under 60 lines. If it grows, you are writing an essay.

## 0. Harness (fill from commands you have RUN)

| Layer | Run it with | Green at the last audit |
|---|---|---|
| `<unit>` | `<command>` | `<X/X>` on `<date>`, commit `<sha>` |
| `<integration>` | `<command>` | `<X/X>` |

Determinism primitives in this project: async wait `<api>` · fake clock `<api>` ·
seeded random `<api>`. "No sleeps" without a named replacement drives sleeps
underground.

## 1. Rules

1. Every code path has a test that **fails when that path breaks**. "It compiles"
   and "it runs" are not coverage.
2. A test name states the **invariant**, not the function:
   `GetFacets_Disjunctive_ExcludesItsOwnFilter`, not `GetFacets`.
3. If you can delete the assertion and the test still passes, the assertion is
   worthless.
4. A contract shared between two layers needs a contract test or a generated
   artifact. Hand-mirroring is forbidden.
5. Never mock the code under test. Mock external systems only.
6. Deterministic always: no sleeps, no wall clock, no DNS, no unseeded random, no
   calls to external services.
7. No feature without a test. No bug fix without a regression test that fails
   before the fix and passes after.
8. Find a bug, write the failing test first, then fix it.

## 2. Forbidden patterns

A test that does any of these is a finding, not a style note.

| # | Pattern | Why it lies |
|---|---|---|
| 1 | Vacuous loop | asserts nothing when the collection is empty |
| 2 | Defensive guard in a test | skips the assertion instead of failing |
| 3 | Soft assertion (`>= 2` when you mean `== 2`) | passes while the count is wrong |
| 4 | Passes for the wrong reason | green for a cause the test does not name |
| 5 | Mocks the thing under test | tests the mock |
| 6 | Tautological assertion | compares the code to itself |
| 7 | Snapshot instead of an assertion | records behaviour, never judges it |
| 8 | Non-deterministic timing | flakes, then gets retried into silence |
| 9 | Loose matcher | matches the failure too |
| 10 | Asserts internal state | passes while the observable output is wrong |

Add a numbered row when a review finds a new way a test lies. Date it and name the
incident. Never delete a row.

## 3. Before you say it is done

- The command above was run, now, from the committed tree.
- A load-bearing new test was **mutation-checked**: break the code, watch the test
  fail, revert.
- New counts recorded in §0.
