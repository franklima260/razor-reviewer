---
name: telemetry-standards
description: Instantiate, audit against, or enforce structured logging + metrics standards in a codebase. Use when the user asks to "set up logging/telemetry standards", "audit our logging", "why is this impossible to debug", mentions observability, structured logging, metrics naming, log levels, or PII in logs — and whenever new components, background workers, network calls, or error paths are being added to a project that has (or should have) a Telemetry_Standards.md. Telemetry is what lets an agent diagnose failures without re-running them; treat missing telemetry on a new failure path as a defect.
---

# Telemetry Standards — instantiate, audit, enforce

Premise: an agent (or human) diagnosing a failure has two options — read a structured
trail the code left behind, or speculate and re-run expensive work. Telemetry standards
exist so that every meaningful failure is self-diagnosing from its logs. This matters
double for agent-first development: an agent that can answer "what happened" from
`key=value` logs spends its context on the fix, not the forensics.

The canonical template lives at `assets/Telemetry_Standards_Template.md` (~600 lines,
language-agnostic, agent instruction block included). It covers: the single-chokepoint
telemetry package rule, helper selection (Info/Warn/Error/Debug/Counter/Gauge/Histogram),
structured key/value (never interpolation), outcome bucketing on every branch exit, PII
and secret sanitization, log-at-transitions-not-continuations, metric naming
(`component.thing{label=value}`, units in names, bounded cardinality), context
propagation, child loggers, error wrapping, level-gated expensive evaluation, agent
diagnostic recipes, a per-PR checklist, an anti-pattern table, and test conventions
(log capture, metric contract pinning, leak detection).

Three modes:

## Mode 1 — BOOTSTRAP (no standards doc)

Copy the template to the project's docs dir and complete its embedded instruction block:
identify the telemetry package (or pick/create the single chokepoint if the project logs
ad hoc — that consolidation is the highest-value single step), rewrite the code examples
in the project's language, fill in the level env var, list components and their metric
prefixes, and enumerate project-specific PII. Judgment calls:

- If the project currently logs through multiple mechanisms (`print`, framework logger,
  bespoke wrappers), the bootstrap includes a migration note: new code uses the
  chokepoint; a task is filed to migrate the rest. Don't declare a standard the codebase
  visibly ignores without a path to compliance.
- §1's case study should be a REAL incident from this project if one exists; otherwise
  mark it illustrative. Never present invented history as history (same rule as the
  testing incident log).
- Diagnostic recipes (§3) are the highest-leverage section for agent workflows: for each
  known failure mode, the grep order and the interpretation table. Seed with the failure
  modes the architecture makes likely; grow with every incident.

## Mode 2 — AUDIT (standards exist; verify the code against them)

1. **Chokepoint audit**: grep for direct logger/print usage outside the telemetry
   package. Every hit is a finding (it bypasses level filtering, test capture, and
   export).
2. **Outcome bucketing audit**: sample the important branch points (network calls, queue
   operations, auth decisions). Does every exit emit — or only failures? Happy-path-silent
   code has no baseline, so anomalies are invisible.
3. **PII/secret sweep**: grep logs calls for paths, tokens, request bodies, user
   identifiers; check against the sanitizer rules and the project's §0.5 list. Findings
   here are severity-high.
4. **Cardinality check**: metric labels carrying unbounded values (paths, IDs, free
   text) break aggregation — findings with the "move to log line, keep label bounded" fix.
5. **Contract pinning**: for each metric a dashboard or recipe depends on, is there a test
   pinning its name and labels (template §6.2)? A silent rename breaks every runbook.
6. **Worker visibility**: every long-lived background worker emits a heartbeat gauge; every
   spawning package has leak detection in tests (§6.3). Silent workers are the
   canonical weeks-later incident.
7. Report findings ranked, with file:line and the grep evidence, in the ledger if the
   project uses one.

## Mode 3 — ENFORCE (ongoing, during every feature/review)

For each new or touched code path, apply the template's §4 checklist literally. The items
that catch the most real defects, in practice:

- Every meaningful branch outcome emits a counter or log — including success.
- Structured key/value only; a `fmt.Sprintf`/f-string inside a log call is a finding.
- Levels match the per-status table (§2.6); hot loops log at Debug or post-loop summary.
- New failure paths added by this change get a diagnostic recipe entry (§3) if an agent
  would plausibly be asked to debug them later — write the grep order NOW while the
  mechanism is fresh, not during the incident.
- Metric names follow `component.thing` with units in the name and bounded labels.
- Any new metric that a recipe or dashboard will rely on gets a contract-pinning test in
  the same commit.
