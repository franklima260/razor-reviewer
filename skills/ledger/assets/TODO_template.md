# <PROJECT / FEATURE NAME> — Implementation TODO

<!-- This file is the single source of truth for state and spec. It records what RAN,
     never what was intended to run. See the razor-reviewer plugin's `ledger` skill for
     the rules that govern it. Delete this comment after bootstrap. -->

## MISSION (memorize this)

<WHAT THE WORK MUST ACHIEVE — 2-4 sentences.>

Everything else must keep working **exactly as before**:

- <INVARIANT 1 — e.g., "default output is byte-identical to stock until phase N ships">
- <INVARIANT 2 — e.g., "no new user-visible flags or syntax, ever">
- <INVARIANT 3>

## THE COMMANDMENTS (violating any of these = the task is wrong, even if tests pass)

1. **Never invent APIs.** These do NOT exist — do not call or create them:
   `<PLAUSIBLE_HALLUCINATION_1>`, `<PLAUSIBLE_HALLUCINATION_2>`. The real names:
   `<REAL_API>` (`<file:line>`).
2. **Never edit `<FILE>` for `<PURPOSE>`.** That logic lives in `<REAL_LOCATION>`.
3. **Never compare floats with `==` in <DOMAIN> logic.** Use the tolerances in <SPEC DOC>.
4. **Never regenerate golden/expected files outside task <TICKET>.** Golden regeneration is
   a human-gated event that happens only when the known-defect list is empty.
5. <ADD MORE — each commandment should trace to a real or confidently predicted failure.>

## PHASE 0 — Baseline (do this before anything else)

- [ ] **T0.1 — Record the green baseline.**
  - Build the project the same way CI does. Run the full test suite.
  - Save pass/fail counts to `<PROJECT>_BASELINE.txt` and commit it.
  - **Done when:** build succeeds; baseline file committed.
  - **Trap:** do not "fix" pre-existing failures; record them. If the tree doesn't build,
    STOP and file a blocker.

## PHASE 1 — <NAME> <(add gates: "GATED: do not start without X")>

**PHASE 1 STATUS BOARD (update whenever a task changes state):**

| Item | State |
|---|---|
| T1.1 <short name> | ⬜ **← START HERE** |
| T1.2 <short name> | ⬜ |

**Working recipe for this environment (keep current — hard-won knowledge evaporates):**
`<exact PATH/setup>` · build: `<command>` · test: `<command>` · current green counts:
`<X/X (config A), Y/Y (config B)>` as of `<commit>`.

- [ ] **T1.1 — <TASK NAME>.** *(spec ref: <PLAN §>)*
  - <Spec details. Real file paths. Real API names.>
  - **Done when:** <observable, executable criteria — commands and expected outcomes>.
  - **Trap:** <the mistake an agent will plausibly make here>.

## WHEN YOU ARE UNSURE

Ask yourself, in order:
1. Does the plan/spec answer it? (Search it before deciding anything.)
2. Does existing stock behavior answer it? (When in doubt, match what exists.)
3. Neither → it's a blocker. Write it to `<PROJECT>_BLOCKERS.md` and STOP. A wrong guess
   costs more than a paused session.
