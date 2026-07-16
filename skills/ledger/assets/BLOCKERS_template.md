# <PROJECT> — Blockers & Deviations Ledger

<!-- Every deviation from plan, discovered limitation, and unresolved question gets an
     entry. STATUS headers are maintained forever — a stale OPEN poisons future risk
     assessment. Repro fixtures are preserved VERBATIM: they are the acceptance tests for
     the fixes. Delete this comment after bootstrap. -->

---

## <TASK ID> — <One-line description of the deviation/limitation> — STATUS: OPEN (<date>)

**What was found:**

<Symptoms, then mechanism. Concrete: commands run, exact error text, file:line.>

**Why this blocks / deviates:**

<Which plan assumption or Done-when it contradicts.>

**Repro (verbatim — the fix must turn this into a passing test):**

```
<minimal executable fixture>
```

**Options considered:**

1. <Option — cost/risk>
2. <Option — cost/risk>

**Resolution:** <filled in when STATUS changes to RESOLVED, with date, commit, and who/what
verified it — "verified" requires an executed run, not intent.>
