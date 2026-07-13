---
name: dev-discover
description: Use when starting any feature, change, or project whose requirements are not yet nailed down — turns a vague idea into a user-approved, evidence-grounded spec before any code is written. Stage 1 of the unified dev pipeline; hands off to dev-plan.
provenance:
  synthesized: 2026-07-14
  sources:
    - superpowers:brainstorming @6.1.1 (workflow spine, HARD-GATE, one-question rule, spec self-review)
    - gstack:spec @1.60.1.0 (five-question intake, code-evidence rule, dedupe, scope lock)
  dropped: visual companion (niche, heavy), Codex quality gate (external dep), telemetry, GitHub issue filing (use gstack spec directly when you need issues)
---

# dev-discover — Requirements Discovery & Spec

Turn "I want X" into a written, approved spec. Nothing downstream (plan, review,
execution) can be better than the spec it started from.

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project,
or take any implementation action until a design has been presented and the
user has approved it. This gate has no exceptions for "simple" projects —
simple projects are where unexamined assumptions waste the most work.
</HARD-GATE>

## When to skip

Skip this skill only when a written spec already exists, or the change is a
single-file reversible edit with an obvious shape. If you skip, say so in one
line and go to dev-plan or straight to work.

## Workflow

Create a task for each step. Do them in order.

### 1. Ground yourself in evidence, not memory

Before asking the user anything, read the project: relevant files, docs,
recent commits. **The evidence rule (from gstack spec):** every technical
question you ask must be backed by at least one thing you actually read —
cite `path:line` when you reference existing behavior. If you searched and
found nothing, say "greenfield, nothing to cite" explicitly. Never ask the
user something the codebase already answers.

### 2. Five-question intake

Answer these five (ask the user only for the ones evidence can't answer):

1. **Who** is affected / who is this for?
2. **What happens today** (current behavior, with `path:line` evidence)?
3. **What should happen** instead?
4. **Why now** — what breaks or is lost by not doing this?
5. **How will we know it's done** — observable success criteria?

**Dedupe check:** before going deeper, search for prior art — existing
issues, TODOs, docs, or code that already solves part of this. Duplicated
work is the most expensive kind.

### 3. Clarify — one question per message

Ask clarifying questions **one at a time**, multiple-choice where possible.
Batching five questions gets you five shallow answers; one good question gets
a real decision. Stop when you can state the requirements without guessing.

If the request spans multiple subsystems, decompose FIRST: each sub-project
gets its own discover → plan → execute cycle. Don't interrogate details of
part 3 before part 1 is scoped.

### 4. Propose approaches

Present 2–3 approaches with trade-offs. Lead with your recommendation and
say why. Always include the minimal-viable option — the user must see what
"smallest thing that works" looks like even if you recommend more.

### 5. Lock scope

State explicitly, and get agreement on:
- **In scope** — what this delivers
- **Out of scope** — named things you are deliberately NOT doing
- **MVP cut line** — if time runs out, what ships first
- **Failure modes** — what can go wrong at runtime, and the rollback story

Scope agreed = scope locked. Later expansion is a new decision, not a drift.

### 6. Write the spec

Write the design to `docs/specs/YYYY-MM-DD-<topic>.md` and commit it.
Sections: Problem / Current behavior (with evidence) / Proposed design /
Alternatives considered / Out of scope / Success criteria.

### 7. Self-review, then user review

One inline pass before showing the user — fix and move on, no re-review loop:
- Placeholder scan: no "TBD", no "figure out later"
- Consistency: does any section contradict another?
- Ambiguity: could a stranger implement from this without asking you anything?
- Scope: does anything in the design exceed what was locked in step 5?

Then the user reviews the written spec. Wait for explicit approval.

## Exit

The ONLY skill you invoke after dev-discover is **dev-plan**, with the
approved spec as input. If the user wants a GitHub issue instead of local
execution, hand off to gstack `spec --file-only` rather than duplicating its
issue machinery here.

## Red flags

- Writing code "just to explore" before the gate → stop, that's implementation
- Asking a question you could answer with grep → read first
- "The user seems in a hurry, skip the spec" → a two-paragraph spec beats none; shrink the artifact, never the gate
