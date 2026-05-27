# Progress Protocol

Generating milestones is easy. Keeping their status *true* is the job — and the reason this is a
skill, not a template. A `MILESTONES.md` that has drifted from reality is worse than none: a cold
agent trusts it and starts in the wrong place. Two disciplines keep it honest.

## 1. Update in the same change

Milestone status changes **only as part of the commit or PR that achieves the change** — never as a
separate "I'll tidy the doc later" step. Later never comes, and the file silently falls behind.

The checkpoint ritual, performed as part of finishing a piece of work:

1. The work that completes (or blocks, or starts) a milestone is staged.
2. In the **same** change, update `MILESTONES.md`: flip the milestone's status, add the evidence
   link (the PR/commit), and refresh the cold-start block.
3. Update the `Last verified` line to the current commit/date.
4. Commit them together.

Concretely, when a milestone's last piece of work lands, the cold-start block moves with it:

```diff
 ## Cold start (read this first)
-- Current phase: M2 — Import pipeline
-- Done: M1   In flight: M2   Next: M3   Blocked: —
-- Start here: finish the malformed-row handling in the importer
+- Current phase: M3 — Reporting
+- Done: M1, M2   In flight: M3   Next: M4   Blocked: —
+- Start here: scaffold the weekly summary endpoint (see requirements §Reporting)
```

If the status update isn't in the same change as the work, it isn't done.

## 2. Verify before trust (cold start)

Before *acting* on `MILESTONES.md`, a cold agent confirms it still matches the repo. The file is a
**strong prior, not gospel.**

Run a cheap reality check against the last verified point:

```bash
git log --oneline "$(grep -m1 'Last verified' MILESTONES.md | grep -oE '[0-9a-f]{7,40}')"..HEAD
```

Then walk this checklist:

- Does each milestone marked `✅ done` actually have its work in the repo (the evidence links
  resolve, the code/feature exists)?
- Does the `🚧 in progress` milestone match what the recent commits are actually touching?
- Does **"Start here"** still name a sensible next action, or has it been overtaken?
- Is anything marked `⛔ blocked` actually still blocked?

If the file has drifted, **correct it first** — in its own commit — before doing any new work. Then
proceed from a file you trust.

## Keeping "Last verified" honest

The `Last verified` line records the commit/date at which the file was last confirmed true. Update
it whenever you verify the file (cold start) or change it (checkpoint). A stale `Last verified` is
itself a signal that the next reader should re-verify.

## What NOT to track here

- **Task-level churn.** Individual tasks are added, split, and reordered too often to track at this
  altitude — they belong to the request-to-implementation flow, tickets, or the in-session task list.
- **Anything a tracker owns.** If the project has a tracker, link to it for task status rather than
  copying status in. Two records of the same status drift; one source of truth doesn't.

## Decisions & scope changes

When a milestone's scope shifts — split, merged, dropped, re-sequenced — log it in the Decisions &
scope changes section with the date and the *why*. If the change is architectural (a hard-to-reverse
choice between real alternatives), record it as an ADR via the `adr-documentation` skill and
reference the ADR from the log entry rather than restating the reasoning.

## Related

- [milestone-design.md](milestone-design.md) — what the statuses are tracking.
- [templates.md](templates.md) — where the cold-start block and `Last verified` line live.
