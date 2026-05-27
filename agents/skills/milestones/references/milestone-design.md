# Milestone Design

How to turn a requirements doc into a small set of milestones that are worth tracking — and how to
write exit criteria a cold agent can actually verify.

## What makes a good milestone

A milestone is a coherent, independently valuable slice of the project with a verifiable end. Good
milestones share four traits:

- **Independently valuable.** Reaching it moves the project to a meaningfully better state — a
  capability works end to end, a risk is retired, a phase is unblocked. Not "wrote half the parser."
- **Completable.** It can actually be finished and declared done, not an open-ended workstream.
- **Verifiable exit.** There is a pass/fail condition that proves it's reached (see below).
- **Right-sized.** Big enough that "done" is a real event; small enough that the project has several
  of them. Aim for **3–8** per project.

Contrast with **tasks**: tasks are smaller, more numerous, and volatile (they get added, split, and
reordered constantly). Tasks are not tracked here — they belong to the project's request-to-
implementation flow, its tickets, or the in-session task list. Milestones are the stable scaffold;
tasks are the churn inside each one.

## Exit criteria

A milestone's exit criteria are the pass/fail conditions that prove it's done. They must:

- Be **verifiable** — a person or command can confirm pass or fail without judgement calls.
- **Trace back** to acceptance criteria in the requirements doc, so "done" means "delivered what was
  asked", not "felt finished".

Rewrite vague exits into verifiable ones. For example, for a CSV-import feature:

- ✗ Vague: "Import works."
- ✓ Verifiable: "A 10k-row CSV with one malformed row imports the 9,999 valid rows, rejects the bad
  row with a line-numbered error, and the importer endpoint returns a 200 with a per-row summary —
  covered by an integration test."

The verifiable form names *what* is observed and *how* it's confirmed (here, an integration test).

## Deriving milestones from requirements

1. **Read the requirements doc** and list the outcomes it asks for.
2. **Group by outcome and dependency.** Cluster requirements that deliver one capability together;
   note which clusters depend on others.
3. **Sequence so each milestone unblocks the next.** Order by dependency, then by risk — pull the
   riskiest or most uncertain work earlier so surprises surface while there's time to react.
4. **Prefer outcome-based slices over technical-layer slices.** "User can log in" (vertical, demoable)
   beats "build the auth database layer" (horizontal, unverifiable on its own).
5. **Surface ordering risks** to the user — if two milestones contend or a dependency is unclear,
   raise it rather than guessing.

## How many, and how big

Aim for **3–8** milestones. Two smells tell you the sizing is off:

- **Too fine:** the list reads like a task checklist (a dozen-plus entries, each a single change).
  Roll them up into phases.
- **Too coarse:** a single "build the thing" milestone. Split by the capabilities or risks inside it.

## Status vocabulary

Use these four states consistently across `MILESTONES.md`:

- `✅ done` — exit criteria met, with evidence (links to the PRs/commits that prove it).
- `🚧 in progress` — actively being worked; the cold-start block should point here.
- `⬜ not started` — defined but not begun.
- `⛔ blocked` — cannot proceed; **must name the blocker** (and link it if a tracker/issue exists).

## Related

- [progress-protocol.md](progress-protocol.md) — keeping these statuses true over time.
- [templates.md](templates.md) — the `MILESTONES.md` structure these milestones live in.
