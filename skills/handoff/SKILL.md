---
name: handoff
description: Write a one-page HANDOFF.md that lets the next agent or session start work immediately. Use when the user says "make me a prompt for the next agent", "prepare the handoff", "we're switching agents/models", "write up where we are for the next session", or when a phase ends and a different agent takes the next one. The output is a brief, not a report.
---

# Handoff

Write one file, `HANDOFF.md`, in the repo root. Paste the same text in chat.

The next agent has no memory of this session. Give it only the facts that it must
have to start work in the next five minutes. Do not explain the project. Do not
repeat what `LEDGER.md` says — point to it.

**Hard limit: 50 lines.** More than that is explanation, not a handoff.

## Template

Copy this. Delete each line that you cannot fill with a fact that you verified now.

````markdown
# HANDOFF — <project> — <date> — <commit sha>

**Do next:** <the one task, and its Done-when criterion.>

**State:** <X/Y tests pass>, at `<sha>`, from `<command you ran>`.
<One line for each open blocker: id — what it stops.>

**Read first** (max 5):
- `<exact path>` — <why, 5 words>
- `<exact path>` — <why, 5 words>

**Run:**
```
<setup>
<build>
<test>
```
<One line for each environment trap that costs more than 10 minutes.>

**Rules:** <up to 3 project rules. `LEDGER.md` holds the rest.>

**Do not:** <the gate, or the mistake most likely to break the work.>
````

## Rules

- Every number comes from a command that you ran now. Do not quote memory. If the
  ledger and your memory disagree, the ledger wins.
- Every path is exact. Do not write "the config file".
- Quote a Done-when criterion word for word. A paraphrase causes drift.
- Give no history, no rationale, no philosophy. Git and the ledger keep those.
- Overwrite the same file at each phase. Do not keep old handoffs.
