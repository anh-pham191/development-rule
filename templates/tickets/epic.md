# Epic Template

Use this template for Epics: work too large to complete in a single sprint that must be broken down into smaller deliverables.

---

## Template

```markdown
## Goal

[One or two sentences describing the overall objective. What are we trying to achieve?]

## Context

[Why is this Epic needed? What problem does it solve or opportunity does it address?]

## Success Criteria

[How will we know the Epic is complete? What outcomes define success?]

- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]
- [ ] [Measurable outcome 3]

## Scope

### In Scope

- [High-level capability or feature 1]
- [High-level capability or feature 2]
- [High-level capability or feature 3]

### Out of Scope

- [Related work explicitly excluded]
- [Future enhancements to consider separately]

## Breakdown

[List the child tasks/stories that will deliver this Epic. Update as work is refined.]

| Ticket | Summary | Status |
|--------|---------|--------|
| [TICKET-XXX] | [Brief description] | To Do |
| [TICKET-XXX] | [Brief description] | To Do |

## Dependencies

- **Blocks:** [Tickets or work that cannot proceed until this Epic is complete]
- **Blocked by:** [Tickets or work that must complete before this Epic can proceed]
- **Related:** [Related Epics or initiatives]

## Risks

[Key risks at the Epic level. Individual tasks will have their own risk fields.]

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk description] | [L/M/H] | [L/M/H] | [Mitigation approach] |

## Notes

[Any additional context, links to design documents, architecture decisions, or stakeholder discussions.]
```

---

## Guidance

### When to Create an Epic

Create an Epic when:
- Work spans multiple sprints
- Multiple distinct tasks or stories are needed
- Work affects multiple systems or teams
- High-level tracking is valuable for stakeholders

### Keeping Epics Updated

- Add child tasks as scope is refined
- Update the Breakdown table as work progresses
- Close the Epic only when all success criteria are met and child work is complete

### Epic vs Task

If work can reasonably be completed in one sprint by one person/pair, it's a Task, not an Epic. Epics represent work that must be decomposed.
