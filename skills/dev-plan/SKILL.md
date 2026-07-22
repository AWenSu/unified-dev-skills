---
name: dev-plan
description: Use when you have an approved spec or clear requirements for a multi-step task, before touching code — produces a plan file precise enough that an engineer with zero codebase context could execute it. Stage 2 of the unified dev pipeline; hands off to dev-plan-review (large tasks) or dev-execute.
provenance:
  synthesized: 2026-07-14
  sources:
    - superpowers:writing-plans @6.1.1 (plan artifact format, Interfaces block, No Placeholders, task right-sizing, type-consistency check)
    - ~/.claude/agents/planner.md (risk grading, sizing guide, success-criteria checklist)
    - mattpocock/skills to-tickets + tdd @1.2.0 (Delivers behavior line, tests-only-at-confirmed-seams, expand–contract wide-refactor sequencing; added 2026-07-22)
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

Read `CONTEXT.md` at the repo root if it exists — the project's domain
glossary. Task names, new symbols, and Interfaces blocks must use its
vocabulary; a plan that renames a glossary concept is planting an
inconsistency.

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

**Delivers:** <the end-to-end behavior this task makes work, stated so it
  can be verified without reading the code — the task's source of truth>
**Files:** exact paths, with line ranges for edits (`src/auth.py:123-145`) —
  location hints, not truth; see the staleness rule below
**Depends on:** Task M / none
**Skills:** <domain skills the implementer must invoke, or "none" — e.g. a
  UI-design skill for visual tasks, a platform skill (Cloudflare, MCP
  builder) for platform idioms. Name them now so execution doesn't have to
  rediscover them.>
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

- **Delivers is the truth; Files are hints (from mattpocock to-tickets).**
  Behavior statements don't go stale; line numbers do. Write **Delivers**
  from the user's perspective, precise enough to verify. Paths and line
  ranges stay — they save the implementer a search — but if reality
  disagrees with them at execution time, the implementer relocates by the
  Delivers line instead of editing whatever now sits at those lines.
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
- **Tests only at confirmed seams.** Every "write failing test" step names
  which seam from the spec's **Test seams** section it exercises. Tests
  verify behavior through that interface — never internals, private methods,
  or side channels. A task that needs a seam the spec didn't confirm →
  stop, take it back to the spec (it's a design change, not a planning
  call). Spec has no Test seams section (pre-seam spec) → propose the seam
  list to the user before writing tasks.

### 3b. Wide refactors — expand–contract, not slices (from mattpocock to-tickets)

A **wide refactor** is one mechanical change — rename a column, retype a
shared symbol — whose blast radius fans across the codebase, so no vertical
task can land green on its own. Don't force it into the normal task shape;
sequence it as **expand–contract**:

1. **Expand** (one task): add the new form beside the old — nothing breaks.
2. **Migrate** (one task per batch): move call sites over in batches sized
   by blast radius (per package, per directory). Every batch depends on the
   expand task; each leaves CI green because the old form still exists.
3. **Contract** (one task, depends on every migrate batch): delete the old
   form once zero callers remain.

If even the batches can't stay green alone, keep the same sequence on a
shared integration branch, all batches feeding one final integrate-and-verify
task — green is promised only there, and the plan says so explicitly.

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
