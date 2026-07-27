# Question bank

Not a checklist to march through. These are the areas where unasked questions have
historically cost rework — use them to notice what the conversation has *not* touched, then
ask the specific version grounded in what you found in Phase 0. A generic question produces
a generic answer.

Regret ordering applies throughout: ask the question whose wrong answer is most expensive
and most likely, first.

---

## Software set (default)

### Scope and boundaries

- What is explicitly out of scope for this change, that a reasonable implementer might
  assume is in scope?
- What currently works that must keep working identically? Which of those has a test today?
- Is this one change or several? If it is several, which one ships first alone?

### Data

- What shapes does the real input actually take — empty, null, unicode, duplicated, absent,
  malformed, and this component's own output fed back in?
- Is existing stored data affected? Does it need migrating or backfilling, and can old and
  new coexist during the rollout?
- What is the volume, and at what volume does the chosen approach stop working?

### Interfaces and callers

- Who calls this today, and what breaks for each of them?
- Is this a surface anyone outside the project depends on? Can it change without warning?
- What contract does this expose, and what is deliberately *not* part of it?

### Failure

- When this fails, does it fail loudly or return something wrong quietly? Which one do we
  want?
- What is retried, what is not, and what happens on the third retry?
- What does partial failure leave behind — half-written records, orphaned files, a queue
  that never drains?
- Is there a state this can get into that requires human intervention to escape?

### Concurrency and ordering

- Can two of these run at once? On the same data?
- Does order matter, and what enforces it?
- What happens if this is interrupted halfway — a crash, a deploy, a killed session?

### Correctness

- Where is the math, and what is the reference we check it against?
- What is the tolerance on any number this produces, and who decided it?
- What is the one result that, if silently wrong, nobody would notice for a month?

### Testing and evidence

- What proves this works — the exact command, and what its output must say?
- What is the current green baseline, so a regression is visible?
- What can only be verified by a human looking at it, and who is that?

### Operations

- How do we know it is working in production without being told by a user?
- What is the rollback, and has anyone done it?
- Is this behind a flag? Who turns it off, and does turning it off restore the old path
  exactly?

### Cost and constraints

- What does this cost to run, and does the cost scale with something we do not control?
- What is the deadline, and what gets cut first if we run out of time?
- What are we choosing not to build now that this design must not make impossible later?

### Ownership

- Who else is working in this area right now?
- Which decisions here are the user's alone, and should become explicit STOP gates?

---

## General fallback

For decisions that are not software. Broader by necessity — use them only when the software
set clearly does not fit.

### The decision itself

- What are we actually deciding, stated as a single sentence?
- What is the alternative we are rejecting, and what would have to be true for it to win?
- Is this reversible? If we are wrong, when do we find out, and what does undoing it cost?

### Constraints

- What is fixed and not up for discussion — budget, time, equipment, a commitment already
  made to someone?
- What resource does this consume that is genuinely scarce?
- What has been tried before, and why did it not stick?

### Success and failure

- Six months out, how do we tell this was the right call?
- What is the most likely way this quietly fails — not the dramatic failure, the boring one?
- What is the early warning sign, and who would notice it?

### People and dependencies

- Who has to do something differently for this to work, and do they know?
- What does this depend on that we do not control?
- Who disagrees with this, and what is their strongest argument?
