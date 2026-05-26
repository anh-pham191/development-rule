# Milestones Skill — Design

**Date:** 2026-05-27
**Status:** Approved (brainstorming)
**Author:** Anh Pham (with Claude)

## Purpose & gap

Requirements docs (produced by the `requirements-elicitation` skill) say *what* must be built.
Tickets and the in-session `TaskList` track *individual tasks within a working session*. Nothing
gives a **cold agent** — one that opens the repo with no prior session context — a durable,
repo-resident answer to:

> What phase is this project in? What's done, what's in flight, what's next, what's blocked?

The `milestones` skill fills that gap. It turns a requirements doc into a milestone breakdown and
maintains a single committed artifact (`MILESTONES.md`) that any fresh agent reads first to orient
itself and continue the work.

## Key decisions

### Source of truth: repo-authoritative, tracker-agnostic

`MILESTONES.md` is the authoritative record of milestone status, committed to the repo. The skill
assumes **no external tracker exists** by default, because the goal — "any fresh agent can pick up
and finish the project" — implies the orienting artifact lives in the repo the agent just cloned,
not behind Jira/Linear/GitHub auth.

This is a *superset* posture: when a project does have a tracker, `MILESTONES.md` links out to it
for task detail while remaining the cold-start entry point. It deliberately avoids a **dual source
of truth** for status (the worst failure mode — two records that drift and leave a fresh agent
unable to tell which to trust).

### Granularity: milestones only; tasks handed off

The skill owns coarse milestones (typically 3–8 phases), each with verifiable exit criteria and a
status. Fine-grained task execution is delegated to the existing `request-to-implementation-flow`,
tickets, and in-session `TaskList`. This complements the ticket flow rather than competing with it,
and keeps the staleness surface small (few status fields to rot).

Explicitly **not** in scope: task-level tracking, full work-breakdown structures, estimation,
scheduling, or Gantt charts.

## The artifact: `MILESTONES.md`

One committed file (repo root or `docs/` — project's choice). Structure:

```markdown
# <Project> — Milestones

> Last verified: <commit SHA / date>. Authoritative record of milestone status.

## Cold start (read this first)
- Current phase: <milestone N — name>
- Done: <list>   In flight: <list>   Next: <list>   Blocked: <list + why>
- Start here: <the single next action a fresh agent should take>

## Milestones
### M1 — <name>  [✅ done | 🚧 in progress | ⬜ not started | ⛔ blocked]
- Goal: <one line; traces to a requirement>
- Exit criteria: <verifiable; mapped to acceptance criteria in the requirements doc>
- Evidence: <PR/commit links proving exit met>   Tracker: <link if one exists>

## Decisions & scope changes
- <date> — <what changed and why> (link an ADR if the change is architectural)

## Links
- Requirements: <path to requirements-elicitation output>   Tracker: <optional>
```

The **Cold start** block is the load-bearing section: it is what a fresh agent reads first, and it
must always reflect reality.

## Skill shape

Mirrors the `requirements-elicitation` house pattern: a slim `SKILL.md` orchestrator plus on-demand
`references/` files, so an agent reads little to operate and loads depth only for the phase it is in
(protecting the context window).

```
agents/skills/milestones/
  SKILL.md                       # slim orchestrator: two operations, pointers, the anti-staleness rule
  references/
    milestone-design.md          # what makes a good milestone; exit criteria; how to derive from requirements
    progress-protocol.md         # the update + verification discipline (anti-staleness)
    templates.md                 # the MILESTONES.md skeleton + cold-start block
```

### Two operations

1. **`generate`** — read the requirements doc(s), propose a milestone breakdown with the user
   (briefly — milestones are higher-judgment than tasks), and write `MILESTONES.md`. Each milestone
   maps its exit criteria back to the requirements' acceptance criteria.
2. **`checkpoint`** — update milestone status against reality. This is where the anti-staleness
   discipline lives (below).

## Anti-staleness (the skill's core value)

Generating milestones is the easy part; keeping "current status" *true* is the hard part and the
reason this is a skill rather than a template. Two mechanisms:

1. **Update-in-the-same-change.** Milestone status changes only as part of the commit/PR that
   achieves the change — never as a separate "remember to update the doc" step. The skill defines
   this as a checkpoint ritual at PR/merge time, so the doc cannot silently fall behind.
2. **Fresh-agent verification first.** Before *trusting* `MILESTONES.md`, a cold agent runs a cheap
   reality check: read `git log` since "Last verified", and confirm the claimed-done work actually
   exists. If the doc has drifted, correct it before acting. The file is treated as a **strong
   prior, not gospel** — the same discipline applied to potentially-stale memory records.

## Relationship to existing process

- **Feeds from:** `requirements-elicitation` (its output is this skill's input).
- **Complements:** `request-to-implementation-flow` — milestones set the phase context; that flow
  handles planning and execution of the work within a milestone.
- **Does not replace:** tickets, `TaskList`, or any tracker. Links to them where they exist.
- **Decisions:** architectural decisions surfaced while planning milestones spin out an ADR (via
  the `adr-documentation` skill) and are noted in the Decisions log.

## Generalisation constraint

Like `requirements-elicitation`, this skill must be fully tool- and domain-agnostic: no company,
programme, product, ticket-prefix, or repo names. Tracker references are generic and optional.

## Validation / success criteria

The skill is successful if:

- A fresh agent, given only the repo, can read `MILESTONES.md` and correctly state what's done,
  what's next, and what to do first — without prior session context.
- The `generate` operation produces milestones whose exit criteria are verifiable and trace to
  requirements.
- The `checkpoint` operation keeps status aligned with the repo's actual state, and the
  verification step catches drift.
- `SKILL.md` stays a slim orchestrator (~100 lines) with depth in references; no domain/tool leaks.

## Out of scope (for this design)

- Automating the checkpoint via git hooks/CI (the skill defines the ritual; automation is a later,
  separate concern).
- Multi-project / portfolio roll-ups.
- Any tracker-specific integration code.
