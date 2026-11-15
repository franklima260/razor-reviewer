---
name: handoff
description: Write a short cold-start prompt for the next agent or session continuing project work. Use when the user says "make me a prompt for the next agent", "prepare the handoff", "we're switching agents/models", or "write up where we are for the next session".
---

# Handoff

The next agent has none of this conversation. Give it the minimum to act correctly, in
chat, **under ~15 lines**. Don't write a handoff file: if the project has a `LEDGER.md`
(see the `ledger` skill), update it and commit; it is the persistent handoff. If it has no
ledger, the prompt carries the state itself.

## Contents

```
You are <implementing|reviewing> <project>. Read LEDGER.md first.
State: <green counts @ commit, verified by you just now>; <uncommitted/WIP, if any>.
Next: <task> — done when <check>. Don't start <gated work>.
Watch out: <the one trap most likely to cause a regression on this task>.
Context not in the ledger: <only what the next agent can't recover from files/git>.
Rules: tick only what you ran; STOP and file a blocker if plan and reality disagree.
```

- Update the ledger before writing the prompt; state only what you verified this session.
- Point to files by path; don't restate what they say.
- Omit any line that would be empty or generic.
