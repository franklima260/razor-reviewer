---
name: ledger
description: Create or update a short LEDGER.md status file so another agent or session can pick up the work. Use only when the user asks to set up or update project status/tracking, or when the handoff skill runs. Do not use to log routine progress, task completions, or reviews during a session.
---

# Ledger

`LEDGER.md` at the repo root is a **snapshot for the next agent**, not a running log.

## When to write it

- When the user asks, or at a handoff / end of session — **once**, not after each task.
- Never during routine work. Progress goes in commits and chat; git is the history.
- Exception: untick a task you found broken (see below).

## Format (whole file ≤ ~40 lines)

```markdown
# <Project> ledger

**Goal:** <one sentence>
**Invariants:** <what must not change — 1-3 bullets>
**Run:** build `<cmd>` · test `<cmd>` · green: <counts> @ <commit>

## Tasks
- [x] T1 <name> — <commit>
- [ ] T2 <name> — done when: <check> ← NEXT
- [ ] T3 <name> — STOP: needs <human> ruling on <X>

## Open
- <blocker, open question, or human decision the next agent must know — one line each>
```

## Rules

- **Tick only what you ran.** A tick means you executed its done-when check against the
  committed tree. The how goes in the commit message, not here.
- **Untick** a task whose check fails: `- [ ] T1 <name> — broken: <which check>`.
- **STOP** when plan and reality disagree and nothing settles it: tell the user in chat. It
  only goes under **Open** if the session ends with it unresolved.
- **Replace, don't append.** Rewrite lines to current state; delete resolved items and
  collapse finished phases to one line. No dates, history, review notes, or narrative.
