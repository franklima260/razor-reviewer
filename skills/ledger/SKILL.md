---
name: ledger
description: Keep a single lean LEDGER.md as the shared status file for work that spans multiple agents or sessions — goal, invariants, run recipe, one-line tasks, open blockers, human decisions. Use when starting multi-session work ("set up a ledger", "track this across sessions"), when ticking/unticking tasks or filing a blocker, or when a project's tracking has drifted from reality.
---

# Ledger

One file, `LEDGER.md` at the repo root, holding **current state only**. Git is the history:
detail lives in commit messages, not in the ledger. If a line doesn't help the next agent
act, it doesn't belong.

The one law: **a tick records what ran, never what was intended to run.**

## Format

```markdown
# <Project> ledger

**Goal:** <one sentence>
**Invariants:** <what must not change / must never be done — a few short bullets max>
**Run:** build `<cmd>` · test `<cmd>` · green: <counts> @ <commit>
**Gotchas:** <environment traps that cost time to rediscover; omit if none>

## Tasks
- [x] T1 <name> — <commit> (`<verify cmd>` → <result>)
- [ ] T2 <name> — done when: <observable check> ← NEXT
- [ ] T3 <name> — STOP: needs human ruling on <X>

## Blockers
- B1 (T2) OPEN <date>: <one line: what contradicts the plan>. Repro: `<path or cmd>`

## Decisions
- <date> <decision> — <who>, basis: <the number/fact it rested on>
```

## Rules

- **Tick** only after running the Done-when check yourself, from the committed tree. The
  tick line carries the commit and the verify command with its observed result — nothing
  more. Explanation goes in the commit message.
- **Untick** (reviewer or anyone) a task whose check fails; replace the evidence with
  `UNTICKED <date>: <which check failed>`.
- **Blockers:** when plan and reality disagree and neither the plan nor existing behavior
  settles it, file a one-line blocker and STOP. Put repros in the repo (a skipped/xfail
  test or a script), and reference the path — don't paste them into the ledger.
  Delete a blocker line once it is resolved; the fixing commit records how.
- **STOP gates:** human-owned decisions (architecture go/no-go, golden-file
  regeneration, accepting a missed target, new user-visible surface, spending money) are
  marked `STOP` on the task. Record the ruling as one line under Decisions, with the
  basis, so later evidence that changes the basis is visibly a reason to re-ask.
- **Prune:** finished phases collapse to one line (`Phase 1 done @ <commit>`). Stay under
  ~60 lines; if you're over, you're writing history, not state.
- Commit ledger edits with the work they describe. A ledger ahead of the tree is lying.

## Bootstrap

1. Agree goal and invariants with the user (push for invariants; agents sacrifice them).
2. Run the build and tests first; record the green baseline in **Run**. If it's red, that
   is blocker B1, not something to fix in passing.
3. List tasks with a Done-when each; put a spike with a STOP gate first if a later task
   depends on an unproven approach.
4. Commit.

## Smells

- A tick without a commit and verify command.
- `← NEXT` missing or on a ticked task.
- A blocker referenced in a commit as fixed but still listed.
- A decision with no basis recorded.
