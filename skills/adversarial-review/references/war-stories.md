# War Stories — the incidents behind the protocol

Every rule in the adversarial-review skill was paid for by a real overturned "done" claim.
These are the generalized forms. Names and domains are stripped; the mechanics transfer to
any stack. When a review feels like paranoid overkill, reread the story that created the rule.

## 1. The benchmark that measured a build nobody ships

A performance-critical architectural decision (adopt or abandon a subsystem) hinged on a
benchmark showing an unacceptable slowdown. Review found the benchmark's build flags were
unrecorded. Re-run in a proper optimized build: the slowdown shrank 4-5x and the decision
flipped. Then a LATER review found the "corrected" flags still didn't match production —
the shipping build deliberately re-enabled runtime assertions via a define the benchmark had
turned off, and the true number was 5x worse than the corrected one. Two separate reviews,
same lesson twice:

- Performance and behavior claims are only valid in the configuration that ships.
- Verify the flags against the real build system (CMake/CI/package config), not against the
  benchmark script's own comment claiming it matches.
- Record exact build commands next to every measurement, and have the binary self-report
  its configuration where possible.

## 2. Nice numbers hide precision bugs (three separate catches)

Tests used tidy fixture values: radius 10 at the origin, angles at 0/90/180/270 (that
project's values were geometric; yours might be $10.00 prices, whole-hour timestamps, or
100-unit quantities — the trap is identical). All green.
A "hostile counterpart" policy (non-integer values, off-origin, arbitrary rotation) was added
to the testing standards — and its very first application found a real bug: a coordinate like
`center + radius` was exactly representable in floating point for the nice fixture (0 + 10)
but not for the hostile one (1.3 + 10.3), and an exact-equality fallback in the code only
handled the representable case. The failure was structurally invisible to every nice-number
test that could ever be written. This pattern recurred three times in one project phase.

- Every invariant test gets a hostile twin: values that don't round-trip exactly, positions
  off the axes, sizes that aren't powers of ten, unicode in strings, empty collections.
- When a test passes on nice inputs and someone says the hostile case is "the same code
  path" — that claim is exactly what the hostile fixture exists to check.

## 3. "Verified" written before verification ran

An implementing agent updated the ledger to "RESOLVED — implemented and verified" (dated,
even) and then died mid-task: work uncommitted, one documented component (a compile guard)
never written despite a comment claiming it existed, and the verification run never
performed. When the reviewer actually ran the suite, it FAILED — the hostile fixture caught
a real bug the "verified" claim had never tested.

- The ledger records what ran, never what was intended to run.
- A "verified" claim with no reproducible run behind it is itself a severity-2 finding.
- Documentation referencing an artifact (guard, check, test) is a claim: grep for the
  artifact.

## 4. The reviewer's own error, propagated as fact

A reviewer ran binaries through a shell whose call-operator quirk swallowed all output,
concluded "binaries fail to launch," and recorded it in the ledger. Later work inherited
that "fact" and scoped tasks around it. The binaries had been fine all along. Cost: a full
remediation cycle spent purging a reviewer artifact from the record.

- Reviewer claims are hypotheses with the same epistemic status as implementer claims.
- Before recording a negative result ("X doesn't work"), reproduce it a second way
  (different shell, explicit exit-code check, log-to-file).
- When your error is found, correct it in the ledger with the same prominence you'd give an
  implementer's error. The protocol survives on symmetry.

## 5. Golden files regenerated while a defect was live

A "flag day" task regenerated all expected-output files to bless a new default behavior —
while an open output-corruption bug was producing subtly wrong values. The regeneration
baked the bug into the goldens: every test now PASSED against corrupted expectations.
Quarantining the attempt and rebuilding the goldens cost a multi-task remediation phase.

- Golden/expected-output regeneration is a privileged, human-gated operation that happens
  only when the known-defect list is empty.
- Review any golden-touching commit by diffing a SAMPLE of goldens against independently
  computed values, never by "tests pass now."

## 6. The unwired component

A sanitization function was written, unit-tested, green — and never called from the code
path it was supposed to protect. The unit tests exercised the component in isolation;
nothing exercised the integration. It shipped disabled, silently.

- Unit-green is not shipped. For every new component, grep the call sites; if the only
  caller is the test file, that's a finding.
- Demand one end-to-end fixture that enters through the real entry point (CLI, HTTP
  handler, UI action) and observes the component's effect from outside.

## 7. The most common input shape was the untested one

A conversion component handled every shape its author's tests constructed — and threw on
the single most common shape real usage produces (in that project, polygons with holes;
in yours it might be nulls, empty lists, duplicate keys, or the component's own output fed
back in). The author's "supported scope" note described a narrow exotic limitation while
this mundane one went undocumented. Bonus finding: the scope-limit test that DID exist was
throwing for a different, shallower reason than the one documented — the exotic mechanism
had never actually been isolated.

- Enumerate the input domain from production's point of view, not the test author's.
- Re-entry (output → input round trip) is a first-class input class.
- When a test asserts "this throws," verify WHAT throws — the documented mechanism or an
  unrelated earlier check masking it.

## 8. Evidence not bound to the tree

Test results were reported from binaries built minutes BEFORE the final commit. Nothing
proved the committed source produced those binaries — content luck (an incremental no-op)
happened to save the evidence chain, but only a post-hoc rebuild demonstrated it.

- Evidence runs happen AFTER the final commit, from the committed tree.
- The reviewer's version: run the incremental build; a no-op binds artifacts to tree;
  anything else means re-run everything.

## 9. "All lanes green" meant one suite in one configuration

A commit message claimed full green across build configurations; the fine print revealed
only the unit-test binary had run — not the full regression suite, not all configurations.
The phrase had historically meant "everything, everywhere," so the claim borrowed unearned
credibility from precedent.

- Pin claims to commands: "X/X assertions in `<exact command>` in configs A, B, C."
- When a claim reuses a phrase with an established stronger meaning, flag the dilution
  even when the work itself is fine.

## 10. STOP gates and design-guess thresholds

A go/no-go threshold ("≤10x slowdown") written as a design guess got treated as a hard
law, nearly killing a viable architecture — and later, the corrected numbers were nearly
accepted without noticing they exceeded the same threshold, because the label ("Go") had
already been granted. Both directions of the error came from confusing the LABEL with the
EVIDENCE.

- Decision thresholds are inputs to a human ruling, not self-executing laws. When evidence
  moves materially, the ruling returns to the human even if the label wouldn't change.
- Record rulings verbatim in the ledger with the numbers they were made against, so a later
  change in numbers visibly invalidates the ruling's basis.
