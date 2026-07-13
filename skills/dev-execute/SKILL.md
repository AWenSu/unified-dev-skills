---
name: dev-execute
description: Use when a reviewed plan is ready to implement — orchestrates subagent-driven execution with per-task review and a crash-safe progress ledger, or falls back to inline execution when subagents aren't available. Stage 4 of the unified dev pipeline; hands off to dev-finish.
provenance:
  synthesized: 2026-07-14
  sources:
    - superpowers:subagent-driven-development @6.1.1 (orchestration spine, ledger, status protocol, model matrix, two-verdict review, BASE rule)
    - superpowers:executing-plans @6.1.1 (inline fallback, stop conditions, branch rule)
    - planning-with-files @3.5.0 (filesystem-as-memory: findings/progress files, 2-action rule)
  dropped: task-brief/review-package helper scripts (plugin-internal paths; inlined their intent as prompt rules)
---

# dev-execute — Plan Execution

Two modes, one decision at the top:

```
Subagents available AND tasks mostly independent?
├─ yes → ORCHESTRATED mode (default; better context isolation, reviewed per task)
└─ no  → INLINE mode (sequential, single context)
Tasks tightly coupled with shared evolving state? → INLINE, or go re-split the plan
```

**Universal rules (both modes):**
- Never start on main/master without explicit user consent — branch first.
- **Continuous execution:** do not pause between tasks to ask "should I
  continue?" — checkpoint questions burn the user's time. Stop only at the
  plan's end or on a genuine blocker.
- **Filesystem is memory** (from planning-with-files): keep `progress.md`
  (what happened) and `findings.md` (what was learned) next to the plan.
  After every ~2 exploratory operations, write findings down. Context windows
  get compacted; files don't. If the planning-with-files plugin is installed,
  its hooks enforce this automatically — don't duplicate, just comply.

## ORCHESTRATED mode

### Per task loop

1. **Extract the task brief** — the task's own text plus the plan header
   (Goal, Global Constraints) plus the Interfaces blocks of completed tasks
   it consumes. Pass briefs as *file paths*, never pasted into the prompt —
   pasted history bloats every downstream dispatch.
2. **Dispatch a fresh implementer subagent** with the brief. It implements,
   tests, commits, and reports status.
3. **Review the diff** with a fresh reviewer subagent. Diff base is the
   commit before the task started — **never `HEAD~1`**, which silently drops
   all but the last commit of a multi-commit task. The reviewer returns TWO
   verdicts: (a) spec compliance — does it do what the task says; (b) code
   quality. Both must pass.
4. **Fix loop:** Critical/Important findings → one fix subagent → re-review.
5. **Ledger append** (see below), then next task.

### Implementer status protocol

| Status | Meaning | Controller action |
|--------|---------|-------------------|
| DONE | Complete, tests pass | Review, proceed |
| DONE_WITH_CONCERNS | Complete but flagged | Review with the concerns in the reviewer brief |
| NEEDS_CONTEXT | Missing information | Answer from plan/spec; if absent there, that's a plan bug — fix the plan |
| BLOCKED | Cannot proceed | Never re-dispatch the same prompt to the same model unchanged; diagnose first |

Anything a subagent reports that you cannot verify from artifacts (diff,
test output) — resolve personally before marking the task complete.
"Agent said success" is not evidence; the diff is.

### Model selection (cost routing)

- **Cheapest tier:** transcription-style tasks — the plan text already
  contains the exact code to write
- **Mid tier (floor for all reviewers):** ordinary implementation
- **Most capable:** final whole-branch review, gnarly debugging
- Always specify the model explicitly; an omitted model silently inherits
  your session's (expensive) one. Turn count beats token price — a cheap
  model that takes 4 retries costs more than a good one that takes 1.

### Progress ledger — crash safety

Append one line per completed task to `.dev-pipeline/progress.md`:

```
Task 3: auth middleware — DONE, reviewed (2 findings fixed), commit a1b2c3d
```

**On session start or after compaction: read the ledger FIRST.** Trust the
ledger and `git log` over your recollection — controllers that lost their
place have re-executed entire completed task sequences, the single most
expensive failure mode observed. The ledger is why this workflow survives
a crashed session.

### Finish

After all tasks: dispatch one final whole-branch reviewer (most capable
model, diff from merge-base) → one fix pass for its findings → **dev-finish**.

## INLINE mode

1. Read the whole plan critically. Concerns → raise them BEFORE starting,
   not at task 7.
2. Create a todo per task. Execute in order: follow each step exactly, run
   each verification, mark complete. Update the ledger the same as
   orchestrated mode — inline sessions crash too.
3. Stop and ask rather than guess when: blocked, the plan has a critical
   gap, an instruction is ambiguous, or a verification keeps failing.
4. After all tasks → **dev-finish**.

## Red flags

- Dispatching parallel implementers on overlapping files → conflicts; serialize them
- Letting the implementer's self-review substitute for review → independent reviewer, always
- "I remember where I was" after compaction → no you don't; read the ledger
- Skipping a task's failing-test step because "the code is obviously right" → the test IS the task
