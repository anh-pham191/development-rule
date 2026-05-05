# ADR Lifecycle Management

How to maintain ADRs as living documents throughout a project's evolution.

## Status Transitions

```
                    ┌─────────────┐
                    │  Proposed   │
                    └──────┬──────┘
                           │ Team agrees
                           ▼
                    ┌─────────────┐
              ┌─────│  Accepted   │─────┐
              │     └──────┬──────┘     │
              │            │            │
        Minor update   Replaced    No longer valid
              │            │            │
              ▼            ▼            ▼
        ┌─────────┐  ┌───────────┐  ┌────────────┐
        │ Amended │  │Superseded │  │ Deprecated │
        └─────────┘  └───────────┘  └────────────┘
```

## Status Definitions

### Proposed
- Under discussion, not yet decided
- May have open questions
- Should include target decision date

```markdown
**Status**: Proposed (decision by 2025-02-01)
```

### Accepted
- Decision made and in effect
- Implementation may be ongoing
- No unresolved questions

```markdown
**Status**: Accepted
**Date**: 2025-01-15
```

### Amended
- Minor clarifications or updates
- Core decision unchanged
- Track amendment history

```markdown
**Status**: Accepted
**Date**: 2025-01-15
**Amended**: 
- 2025-02-20: Clarified error handling approach
- 2025-03-10: Added example for edge case
```

### Superseded
- Replaced by a newer ADR
- Keep for historical context
- Always link to replacement

```markdown
**Status**: Superseded by ADR-012
**Date**: 2025-01-15
**Superseded**: 2025-04-01
```

### Deprecated
- No longer recommended
- Not replaced by another ADR
- Explain why deprecated

```markdown
**Status**: Deprecated
**Date**: 2025-01-15
**Deprecated**: 2025-06-01
**Reason**: Technology X is no longer supported
```

## Maintaining Cross-References

When ADRs relate to each other, maintain bidirectional links:

### In the superseding ADR:
```markdown
**Supersedes**: ADR-003

## Context
This decision replaces ADR-003 because [reason].
```

### In the superseded ADR:
```markdown
**Status**: Superseded by ADR-007

> **Note**: This ADR has been superseded. See ADR-007 for the current approach.
```

### Related ADRs Section:
```markdown
## Related ADRs
- **ADR-002**: Provides foundation for this decision
- **ADR-005**: Depends on this decision
- **ADR-008**: Alternative approach for different context
```

## Closing Open Questions

ADRs in "Proposed" status may have open questions:

```markdown
## Open Questions
- [ ] How should we handle rate limiting?
- [ ] What's the migration path for existing data?
```

Before moving to "Accepted":
1. Resolve all open questions
2. Document the resolution inline or in a new section
3. Remove or check off the questions

```markdown
## Resolved Questions

### Rate limiting approach
Decided to use token bucket algorithm. See implementation in `pkg/ratelimit`.

### Migration path
Existing data will be migrated via script. See `scripts/migrate_v2.sql`.
```

## Date Consistency

Maintain accurate dates throughout:

| Field | Meaning |
|-------|---------|
| **Date** | When the decision was made (not drafted) |
| **Amended** | When clarifications were added |
| **Superseded** | When this ADR was replaced |
| **Deprecated** | When this ADR was deprecated |

### Common Mistakes

```markdown
# Wrong: Date is when draft started
**Date**: 2025-01-01  # Started writing
**Status**: Accepted   # Accepted 2025-01-20

# Right: Date reflects decision
**Date**: 2025-01-20  # When accepted
```

## Index Maintenance

Keep your ADR index current:

### After creating a new ADR:
1. Add entry to index
2. Update related ADRs if needed

### After superseding an ADR:
1. Update old ADR status
2. Add "Superseded by" link
3. Update index to show status

### Quarterly Review:
- Check for stale "Proposed" ADRs
- Verify all cross-references are valid
- Ensure deprecated/superseded ADRs are marked

## ADR Review Checklist

Before accepting a new ADR:

```markdown
- [ ] Context fully explains the problem
- [ ] Decision is unambiguous
- [ ] Consequences include both positive and negative
- [ ] Alternatives were genuinely considered
- [ ] No open questions remain
- [ ] Cross-references to related ADRs added
- [ ] Index updated
- [ ] Date reflects acceptance date
```

## Archival Strategy

For long-running projects:

1. **Keep all ADRs**: Even deprecated/superseded ones provide history
2. **Use folders**: Group by year or domain if collection grows large
3. **Tag in index**: Mark superseded/deprecated clearly
4. **Consider tooling**: Tools like `adr-tools` can help manage lifecycle
