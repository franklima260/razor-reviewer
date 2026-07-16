# Testing Standards

> **Template Version:** 1.0
> **Last Updated:** <!-- FILL IN DATE -->
> **Project:** <!-- FILL IN PROJECT NAME -->

**Audience:** every engineer and AI agent working on this codebase.

This document is binding, not aspirational. Code that ships without satisfying these standards is not done.

The standards exist because we have been bitten by every category of failure they prevent. Read this end-to-end before writing or reviewing tests in this repo.

---

<!-- ============================================================
AGENT INSTRUCTION — COMPLETE SECTION 0 BEFORE USING THIS DOCUMENT
==============================================================

You are an AI coding agent onboarding to a new project. This document
contains language-agnostic testing standards. Section 0 is intentionally
sparse — you must fill it in based on this project's actual stack before
this document can be used.

Work through each step below in order. When all steps are done, delete
this instruction block.

STEP 1 — IDENTIFY LANGUAGES AND FRAMEWORKS
   Inspect the repository (package manifests, config files, source
   directories) to determine:
   - What language(s) the project uses per layer (backend, frontend,
     workers, CLI, etc.)
   - What test framework and runner each layer uses
   - What command runs each layer's tests
   Fill in Section 0.1.

STEP 2 — IDENTIFY OR DESIGN TEST HARNESSES
   A test harness is a shared setup helper that provisions real
   infrastructure for tests (database, HTTP server, message broker, etc.).
   Inspect the test files to find existing harnesses. If none exist, note
   that they must be created and describe what each layer needs.
   Fill in Section 0.2.

STEP 3 — IDENTIFY DETERMINISM PRIMITIVES
   For each source of non-determinism in the project (async I/O, timers,
   randomness, external network calls), identify the banned pattern and
   the required replacement using this project's language and framework.
   Fill in Section 0.3.

STEP 4 — IDENTIFY STATIC ANALYSIS TOOLS
   List every lint, type-check, or static-analysis command that must pass
   before a PR merges, with the exact command to run each one.
   Fill in Section 0.4.

STEP 5 — DOCUMENT CROSS-LAYER CONTRACTS
   Determine whether the project has shared contracts between layers (API
   response shapes, event/message payloads, shared enums, URL parameters).
   For each, document the source of truth, the generation or validation
   mechanism, and the change workflow. Fill in Section 6. If no contracts
   exist yet, document the pattern that MUST be followed when they are
   introduced.

STEP 6 — POPULATE LAYER SECTIONS (Sections 4 and 5)
   Sections 4 and 5 are scaffold layer sections. Replace them with one
   section per architectural layer in this project. For each layer,
   document:
   - Test file layout conventions
   - Which harness to use and how
   - How concurrency is tested deterministically
   - How background-task / thread leaks are detected and enforced
   - Any framework-specific rules (component testing, mock boundaries,
     state reset, reactive assertions, fake timers)
   Use the project's actual language syntax in all code examples.
   Remove any scaffold section that does not apply to this project.
   Renumber subsequent sections if needed.

STEP 7 — REPLACE THE PR CHECKLIST (Section 8)
   Update Section 8 with the real commands and requirements for this
   project. Replace every <PLACEHOLDER> with an actual value.

STEP 8 — POPULATE THE INCIDENT LOG (Section 10)
   Replace the placeholder entries with real bugs from this project that
   illustrate why these standards exist. If the project is new, populate
   Section 10 as incidents occur.

STEP 9 — FINAL CLEANUP
   Search for every remaining <LIKE_THIS> placeholder and fill it in.
   Then delete this instruction block.

============================================================ -->

---

## 0. Project-Specific Requirements

> **AGENT: This section must be completed before this document is used.
> Follow the instruction block above step by step.**

### 0.1 Languages and Test Frameworks

| Layer | Language | Test Framework | Run Command |
|---|---|---|---|
| `<LAYER>` | `<LANGUAGE>` | `<FRAMEWORK>` | `<COMMAND>` |
| `<LAYER>` | `<LANGUAGE>` | `<FRAMEWORK>` | `<COMMAND>` |
| `<ADD MORE ROWS AS NEEDED>` | | | |

### 0.2 Test Harnesses

Test harnesses are shared helpers that provision real infrastructure for
tests. Using them consistently is mandatory — ad-hoc setup creates gaps
between test environments and production.

| Harness | Layer | What it provides | How to invoke |
|---|---|---|---|
| `<HARNESS NAME>` | `<LAYER>` | `<e.g., real database with migrations applied>` | `<INVOCATION EXAMPLE>` |
| `<HARNESS NAME>` | `<LAYER>` | `<e.g., running HTTP server on a random port>` | `<INVOCATION EXAMPLE>` |
| `<ADD MORE ROWS AS NEEDED>` | | | |

**Rule:** Never substitute a mock or in-memory fake for a harness-provided
resource unless the harness itself offers that option. See Section 4 for
details.

### 0.3 Determinism Primitives

| Source of non-determinism | Banned pattern | Required replacement |
|---|---|---|
| Waiting for async work | `<e.g., sleep(50)>` | `<e.g., explicit sync primitive>` |
| Time-dependent logic | `<e.g., DateTime.now()>` | `<e.g., injected clock / fake timer API>` |
| Random values | `<e.g., unseeded rand()>` | `<e.g., seeded RNG or fixed fixture>` |
| External network calls | `<e.g., real HTTP to third-party>` | `<e.g., recorded fixture at network boundary>` |
| `<ADD MORE ROWS AS NEEDED>` | | |

### 0.4 Static Analysis and Type Checks

The following must all pass with zero errors before any PR is merged:

- `<COMMAND>` — `<what it checks>`
- `<COMMAND>` — `<what it checks>`
- `<ADD MORE>`

---

## 1. The Non-Negotiable Rules

1. **Every code path must be covered by a test that fails when the code path breaks.** "It compiles" is not coverage. "It runs without crashing" is not coverage.
2. **A test name must describe the invariant it protects, not the function it calls.** `GetFacets_Disjunctive_ExcludesItsOwnFilter` — yes. `GetFacets` — no.
3. **A test must fail if the production code's behavior changes, and pass otherwise.** If you can delete the assertion and the test still "passes," the assertion is worthless.
4. **No layer is allowed to test only itself when the contract is shared with another layer.** Cross-layer contracts require a contract test or a generated artifact. Hand-mirroring is forbidden — see Section 6.
5. **No mocking of code under test.** Mock external systems (network, filesystem, time when needed). Never mock the function whose behavior the test is verifying.
6. **Tests must be deterministic.** No sleep statements, no real wall clocks, no real DNS, no non-seeded randomness, no network calls to external services. See Section 0.3 for project-specific replacements.
7. **A new feature is not merged without tests.** A bug fix is not merged without a regression test that fails before the fix and passes after.
8. **If you find a bug, write a test that catches it before you write the fix.**

---

## 2. What "Tested" Actually Means

A function is tested if **every meaningful branch, error path, and boundary condition has a dedicated assertion**.

### 2.1 Required Coverage Per Function

For any non-trivial function, the test file must include all of:

- **Happy path** with realistic input.
- **Each conditional branch** taken at least once.
- **Each error return / thrown exception** triggered at least once.
- **Every boundary** (empty input, single-element input, max-size input, off-by-one).
- **Each input that gets clamped, sanitized, or rejected.**
- **Idempotency**, if the function is supposed to be idempotent — call it twice and assert the same result both times.

If a function has any conditional branch or early return, each arm gets a test. No exceptions.

### 2.2 Required Coverage Per Data Path

For any data type that crosses a boundary (database row to object, HTTP request to handler, event payload to consumer, URL to state):

- **Round-trip** through serialize→deserialize must equal the input.
- **Malformed input** must be rejected predictably — a typed error, not a crash or a silent default.
- **Optional / nullable fields** must be exercised both present and absent.
- **Default values** when fields are missing must be asserted explicitly.

### 2.3 Required Coverage Per UI Component

> **AGENT:** If this project has no UI layer, delete this section.
> If it does, rewrite it using the project's actual component framework
> and testing library. Replace every generic bullet point with the
> specific directives, helpers, and assertion patterns that apply.

Every UI component must have a test file that covers:

- **Renders without crashing** with minimum required props.
- **Each conditional render branch** (conditional blocks, empty vs. non-empty lists).
- **Each user interaction** (click, input, keypress) triggers the correct handler with the correct argument.
- **Each prop variant** that affects rendered output.
- **Edge cases the component explicitly handles** (e.g., undefined input does not crash).

If the component has visible state (loading, empty state, error message), each state must be asserted.

---

## 3. Forbidden Test Anti-Patterns

Reviewers will reject any PR containing these. AI agents must refuse to write them.

### 3.1 Vacuous Loops

```
// WRONG — if the collection is empty (the exact bug you are testing for),
// the loop body runs zero times and the test passes silently.
for each item in result.items:
    assert item.field == "expected"
```

Always assert the collection length before iterating:

```
// RIGHT
assert len(result.items) == 1
assert result.items[0].field == "expected"
```

### 3.2 Defensive Guards in Tests

```
// WRONG — the conditional silently skips the core assertion
// when the value is null / nil / undefined.
value = mock.calls[0]
if value is not null:
    assert value <= 499
```

**Tests assert; they do not branch.** If the value could legitimately be null, assert that it is not null first, then assert the value — two separate, unconditional assertions.

### 3.3 Soft Assertions (`>= 2` when you mean `== 2`)

```
// WRONG — passes for the correct answer of 2, and also for 7 phantom results
assert len(result.items) >= 2
```

Use the exact expected count. A test that passes for a wildly wrong answer does not protect the invariant.

### 3.4 Tests That Pass for the Wrong Reason

When testing a typed or structured condition, construct the test value with exactly the type and shape the production code identifies it by. If production code checks both a type and a property, the test input must satisfy both checks — not just the one that happens to be evaluated first.

**Defense:** after writing a test, verify it can fail by temporarily mutating the production code to break the invariant. Confirm the test catches it, then revert. A test that has never been observed to fail provides zero confidence.

### 3.5 Tests That Mock the Thing Under Test

```
// WRONG — the test verifies the mock, not the function
mock(myModule.functionUnderTest)
// ... test that calls functionUnderTest ...
```

Mocks are reserved for dependencies (network, database, external services, time). Never mock the unit under test.

### 3.6 Tautological Assertions

```
// WRONG — obviously circular
assert result == result

// WRONG — subtly circular: expected is derived by re-running the production logic
expected = computeExpectedValue(input)  // same logic as the code under test
assert actualOutput == expected
```

Expected values must be hard-coded or derived independently — never by re-running the code under test.

### 3.7 Snapshot Tests as a Substitute for Assertions

Snapshots are acceptable for large structured outputs where the exact shape is the point (e.g., generated code, serialized configuration). They are not acceptable as a substitute for understanding what the correct output should be. Freezing whatever the code currently produces is not a testing strategy.

### 3.8 Non-Deterministic Timing

Sleep statements, real-clock polling, and fixed-delay waits are banned in test bodies. Use the determinism primitives in Section 0.3. If you need a sleep to wait for background work to finish, you have a missing synchronization primitive in production code — fix the production code, do not add a sleep.

### 3.9 Loose Matchers

```
// WRONG — the pattern matches too many values; ">= 1" accepts almost any output
assert count(elements_matching(/loose-pattern/)) >= 1
```

Match the exact expected text or target a specific element by its unique selector. Matchers must be as tight as the invariant being protected.

### 3.10 Asserting on Internal State Instead of Observable Behavior

UI tests must verify what the user sees in the rendered output, not what an internal component variable holds. Backend tests must verify what the HTTP response contains, not which private helper was called. Tests that couple to internals break on every refactor and provide no real safety net.

---

## 4. Layer Standards: `<LAYER NAME>`

> **AGENT:** Replace this entire section with the standards for the first
> architectural layer of this project (e.g., Backend, API Service, Worker).
> Use the project's actual language and framework throughout — including in
> all code examples. Cover at minimum:
>
> - Test file layout and naming conventions
> - Which harness(es) from Section 0.2 to use and how
> - How concurrency or async work is tested deterministically (no sleeps)
> - How leaked threads / background tasks are detected and enforced
> - Any layer-specific rules not covered by Sections 1–3
>
> If the project has more than two layers, add additional numbered sections
> and renumber what follows accordingly.

### 4.1 Test Layout

`<Describe where test files live relative to the code they test, file naming
conventions, and how shared helpers and fixtures are organized.>`

### 4.2 Infrastructure Harness

`<Identify which harness from Section 0.2 this layer uses. Show a correct
setup and teardown example in the project's language. State explicitly what
must NOT be mocked (e.g., the database, the message broker) and why.>`

### 4.3 Concurrency and Async

`<Describe how tests in this layer wait for concurrent or asynchronous work
to complete without using sleep. Reference the specific primitive from
Section 0.3.>`

### 4.4 Background Task and Thread Leak Detection

`<Describe the mechanism that catches leaked threads or background tasks,
and where it must be enabled (e.g., a global test setup hook, a per-package
teardown, or a shared test fixture).>`

---

## 5. Layer Standards: `<LAYER NAME>`

> **AGENT:** Replace this section with the standards for the second
> architectural layer (e.g., Frontend, CLI, Mobile). Follow the same
> structure as Section 4. If this project has only one layer, delete this
> section. If it has three or more, add further sections as needed.

### 5.1 Component or Unit Test Setup

`<Describe the testing library used, how units or components are rendered
or instantiated, and the rule for what to assert on (observable output,
not internal state).>`

### 5.2 Mock Boundaries

`<Describe where the mock boundary sits (e.g., the network layer, an
external SDK), how mocks are declared, and the rule that every dependency
used by the unit under test must appear in the mock setup.>`

### 5.3 Shared State Reset

`<If this layer uses a shared store or singleton, describe the reset pattern
that must run before each test to prevent state leaking between tests.>`

### 5.4 Reactive or Proxy Assertions

`<If the framework wraps state in reactive proxies, describe the correct
assertion method and why reference equality fails.>`

### 5.5 Async Waits

`<Describe how to wait for async rendering or data-fetching to settle
before asserting, and what pattern is banned.>`

### 5.6 Fake Timers

`<Describe how to replace real timers with controllable fakes for tests
involving debounced or scheduled callbacks, and how to restore real
timers afterward.>`

---

## 6. Cross-Layer Contract Testing

> **AGENT:** This section is the most important to complete and the most
> costly to leave empty. Leaving contracts hand-mirrored between layers
> is the most common source of silent production failures in multi-layer
> systems.
>
> For each contract type below, either document the existing enforcement
> mechanism or document the pattern that MUST be followed when that
> contract type is introduced. Do not leave any entry as a placeholder
> after onboarding.

Cross-layer contracts — shared data shapes, event payloads, status enums,
URL parameters — cannot be validated by layer-local tests alone. Every
contract must have a single source of truth and a mechanism to detect drift.

### 6.1 API Response Shapes

**Source of truth:** `<Where the canonical type definition lives>`

**Enforcement mechanism:** `<How consuming layers stay in sync — e.g., code
generation, OpenAPI schema validation, contract tests>`

**Workflow when changing a shape:**
1. `<Edit the source-of-truth definition.>`
2. `<Run any generator or schema export command.>`
3. `<Commit the source change and any generated artifacts together.>`
4. `<CI runs CONTRACT_TEST_NAME and fails if artifacts are stale.>`

**Rule:** Never hand-write a type on the consuming side to match a type on the
producing side. Extend the generation or validation mechanism instead.

### 6.2 URL and Query Parameters

> String-keyed parameters are invisible to type generation — a rename on one
> side silently breaks the other.

**Contract test:** `<Name and location of the test that verifies every
parameter name used by the consumer maps to the correct field on the producer>`

**Workflow when adding a new parameter:**
1. `<Add to the producer's parser and its type.>`
2. `<Add to the consumer's serializer and its type.>`
3. `<Add to the contract test.>`
4. `<Add a unit test for the consumer's serializer.>`

Skipping any step leaves the system silently broken in production.

### 6.3 Event and Message Payloads

> Applies to any structured message crossing a process boundary: WebSocket
> messages, job queue payloads, pub/sub events, inter-service calls.

**Source of truth:** `<Where the canonical payload type lives>`

**Enforcement:** `<How consumers verify they match the producer — e.g.,
generated types, schema registry, contract tests>`

### 6.4 Shared Enumerations and Status Codes

**Source of truth:** `<Where the canonical enum or constant definition lives>`

**Workflow when adding a new value:**
1. `<Add the constant to the source.>`
2. `<Propagate to all consumers (run generator or update manually with a test).>`
3. `<Update any switch / match statements or label maps on all affected sides.>`
4. `<Update tests for any layer that renders or branches on the new value.>`

### 6.5 Known Contract Gaps

Document contracts that are not yet generated or formally validated, so
agents and engineers know where manual discipline is required.

| Contract | Gap | Recommended fix |
|---|---|---|
| `<e.g., HTTP error response shape>` | `<e.g., hand-mirrored on both sides>` | `<e.g., add to schema generator>` |
| `<ADD MORE>` | `<DESCRIPTION>` | `<FIX>` |

When you encounter a gap, extend the generator or add a contract test rather
than adding another hand-mirrored definition.

---

## 7. The Test Pyramid

- **Unit tests** — the majority of the suite. Fast, isolated, no network, no real database. Targets: pure functions, single components, individual handlers with mocked external dependencies.
- **Integration tests** — tests against real infrastructure provided by the harnesses in Section 0.2. Slower but realistic. Required for any new endpoint, database query, or background worker.
- **Contract tests** — type-generation drift checks and cross-layer contract tests from Section 6. Must run as part of the normal test suite, not as a separate optional step.
- **End-to-end tests** — `<E2E TOOL FROM SECTION 0.1>` tests for complete user flows. Not required for every change; required for any user-flow change that crosses multiple components or layers.

When in doubt, write the integration test. Pure unit tests with extensive
mocks have repeatedly failed to catch real bugs caused by infrastructure
behavior that mocks do not replicate.

---

## 8. PR Checklist

Before requesting review, every box must be checked:

- [ ] Every new function has a test covering: happy path + each branch + each error path.
- [ ] Every modified function has a test that would have failed against the old behavior.
- [ ] Every new UI component has a test file. *(Remove this line if no UI layer.)*
- [ ] If any cross-layer contract changed: the source of truth was updated, any generator was run, generated artifacts are committed, and the contract test passes.
- [ ] No mock returns the same shape the production code returns (tautology check — Section 3.6).
- [ ] No sleep or fixed-delay waits in any test body (Section 3.8).
- [ ] All tests pass with caching disabled: `<COMMAND FROM SECTION 0.1>`.
- [ ] All static analysis checks pass: `<COMMANDS FROM SECTION 0.4>`.
- [ ] `<ADD PROJECT-SPECIFIC ITEMS>`

---

## 9. Process for AI Agents

If you are an AI agent working on this codebase:

1. **Before writing any code that crosses layer boundaries, read Section 6 in full.** If the change involves an API shape, URL parameter, or event payload, update the source of truth and run any generator. Do not hand-mirror types.
2. **Before writing tests, read Section 3 in full.** Every anti-pattern there has been caught in real projects. Do not reintroduce them.
3. **When a test you write passes on the first try, verify it can fail.** Temporarily mutate the production code to break the invariant; re-run the test; confirm it fails; revert. A test that has never been observed to fail provides zero confidence. Better still: write the test before the production code, watch it fail, then make it pass.
4. **When asked to "add tests for X," do not just call X and assert on the return value.** Enumerate every branch, error path, boundary condition, optional field, and empty-collection case — then write one focused test for each.
5. **If you cannot determine what the test should assert, stop and ask.** Do not generate a tautological test. Do not use a snapshot as a substitute for a real assertion. State the ambiguity and request clarification.
6. **Never reduce test strictness to make a failing test pass.** If a test reveals a bug, fix the bug. If a test is overly specific in an irrelevant way, change it deliberately and explain why. Silently loosening an assertion to silence a failure is forbidden.

---

## 10. Why This Document Exists

> **AGENT:** Replace these placeholders with real bugs from this project's
> history. Each entry should state what went wrong, which anti-pattern from
> Section 3 caused it, and which rule would have caught it. If the project
> is new, populate this section as incidents occur — it is a living record.

In the lifetime of this project, we have shipped bugs that had passing tests:

- **`<INCIDENT TITLE>`** — `<What went wrong. Which anti-pattern (Section 3.X) caused it. Which rule (Section 1.X or 2.X) would have caught it.>`
- **`<INCIDENT TITLE>`** — `<DESCRIPTION>`
- **`<INCIDENT TITLE>`** — `<DESCRIPTION>`

Every one of these had passing tests before it shipped. Every one of them
would have been caught by the rules in this document. The rules are
non-negotiable because the alternative is these bugs, on repeat, forever.
