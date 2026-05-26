---
name: milestones
description: >
  Turn a requirements doc into a repo-authoritative MILESTONES.md that any fresh agent reads to
  orient itself and continue the work, and keep that file true over time. Use when: (1) A project
  has a requirements doc and needs a phased plan a cold agent can pick up, (2) Onboarding onto an
  unfamiliar repo and asking "what's done, what's next, what's blocked?", (3) Recording that a
  milestone is complete after merging work, (4) Re-orienting at the start of a session before
  continuing a multi-session project. Tracks milestones only — hands task detail to the project's
  ticket flow or in-session task list.
---

# Milestones

Turn requirements into a small set of milestones a fresh agent can act on, captured in a single
committed `MILESTONES.md` that stays true to the repo's actual state.

The artifact exists to orient a **cold agent** — one that opens the repo with no prior session
context — in seconds: what phase the project is in, what's done, what's next, what's blocked, and
the single next action to take. Generating the milestone list is the easy part; the skill's real
job is keeping that status *true* as the work moves.

## The one rule

**Status changes only as part of the change that earns it — and never trust `MILESTONES.md`
before verifying it against the repo.** A status field updated in a separate "I'll tidy the doc
later" step is a status field that lies; fold the update into the same commit/PR that did the
work. And a cold agent treats the file as a strong prior, not gospel: verify it against the repo's
actual state before acting on it. See [progress-protocol.md](references/progress-protocol.md).

## Two operations

- **`generate`** — read the requirements doc(s), propose a milestone breakdown with the user
  (milestones are higher-judgment than tasks, so this is a short discussion, not a silent dump),
  and write `MILESTONES.md`. How to shape good milestones and exit criteria:
  [milestone-design.md](references/milestone-design.md).
- **`checkpoint`** — update milestone status against the repo's reality, record evidence, and log
  any scope changes. This is where the anti-staleness discipline lives:
  [progress-protocol.md](references/progress-protocol.md).

## Workflow

```
generate:   read requirements → propose milestones (with user) → write MILESTONES.md → commit
checkpoint: verify file vs repo → update status + evidence → log decisions → commit with the change
```

The `MILESTONES.md` structure and a filled example: [templates.md](references/templates.md).

## Operating rules

- **Milestones only.** Track coarse phases (aim 3–8), not individual tasks. Hand task execution to
  the project's request-to-implementation flow, tickets, or the in-session task list.
- **The repo is the source of truth.** `MILESTONES.md` is authoritative for status. If the project
  uses a tracker, link out to it for task detail — never mirror status into two places, because the
  two will drift and a cold agent can't tell which to trust.
- **Keep the cold-start block honest.** It is the first thing a fresh agent reads; if it is wrong,
  the agent starts in the wrong place. Correct it the moment it drifts.
- **Decisions spin out an ADR.** When milestone planning produces an architectural decision, record
  it via the `adr-documentation` skill and note it in the Decisions log — don't bury it in prose.
- **British/Commonwealth spelling** throughout (optimise, behaviour, prioritise, organise).

## References

- **[milestone-design.md](references/milestone-design.md)** — what makes a good milestone, exit criteria, and how to derive milestones from a requirements doc.
- **[progress-protocol.md](references/progress-protocol.md)** — the update-and-verify discipline that keeps status true (the anti-staleness core).
- **[templates.md](references/templates.md)** — the `MILESTONES.md` skeleton, cold-start block, and decisions log.
