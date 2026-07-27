---
name: commenting-standards
description: Instantiate, audit against, or enforce binding commenting and documentation standards in a codebase. Use when the user asks to "set up commenting standards", "audit our comments", "clean up the AI comment noise", mentions stale/misleading comments, docstring conventions, TODO hygiene, self-documenting code, or README/CLAUDE.md drift — and whenever code is being written in a project that has (or should have) a COMMENTING_STANDARDS.md. In agent-written codebases a wrong comment misdirects every future agent that reads the file; treat a comment contradicted by the code as a defect, not as clutter.
---

# Commenting Standards — instantiate, audit, enforce

Premise: code is the only authority on what a system does — it is the only artifact
that gets executed. Comments, docstrings, READMEs, plans and ledgers are unverified
claims, checked by nothing, and they enter an agent's context with the same visual
authority as the code beside them. A wrong comment is therefore a defect with a long
fuse: it will misdirect every agent that reads the file, and nothing will ever fail
because of it until something fails badly. But code is also not an authority on
*intent* — why a decision was made, what was rejected, what external fact forced it.
Nothing but prose carries that. This skill maintains a binding standards document that
lets comments carry intent without accumulating lies.

The canonical template lives at `assets/Commenting_Standards_Template.md` (~1,050 lines,
language-agnostic, agent instruction block baked in). It covers: the trust hierarchy
across code/tests/types/comments/docs/ledgers, the always-do list, a forbidden
anti-pattern catalog (restatement, session narration, temporal language, unearned
status claims, unenforced invariants, positional references, commented-out code, bare
markers, naked suppressions, decoration, aspirational comments, orphan references,
impossible-case comments), doc-comment contracts for public surfaces, test-comment
rules, epistemic status headers for prose artifacts (Binding / Descriptive /
Historical), a drift-control procedure, a sweep specification with severity splits, a
contract for the project-built enforcement checker, an adoption ratchet for existing
codebases, a PR checklist, agent read/write process rules, and a living incident log.

Most of these rules are mechanically checkable, which is what separates this document
from a style essay. The template does not ship a script — it specifies one (§7.4) and
instructs the onboarding agent to build it in the project's own language, wired into
whatever already runs in CI, with the current per-rule hit counts recorded as a
ratchet. The catalog and the checker are maintained together: a §3 entry without a
check is a rule enforced only by whoever happened to read the prose.

This skill has three modes. Pick by situation:

## Mode 1 — BOOTSTRAP (project has no standards doc)

Copy the template to `doc/COMMENTING_STANDARDS.md` (or the project's docs dir) and
complete its embedded agent instruction block — the 10 steps that turn the template
into a binding, project-specific document. Key judgment calls the template can't make
for you:

- **§0.3's marker vocabulary is a closed list, and closing it is the point.** Survey
  what the codebase already uses (`TODO`, `FIXME`, `XXX`, `NOTE`, `HACK`, ad-hoc
  inventions), pick the set worth keeping, and define the owner-or-ticket format. An
  open vocabulary is an unsearchable one.
- **§0.4's suppression inventory is the highest-severity section.** Find every
  directive in this stack that silences a lint rule, type check, coverage
  requirement, or security scanner, then grep for existing uses. A suppression is the
  only comment that changes program behavior; naked ones are findings on day one, not
  future work.
- **§0.5's prose inventory decides what agents are allowed to believe.** List every
  document an agent may read and assign Binding / Descriptive / Historical. The
  common mistake is marking a stale architecture description Binding because it lives
  in `CLAUDE.md` — that turns a decaying description into a rule to believe something
  false. Instruction files hold rules; descriptions go in Descriptive docs with a
  verification date.
- **Scope to the surface being built.** If the project extends an existing codebase,
  scope the documented surface (§0.2) to the new/touched code. A standard that
  demands doc comments on 4,000 pre-existing symbols is one nobody will apply.
- **Building the checker (§7.4) is the step that makes the rest real, and the step
  most likely to be skipped.** Look first for a home in a tool the project already
  runs in CI — custom lint rules beat a bespoke script, because the bespoke script is
  the one that quietly stops being run. Get the severity split right: `error` only for
  rules where a hit is essentially always a defect (bare markers, naked suppressions,
  commented-out code, orphan references, missing docs on the §0.2 surface), `warn` for
  the pattern-matching rules that will misfire on ordinary prose (temporal language,
  "always/never", restatement). Promoting warns to errors to look strict gets the
  whole checker disabled, after which nothing is enforced. Build the waiver mechanism
  in from the start, and test the checker — each rule needs a fixture that trips it
  and one that doesn't, or you ship a green tick that matches nothing.
- **The two checks worth real implementation effort** are orphan references (extract
  identifiers from comments, diff against the toolchain's symbol table — renames are
  the largest single source of confidently-wrong comments) and restatement (comment
  tokens that are a subset of the next code line's identifiers — the most common
  defect in agent-written code).
- **§0.6 is the contract with reality.** Record commands you have RUN and their
  current output, including per-rule hit counts. Those counts are the §7.5 adoption
  ratchet: on an existing codebase they start high and may only fall, which is what
  lets the standard be adopted without a wall of pre-existing violations blocking
  every merge. A "zero hits" claim that was never executed is exactly the kind of
  unverified prose this document exists to prevent.
- **The incident log (§10) starts honest.** New project → leave it explicitly empty.
  Never seed it with hypotheticals presented as history.
- **§3 is extensible, and extending it is the point.** The catalog (§3.1–3.13) is the
  floor. When a comment misleads someone, add it as a numbered entry, dated and
  traced to the incident. Cite entries by the project doc's numbers; never cite a §3
  number the doc doesn't contain.
- Delete every scaffold section that doesn't apply, renumber, and remove the
  instruction block when done.

## Mode 2 — AUDIT (standards exist; verify the prose against the code)

Evidence-based sweep. Comments are claims; bring each one back to the code.

1. **Run the checker in `--all` mode yourself, now, and record counts.** Compare
   against the §0.6 baseline — a rising count is a finding regardless of who caused
   it. If the project has no checker, that is finding #1 and the audit's main
   recommendation; run the §7.3 sweeps by hand this once to size the problem.
2. **Audit the checker before trusting it.** Break a rule deliberately in a scratch
   file, confirm the checker catches it, revert. Check what it excludes — an ignore
   list that has quietly grown to cover half the repo reports green over nothing.
   Count waivers and read a sample: waivers without justifications are §3.9 findings,
   and a rising waiver count is a rule being papered over rather than followed. Then
   check the §3 catalog against the implemented rules; entries with no check and no
   review-only note are rules that exist only on paper.
3. **Orphan-reference sweep — the highest-yield check.** Extract identifiers named in
   comments and docs, check each against the symbol table. Renames are the single
   largest source of confidently-wrong comments, and every hit is a comment that will
   send an agent to code that no longer exists.
4. **Verify the load-bearing claims, don't just count patterns.** Sample the comments
   that would change an implementation if believed — invariants ("always non-null"),
   ownership, concurrency ("not thread-safe"), and units — and check each against the
   code. For each, confirm the enforcement named in §2.6 actually exists; a claim
   whose "enforcing" test doesn't exist is severity-high, because it reads as proof.
5. **Suppression audit.** Every lint disable / type ignore / scanner waiver: does it
   carry a justification, and is the justification still true? File-level and
   project-level blanket suppressions get named explicitly — they hide unknown
   quantities of the codebase from tooling.
6. **Doc-comment coverage on the §0.2 surface**: symbols missing docs, plus docs
   missing the §4.1 items that bite hardest — errors, units, side effects,
   concurrency.
7. **Prose artifact status check**: does every §0.5 document carry its status header?
   Are Descriptive docs dated, and how stale? Spot-check the commands and paths in
   the README by running them — a README command that fails is a finding with the
   executed evidence attached.
8. **Test-comment check**: skipped/disabled tests without a TOMBSTONE (reason,
   ticket, re-enable condition), and any comment explaining away a weakened
   assertion — cross-file that one into the testing standards audit.
9. Write the audit as a dated report (in the ledger if the project uses one):
   findings ranked by severity, each with file:line and executed evidence, plus
   explicit credit for prose that is genuinely carrying its weight. Where a finding
   class was mechanical, the fix is a checker rule, not a list of files — say so, so
   the audit ratchets the tooling forward instead of being re-run by hand next time.

## Mode 3 — ENFORCE (ongoing, during every feature/review)

When new code lands (yours or another agent's):

- **Delete before defending.** The default verdict on a new comment is deletion. It
  survives only by buying *why*, *non-local context*, or *interface abstraction*
  (§1.4). Restatement is the most common defect in agent-written code by a wide
  margin, because it's the cheapest text a model can emit.
- **Screen every added comment against the §3 catalog before commit** — especially
  session narration ("as requested", "I've refactored this", "Step 1/2/3") and
  temporal language ("now", "new", "previously"), which are the clearest fingerprints
  of unreviewed agent output.
- **Every "must / always / never" gets enforcement first, then a comment pointing at
  it.** A comment claiming behavior no test asserts is a coverage gap wearing a
  disguise — raise it as a testing-standards finding, not a wording nit.
- **Walk the §7.1 blast radius before declaring a change done**: the function's docs,
  the file header, a grep for every renamed symbol, the five contract fields
  (signature, errors, units, defaults, concurrency), and any Descriptive doc the
  change invalidated. A change that leaves a contradicting comment behind is
  incomplete, tests green or not.
- **Fix contradictions in the current change, never as a follow-up.** Corrected if
  you know the truth, deleted if you don't. A comment you cannot verify is worse than
  no comment; leaving it because "it might mean something" hands the problem to an
  agent with less context than you have.
- **Never write status, progress, or confidence into a comment.** "Production-ready",
  "fully tested", "handles all edge cases" are the exact claims the adversarial-review
  discipline exists to strip. Status is earned by execution and lives in the ledger.
- **Run the checker on the change and read its warnings.** A `warn` scrolled past is a
  rule silently declined. Never widen a rule, waive in bulk, or rebaseline upward to
  get a green run — that is the unearned-status failure applied to tooling, and it
  ends with a check that reports success over nothing.
- **When a comment misleads you or another agent, log it in §10** with the
  anti-pattern that let it through — and extend the machine in the same change: the
  §3 entry, the §7.3 sweep, and the checker rule with its two fixtures. A rule added
  to prose alone is enforced by whoever happens to re-read the prose. The log is the
  document's immune memory; the checker is what gives it teeth for agents who never
  read this conversation.
