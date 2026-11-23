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
| `skills/ledger` | One `LEDGER.md`, 100 lines maximum: mission, invariants, up to five rules, a task table with Done-when and evidence, a blocker table. Tick rules, STOP gates, bootstrap procedure. |
| `skills/testing-standards` | Instantiate / audit / enforce a binding TESTING_STANDARDS.md, capped at 60 lines: eight rules, ten forbidden patterns, the greps that find them, and mutation checking. |
| `skills/grill-me` | Spec-first interrogation before any code exists: regret-ordered interview rounds alternating with attack passes run by a fresh agent that sees only the spec, looping until two consecutive attacks come back empty. Writes a dated spec to `docs/specs/`. Template included. |
| `skills/handoff` | One `HANDOFF.md`, 50 lines maximum: the next task, verified state, up to five files to read, the commands to run, the traps. Overwritten each phase. |
| `agents/razor.md` | The reviewer as an invocable subagent. |
| `tools/check-docs` | Deterministic checks on the ledger (100 lines), handoff (50), testing standards (60) and specs (120): size caps, a command in every Evidence cell, no success claim without a command, no padding phrases, no prose blocks. Counts and greps only, so it costs no tokens. |
| `tools/githooks/pre-commit` | Runs those checks on the staged content, refuses a **new** status document (`PROGRESS.md`, `AUDIT_REPORT.md`, …) so a cap cannot be dodged by opening a second file, and makes a new spec supersede the old one instead of accumulating. `.githooks/allow-docs` lists deliberate exceptions. |
| `tools/githooks/commit-msg` | One task, one commit: a commit that changes the ledger must name the task (`T2.3`). |
| `tools/install-checks` | Puts all of it into a project (`core.hooksPath=.githooks`). One command, every agent. |
| `tools/ensure-hooks` | Arms those hooks at session start, because `core.hooksPath` does not survive a clone. Register it once per agent on `SessionStart`; it is silent. |

## The workflow in one paragraph

Grill the spec first (`grill-me`) — interview and independent attack, cycling until the
attacks go quiet, producing a dated spec whose assumptions all carry a stated blast radius.
Bootstrap a project from it with the `ledger` skill (one `LEDGER.md`: mission, invariants,
rules, baseline, tasks with Done-when) and `testing-standards`. Implementing agents work
one task per commit, harness-before-feature, ticking
only what they executed. At every milestone, `razor` (or the `adversarial-review` skill
inline) re-executes every claim, attacks with hostile repros, removes unearned ticks, and
files blocker rows that name the repro tests. Human decisions sit behind explicit STOP
gates, with the ruling recorded against the numbers it was made on. Phase boundaries
produce a one-page `handoff` so the next agent starts hot.

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

Installed, the plugin adds five skills and one agent (each skill or agent pays its
on-invoke cost only when it fires; the always-on cost is the descriptions alone):

- **`grill-me`** skill — the spec-hardening interrogation that runs before code exists.
- **`adversarial-review`** skill and **`razor`** agent — the verification protocol.
- **`ledger`**, **`handoff`**, **`testing-standards`** skills — the project discipline
  and the standards it enforces. The ledger and the handoff are capped at 100 and 50
  lines, and `tools/` enforces those caps on commit for every agent.

Most skills trigger automatically from their descriptions (e.g. asking Claude to "review the
agent's work" or "set up testing standards"); the reviewer is also invocable as the
`razor` subagent. `grill-me` is the exception — it never starts on its own, though Claude
may mention once that a grilling would pay before substantial work with no spec.

## License

MIT — see [LICENSE](LICENSE).
