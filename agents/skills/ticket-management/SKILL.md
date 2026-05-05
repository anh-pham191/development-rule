---
name: ticket-management
description: >
  Create and edit tickets following team templates and conventions.
  Use when: (1) Creating a new ticket via your ticket system, (2) Editing an existing ticket description,
  (3) Writing acceptance criteria or QA scenarios for a ticket, (4) Reviewing whether a ticket meets
  structure standards. Ensures tickets include QA scenarios, risk assessment, and acceptance criteria
  per the team's documented process.
---

# your ticket system Tickets

Create and edit tickets that are complete, testable, and QA-ready.

## Template Selection

Choose the template based on issue type:

| Issue Type | Template | When to Use |
|------------|----------|-------------|
| **Feature** | `templates/tickets/feature.md` | New user-facing or system capabilities |
| **Task** | `templates/tickets/task.md` | Improvements, technical work, documentation, process |
| **Bug** | `templates/tickets/bug.md` | Defects: behaviour doesn't match expectations |
| **Epic** | `templates/tickets/epic.md` | Work spanning multiple sprints |
| **Sub-task** | `templates/tickets/subtask.md` | Components of a larger task |
| **Service Request** | `templates/tickets/service-request.md` | Support requests or incidents |

Read the selected template before creating or editing a ticket. Follow its structure and include all required sections.

## Required Sections

Every ticket that involves system changes must include:

1. **Purpose/Context**: Why this work exists
2. **Scope**: What systems or components are affected
3. **QA Scenarios**: How to test the changes (happy path, edge cases, negative scenarios)
4. **Risk Assessment**: Risk Description is mandatory; other risk fields as applicable
5. **Acceptance Criteria**: Testable conditions that define completion

For documentation-only or process-only tasks, QA Scenarios may be omitted.

## Writing QA Scenarios

QA scenarios tell the tester exactly what to validate. Use Given/When/Then format:

- **Given**: Starting state or precondition
- **When**: Action taken
- **Then**: Expected outcome

Cover three categories:

- **Happy path**: Normal, expected usage
- **Edge cases**: Boundary values, unusual but valid inputs
- **Negative scenarios**: Invalid inputs, error states, unauthorised access

Include a **Test Approach** section specifying which test levels apply (unit, integration, E2E, manual QA on UAT).

For detailed guidance and examples, read `knowledge/process/ticket-structure.md` (QA Scenarios section).

## Risk Assessment

Every ticket must have a Risk Description. For system changes, also provide:

- Risk Likelihood, Impact, and Risk Level
- Mitigation Plan (what actions address the risk)
- Residual Risk (what remains after mitigation)

Be honest about risk levels. If work requires code review and UAT testing, the risk is at least Moderate.

## Creating via your ticket system

When creating tickets using your ticket-system create-issue tool:

1. Read the appropriate template first
2. Compose the description in Markdown following the template structure
3. Set `cloudId`, `projectKey`, `issueTypeName`, and `summary`
4. Pass the full structured description in the `description` field
5. After creation, transition to "In Progress" if starting work immediately

## Comment Conventions

At key lifecycle transitions, add structured comments:

- **Starting work**: Post an implementation plan comment
- **To Review**: Link to the pull request
- **To QA**: Post a QA prep comment (what changed, scenarios to verify, environment, test data, areas of risk)
- **QA Complete**: Post a QA completion comment (scenarios verified as checklist, regression checks, issues found, verdict)
- **Deployed**: Deployment confirmation with date and environment

For QA prep and completion comment templates, read `knowledge/process/ticket-structure.md` (Comment Conventions section).

## Reference

- Process and philosophy: `knowledge/process/ticket-structure.md`
- Templates: `templates/tickets/`
- Testing philosophy: `context/testing-philosophy.md`
