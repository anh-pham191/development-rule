# Task Template

Use this template for Tasks: discrete, actionable pieces of work such as improvements, technical work, documentation, or process work. Tasks do not necessarily result in system changes.

---

## Template

```markdown
## Purpose

[Why does this work exist? What problem does it solve or value does it add?]

## Scope

[What systems, repositories, or components are affected? If known, list the services or APIs that may need changes. It is acceptable to leave this incomplete if implementation decisions have not yet been made.]

- [System/component 1]
- [System/component 2]

## Requirements

[What must be done? For multi-part work, use a To-do list with system prefixes if applicable.]

### To-do

- [ ] [Actionable item 1]
- [ ] [Actionable item 2]
- [ ] [Actionable item 3]

## Business Decisions

[Document key choices and their rationale. Why are we doing it this way?]

| Decision | Rationale |
|----------|-----------|
| [Choice made] | [Why this choice] |

## Out of Scope

[Explicitly list what this ticket does NOT cover. Creates tickets for deferred work.]

- [Related work to be done separately]
- [Future enhancement] (see: [TICKET-XXX])

## QA Scenarios

[Required when the task involves system changes. Omit for documentation-only or process-only tasks.]

[Test scenarios that verify the changes work correctly.]

### Happy Path

- [ ] Given [precondition], when [action], then [expected result]

### Edge Cases

- [ ] Given [boundary condition], when [action], then [expected result]

### Negative Scenarios

- [ ] Given [invalid input or error state], when [action], then [expected result]

### Test Approach

[Which levels of testing apply?]

- **Unit tests:** [Areas covered]
- **Integration tests:** [Boundaries to test]
- **E2E tests:** [User flows, if applicable]
- **Manual QA on UAT:** [Scenarios requiring manual verification]

## Risk Assessment

**Risk Description:** [What could go wrong with the code changes? Or "Data update only — no system change required" for data-only work.]

**Risk Likelihood:** [Rare / Unlikely / Possible / Likely / Almost Certain]
**Impact:** [What is the impact if the changes go wrong?]
**Risk Level:** [Low / Medium / High / Critical — Low means no mitigation required; use at least Medium if code review or UAT testing is needed]

**Mitigation Plan:** [What are we doing to mitigate the risk of the changes? e.g., "Code review + automated test coverage + regression testing on UAT"]

**Residual Risk Level:** [Low / Medium / High / Critical — usually Low after mitigation]
**Status:** [Open / Under Mitigation / Mitigated / Accepted / Closed]

## Acceptance Criteria

[Testable conditions that define when the work is complete.]

- [ ] [Specific, testable criterion 1]
- [ ] [Specific, testable criterion 2]
- [ ] [Specific, testable criterion 3]
```

---

## Guidance

### Required Sections

At minimum, every Task should have:
- **Purpose**: Why the work exists
- **Scope**: What's affected
- **Risk Assessment**: All risk fields must be populated (describes risk of the code changes, not the underlying problem)
- **Acceptance Criteria**: How we know it's done

### Optional Sections

Include when relevant:
- **Requirements/To-do**: For multi-step work
- **Business Decisions**: When choices need documentation
- **Out of Scope**: When scope boundaries need clarification
- **QA Scenarios**: Required when the task involves system changes; omit for documentation-only or process-only tasks

### When to Include QA Scenarios

Include QA Scenarios when the task involves **system changes** (code, configuration, infrastructure). The section helps QA understand what to verify without needing to read the code.

Omit QA Scenarios when the task is purely:
- Documentation or process work
- Investigation or research (no deliverable changes)
- Administrative work

When included, use Given/When/Then format and cover happy path, edge cases, and negative scenarios. See the [Feature template](feature.md) for detailed guidance on writing scenarios.

### Writing Good Acceptance Criteria

Acceptance criteria should be:
- **Testable**: Can be verified as pass/fail
- **Specific**: Not open to interpretation
- **Complete**: Cover all requirements

**Good:** "User receives email within 5 minutes of password reset request"
**Bad:** "Email works properly"

### Story Points

Estimate complexity using the Fibonacci scale:
- **1**: Trivial, minimal complexity
- **2**: Small, straightforward
- **3**: Standard, well-understood
- **5**: Complex, multiple considerations
- **8**: Significant, needs careful planning
- **13/21**: Too large, break it down
