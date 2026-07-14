---
name: handoff
description: Generate a kickoff/handoff prompt for the next agent (or session) continuing multi-session project work. Use when the user says "make me a prompt for the next agent", "prepare the handoff", "we're switching agents/models", "write up where we are for the next session", or when a work phase completes and a different agent will pick up the next one. Produces a self-contained brief a cold agent can execute without access to this conversation.
---

# Agent Handoff

The next agent starts with **zero shared context** — it hasn't seen this conversation,
doesn't know the project's private vocabulary, and will happily re-derive (wrongly) anything
you leave implicit. A handoff prompt is a cold-start executable: everything load-bearing is
either stated in it or pointed to by exact file path.

The failure this prevents: each session re-explains the project ad hoc, each explanation
drifts, and by the fifth agent the "known facts" include three telephone-game mutations.
The ledger files are the antidote (see the `ledger` skill) — the handoff's main job is to
route the agent into them in the right order and frame the discipline expectations.

## Structure (use these sections, adapt lengths to the work)

1. **Role and accountability, first paragraph.** Whether the agent is implementing or
   reviewing, and that an adversarial reviewer will re-execute every claim it makes. Say
   plainly: the ledger records what ran, never what was intended to run. If prior agents
   have had ticks removed, say so — it calibrates honesty better than any rule list.

2. **Orientation reading list, ordered, with WHY per item.** The ledger TODO (status board
   first, then the active task's full spec), the blockers ledger, the design plan/spec,
   the testing standards doc, key evidence docs, and the specific source files the task
   touches. Every entry is an exact path. Keep it to what the task needs — ten documents
   nobody reads is worse than five that get read.

3. **Verified current state.** Green counts per configuration, at which commit, verified by
   whom and how. Decisions already made — rulings recorded verbatim, with their basis —
   so the agent doesn't relitigate them. Known open limitations with their blocker-ledger
   entries. Only state things here that YOU have verified or that carry a citation into a
   ledger; a handoff that launders unverified claims poisons the next session at birth.

4. **The task queue, strictly ordered, with gates.** Which task is active, what its
   Done-when is, what is explicitly NOT to be started (STOP gates, gated phases), and any
   parallelizable side work. Repeat the single highest-risk trap for the active task
   inline — the one thing most likely to ship a regression.

5. **Environment recipe.** Exact PATH/setup incantations, build and test commands, quirks
   that cost hours to rediscover (temp-dir workarounds, shell gotchas, flaky steps and
   their fixes), and where current green counts live. This section has the highest
   value-per-line of the whole handoff; hard-won environment knowledge otherwise
   evaporates between sessions.

6. **Discipline expectations.** One task one commit; commit message cites the task ID and
   the project's trailer conventions; tick only what was executed; harness before feature;
   blockers ledger + STOP when the plan and reality disagree; evidence runs after the
   final commit from the committed tree.

## Rules of construction

- **Pull numbers from the ledgers at generation time** — don't quote from memory. If the
  status board and your memory disagree, the ledgers win (and the discrepancy is worth a
  note of its own).
- **Quote specs verbatim where exactness matters** (fixture code, Done-when criteria,
  commandments). Paraphrase is where drift enters.
- **Name the anti-patterns by their war stories** when the project has them — "the last
  premature-'verified' cost a full review cycle" lands harder than "be honest".
- Write it to a file in the repo (e.g. `HANDOFF_<PHASE>.md`) so it survives the session,
  AND paste it in chat for immediate use. Refresh the file at each phase boundary rather
  than accreting stale generations.
- End with the first concrete action: "Start by reading the documents above, then confirm
  your integration plan (call sites verified by grep, fixtures written and failing) before
  writing code."
