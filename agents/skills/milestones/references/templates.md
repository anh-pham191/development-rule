# Templates

The `MILESTONES.md` skeleton and a filled example. Adapt names to the project; drop sections that
genuinely don't apply, but record *why* for any major omission.

## `MILESTONES.md` skeleton

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
- Requirements: <path to requirements doc>   Tracker: <optional>
```

## Filled example

A small "CSV import" feature, mid-flight — one milestone done, one in progress. This shows the
intended fidelity: the cold-start block reads cleanly on its own, done milestones carry evidence
links, and the in-progress one names a concrete next action.

```markdown
# CSV Import — Milestones

> Last verified: a1b2c3d (2026-05-27). Authoritative record of milestone status.

## Cold start (read this first)
- Current phase: M2 — Validation & error reporting
- Done: M1   In flight: M2   Next: M3   Blocked: —
- Start here: add the per-row error summary to the import endpoint response (requirements §Errors)

## Milestones
### M1 — Upload & parse  [✅ done]
- Goal: accept a CSV upload and parse it into typed rows.
- Exit criteria: a 10k-row file parses into typed rows; oversized files are rejected with a 413;
  covered by integration tests.
- Evidence: #102, #109   Tracker: —

### M2 — Validation & error reporting  [🚧 in progress]
- Goal: validate rows and report failures without aborting the whole import.
- Exit criteria: a file with one malformed row imports the valid rows and returns a 200 with a
  line-numbered per-row error summary; covered by an integration test.
- Evidence: #114 (partial)   Tracker: —

### M3 — Idempotent re-import  [⬜ not started]
- Goal: re-importing the same file does not duplicate rows.
- Exit criteria: importing an identical file twice yields one set of rows; dedup key documented;
  covered by a test.
- Evidence: —   Tracker: —

## Decisions & scope changes
- 2026-05-27 — Split the original "Import" milestone into M1/M2/M3 so error handling and
  idempotency each have a verifiable exit.

## Links
- Requirements: docs/requirements/csv-import.md   Tracker: —
```

Keep examples domain-neutral. The skeleton's section order and the cold-start block are the parts
that matter — match them; the prose around each milestone adapts to the project.
