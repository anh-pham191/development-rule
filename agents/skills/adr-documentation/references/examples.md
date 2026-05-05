# ADR Examples

Real-world examples demonstrating effective ADR practices.

## Example 1: Technology Selection

A well-structured ADR for choosing a database:

```markdown
# ADR-004: PostgreSQL for Primary Database

**Status**: Accepted
**Date**: 2025-01-20
**Deciders**: Backend Team, Infrastructure Lead

## Context

We need to select a primary database for the tax calculation service. Requirements:
- ACID compliance for financial data
- Complex query support for reporting
- Horizontal read scaling
- Team familiarity

Current data volume: ~10M records, growing 20% annually.

## Decision

We will use **PostgreSQL 16** as our primary database.

Primary reasons:
1. Strong ACID guarantees essential for financial calculations
2. Team has 5+ years PostgreSQL experience
3. Excellent query planner for our analytical queries
4. Read replicas provide sufficient scaling for our needs

## Evaluation Matrix

| Criterion | PostgreSQL | MySQL | MongoDB |
|-----------|------------|-------|---------|
| ACID compliance | ✓ Full | ✓ Full | Partial |
| Complex queries | ✓ Excellent | Good | Limited |
| Team experience | ✓ 5+ years | 2 years | 1 year |
| Scaling model | Read replicas | Read replicas | Sharding |
| **Fit score** | **9/10** | 7/10 | 5/10 |

## Consequences

### Positive
- Proven reliability for financial systems
- Rich ecosystem (pg_cron, PostGIS if needed)
- Easy to hire PostgreSQL developers
- Extensive monitoring/tooling support

### Negative
- Write scaling limited to vertical (acceptable for our volume)
- Requires DBA expertise for production tuning
- No native multi-region active-active

### Migration Path
None required—greenfield project.

## Alternatives Considered

### MySQL 8
Strong candidate but team less experienced. Query optimizer less capable for our analytical patterns.

### MongoDB
Rejected due to weaker consistency guarantees for financial data. Our data is highly relational.
```

## Example 2: Domain Pattern

An ADR establishing a critical domain pattern:

```markdown
# ADR-002: Income Year vs Calendar Period Separation

**Status**: Accepted
**Date**: 2025-01-15
**Deciders**: Development Team, Domain Expert

## Context

The NZ IRD uses an "income year" labeling convention that creates ambiguity for October, November, and December balance dates:

- "2025 income year" for December balance runs 1 Jan 2024 – 31 Dec 2024
- All provisional payments occur in 2024, not 2025

This ambiguity has caused systematic calculation errors.

## Decision

Maintain strict separation between Income Year Labels and Calendar Periods.

### Type System
```go
type IncomeYear struct {
    Label        int  // IRD convention (e.g., 2025)
    BalanceMonth int  // 1-12
}

type CalendarPeriod struct {
    StartDate NZDate
    EndDate   NZDate
}
```

### Mapping Rule
```
E = (BalanceMonth in {10,11,12}) ? (Label-1) : Label
```

All date arithmetic uses `CalendarPeriod`, never `Label` directly.

## Consequences

### Positive
- Eliminates year calculation bugs for Oct/Nov/Dec
- Type safety prevents accidental mixing
- Each mapping independently testable

### Negative
- Additional complexity in mapping layer
- Learning curve for new developers

## Validation

Test against all 12 balance months:
- Jan-Sep: Label == Calendar end year
- Oct-Dec: Label == Calendar end year + 1
```

## Example 3: Supersession Chain

How ADRs evolve over time:

```markdown
# ADR-003: JWT for Authentication

**Status**: Superseded by ADR-007
**Date**: 2025-01-10
**Superseded**: 2025-03-15

> **Note**: This ADR has been superseded. See ADR-007 for current auth approach.

## Context
[Original context...]

## Decision
Use JWT tokens with 24-hour expiry...

---

# ADR-007: Session-Based Authentication

**Status**: Accepted
**Date**: 2025-03-15
**Supersedes**: ADR-003

## Context

ADR-003 established JWT authentication. After 2 months in production:
- Token revocation is complex (requires blocklist)
- 24-hour tokens too long for our security requirements
- Refresh token flow adds client complexity

## Decision

Replace JWT with server-side sessions.

### Migration Plan
1. Deploy session infrastructure alongside JWT
2. New clients use sessions, existing clients continue JWT
3. After 30 days, require session auth
4. Remove JWT support

## Consequences

### Positive
- Immediate revocation capability
- Simpler client implementation
- Granular session management

### Negative  
- Server state required (Redis)
- Horizontal scaling needs session affinity or shared store

## Related ADRs
- **ADR-003**: Previous approach (superseded)
- **ADR-005**: Redis infrastructure (enables this)
```

## Example 4: Lightweight ADR

For smaller decisions:

```markdown
# ADR-011: Structured Logging with Zap

**Status**: Accepted
**Date**: 2025-02-01

## Context
Need consistent, performant logging across services.

## Decision
Use `go.uber.org/zap` for all logging:
- Production: JSON format
- Development: Console format
- Always include: timestamp, level, caller, request_id

## Consequences
- Consistent log format enables centralized analysis
- Team must learn zap's API (minor)
```

## Example 5: ADR with Open Questions Resolved

Showing the progression from proposed to accepted:

```markdown
# ADR-009: Event Sourcing for Audit Trail

**Status**: Accepted
**Date**: 2025-02-20
**Deciders**: Architecture Team

## Context
Regulatory requirement for complete audit trail of all state changes.

## Decision
Implement event sourcing for the calculation domain:
- Events stored in append-only log
- Current state derived from event replay
- Snapshots every 1000 events for performance

## Resolved Questions

### Event schema versioning
**Resolution**: Use explicit version field in events. Upcasters transform old events to current schema on read.

### Storage backend
**Resolution**: PostgreSQL with JSONB for events. Evaluated EventStoreDB but team familiarity with PostgreSQL won out.

### Snapshot frequency
**Resolution**: 1000 events based on load testing. Replay of 1000 events takes <100ms.

## Consequences
[...]
```

## Anti-Patterns to Avoid

### Too Vague
```markdown
# Bad: ADR-X: Use Better Architecture

## Decision
We will improve our architecture to be more scalable.
```

### No Consequences
```markdown
# Bad: Missing trade-offs

## Consequences
### Positive
- Everything will be better
- No downsides

### Negative
- None!  # <-- This is never true
```

### Stale Status
```markdown
# Bad: Status doesn't match reality

**Status**: Proposed  # But it's been in production for 6 months
```

### Missing Context
```markdown
# Bad: Assumes reader knowledge

## Context
Same issues as before.

## Decision
Use the new approach instead.
```
