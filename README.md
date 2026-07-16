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
`.claude/agents/`. Skills are self-contained; the two standards templates live under the
respective skills' `assets/`.

## What you get

Installed, the plugin adds five skills and one agent (~800 always-on tokens; each skill
or agent pays its on-invoke cost only when it fires):

- **`adversarial-review`** skill and **`razor`** agent — the verification protocol.
- **`ledger`**, **`testing-standards`**, **`telemetry-standards`**, **`handoff`** skills —
  the project discipline and the standards it enforces.

Skills trigger automatically from their descriptions (e.g. asking Claude to "review the
agent's work" or "set up testing standards"); the reviewer is also invocable as the
`razor` subagent.

## License

MIT — see [LICENSE](LICENSE).
