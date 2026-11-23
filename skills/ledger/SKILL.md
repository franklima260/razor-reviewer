---
name: ledger
description: Keep one lean LEDGER.md that holds a project's mission, rules, task table and blockers for multi-agent or multi-session work. Use when starting substantial work ("set up the project files", "make a plan we can execute across sessions"), when work spans agents or sessions, when the user mentions status boards, ticks or blockers, or when the tracking has drifted from reality. Consult it before you tick, untick or restructure any ledger entry.
---

# Ledger

One file, `LEDGER.md`, in the repo root. It is the only status document.

Its law: **the ledger records what ran, not what was intended to run.**

Do not create TODO, BLOCKERS, evidence, status, progress or summary documents
beside it. If a project already has split ledger files, leave them as they are
and do not convert them unless the user asks.

**Hard limit: 100 lines.** More than that is narrative, and narrative must go.

## The file

This is the whole structure. Add nothing to it.

````markdown
# <project> — Ledger

**Mission:** <1-2 sentences: what the work must achieve.>
**Must not change:** <2-3 invariants. Agents sacrifice these for the goal unless you name them.>

**Rules** (max 5; write each one from a real failure, not from general advice)
1. <e.g. `<fake_api>` does not exist. Use `<real_api>` (`<file:line>`).>
2. <e.g. Never regenerate golden files. That is a STOP gate.>

**Environment:** build `<cmd>` · test `<cmd>` · green `<X/Y>` at `<sha>`

| ID | Task | Done when | State | Evidence |
|---|---|---|---|---|
| T1 | <task> | <executable criterion> | done | `<cmd>` → 48/48, `<sha>` |
| T2 | <task> | <executable criterion> | NEXT | |
| T3 | <task> | <executable criterion> | STOP: human picks A or B | |

**Blockers**

| ID | Problem | Status | Repro / fix |
|---|---|---|---|
| B1 | <one line: symptom, then cause> | OPEN <date> | `<test name or command>` |
````

## Rules of use

- **One row for each task.** The Done-when cell is the spec. If the spec needs more
  than two lines, put it in the code or an issue and link it from the cell.
- **Evidence or no tick.** Write `done` only after you ran the Done-when command in
  this session, from the committed tree. The Evidence cell holds that command, its
  result and the commit. An empty Evidence cell means the task is not done.
- **A reviewer who cannot repeat the result** sets the State back to `NEXT` and writes
  the criterion that failed in the Evidence cell. An honest `NEXT` is progress. A
  false `done` is damage.
- **One task, one commit.** The commit message gives the task ID. Commit the ledger
  change together with the work it describes.
- **STOP gates.** Write `STOP: <decision> — human` in the State cell. Do not go past
  it, however obvious the answer looks. Record the answer in the same cell, with the
  date and the numbers it was made against.
- **Blockers.** When the plan and the code disagree, and neither the plan nor the
  existing behaviour answers it, add a blocker row and stop. Keep the repro as a test
  in the repo; the row holds the test name, not the test code.
- **One document.** Never open a second status file. A cap you cannot meet means you
  must delete rows, not start `PROGRESS.md`. The commit hook refuses the usual names; if
  one is genuinely needed, add it to `.githooks/allow-docs` and say why in the commit.
- **Do not restate the work in chat.** The row is the record. Give the task ID, the
  command and its result. No summary essay, no walk through the diff.
- **Never pass `--no-verify`.** The pre-commit check is the limit, not advice.
- **Delete, do not accumulate.** When a phase ends, delete its `done` rows. Git keeps
  the history. The ledger shows the present.

## Bootstrap

1. Write Mission and the invariants with the user.
2. Run the test suite. Put the counts in the Environment line. If the tree is red or
   does not build, that is blocker B1. Stop there.
3. Write the tasks as rows. Each Done-when must be a command with an expected result.
4. Write the rules from the hazards that you found in step 2 and 3.
5. Install the checks: `sh <plugin>/tools/install-checks .` They refuse a commit that
   breaks a size limit, ticks a task with no command, opens a second status document, or
   changes the ledger without naming a task. They cost nothing to run, and they are the
   same checks for every agent.
6. Commit `LEDGER.md`, `.githooks/` and `.gitattributes`.

## Audit each review

- A `done` row with an empty Evidence cell, or evidence dated before its commit.
- A blocker that says OPEN while a task says it is fixed.
- "verified", "green" or "passes" with no command next to it.
- A ledger change in a commit that contains no work, or work with no ledger change.
- The Environment line still shows counts from an older commit.
