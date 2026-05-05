# Feature Template

Use this template for Features (or New Features): new functionality to be implemented in the system.

---

## Template

```markdown
## Purpose

[Why does this feature exist? What problem does it solve or value does it add for users?]

## Scope

[What systems, repositories, or components are affected? If known, list the services or APIs that may need changes. It is acceptable to leave this incomplete if implementation decisions have not yet been made.]

- [System/component 1]
- [System/component 2]

## User Story

[Optional: Describe the feature from the user's perspective.]

As a [type of user], I want [capability] so that [benefit].

## Requirements

[What must the feature do? Be specific about the expected behaviour.]

### Functional Requirements

- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

### Non-functional Requirements

[Performance, security, accessibility, or other quality requirements.]

- [Requirement 1]
- [Requirement 2]

## Business Decisions

[Document key choices and their rationale. Why are we doing it this way?]

| Decision | Rationale |
|----------|-----------|
| [Choice made] | [Why this choice] |

## Out of Scope

[Explicitly list what this feature does NOT include. Creates tickets for deferred work.]

- [Related work to be done separately]
- [Future enhancement] (see: [TICKET-XXX])

## QA Scenarios

[Test scenarios that verify the feature works correctly. These tell the tester exactly what to validate.]

### Happy Path

- [ ] Given [precondition], when [action], then [expected result]
- [ ] Given [precondition], when [action], then [expected result]

### Edge Cases

- [ ] Given [boundary condition], when [action], then [expected result]

### Negative Scenarios

- [ ] Given [invalid input or error state], when [action], then [expected result]

### Test Approach

[Which levels of testing apply to this feature?]

- **Unit tests:** [Areas covered by unit tests]
- **Integration tests:** [API or service boundaries to test]
- **E2E tests:** [User flows to automate]
- **Manual QA on UAT:** [Scenarios requiring manual verification]

## Risk Assessment

**Risk Description:** [What could go wrong when the changes are deployed? e.g., "New feature does not behave as per the supplied requirement"]

**Risk Likelihood:** [Rare / Unlikely / Possible / Likely / Almost Certain]
**Impact:** [What is the impact if the changes go wrong?]
**Risk Level:** [Low / Medium / High / Critical — Low means no mitigation required; use at least Medium if code review or UAT testing is needed]

**Mitigation Plan:** [What are we doing to mitigate the risk of the changes? e.g., "Code review + automated test coverage + UAT testing"]

**Residual Risk Level:** [Low / Medium / High / Critical — usually Low after mitigation]
**Status:** [Open / Under Mitigation / Mitigated / Accepted / Closed]

## Acceptance Criteria

[Testable conditions that define when the feature is complete.]

- [ ] [Specific, testable criterion 1]
- [ ] [Specific, testable criterion 2]
- [ ] [Specific, testable criterion 3]
```

---

## Guidance

### Required Sections

At minimum, every Feature should have:
- **Purpose**: Why the feature exists
- **Scope**: What's affected
- **Requirements**: What the feature must do
- **QA Scenarios**: How to test it (happy path, edge cases, negative scenarios)
- **Risk Assessment**: All risk fields must be populated (describes risk of the code changes being deployed)
- **Acceptance Criteria**: How we know it's done

### Optional Sections

Include when relevant:
- **User Story**: For user-facing features
- **Business Decisions**: When choices need documentation
- **Out of Scope**: When scope boundaries need clarification
- **Non-functional Requirements**: For performance, security, etc.

### Feature vs Task

Use **Feature** when:
- Implementing new user-facing or system capabilities
- Adding new behaviour to the system
- The work results in something that didn't exist before

Use **Task** when:
- Improving existing functionality
- Technical work that doesn't add new capabilities
- Documentation, process, or administrative work

### Writing QA Scenarios

QA scenarios specify *how to prove* the feature works. They differ from acceptance criteria: acceptance criteria define *what done looks like*; QA scenarios define *how to test it*.

Use **Given / When / Then** format:
- **Given**: The starting state or precondition
- **When**: The action taken
- **Then**: The expected outcome

Cover three categories:

1. **Happy path**: The expected, normal usage flow
2. **Edge cases**: Boundary values, unusual but valid inputs, timing conditions
3. **Negative scenarios**: Invalid inputs, unauthorised access, error states

**Good:**
- "Given a user with an expired password reset link, when they click the link, then they see an error message and a prompt to request a new link"

**Bad:**
- "Test that password reset works"

Include a **Test Approach** section to clarify which levels of testing apply (unit, integration, E2E, manual). This helps developers and QA coordinate on coverage.

### Writing Good Acceptance Criteria

Acceptance criteria should be:
- **Testable**: Can be verified as pass/fail
- **Specific**: Not open to interpretation
- **Complete**: Cover all requirements

**Good:** "User can reset password via email link that expires after 24 hours"
**Bad:** "Password reset works"

### Story Points

Estimate complexity using the Fibonacci scale:
- **1**: Trivial, minimal complexity
- **2**: Small, straightforward
- **3**: Standard, well-understood
- **5**: Complex, multiple considerations
- **8**: Significant, needs careful planning
- **13/21**: Too large, break it down
