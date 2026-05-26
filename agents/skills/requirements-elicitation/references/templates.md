# Templates

Ready-to-fill markdown skeletons. Adapt the header to the project's house style (see the worked
example in [documentation.md](documentation.md)) — these are the generic forms. Drop sections that
genuinely don't apply rather than leaving them empty, but record *why* a major section is omitted.

## Functional requirements doc

```markdown
# <Component> — <Topic> Requirements

> <One-line context: programme/project, what this covers, for which component.>
> Ticket: <link, if any>.
> Session decisions: <YYYY-MM-DD>[, <YYYY-MM-DD>, …]  (append each elicitation session's date).

## Purpose
<Why this exists and the outcome it serves. The measurable success signal.>

## Scope
**In scope:** <what this covers.>
**Non-goals:** <what a reader might assume is included but isn't; what's delegated elsewhere.>

## Behaviour
<The functional requirements as input → output, including the rules applied. Use Given/When/Then
for anything testable.>

### Rules
<Each business rule and how/when it applies.>

### Boundaries & edge cases
<Zero, empty, null, min, max, first, last, overflow, precision, timezone/leap cases — and the
correct behaviour for each.>

### Failure behaviour
<What happens on bad input, dependency failure, partial failure. Retry/idempotency. What is told
to whom when the right answer can't be produced.>

### Temporal behaviour
<How this behaves as the rules/rates/data change over time. Back-dating, future-dating, rollout.
State "N/A — no time-varying rules" if genuinely not applicable.>

## Data
<Entities, source of truth, precision, retention, PII classification, provenance.>

## Interfaces
<APIs/events in and out, error contract, versioning/compatibility, dependency timeout behaviour.>

## Acceptance criteria
- [ ] <Testable condition — a test could pass or fail against this.>
- [ ] <…>

## Assumptions
- <Stated assumption the design rests on.>

## Open questions
| # | Question | Owner | Needed by |
|---|----------|-------|-----------|
| 1 | <…>      | <who> | <when>    |

## Related
- <Links to meta/non-functional docs, ADRs, upstream specs — link, don't duplicate.>
```

## Non-functional requirements doc

```markdown
# <Component> — Non-Functional Requirements

> <One-line context.>

## Performance
- Latency budget: p50 <…>, p99 <…>. Behaviour when the budget would be exceeded: <queue/shed/degrade>.
- Throughput: <…>. Payload limits: <…>.

## Availability
- Target: <…>. Acceptable downtime: <…>. Degradation behaviour: <…>.

## Scalability
- Volume now: <…>. Projected: <…>. First bottleneck under load: <…>.

## Security
- Authn/authz: <…>. Trust boundary: <…>. Threat notes: <…>.
- Must never be logged/returned: <secrets, PII>.

## Auditability / compliance
- What must be provable after the fact: <…>. Evidence retention: <…>.

## Observability
- Health signal: <…>. Alarm condition: <…>. Key metrics: <…>.
```

## Open-questions register (standalone, if not embedded)

```markdown
# <Topic> — Open Questions

| # | Question | Why it matters | Owner | Needed by | Status |
|---|----------|----------------|-------|-----------|--------|
| 1 | <…>      | <impact if unanswered> | <who> | <when> | Open |
```

## Decision → ADR stub

When the discussion produces an architectural decision, capture it as an ADR (see the
`adr-documentation` skill for the full template) and link it from the requirements doc:

```markdown
# ADR-NNN: <Decision title>

**Status**: Proposed
**Date**: <YYYY-MM-DD>

## Context
<The forces at play, surfaced during requirements elicitation.>

## Decision
<What was decided.>

## Consequences
### Positive
<…>
### Negative
<…>

## Alternatives Considered
<The options weighed during the discussion and why they were rejected.>
```
