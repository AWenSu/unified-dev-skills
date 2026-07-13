# unified-dev-skills

Five stage skills for the software development lifecycle, each synthesized
from the best mechanisms of overlapping community skills — distilled, not
concatenated.

```
dev-discover ──▶ dev-plan ──▶ dev-plan-review ──▶ dev-execute ──▶ dev-finish
 (spec, gated)   (artifact)    (large/risky only)  (orchestrated    (evidence
                                                    or inline)       gate + merge)
```

## Why this exists

A well-equipped Claude Code setup accumulates overlapping planning skills:
superpowers' lifecycle chain, gstack's heavyweight review suite,
planning-with-files' persistence layer, custom planner agents. They're each
excellent, but the overlap costs you: four ways to "make a plan," no single
obvious flow, and routing tables to memorize.

This repo keeps **one skill per stage**. Each absorbs the mechanisms that
earned their place (hard gates, evidence rules, decision taxonomies, progress
ledgers, verification laws) and drops what didn't (external CLI dependencies,
telemetry, duplicated prose).

## The five skills

| Stage | Skill | Spine | Key grafts |
|-------|-------|-------|-----------|
| 1 Discover | `dev-discover` | superpowers:brainstorming | gstack spec's code-evidence rule (`path:line` before questions), five-question intake, scope lock |
| 2 Plan | `dev-plan` | superpowers:writing-plans | planner agent's risk grading + sizing guide; Interfaces blocks; No Placeholders |
| 3 Review | `dev-plan-review` | gstack autoplan (decision system) | doubt-driven adversarial refutation of findings; 6 auto-decision principles; Mechanical/Taste/UserChallenge taxonomy |
| 4 Execute | `dev-execute` | superpowers:subagent-driven-development | executing-plans inline fallback; planning-with-files filesystem-as-memory; crash-safe ledger; model cost routing |
| 5 Finish | `dev-finish` | superpowers:verification-before-completion | drive-the-real-flow requirement; branch integration options |

Skip rules are built in: small tasks bypass stages 1–3 entirely; only large
or risky plans go through review. Planning overhead should never exceed ~20%
of the task itself.

**Per-project-type defaults** (which stages to run, which lenses fire, what
to layer on top for web apps, APIs, CLIs, MCP servers, serverless, docs,
scraping): see [PROJECT-TYPE-GUIDE.md](PROJECT-TYPE-GUIDE.md).

## Install

Copy `skills/` into any location Claude Code loads skills from:

```bash
# per-project
cp -R skills/* your-repo/.claude/skills/
# or global
cp -R skills/* ~/.claude/skills/
```

Each skill is a single self-contained SKILL.md — no scripts, no hooks, no
external dependencies.

## Provenance & upstream sync

Every SKILL.md records its sources and their versions in frontmatter.
Snapshot at synthesis time (2026-07-14):

| Upstream | Version | Repo |
|----------|---------|------|
| superpowers | 6.1.1 | github.com/obra/superpowers |
| gstack | 1.60.1.0 | github.com/garrytan/gstack |
| planning-with-files | 3.5.0 | github.com/OthmanAdi/planning-with-files |

These are syntheses, not forks — upstream keeps evolving. To sync:

1. Check upstream releases against the versions above.
2. Read their changelog for *mechanism* changes (new gates, new protocols).
   Prose rewrites and fixes to machinery we dropped (Codex hooks, telemetry,
   mockup boards) don't apply here.
3. Port mechanism changes into the affected stage skill; bump the version
   noted in its frontmatter.

If you use the upstream skills directly alongside these, prefer the
heavyweight originals when their extra machinery earns its cost — e.g.
gstack `/autoplan` for >15-file plans (dual-model consensus), gstack `spec`
when the output should be a GitHub issue.

## Design rules applied throughout

- **Distill, don't concatenate** — a mechanism gets in by being load-bearing,
  not by existing
- **Gates are sacred, artifacts can shrink** — under time pressure, write a
  smaller spec; never skip approval
- **Evidence over reports** — a subagent's "success", a stale test run, and
  "should work" are not evidence; diffs and fresh command output are
- **Filesystem over context window** — anything that must survive compaction
  goes in a file

## License

MIT
