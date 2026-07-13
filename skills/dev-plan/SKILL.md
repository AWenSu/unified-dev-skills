---
name: dev-plan
description: Use when you have an approved spec or clear requirements for a multi-step task, before touching code — produces a plan file precise enough that an engineer with zero codebase context could execute it. Stage 2 of the unified dev pipeline; hands off to dev-plan-review (large tasks) or dev-execute.
provenance:
  synthesized: 2026-07-14
  sources:
    - superpowers:writing-plans @6.1.1 (plan artifact format, Interfaces block, No Placeholders, task right-sizing, type-consistency check)
    - ~/.claude/agents/planner.md (risk grading, sizing guide, success-criteria checklist)
  dropped: nothing significant — these two composed cleanly
---

# dev-plan — Implementation Planning

Write the plan for an engineer with zero context for this codebase and
questionable taste. Every ambiguity you leave is a decision they will make
wrong.

## When to skip

Single-file reversible changes don't need a plan file. Medium tasks
(multi-file, under half a day) may use the lightweight path: dispatch the
`planner` agent for a one-shot plan you review in-conversation, skip the
artifact. This skill's full artifact is for work that will be executed over
multiple sessions or by subagents.

## Sizing guide (from planner)

| Size | Phases | Steps | Plan artifact? |
|------|--------|-------|----------------|
| Small | 1 | 3–5 | No — just do it with a todo list |
| Medium | 2–3 | 8–15 | Optional — planner agent one-shot usually enough |
| Large | 3–5 | 15–30 | Required — this skill, full format |

If you're above 5 phases or 30 steps, the spec covers too much: go back and
split it into separate specs first.

## Workflow

### 1. Scope check, then map files

Multi-subsystem spec → separate plans. Then, before writing any task, map the
file structure the work will touch — this is where decomposition decisions
get locked in. Read enough real code to name exact paths.

### 2. Write the plan header

Save to `docs/plans/YYYY-MM-DD-<feature>.md` with this header:

```markdown
# Plan: <feature>

> **For agentic workers:** execute with dev-execute.

**Goal:** <one sentence>
**Spec:** <link to the dev-discover spec>
**Architecture:** <2-3 sentences, ASCII diagram if data flows>
**Global Constraints:** <exact values copied verbatim from the spec —
  limits, formats, naming, versions. Never paraphrase.>
**Success Criteria:** <checklist copied from the spec's "how will we know
  it's done" — the final gate checks these boxes>
```

### 3. Write tasks

Each task uses this template:

```markdown
## Task N: <name>   [Risk: Low|Med|High]

**Files:** exact paths, with line ranges for edits (`src/auth.py:123-145`)
**Depends on:** Task M / none
**Interfaces:**
  - Consumes: <exact names + signatures this task uses from earlier tasks>
  - Produces: <exact names + signatures later tasks will use>

- [ ] Write failing test for <specific behavior>
- [ ] Run it, verify it fails
- [ ] Implement minimal code to pass
- [ ] Run it, verify it passes
- [ ] Commit
```

Rules that make plans executable:

- **Each step is one action (2–5 minutes).** A step you can't verify in one
  command is two steps.
- **Interfaces block is mandatory** wherever tasks share symbols. A task's
  implementer sees only their own task — this block is how they learn what
  their neighbors named things.
- **Risk grade every task** (from planner): High = data migration,
  auth/security paths, irreversible operations. High-risk tasks get a named
  rollback step.
- **Right-sizing:** split only where a reviewer could meaningfully reject one
  task while approving its neighbor. Smaller is not automatically better.

### 4. No placeholders — banned phrases

A plan containing any of these is unfinished:
- "TBD" / "figure out during implementation"
- "Add appropriate error handling" (name the errors and what happens)
- "Similar to Task N" (repeat the code — implementers don't see Task N)
- References to types/functions no task defines

### 5. Self-review

One pass, fix and move on:
- Every spec requirement maps to a task (coverage)
- Placeholder scan (step 4 list)
- **Type-consistency across tasks:** `clearLayers()` in Task 3 but
  `clearFullLayers()` in Task 7 is a bug you're planting now
- Each phase leaves the system working (incremental, from planner)

## Exit

Offer exactly two paths:

1. **Large / risky plan** → **dev-plan-review** first (recommended when: >8
   files, new architecture, production data, or you have any Taste-level
   doubt about the approach)
2. **Plan is straightforward** → **dev-execute** directly

Return only the plan. Do not begin implementation — that's dev-execute's job,
after any review.
