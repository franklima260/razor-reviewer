---
name: handoff
description: Write a short cold-start prompt for the next agent or session continuing project work. Use when the user says "make me a prompt for the next agent", "prepare the handoff", "we're switching agents/models", or "write up where we are for the next session".
---

# Handoff

Output a prompt in chat, **≤ 10 lines**. Don't create a handoff file.

```
You are <implementing|reviewing> <project>. <Read LEDGER.md first.>
State: <green counts @ commit, verified this session>; <uncommitted work, if any>.
Next: <task> — done when <check>. Don't start <gated work>.
Watch out: <the one trap most likely to bite>.
Rules: tick only what you ran; STOP and ask if plan and reality disagree.
```

- If the project has a `LEDGER.md`, refresh it once (per the `ledger` skill) and commit.
  If it doesn't, don't create one unless the user asks; the prompt carries the state.
- Point at files by path; don't restate them. Omit empty or generic lines.
