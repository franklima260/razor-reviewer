# Telemetry Standards

> **Template Version:** 1.0  
> **Last Updated:** <!-- FILL IN DATE -->  
> **Project:** <!-- FILL IN PROJECT NAME -->

This document is the source of truth for **what to log, when, how, and why**
in `[PROJECT_NAME]`. It is written primarily for AI coding agents — the assumption
that will frame every decision below — but applies equally to human contributors.

---

<!-- ============================================================
AGENT INSTRUCTION — PROJECT-SPECIFIC REQUIREMENTS
==============================================================

You are an AI coding agent onboarding to a new project. This document is a
standardized telemetry template. Before using it, you MUST complete Section 0
below with project-specific information.

To fill in Section 0, do the following:

1. Identify the project's telemetry/logging package or library (e.g., the
   equivalent of `telemetry.Info`, `telemetry.Counter`, etc.) and document
   the correct import path and function signatures.

2. Identify the project's programming language(s) and update all code examples
   in §2 and §6 to use the correct syntax and idioms for that language.

3. Identify the runtime environment variable (or config key) used to control
   log level at runtime and document it in §7.

4. Identify the top-level components/subsystems in the codebase (e.g., api,
   auth, pipeline, scanner) and list them in §0.4, then add component-specific
   diagnostic recipes to §3 following the existing recipe format.

5. Identify any PII or sensitive data types specific to this project (beyond
   the generic examples in §2.5) and add them to §0.5.

6. List all related documents (architecture docs, runbooks, incident reports)
   in §9.

7. Replace every placeholder in angle brackets — <LIKE_THIS> — with the
   correct project-specific value. Search for `<` to find them all.

8. Delete this instruction block when Section 0 is complete.

============================================================ -->

---

## 0. Project-Specific Requirements

> **⚠️ AGENT: This section must be completed before this document is used in a project.
> See the instruction block above for how to fill it in.**

### 0.1 Telemetry Package

This project uses `<TELEMETRY_PACKAGE_IMPORT_PATH>` as its telemetry abstraction.
All logging and metric calls must go through this package — never through the
underlying logging library directly.

| Function | Purpose | Signature |
|---|---|---|
| `<Info>` | State transitions / events | `<Info(ctx, msg, key, val, ...)>` |
| `<Warn>` | Recoverable problems | `<Warn(ctx, msg, key, val, ...)>` |
| `<Error>` | Unrecoverable problems | `<Error(ctx, msg, key, val, ...)>` |
| `<Debug>` | Step-by-step trace | `<Debug(ctx, msg, key, val, ...)>` |
| `<Counter>` | Counted events | `<Counter(ctx, name, key, val, ...)>` |
| `<Gauge>` | Point-in-time values | `<Gauge(ctx, name, val)>` |
| `<Histogram>` | Latency / size distributions | `<Histogram(ctx, name, val)>` |
| `<With>` | Component-scoped child logger | `<With(key, val, ...)>` |
| `<Enabled>` | Level-gate expensive work | `<Enabled(ctx, level)>` |
| `<CaptureForTests>` | Capture logs in tests | `<CaptureForTests(handler)>` |

### 0.2 Language and Runtime

- **Language:** `<LANGUAGE / VERSION>`
- **Log level environment variable:** `<LOG_LEVEL_ENV_VAR>` (e.g., `APP_LOG_LEVEL`)
- **Default log level:** `<DEFAULT_LEVEL>` (recommended: `info`)
- **Log format (production):** `<JSON | logfmt | OTHER>`
- **Log format (development):** `<logfmt | pretty | OTHER>`

### 0.3 Metrics Export

- **Metrics endpoint:** `<URL_PATH>` (e.g., `/metrics`)
- **Format:** `<Prometheus | StatsD | CloudWatch | OTHER>`
- **Scrape interval:** `<INTERVAL>`

### 0.4 Known Components and Metric Prefixes

| Component | Metric prefix | Description |
|---|---|---|
| `<api>` | `api.` | `<HTTP request handling>` |
| `<auth>` | `auth.` | `<Authentication and sessions>` |
| `<ADD MORE>` | `<prefix>.` | `<description>` |

New components must be registered here before metrics are added.

### 0.5 Project-Specific PII and Sensitive Data

In addition to the generic rules in §2.5, the following data types are
considered sensitive in this project and must never appear in logs:

- `<e.g., customer_id — use hashed or last-4 only>`
- `<e.g., payment_token — always REDACTED>`
- `<ADD MORE>`

---

## 1. Why Rigorous Telemetry Matters

### 1.1 The Cost of Missing Telemetry: A Case Study

> **AGENT:** Replace this section with a real or illustrative incident from
> the project that demonstrates the cost of missing telemetry. The narrative
> should follow this structure:
> 1. The symptom reported by the user.
> 2. The multi-step investigation required because telemetry was absent.
> 3. The root cause eventually found.
> 4. What the log output *would have looked like* with proper telemetry, and
>    how quickly it would have led to diagnosis.

**[INCIDENT TITLE — e.g., "The Auth Token Expiry Investigation"]**

Before rigorous telemetry was in place, a user reported:

> `<INSERT SYMPTOM — e.g., "users are getting logged out randomly">`

The investigation took **<N> sessions**. The root cause was `<ROOT_CAUSE>`.

With the telemetry now required by this document, the original report would
have been self-diagnosing:

```
<INSERT EXAMPLE LOG OUTPUT SHOWING THE DIAGNOSIS>
```

From those lines an agent could establish the cause without reading any code.

### 1.2 What Rigorous Telemetry Buys

- **Diagnosis without re-runs.** A single failure leaves a complete, structured trail.
- **Pattern detection.** Counters surface error-rate trends that individual log lines cannot.
- **Regression detection.** Histogram shifts after a refactor are visible immediately.
- **Reduced agent context cost.** An agent that doesn't need to read source files to interpret a log line has more budget for the actual fix.
- **Audit trail.** Settings changes, auth failures, and crashes are recorded for forensic review.

### 1.3 What Missing Telemetry Costs

- **Silent failures.** Background goroutines / workers can leak for weeks before anyone notices.
- **Speculation.** Without outcome counters, distinguishing a bug from an attack is impossible.
- **Re-running expensive work** just to capture a print statement.
- **Wrong fixes.** An agent investigating the wrong subsystem because counters for the real cause don't exist.

---

## 2. Standards: The Always-Do List

These are not suggestions. Code that violates them must be rejected at review.

### 2.1 Use the Telemetry Package; Never the Underlying Logger Directly

```
// WRONG
<log.Printf("scan complete: %d files", count)>
<fmt.Println("dropping client")>

// RIGHT
<telemetry.Info(ctx, "scan complete", "files", count)>
<telemetry.Info(ctx, "ws: dropping slow client", "client_id", id)>
```

The telemetry package is the single chokepoint for level filtering, test
capture, and metrics export. Calls outside it bypass all of these.

### 2.2 Use the Right Helper for the Right Shape

| Shape | Helper | Example |
|---|---|---|
| State transition / event | `<Info>` | `"scan started"`, `"db: migration applied"` |
| Recoverable problem | `<Warn>` | `"primary backend failed, using fallback"` |
| Unrecoverable problem | `<Error>` | `"failed to bind port"` |
| Step-by-step trace | `<Debug>` | `"sending batch"` with token/item counts |
| Counted event | `<Counter>` | `"api.request{outcome=ok}"` |
| Point-in-time value | `<Gauge>` | `"pipeline.queue_depth"` |
| Latency / size distribution | `<Histogram>` | `"db.query_ms"` |

All logging helpers must pass a context object as their first parameter (see §2.9).

Never emit a metric by formatting a log line manually. Use the dedicated
`<Counter>` / `<Gauge>` / `<Histogram>` helpers so every metric record has a
consistent, machine-readable shape.

### 2.3 Use Structured Key/Value, Never String Interpolation

```
// WRONG — values are unsearchable; agent must parse free text
<telemetry.Info(ctx, fmt.Sprintf("processed %d files in %dms for user %s", n, ms, user))>

// RIGHT — every field is grep-able as key=value
<telemetry.Info(ctx, "scan: batch complete", "files", n, "duration_ms", ms, "user", user)>
```

Structured fields let an agent answer questions like "how many files were in
the slow batches?" with a simple filter instead of a regex against free text.

### 2.4 Bucket Outcomes; Do Not Log Only the Happy Path

Every meaningful branch should emit a counter or log on **every** exit —
not just on success or just on failure.

```
// WRONG — only one outcome is observable
<resp, err := client.Do(req)
if err != nil {
    telemetry.Error(ctx, "request failed", "err", err)
    return err
}
// success path emits nothing>

// RIGHT — every outcome is bucketed
<switch {
case err != nil:
    telemetry.Counter(ctx, "<component>.request", "outcome", "network_err")
    telemetry.Error(ctx, "<component> connection failed", "err", err)
case resp.StatusCode >= 500:
    telemetry.Counter(ctx, "<component>.request", "outcome", "5xx")
case resp.StatusCode != 200:
    telemetry.Counter(ctx, "<component>.request", "outcome", "4xx")
default:
    telemetry.Counter(ctx, "<component>.request", "outcome", "ok")
}>
```

A counter with no exits in production tells the agent the path is not firing.
A counter with all exits shows the agent the rates. "Logged on failure only"
provides neither.

### 2.5 Sanitize Before Logging (No PII, Secrets, or Noise)

Three rules apply everywhere:

1. **Run error messages through the sanitizer** before logging. File paths
   must be reduced to basenames; never log absolute paths.
2. **Redact secrets.** Any setting or header whose key matches a sensitive
   pattern (e.g., `*_api_key`, `*_token`, `*_secret`, `*_password`) must
   appear as `[REDACTED]`.
3. **Never log a request body, prompt, or binary payload at INFO or above.**
   Acceptable at DEBUG if plain text under ~1 KB. Binary / image data must
   never appear in logs — log the byte count instead.

See §0.5 for project-specific sensitive data types.

### 2.6 Use the Per-Status Log Level

| Status | Level | Rationale |
|---|---|---|
| `5xx` | `Error` | Server bug; always log |
| `400` | `Warn` | Request validation failure |
| `401`, `403`, `404` | `Info` | Routine; don't spam Warn/Error |
| `2xx` / `3xx` non-static | `Info` | One line per request |
| Static assets | `Debug` | Skip at default INFO level |

Do not bypass this mapping in new endpoints.

### 2.7 Log at State Transitions, Not State Continuations

```
// WRONG — one line per item floods logs
<for _, f := range files {
    telemetry.Info(ctx, "scanning file", "name", f.Name)
    process(ctx, f)
}>

// RIGHT — one line per batch, count as data
<telemetry.Info(ctx, "scan: starting batch", "files", len(files))
for _, f := range files {
    process(ctx, f)
}
telemetry.Info(ctx, "scan: batch complete", "files", len(files), "duration_ms", ms)>
```

Exception: if a per-item log line is what an agent will grep for when
diagnosing a specific item's failure, emit it at `Debug`.

### 2.8 Name Metrics with `component.thing{label=value}` Style

| Pattern | Example |
|---|---|
| `<component>.<event>` (counter) | `auth.login`, `api.request`, `<component>.task` |
| `<component>.<thing>` (gauge) | `pipeline.queue_depth`, `runtime.goroutines` |
| `<component>.<thing>_<unit>` (histogram) | `db.query_ms`, `api.request_duration_ms` |
| Label keys | `outcome`, `provider`, `reason`, `backend`, `missing` |

Rules:
- Units in the metric name, never in the value: `_ms`, `_bytes`, `_mb`.
- Label values must have bounded cardinality. `outcome=ok` ✓. `path=...` ✗.
- New components get their own prefix; don't pile everything under one prefix.

### 2.9 Always Pass a Context Object

Every logging function must accept a context as its first parameter. This
enables request ID and trace ID propagation across concurrent systems.

```
// WRONG — impossible to correlate with a specific request
<telemetry.Info("user logged in", "user_id", id)>

// RIGHT
<telemetry.Info(ctx, "user logged in", "user_id", id)>
```

### 2.10 Use Component-Scoped Child Loggers

When logging repeatedly within a loop or subsystem, use `<telemetry.With>`
to create a scoped logger rather than repeating the same fields.

```
// WRONG — same "file" field repeated on every line
<telemetry.Debug(ctx, "parsing started", "file", f.Name)
telemetry.Info(ctx, "parsing complete", "file", f.Name)>

// RIGHT
<log := telemetry.With("file", f.Name)
log.Debug(ctx, "parsing started")
log.Info(ctx, "parsing complete")>
```

### 2.11 Wrap Errors and Capture Stack Traces

When logging at `Error` level, the error must include location context — either
by wrapping it with a message (e.g., `fmt.Errorf("doing X: %w", err)` in Go)
or by using a stack-trace-aware error package. Raw `err` alone provides no
location information to the agent diagnosing the failure.

### 2.12 Protect Expensive Log Evaluations

Do not compute expensive fields unless the target log level is active.

```
// WRONG — calculateExpensiveStats() runs even when DEBUG is off
<telemetry.Debug(ctx, "processing complete", "stats", calculateExpensiveStats())>

// RIGHT
<if telemetry.Enabled(ctx, LevelDebug) {
    telemetry.Debug(ctx, "processing complete", "stats", calculateExpensiveStats())
}>
```

---

## 3. Agent Diagnostic Recipes

> **If you are an agent, this is the first section to read when diagnosing a problem.**  
> **AGENT:** Add a recipe here for every significant failure mode in the project.
> Follow the format of the existing recipes. Each recipe must include a grep
> order and an interpretation table.

Each recipe follows this format:

1. **Grep order** — the sequence of patterns to search for, from most to
   least specific. Start with metric counters; fall back to log messages.
2. **Interpretation** — what each combination of findings means, and where
   to look next.

### 3.1 `<FAILURE MODE — e.g., "API requests are failing">`

Grep order:
```
metric.counter name=<component>.request outcome=5xx
metric.counter name=<component>.request outcome=network_err
"<component>: connection failed"
"level=ERROR"
```

Interpretation:
- `<component>.request{outcome=5xx}` rising → server-side error in
  `<component>`; check Error logs for stack traces around the same timestamp.
- `{outcome=network_err}` rising → upstream dependency is unreachable; check
  infrastructure / DNS.
- No counters at all → the request path isn't being reached; check routing
  and auth logs.

### 3.2 `<FAILURE MODE — e.g., "Background worker is stuck">`

Grep order:
```
metric.gauge name=<component>.queue_depth
metric.gauge name=<component>.workers_busy
"<component>: work channel full"
```

Interpretation:
- `queue_depth` rising, `workers_busy` at max → workers are hung on a specific
  item; look for the last `Debug` log before the stall.
- `queue_depth` rising, `workers_busy` at 0 → workers have crashed; look for
  Error logs around the time the gauge flatlined.
- `queue_depth` stable at 0 → producer is not enqueuing; check the upstream
  trigger (scheduler, event listener, etc.).

### 3.3 `<FAILURE MODE — e.g., "Goroutine / thread count keeps climbing">`

Grep order:
```
metric.gauge name=runtime.goroutines
metric.gauge name=<component>.active_connections
```

Interpretation:
- `runtime.goroutines` climbing alongside `active_connections` → connections
  are not being cleaned up on close. Verify teardown logic.
- `runtime.goroutines` climbing in isolation → leak in an uninstrumented
  subsystem; add a gauge inside its event loop to surface it.

> **AGENT:** Add more recipes here for every component listed in §0.4.

---

## 4. Adding New Telemetry: Checklist

Before merging code that adds a feature or touches a hot path, the
author (or reviewing agent) must answer **yes** to each:

- [ ] Does every meaningful branch outcome emit a counter or log?
- [ ] Is every emission passing a context object to the telemetry functions?
- [ ] Are repeating attributes across a component deduplicated using `<telemetry.With>`?
- [ ] Are field names structured key/value, not interpolated strings?
- [ ] Does the log level match §2.6's table?
- [ ] Are PII-bearing values (paths, headers, settings, prompts, binary data)
      sanitized or omitted per §2.5 and §0.5?
- [ ] If the code spawns a background worker or goroutine, does it emit a
      periodic gauge that would surface the worker's existence in
      `runtime.goroutines` drift?
- [ ] Are expensive log parameters guarded by `<telemetry.Enabled>`?
- [ ] Do unrecoverable `Error` level logs include wrapped errors / stack traces?
- [ ] Is there a unit test (using `<telemetry.CaptureForTests>`) that pins
      the contract of any new metric that dashboards or diagnostic recipes
      will rely on?
- [ ] Have you added or updated a recipe in §3 if the new metric supports
      a new diagnostic scenario?

A `no` to any item is an open issue. Fix it before merging, or file a
follow-up task and link it from the code with a `TODO(telemetry)` comment.

---

## 5. Anti-Patterns (Do Not Do)

| Anti-pattern | Why bad | Fix |
|---|---|---|
| Using the underlying logger directly | Bypasses level filtering and test capture | Use `<telemetry.Info(ctx, ...)>` |
| `<telemetry.Info(ctx, "failed: " + err.Error())>` | Unstructured; unsearchable | `<telemetry.Error(ctx, "failed", "err", err)>` |
| Logging only on error | Hides success rate; no baseline for anomaly detection | Bucket every outcome (§2.4) |
| `<telemetry.Info(ctx, "got %d", n)>` | Unstructured | `<telemetry.Info(ctx, "got", "n", n)>` |
| `<telemetry.Info(ctx, "counter: x=1")>` | Not a real metric; invisible to scraping | `<telemetry.Counter(ctx, "x")>` |
| Logging without a context object | Breaks trace and request ID correlation | Always pass `ctx` as first arg |
| `name="user_login_event_counter"` | Verbose, inconsistent naming | `name="auth.login"` (§2.8) |
| Mixed label key styles (`userPath`, `userpath`, `user_path`) | Inconsistent; breaks grep patterns | Pick one style (snake_case); enforce in review |
| Logging absolute file paths or user-supplied input | Leaks PII | Sanitize per §2.5 |
| Logging binary payloads (images, blobs) | Floods logs with base64 | Log `bytes_len` instead |
| `<telemetry.Info(ctx, "step")>` with no fields | Tells the agent nothing | Include relevant IDs and counts |
| Background worker with no gauge / heartbeat | Invisible failure mode | Emit a gauge from inside the worker loop |
| Metric label with unbounded cardinality (paths, IDs) | Breaks aggregation | Move high-cardinality data to log line; keep label tight |
| INFO-level logging inside hot loops | Floods logs; hides signal | Move to DEBUG or emit a post-loop summary |
| Repeating the same fields instead of using `<telemetry.With>` | Error-prone boilerplate | Use a scoped child logger (§2.10) |
| Computing expensive stats for a guarded log level | Silent CPU regression | Guard with `<telemetry.Enabled>` (§2.12) |

---

## 6. Test Conventions

### 6.1 Use `<telemetry.CaptureForTests>` for Log Assertions

```
<buf := &bytes.Buffer{}
restore := telemetry.CaptureForTests(slog.NewTextHandler(buf, &slog.HandlerOptions{Level: slog.LevelDebug}))
defer restore()

// exercise code under test

out := buf.String()
if !strings.Contains(out, "level=ERROR") {
    t.Errorf("expected ERROR log, got:\n%s", out)
}
if strings.Contains(out, secretValue) {
    t.Errorf("secret leaked into logs:\n%s", out)
}>
```

Do not invent your own log-capture mechanism.

### 6.2 Pin Metric Contracts in Tests

If a diagnostic recipe in §3 says "grep for `metric.counter name=auth.login`",
a regression that renames it to `auth.logins` silently breaks every runbook.
Pin the contract:

```
<func Test_Auth_EmitsLoginCounter(t *testing.T) {
    buf := captureMetric(t, LevelInfo)
    // ... drive a successful login ...
    if !strings.Contains(buf.String(), "name=auth.login") {
        t.Errorf("auth.login counter not emitted")
    }
}>
```

### 6.3 Goroutine / Thread Leak Detection

Every package that spawns long-lived background workers must include a
leak-detection mechanism in its test suite (e.g., `goleak.VerifyTestMain`
in Go). A test that forgets to close or cancel a resource must fail the
package run. This is non-negotiable.

---

## 7. Levels: The Runtime View

`<LOG_LEVEL_ENV_VAR>` controls what is visible in production.

| Level | What's visible | When to use |
|---|---|---|
| `off` | Nothing | Tests, CI, intentionally silent deploys |
| `error` | Only `Error` and `Fatal` | Extreme noise reduction |
| `warn` | `Warn` and above | Post-stabilization deployments |
| `info` | `Info` and above, including all metric records | **Default** |
| `debug` | Everything, including per-call traces | Active diagnosis sessions |

Metrics (`Counter`, `Gauge`, `Histogram`) emit at `Info`. They are part of
the default visible output. Setting the level to `warn` silences metrics —
a deliberate choice made only for minimal-output scenarios.

### 7.1 Output Formats and Rotation

By default, logs write to standard error. In production, logs must be written
in **JSON format** with rotation enabled (e.g., 50 MB max size, 3 backups) to
prevent disk exhaustion and ensure agents can natively parse multiline stack
traces. For local development, `logfmt` or a human-readable text format is
acceptable.

### 7.2 Metrics Export

Metrics are emitted as structured log records (`msg=metric.counter`, etc.)
and must also be exposed via the `/metrics` endpoint defined in §0.3. This
decouples telemetry recording from standard application logging.

### 7.3 Audit Logs

Security and audit events (settings changes, auth failures, privilege
escalation) must be routed to a persistent, dedicated sink separate from
volatile debug logs, ensuring retention even as ephemeral logs rotate.

---

## 8. Glossary

- **Agent / agentic** — The AI coding assistant working on the codebase.
  "Agent-first" means decisions are weighted toward what makes diagnosis
  fast and unambiguous for an agent with a fresh context window.
- **Structured logging** — Log records expressed as key/value pairs, not
  formatted strings.
- **Counter / Gauge / Histogram** — Three metric shapes. A *counter* only
  ever increases (e.g., requests served). A *gauge* is a point-in-time value
  that can rise or fall (e.g., queue depth). A *histogram* observes a
  distribution (e.g., latencies).
- **Cardinality** — The number of distinct label-value combinations a metric
  can produce. Low cardinality (`outcome=ok|err`) is good; high cardinality
  (`user_id`, `path`) breaks aggregation systems.
- **Tombstone test** — A skipped test that documents a known gap so future
  readers (and agents) don't forget about it.
- **Sanitizer** — A utility function that strips or masks PII and secrets from
  values before they are included in log records.

---

## 9. Related Documents

> **AGENT:** Replace the placeholder entries below with links to the actual
> documents for this project. Include architecture docs, prior incident
> reports, runbooks, and any observability plans that informed this document.

- `<architecture.md>` — System architecture overview. Describes component
  boundaries and data flows that inform metric prefix assignments in §0.4.
- `<incident_reports/>` — Post-mortems and incident reports. Each major
  incident should have a corresponding diagnostic recipe added to §3.
- `<runbooks/>` — Operational runbooks that reference metric names defined
  in this document.
- `<observability_plan.md>` — (If applicable) The phased plan that landed
  the current telemetry infrastructure. Explains *why* the system has the
  metrics it has.
