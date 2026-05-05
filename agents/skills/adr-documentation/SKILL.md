---
name: adr-documentation
description: >
  Architecture Decision Records (ADR) best practices for high-performance software development.
  Use when: (1) Creating new ADRs, (2) Reviewing existing ADRs, (3) Maintaining ADR collections,
  (4) Discussing architectural decisions, (5) Setting up ADR processes for a project.
  Provides templates, lifecycle management, and quality guidelines for effective ADRs.
---

# ADR Documentation

Best practices for Architecture Decision Records in high-performance software development.

## What is an ADR?

An ADR captures a significant architectural decision along with its context and consequences. ADRs create a decision log that helps future developers understand *why* the system is built the way it is.

## When to Write an ADR

Write an ADR when:
- Making a decision that affects the system's structure
- Choosing between significant alternatives
- Establishing patterns that others should follow
- Making trade-offs that need explanation
- Deciding something that would be hard to reverse

Skip an ADR for:
- Trivial implementation choices
- Decisions easily changed later
- Standard language/framework conventions

## ADR Structure

```markdown
# ADR-NNN: Title

**Status**: [Proposed | Accepted | Deprecated | Superseded]
**Date**: YYYY-MM-DD
**Deciders**: [List of people involved]

## Context
[What is the issue? What forces are at play?]

## Decision
[What is the change being proposed/made?]

## Consequences
### Positive
[Benefits of this decision]

### Negative
[Drawbacks and costs]

## Alternatives Considered
[What other options were evaluated?]
```

For detailed templates, see [templates.md](references/templates.md).

## ADR Lifecycle

```
Proposed → Accepted → [Deprecated | Superseded]
                ↓
            Amended (minor updates)
```

| Status | Meaning |
|--------|---------|
| Proposed | Under discussion, not yet decided |
| Accepted | Decision made, in effect |
| Deprecated | No longer recommended, kept for history |
| Superseded | Replaced by another ADR (link to it) |

Always include status transitions:
```markdown
**Supersedes**: ADR-003
**Superseded by**: ADR-012
**Amended**: 2025-03-15 (clarified error handling)
```

For lifecycle management details, see [lifecycle.md](references/lifecycle.md).

## Maintaining an ADR Index

Create `ADR-000-index.md` or a table in the docs README:

```markdown
| ADR | Title | Status | Date | Supersedes |
|-----|-------|--------|------|------------|
| 001 | Architecture Overview | Accepted | 2025-01-15 | - |
| 002 | Database Selection | Accepted | 2025-01-20 | - |
| 003 | Auth Strategy | Superseded | 2025-02-01 | - |
| 004 | Auth Strategy v2 | Accepted | 2025-03-01 | 003 |
```

## Quality Checklist

Before finalizing an ADR:

- [ ] **Context is complete**: Reader understands the problem without prior knowledge
- [ ] **Decision is clear**: Unambiguous what was decided
- [ ] **Consequences are honest**: Both positive AND negative listed
- [ ] **Alternatives documented**: Shows due diligence
- [ ] **Status is current**: Reflects actual state
- [ ] **Cross-references updated**: Related ADRs link to each other
- [ ] **Open questions resolved**: No dangling "TBD" items in accepted ADRs
- [ ] **Date is accurate**: Reflects decision date, not draft date

## References

- **[templates.md](references/templates.md)**: ADR templates for different scenarios
- **[lifecycle.md](references/lifecycle.md)**: Status management and maintenance
- **[examples.md](references/examples.md)**: Real-world ADR examples
