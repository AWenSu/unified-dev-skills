---
name: dev-plan-review
description: Use after dev-plan produces a plan for large or risky work — runs multi-lens automated review (business, engineering, plus design/DX when in scope) with adversarial verification, auto-deciding routine choices and escalating only genuine judgment calls. Stage 3 of the unified dev pipeline; hands off to dev-execute.
provenance:
  synthesized: 2026-07-14
  sources:
    - gstack:autoplan @1.60.1.0 (6 decision principles, Mechanical/Taste/UserChallenge taxonomy, sequential lens order, scope-conditional lenses, exit gate)
    - doubt-driven-development (fresh-context adversarial refutation)
  dropped: Codex dual-voice (external CLI dep), telemetry, restore points, comparison-board mockups. For the full heavyweight version with dual-model consensus, use gstack /autoplan directly.
---

# dev-plan-review — Multi-Lens Plan Review

Rough plan in, reviewed plan out. Auto-decide only the Mechanical — findings
with one defensible answer. Every genuine judgment call goes to the user as
its own separate, fully-detailed question (Step 5); never batch them into
one summary question.

**Weight check:** this is the lean single-model version. For very large plans
(>15 files, new product surface) where gstack is installed, prefer `/autoplan`
— its dual-model consensus and mockup generation earn their cost there.

## Step 0: Premises — the ONE mandatory user question

Before any review, confirm premises with the user in a single question:
"This plan assumes <X, Y, Z>. Correct?" Wrong premises make every downstream
finding worthless. This is the only question that must always be asked.

## Step 1: Detect scope → select lenses

Always run: **CEO lens**, **Eng lens**.
Conditionally add (2+ keyword hits in the plan):
- **Design lens** — view/rendering/UI/component/screen vocabulary
- **DX lens** — API/CLI/SDK/docs/MCP vocabulary (product is developer-facing)

## Step 2: Run lenses — sequential, fresh-context

Run lenses **in strict order: CEO → Design → Eng → DX**. Each builds on the
previous; never parallel. Each lens is a **fresh-context subagent** (Agent
tool) that receives ONLY the plan file and spec — no conversation history.
Independence is the point: a reviewer marinated in your reasoning will agree
with your mistakes.

Lens briefs (give each subagent its brief + the plan):

- **CEO:** Should this exist at all? Challenge the premise, the scope
  ambition (expand/hold/reduce), alternatives not considered, duplication
  with existing capability. You have authority to say "scrap it."
- **Eng:** Can this be built as written? Architecture, data flow (happy path
  + nil/empty + upstream-error for every flow), edge cases, test strategy,
  performance. Complexity smell: >8 files or >2 new classes/services for the
  stated goal → flag it.
- **Design:** Every user-visible state named? Empty states, error states,
  loading. Specificity over vibes — "clean UI" is not a finding.
- **DX:** Time-to-hello-world, error messages (problem + cause + fix),
  can a stranger onboard from the artifacts this plan produces?

Each lens returns: score 0–10, findings list, and for each finding a
**concrete edit** to the plan (not just a complaint).

## Step 3: Classify every finding — the decision taxonomy

| Class | Definition | Handling |
|-------|-----------|----------|
| **Mechanical** | One defensible answer exists | Auto-decide silently, apply the edit |
| **Taste** | Reasonable people could differ | Ask the user — one question per decision (see Step 5) |
| **User Challenge** | Review says the user's stated direction is wrong | NEVER auto-decide — ask with the 5-field format |

Auto-decisions follow the **6 principles** (from autoplan, verbatim intent):
1. **Choose completeness** — prefer the option covering more edge cases
2. **Boil lakes, not oceans** — expansions inside the blast radius (<5 files,
   no new infra, <1 day) auto-approve; bigger expansions become Taste
3. **Pragmatic** — equivalent options: pick the cleaner one in 5 seconds
4. **DRY** — duplicates an existing capability → reject
5. **Explicit over clever** — 10 obvious lines beat 200 abstract ones
6. **Bias toward action** — flag concerns without blocking

User Challenge format (all five fields, user's direction stays default):
> **What you said** / **What review recommends** / **Why** /
> **What we might be missing** / **If we're wrong, the cost is**

## Step 4: Adversarial verification (from doubt-driven-development)

Before applying findings, take every Critical/High finding and dispatch ONE
fresh-context skeptic subagent per finding with the instruction: "Try to
refute this finding. Default to refuted if the evidence is weak." Findings
the skeptic kills are dropped with a one-line note. This is what separates
"plausible-sounding review" from review.

Skip this step only for plans under 5 files — there the findings are cheap
enough to just evaluate inline.

## Step 5: Ask the user, then apply edits + final gate

Apply surviving **Mechanical** edits directly to the plan file.

Then walk the user through every surviving **Taste** finding and **User
Challenge** — **one question at a time, one finding per question** (use
AskUserQuestion where available). Never batch them into a single summary
question, and never proceed on an unanswered one. Each question carries full
detail:

- **Context** — which lens raised it, what part of the plan it touches
  (quote the plan line), and the evidence behind it
- **Options** — the concrete alternatives (2-4), each with its trade-off,
  your recommended one first and marked
- **Consequence** — what happens downstream if this goes the other way
- User Challenges additionally use the 5-field format above, with the
  user's stated direction as the default option

Apply each answer to the plan file as it lands, before asking the next —
answers often change what the next question should be; drop questions an
earlier answer already settled.

After all questions are resolved, present ONE closing summary:

- Lens scores (before → after edits)
- Decisions made, one line each (Mechanical auto-applied + user-answered)

Append to the plan file:

```markdown
## REVIEW REPORT
Lenses: CEO 7→9, Eng 6→9 [, Design, DX]
Decisions: <list — auto-applied Mechanical + user-answered Taste/Challenges>
NO UNRESOLVED DECISIONS   ← or list them; execution is blocked until none remain
```

**Exit gate:** dev-execute may not start while the report lists unresolved
decisions. Max 3 review rounds — if round 3 still has unresolved items, the
plan is fighting the spec; go back to dev-discover.

## Red flags

- A lens returning "no issues found" without saying what it checked → rerun it
- Writing "do not flag X" into a lens brief → you're pre-judging the review
- Skipping the skeptic pass because findings "look obviously right" → that's exactly when they aren't
