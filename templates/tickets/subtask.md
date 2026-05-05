# Sub-task Template

Use this template for Sub-tasks: components of a larger Task that can be worked on independently, often in different systems or repositories.

---

## Template

```markdown
## Context

[Brief reference to the parent task. What is this sub-task contributing to?]

Parent: [TICKET-XXX] - [Parent task summary]

## Scope

[What specific system, component, or repository does this sub-task cover?]

**System:** [e.g., API, Frontend, Database, IR Landing]

## Requirements

[What must be done in this sub-task?]

- [ ] [Specific action 1]
- [ ] [Specific action 2]
- [ ] [Specific action 3]

## Technical Notes

[Implementation details, constraints, or considerations specific to this sub-task.]

## QA Verification

[How will this sub-task be verified? Reference parent ticket QA scenarios if applicable.]

- [ ] [Verification step specific to this sub-task's system/component]
- [ ] [Regression check: no impact on existing behaviour in this system]

**Parent QA Scenarios:** [List which scenarios from the parent ticket this sub-task contributes to, or "See parent ticket for full QA scenarios"]

## Risk Assessment

**Risk Description:** [What could go wrong with the code changes? Or "No system change required" if applicable.]

**Risk Likelihood:** [Rare / Unlikely / Possible / Likely / Almost Certain]
**Impact:** [What is the impact if the changes go wrong?]
**Risk Level:** [Low / Medium / High / Critical — use at least Medium if code review or UAT testing is needed]

**Mitigation Plan:** [What are we doing to mitigate the risk of the changes?]

**Residual Risk Level:** [Low / Medium / High / Critical — usually Low after mitigation]
**Status:** [Open / Under Mitigation / Mitigated / Accepted / Closed]

## Acceptance Criteria

[How we verify this sub-task is complete. Should align with parent task criteria.]

- [ ] [Testable criterion 1]
- [ ] [Testable criterion 2]
```

---

## Guidance

### Brevity is Acceptable

Sub-tasks can be briefer than parent tasks because:
- Context exists in the parent
- Scope is narrower by definition
- Dependencies are clearer

However, sub-tasks should still have:
- Clear scope (which system)
- Requirements (what to do)
- QA verification (how to test this component)
- Risk assessment (risk fields are required on all tickets)
- Acceptance criteria (how to verify)

### Referencing the Parent

Always link to the parent task. This:
- Provides context without duplication
- Allows navigating the full scope
- Ensures sub-task is tracked under parent in reports

### Coordination Across Sub-tasks

When sub-tasks have dependencies on each other (e.g., API must be done before Frontend), document this:
- Use blocks/blocked-by links between sub-tasks
- Note dependencies in the parent task
- Coordinate on shared contracts (API specifications)

### QA Verification

Sub-task QA verification is lighter than parent-level QA scenarios. Focus on:
- **Component-specific checks**: What can be verified in isolation for this system
- **Regression**: Confirm no unintended impact on existing behaviour
- **Parent linkage**: Reference which parent QA scenarios this sub-task contributes to

Full end-to-end QA scenarios belong on the parent ticket. Sub-tasks document what is testable at their scope.

### When to Create Sub-tasks vs Separate Tasks

**Create sub-tasks when:**
- Work is clearly part of a larger deliverable
- Parent task would be too large without breakdown
- Progress tracking at component level is valuable

**Create separate tasks when:**
- Work could be delivered independently
- Different sprints might be involved
- Scope is substantial enough to stand alone
