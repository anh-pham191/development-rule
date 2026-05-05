# ADR Templates

Templates for different types of architectural decisions.

## Standard ADR Template

```markdown
# ADR-NNN: [Short Title]

**Status**: Proposed | Accepted | Deprecated | Superseded by ADR-XXX
**Date**: YYYY-MM-DD
**Deciders**: [Names or roles]

## Context

[Describe the issue motivating this decision. Include:
- What problem are we solving?
- What constraints exist?
- What forces are at play (technical, business, team)?]

## Decision

[State the decision clearly. Use active voice:
"We will use X for Y because Z."]

## Consequences

### Positive
- [Benefit 1]
- [Benefit 2]

### Negative
- [Cost or risk 1]
- [Cost or risk 2]

### Neutral
- [Side effects that aren't clearly good or bad]

## Alternatives Considered

### [Alternative 1]
[Brief description and why rejected]

### [Alternative 2]
[Brief description and why rejected]

## References

- [Links to relevant docs, issues, discussions]
```

## Lightweight ADR Template

For smaller decisions that still warrant documentation:

```markdown
# ADR-NNN: [Title]

**Status**: Accepted
**Date**: YYYY-MM-DD

## Context
[2-3 sentences on the problem]

## Decision
[1-2 sentences on what we're doing]

## Consequences
- [Key consequence 1]
- [Key consequence 2]
```

## Technical Choice ADR

For technology/library selection:

```markdown
# ADR-NNN: [Technology] for [Purpose]

**Status**: Accepted
**Date**: YYYY-MM-DD
**Deciders**: [Team]

## Context

We need to select a [category] for [purpose].

Requirements:
- [Requirement 1]
- [Requirement 2]

## Decision

We will use **[Technology]** because:
1. [Primary reason]
2. [Secondary reason]

## Evaluation Matrix

| Criterion | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| [Criterion 1] | Score | Score | Score |
| [Criterion 2] | Score | Score | Score |
| **Total** | X | Y | Z |

## Consequences

### Positive
- [Benefit]

### Negative
- [Trade-off]

### Migration Path
[How to adopt, any migration needed]
```

## Pattern ADR Template

For establishing coding/architectural patterns:

```markdown
# ADR-NNN: [Pattern Name] Pattern

**Status**: Accepted
**Date**: YYYY-MM-DD

## Context

[What problem does this pattern solve?]

## Decision

Adopt the [Pattern Name] pattern for [scope/area].

### Pattern Definition

[Description of the pattern]

### Implementation

```[language]
// Example code showing the pattern
```

### When to Use
- [Scenario 1]
- [Scenario 2]

### When NOT to Use
- [Anti-pattern scenario]

## Consequences

### Positive
- [Consistency benefit]
- [Maintainability benefit]

### Negative
- [Learning curve]
- [Overhead]
```

## Domain-Specific ADR Template

For domain/business logic decisions:

```markdown
# ADR-NNN: [Domain Concept] Handling

**Status**: Accepted
**Date**: YYYY-MM-DD
**Deciders**: [Including domain experts]

## Context

### Business Background
[Explain the domain context]

### Technical Challenge
[What makes this technically interesting]

## Decision

[How we model/handle this domain concept]

### Domain Model

```
[Diagram or type definitions]
```

### Business Rules
1. [Rule 1]
2. [Rule 2]

### Edge Cases
| Scenario | Handling |
|----------|----------|
| [Edge case 1] | [How handled] |
| [Edge case 2] | [How handled] |

## Consequences

### Positive
- [Correctness benefit]
- [Alignment with business]

### Negative
- [Complexity introduced]

## Validation
[How to verify this decision is correct]
```

## RFC-Style ADR

For decisions requiring broader input:

```markdown
# ADR-NNN: [Title]

**Status**: Proposed (RFC until YYYY-MM-DD)
**Date**: YYYY-MM-DD
**Author**: [Name]
**Reviewers**: [Names]

## Summary

[One paragraph summary]

## Motivation

[Why is this change needed?]

## Detailed Design

[Technical details of the proposal]

## Drawbacks

[Why should we NOT do this?]

## Alternatives

[What other designs were considered?]

## Unresolved Questions

- [ ] [Question 1]
- [ ] [Question 2]

## Implementation Plan

1. [Step 1]
2. [Step 2]

---
## Feedback

### From [Reviewer 1] (YYYY-MM-DD)
[Feedback]

### From [Reviewer 2] (YYYY-MM-DD)
[Feedback]
```
