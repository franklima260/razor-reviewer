# razor-reviewer

Adversarial-review, ledger-driven, testing-first development workflow for AI-assisted
("vibe code") projects — formalized from a real multi-phase project where sixteen
adversarial review rounds repeatedly overturned "100% done" claims and caught bugs that
green test suites had certified.

**Core thesis:** in agent-written codebases, the tests are the only part of the system that
gets independently re-executed, so (1) test quality is product quality, and (2) every
"done" claim is a hypothesis until someone re-runs it. This plugin packages both halves:
the verification discipline and the standards it enforces.

## Contents

| Piece | What it does |
|---|---|
| `skills/adversarial-review` | The Razor protocol: claim inventory → evidence binding → re-execution → executable repros → ranked findings → tick discipline. Ships a war-stories catalog of ten generalized real incidents. |
| `skills/ledger` | Three-file project discipline (TODO + status board + commandments, BLOCKERS deviation ledger, evidence docs), tick rules, STOP gates, bootstrap procedure. Templates included. |
| `skills/testing-standards` | Instantiate / audit / enforce a binding TESTING_STANDARDS.md. Vendors the full language-agnostic template (anti-pattern catalog, contract testing, PR checklist, incident log). |
| `skills/telemetry-standards` | Same three modes for structured logging + metrics standards (single chokepoint, outcome bucketing, PII rules, diagnostic recipes). Template vendored. |
| `skills/handoff` | Cold-start kickoff prompts for the next agent/session: orientation list, verified state, task queue with gates, environment recipes, discipline expectations. |
| `agents/razor.md` | The reviewer as an invocable subagent. |

## The workflow in one paragraph

Bootstrap a project with the `ledger` skill (mission, commandments, baseline, phased tasks
with Done-when) and the two standards skills (testing, telemetry). Implementing agents work
one task per commit, harness-before-feature, ticking only what they executed. At every
milestone, `razor` (or the `adversarial-review` skill inline) re-executes every claim,
attacks with hostile repros, removes unearned ticks, and files findings with verbatim
fixtures in the blockers ledger. Human decisions sit behind explicit STOP gates with
rulings recorded verbatim against the numbers they were made on. Phase boundaries produce
`handoff` prompts so the next agent starts hot.

## Install

Add this directory as a local plugin (marketplace entry or `--plugin-dir`), or copy the
`skills/*` folders into `.claude/skills/` and `agents/razor.md` into `.claude/agents/` of
a project. Skills are self-contained; the two standards templates live under the
respective skills' `assets/`.
