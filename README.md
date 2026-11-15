# razor-reviewer

A [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) plugin for
adversarial-review, ledger-driven, testing-first development on AI-assisted
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
| `skills/ledger` | One lean `LEDGER.md`: goal, invariants, run recipe, one-line tasks with evidence, blockers, human decisions. Current state only — git holds the history. |
| `skills/testing-standards` | Instantiate / audit / enforce a binding TESTING_STANDARDS.md. Vendors the full language-agnostic template (anti-pattern catalog, contract testing, PR checklist, incident log). |
| `skills/telemetry-standards` | Same three modes for structured logging + metrics standards (single chokepoint, outcome bucketing, PII rules, diagnostic recipes). Template vendored. |
| `skills/commenting-standards` | Same three modes for comments and docs. Treats prose as an unverified claim: trust hierarchy, anti-pattern catalog, doc-comment contracts, epistemic status headers for READMEs/instruction files/ledgers. Specifies an enforcement checker the onboarding agent builds in the project's own stack, with a severity split and an adoption ratchet. Template vendored. |
| `skills/grill-me` | Spec-first interrogation before any code exists: regret-ordered interview rounds alternating with attack passes run by a fresh agent that sees only the spec, looping until two consecutive attacks come back empty. Writes a dated spec to `docs/specs/`. Template, question bank, and attack briefs included. |
| `skills/handoff` | A ~15-line cold-start prompt for the next agent/session, pointing at the ledger. |
| `agents/razor.md` | The reviewer as an invocable subagent. |

## The workflow in one paragraph

Grill the spec first (`grill-me`) — interview and independent attack, cycling until the
attacks go quiet, producing a dated spec whose assumptions all carry a stated blast radius.
Bootstrap a project from it with the `ledger` skill (goal, invariants, baseline, tasks with
Done-when) and the three standards skills (testing, telemetry, commenting). Implementing agents work
one task per commit, harness-before-feature, ticking only what they executed. At every
milestone, `razor` (or the `adversarial-review` skill inline) re-executes every claim,
attacks with hostile repros, removes unearned ticks, and files blockers pointing at
committed repros. Human decisions sit behind explicit STOP gates with
rulings recorded against the numbers they were made on. Agent switches get a short `handoff`
prompt.

## Install

Requires Claude Code with plugin support (`claude plugin --help` should list the
subcommands).

### From GitHub (recommended)

```bash
claude plugin marketplace add franklima260/razor-reviewer
claude plugin install razor-reviewer@ai-vibe-coding
```

Restart Claude Code afterward so the new skills and the `razor` agent load. Verify with
`claude plugin list` and `claude plugin details razor-reviewer@ai-vibe-coding`.

### From a local clone

```bash
git clone https://github.com/franklima260/razor-reviewer.git
claude plugin marketplace add ./razor-reviewer
claude plugin install razor-reviewer@ai-vibe-coding
```

### Manual (no marketplace)

Copy the `skills/*` folders into a project's `.claude/skills/` and `agents/razor.md` into
`.claude/agents/`. Skills are self-contained; the three standards templates live under the
respective skills' `assets/`.

## What you get

Installed, the plugin adds seven skills and one agent (each skill or agent pays its
on-invoke cost only when it fires; the always-on cost is the descriptions alone):

- **`grill-me`** skill — the spec-hardening interrogation that runs before code exists.
- **`adversarial-review`** skill and **`razor`** agent — the verification protocol.
- **`ledger`**, **`testing-standards`**, **`telemetry-standards`**,
  **`commenting-standards`**, **`handoff`** skills — the project discipline and the
  standards it enforces.

Most skills trigger automatically from their descriptions (e.g. asking Claude to "review the
agent's work" or "set up testing standards"); the reviewer is also invocable as the
`razor` subagent. `grill-me` is the exception — it never starts on its own, though Claude
may mention once that a grilling would pay before substantial work with no spec.

## License

MIT — see [LICENSE](LICENSE).
