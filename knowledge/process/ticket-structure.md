# your ticket system Ticket Philosophy, Structure, and Process

This document defines how we write and manage tickets. It serves as the specification for AI agents working with your ticket system and as a reference for the development team.

## Purpose

tickets are more than task lists. They are **specifications** and **contracts** for work. A well-written ticket:

- Provides complete context for anyone (human or agent) picking up the work
- Defines clear boundaries for what is and isn't in scope
- Documents decisions and rationale for future reference
- Enables asynchronous collaboration without constant clarification
- Creates an audit trail of how work evolved

The goal is to write tickets that could be handed to someone with no prior context and still be actionable.

---

## Philosophy

### Tickets as Specifications

A ticket should fully specify the work to be done. The description is the source of truth: if something isn't in the ticket, it isn't part of the work. This means:

- **Be explicit**: State assumptions rather than assuming shared context
- **Be complete**: Include all information needed to start work
- **Be bounded**: Clearly define what is out of scope

### Tickets as Contracts

Once work begins, the ticket represents an agreement about what will be delivered. Changes to scope should be:

- Discussed and agreed before implementation
- Reflected in updates to the ticket description
- Documented in comments with rationale

### Balance Detail with Maintainability

Tickets should be detailed enough to be actionable but not so detailed that they become burdensome to maintain. Use judgement:

- Core requirements belong in the description
- Implementation details often belong in comments
- Transient information (progress updates, blockers) belongs in comments
- Decisions that affect future work should be preserved in the description

---

## Ticket Types

### Epic

An Epic represents work too large to complete in a single sprint. Epics must be broken down into smaller, deliverable pieces.

**When to use:**
- Feature spans multiple components or teams
- Work requires multiple releases or phases
- Scope is understood at a high level but details need elaboration

**Key characteristics:**
- Describes the overall goal and success criteria
- Links to child tasks/stories that deliver the Epic
- Updated as child work is completed

See: [Epic Template](../../templates/tickets/epic.md)

### Feature

A Feature (or New Feature) represents new functionality to be implemented in the system.

**When to use:**
- New user-facing capabilities
- New system capabilities or integrations
- Significant enhancements that add new behaviour

**Key characteristics:**
- Describes what the new functionality should do
- Has clear acceptance criteria
- May be broken into sub-tasks for implementation

See: [Feature Template](../../templates/tickets/feature.md)

### Task

A Task represents a discrete, actionable piece of work. Tasks do not necessarily result in system changes.

**When to use:**
- Improvements to existing features
- Technical work (refactoring, infrastructure, tooling)
- Documentation or process work
- Investigation or research
- Administrative or operational work

**Key characteristics:**
- Actionable and specific
- Specific enough to be completed in one sprint
- Has clear acceptance criteria
- Can be assigned to one person or pair

See: [Task Template](../../templates/tickets/task.md)

### Bug

A Bug represents a defect: behaviour that doesn't match expectations or requirements.

**When to use:**
- System behaves differently than specified
- Regression from previous working behaviour
- Error conditions not handled correctly

**Key characteristics:**
- Describes expected vs actual behaviour
- Includes reproduction steps where applicable
- Documents impact to prioritise correctly

See: [Bug Template](../../templates/tickets/bug.md)

### Sub-task

A Sub-task breaks down a larger Task into component pieces, often across different systems or repositories.

**When to use:**
- Task requires changes in multiple systems
- Work can be parallelised across team members
- Tracking progress on distinct phases is valuable

**Key characteristics:**
- Can be briefer than parent (references parent for context)
- Still requires acceptance criteria
- Clearly indicates which system or component is affected in the description

See: [Sub-task Template](../../templates/tickets/subtask.md)

### Service Request

Service Requests come from service desk projects for support or incidents.

**When to use:**
- Internal support requests
- Incident reports requiring investigation

**Key characteristics:**
- May originate from non-technical users
- Often requires triage before becoming actionable
- May include development work directly, or spawn separate development tickets for larger changes

See: [Service Request Template](../../templates/tickets/service-request.md)

---

## Structure Standards

### Description Sections by Type

Each ticket type has recommended sections. See the templates for full details.

**Tasks/Features:**
1. Purpose/Context: Why this work exists
2. Scope: What systems/repos are affected (include affected services/APIs if known, but this can be incomplete if implementation decisions have not been made)
3. Requirements: What must be done (use To-do lists for multi-part work)
4. Business Decisions: Key choices and rationale
5. Out of Scope: What this ticket does NOT cover
6. QA Scenarios: How to test it (required for system changes; see below)
7. Acceptance Criteria: Testable conditions for completion

**Bugs:** Use the [Bug Template](../../templates/tickets/bug.md) for structure and required sections (enhanced for QA in TICKET-80: bug type, evidence in your ticket system, intermittent bugs).

### Acceptance Criteria

Acceptance criteria define when work is complete. They should be:

- **Testable**: Can be verified as pass/fail
- **Specific**: Not open to interpretation
- **Complete**: Cover all requirements in the ticket

Format as checkboxes:

```
## Acceptance Criteria

- [ ] User can reset password via email link
- [ ] Reset link expires after 24 hours
- [ ] Invalid/expired links show appropriate error message
- [ ] Password change is logged in audit trail
```

### QA Scenarios

QA scenarios tell the tester exactly what to validate. They differ from acceptance criteria:

- **Acceptance criteria** define *what done looks like* (the contract)
- **QA scenarios** define *how to prove it works* (the test plan)

A ticket can have acceptance criteria met but still fail QA if edge cases or negative scenarios were not considered. QA scenarios close this gap.

#### When to Include

QA scenarios are required for any ticket that involves **system changes** (code, configuration, infrastructure). They are not required for:
- Documentation-only or process-only work
- Investigation or research tickets
- Administrative tasks

#### Format

Use **Given / When / Then** to structure each scenario:

- **Given**: The starting state or precondition
- **When**: The action taken by a user or system
- **Then**: The expected outcome

#### Scenario Categories

Cover these categories for thorough coverage:

**Happy path**: The expected, normal usage flow. Start here. If the happy path fails, nothing else matters.

**Edge cases**: Boundary values, unusual but valid inputs, timing conditions. These are where production bugs hide. Consider:
- Minimum and maximum values
- Empty inputs vs missing inputs
- Concurrent operations
- Calendar boundaries (e.g., calendar year vs tax year)

**Negative scenarios**: Invalid inputs, unauthorised access, error states. Verify the system fails gracefully:
- Invalid data formats
- Expired sessions or tokens
- Permissions violations
- Network failures or timeouts

**Regression checks**: Verify that existing functionality is not broken by the change. Particularly important for changes to shared components.

#### Test Approach

Each ticket should specify which levels of testing apply:

- **Unit tests**: Logic and business rules in isolation
- **Integration tests**: API boundaries, database interactions, service communication
- **E2E tests**: Full user flows through the system
- **Manual QA on UAT**: Scenarios that require human judgement, visual verification, or complex data setup

#### Examples

**Good QA scenarios (password reset feature):**

```
### Happy Path
- [ ] Given a registered user, when they request a password reset, then they receive an email within 5 minutes
- [ ] Given a valid reset link, when the user sets a new password, then they can log in with the new password

### Edge Cases
- [ ] Given a user who requests multiple resets, when they use the most recent link, then it works and previous links are invalidated
- [ ] Given a reset link that is 23 hours old, when the user clicks it, then it still works (expires at 24 hours)

### Negative Scenarios
- [ ] Given an expired reset link (>24 hours), when the user clicks it, then they see an error and a prompt to request a new link
- [ ] Given an unregistered email address, when a reset is requested, then no email is sent but the UI shows the same confirmation (no user enumeration)

### Test Approach
- Unit tests: Password validation rules, link expiry logic
- Integration tests: Email service integration, database token storage
- E2E tests: Full reset flow from request to login
- Manual QA on UAT: Email rendering across clients, link behaviour in different browsers
```

**Bad QA scenarios:**

- "Test that password reset works" (not specific, not testable)
- "Check edge cases" (which ones?)
- No negative scenarios (what happens when things go wrong?)

### Out of Scope

Explicitly listing what is NOT in scope prevents scope creep and creates a record of deferred work:

```
## Out of Scope

- Password complexity requirements (separate ticket: TICKET-1234)
- Two-factor authentication integration
- Admin password reset capability
```

---

## Fields

### Summary

The summary should be concise but descriptive:

- Start with action verb for tasks: "Add...", "Update...", "Fix...", "Remove..."
- Include the affected component or area
- Be specific enough to distinguish from similar tickets

**Good:** "Add password reset email with expiry link"
**Bad:** "Password reset" or "Fix bug"

### Story Points

We use the Fibonacci scale for relative estimation:

| Points | Meaning |
|--------|---------|
| **1** | Trivial change, minimal complexity |
| **2** | Small task, straightforward implementation |
| **3** | Standard task, well-understood scope |
| **5** | Complex task, multiple components or considerations |
| **8** | Significant work, requires careful planning |
| **13** | Large; consider breaking down |
| **21** | Too large; must be broken down |

Story points measure **relative complexity**, not time. A 2-point task is roughly twice as complex as a 1-point task.

### Risk Fields

Risk assessment is required for all tickets. These fields assess the **risk introduced by the code changes and deployment done in response to the ticket** — not the risk or severity of the underlying problem the ticket addresses.

This distinction matters. For a bug, the severity of the bug itself is captured in **Priority** (triage). The risk fields describe what could go wrong when we **deploy the fix**. A critical bug might have a straightforward, low-risk fix. A minor bug might require a high-risk fix that touches core infrastructure. These are separate assessments.

Risk fields are not optional bureaucracy — they serve as a practical tool for determining and recording what is genuinely required for each piece of work.

All of the following fields must be populated on every ticket.

#### Risk Description

your ticket system field: `Risk Description` (`customfield_10136`, free text)

Every ticket must have a Risk Description. **Without this field, none of the other risk fields have any meaning.**

This is either:

- **No system change:** e.g., "Data update only — no system change required"
- **Identifiable risk:** A specific description of what could go wrong **with the changes being deployed**

Examples of identifiable risks:
- "New feature does not behave as per the supplied requirement"
- "Fix introduces an unexpected regression"
- "Change does not perform as expected and/or introduces a regression in the existing API endpoint / processor / feature / UX flow"

Be specific. Name the thing that could go wrong with the code change. Generic descriptions like "something might break" do not help anyone assess likelihood or plan mitigation.

> **Common mistake:** For bugs, do not describe the risk of the bug itself (e.g., "Users cannot complete checkout"). Describe the risk of your fix (e.g., "Fix to checkout validation introduces a regression in the payment flow"). The bug's severity is already captured in Priority.

#### Risk Likelihood

your ticket system field: `Risk Likelihood` (`customfield_10137`, select)

How likely is the identified risk to occur? Remember: this is the likelihood that your **code changes** cause problems, not the likelihood of the original issue recurring.

- **Rare**: The risk is highly unlikely to occur
- **Unlikely**: The risk may occur but is not expected
- **Possible**: The risk might occur under certain circumstances
- **Likely**: The risk is expected to occur at some point
- **Almost Certain**: The risk is highly likely to occur

For new features and standard changes, **Unlikely** or **Possible** are typically the most appropriate choices.

#### Impact

your ticket system field: `Impact` (`customfield_10138`, select)

What is the impact if the code change introduces the identified risk? Think about the blast radius of **your changes going wrong**:

- A change to an isolated feature or component has limited impact
- A change to a major feature area or an entire system has significant impact
- A change to the core purpose of a system that other systems depend on (e.g., the Taxpayer Service Account Processor) has major or critical impact

Consider downstream dependencies: if your change causes a problem, what else stops working?

> **Note:** This field exists in your ticket system (`customfield_10138`) but may not appear on all issue type screens. If it is not visible on your ticket, raise this with the your ticket system admin.

#### Risk Level

your ticket system field: `Risk Level` (`customfield_10139`, select)

The overall risk level, derived from the combination of Likelihood and Impact:

- **Low**: Minimal concern; no immediate action required
- **Medium**: Moderate concern; mitigation needed within a reasonable time
- **High**: Significant concern; immediate action required
- **Critical**: Extreme concern; top-priority action needed

Use the matrix below as a guide:

| | Low Impact | Medium Impact | High Impact | Critical Impact |
|--|:--:|:--:|:--:|:--:|
| **Rare** | Low | Low | Medium | Medium |
| **Unlikely** | Low | Medium | Medium | High |
| **Possible** | Medium | Medium | High | High |
| **Likely** | Medium | High | High | Critical |
| **Almost Certain** | High | High | Critical | Critical |

**Be honest here.** A Risk Level of **Low** means no mitigating action is required. For the vast majority of development work, at least code review and testing on a UAT environment are required — that is not "Low". Choose at least **Medium** unless the change can genuinely be released without the usual QA steps (e.g., a data-only update, a documentation change, or a configuration tweak with no behavioural effect).

#### Mitigation Plan

your ticket system field: `Mitigation Plan` (`customfield_10140`, free text)

What are we doing to address the specific risk of the code changes identified in the Risk Description?

Examples:
- "Code review + automated test coverage"
- "Code review. Automated test coverage including E2E tests. Addition of new automated test coverage. Manual regression and exploratory testing on UAT. Follow-up checks and smoke testing post-deployment."

This field is important because:

1. **It documents what is actually required for the ticket.** We often discuss testing and QA requirements in conversation but have no record of the agreement. The Mitigation Plan is that record.
2. **It creates accountability.** If the plan says "additional automated tests", there must be evidence in the code changes or pull requests that this was actually done. The plan is a commitment, not a wish.

Match the mitigation to the risk. A Low-risk data update might only need "Peer review of data change". A High-risk change to a core processing pipeline needs a comprehensive plan covering code review, automated tests (including new coverage), manual regression testing, and post-deployment verification.

#### Residual Risk Level

your ticket system field: `Residual Risk Level` (`customfield_10141`, select)

What risk remains after all mitigation steps are complete?

- **Low**: Risk reduced to acceptable levels
- **Medium**: Some risk remains but is manageable
- **High**: Significant risk remains despite mitigation
- **Critical**: Risk remains unacceptable; further action required

Usually **Low**, unless there are legitimate reasons for ongoing risk — for example, changes that cannot be fully verified until they are running in production, known edge cases that remain untested, or dependencies on third-party systems with unpredictable behaviour.

#### Status (Risk)

your ticket system field: `Status` (`customfield_10142`, select)

Track the status of risk mitigation as the ticket progresses:

- **Open**: Risk has been identified but not addressed yet
- **Under Mitigation**: Actions are being taken to mitigate the risk
- **Mitigated**: Mitigation measures are complete; risk is reduced
- **Accepted**: Risk has been acknowledged and accepted without further mitigation
- **Closed**: Risk is no longer applicable or relevant

**Update this field as work progresses.** When the ticket is complete and all mitigation steps are done, mark it as **Mitigated** or **Closed** (if no longer relevant). If there is some ongoing risk that has been acknowledged but does not require follow-up, use **Accepted**. If mitigation is still underway (e.g., deployed but awaiting post-deployment verification), use **Under Mitigation**.

#### Common Mistakes

| Mistake | Why it's wrong | Correct approach |
|---------|---------------|-----------------|
| Describing the bug as the risk | Risk fields are about the deployment, not the problem | Describe what could go wrong with your **fix** |
| Defaulting Risk Level to Low | Low means no mitigation needed — rarely true | Use at least Medium if code review or UAT is required |
| Generic Risk Description | "Something might break" helps no one | Name the specific component, endpoint, or flow at risk |
| Mitigation Plan says "tests" with no evidence | The plan is a commitment | Merge request must contain the tests you committed to |
| Leaving Risk Status as Open on completed tickets | Status should track the lifecycle | Update to Mitigated, Accepted, or Closed when done |

### Priority

Use priority to indicate urgency relative to other work. Align with your team's definitions.

### Labels and Components

Use consistently within your project:
- **Labels**: Cross-cutting concerns (e.g., `security`, `tech-debt`, `documentation`)
- **Components**: System areas (e.g., `api`, `frontend`, `database`)

---

## Dependencies and Linking

### Ticket Relationships

Use your ticket system's link types appropriately:

- **Blocks / Is blocked by**: Work cannot proceed until blocker is resolved
- **Relates to**: Related context, but no dependency
- **Is parent of / Is child of**: Hierarchy (Epic > Task > Sub-task)

Add blocking links proactively. This surfaces dependencies in boards and reports.

### Linking to Code

We use GitHub integration for automatic linking:

**Branch naming:** Include the ticket key in branch names:
```
TICKET-36-ticket-documentation
```

**Commit messages:** Reference the ticket key:
```
TICKET-36: Add knowledge document for ticket standards

Document philosophy, structure, and process for writing tickets.
```

**Merge request URLs:** Add as comments on the ticket when creating PRs:
```
PR ready for review: https://github.com/org/repo/-/merge_requests/123
```

### Linking to Documentation

Reference relevant documentation in the ticket description:
- Confluence pages
- README files
- API specifications
- Design documents

---

## Lifecycle

### Status Workflow

Standard workflow for software projects:

```
To Do → In Progress → In Review → QA → Ready for Production → Done
```

| Status | Meaning |
|--------|---------|
| **To Do** | Work not yet started |
| **In Progress** | Active development |
| **In Review** | Code complete, awaiting review |
| **QA** | Review complete, awaiting QA verification |
| **Ready for Production** | QA passed, awaiting deployment |
| **Done** | Deployed and verified |

Transition tickets promptly. An accurate board helps the whole team.

### Definition of Ready

A ticket must satisfy the [Definition of Ready](definition-of-ready.md) before implementation begins. The DoR is the entry gate; the Definition of Done (below) is the exit gate.

### Definition of Done

A ticket is **Done** when:

- [ ] All acceptance criteria are met
- [ ] Code changes have been reviewed and approved
- [ ] Automated tests pass (unit, integration, E2E as applicable)
- [ ] New/changed functionality has appropriate test coverage
- [ ] QA scenarios verified (all happy path, edge case, and negative scenarios pass)
- [ ] Risk mitigation steps documented in the ticket have been completed
- [ ] QA completion comment posted with evidence of verification
- [ ] Deployed to production (or Ready for Production, per release schedule)
- [ ] Residual Risk Level field updated
- [ ] Any follow-up tickets created for out-of-scope work discovered

*This Definition of Done is a starting point and should evolve based on team experience.*

### When to Create vs Reuse Tickets

**Create a new ticket when:**
- The work is distinct and separately trackable
- Different people might work on it
- It could be deprioritised independently

**Reuse/update an existing ticket when:**
- The work is a refinement of the original scope
- It was always part of the original intent
- Splitting would create artificial overhead

---

## Comment Conventions

### Implementation Plans

Before beginning implementation, check the ticket for existing implementation plan comments. If none exists, create one and save it as a comment. This provides:

- **Audit trail**: Plans preserved alongside the ticket
- **Agent coordination**: Other agents can check for existing plans
- **Visibility**: Stakeholders can review planned approaches
- **Continuity**: If work is interrupted, the plan remains

### Code Review

When moving a ticket to In Review (for code changes), add a comment with:

- Link to the pull request URL in GitHub

This comment is required for compliance. It links the ticket to the code changes being reviewed.

### QA Prep

Before moving a ticket to QA, add a QA prep comment documenting what the tester needs to verify. This comment bridges the gap between the developer's work and QA verification.

This comment is required for compliance. It provides the tester with clear guidance and creates a record of what was expected to be verified.

The QA scenarios in the ticket description are the *specification* (what should be tested). The QA prep comment is the *handoff* (how to test it in the current environment). Use this structure:

```
QA prep for [TICKET-XXX]:

**What changed:** [Brief summary of the changes made]

**Environment:** [UAT / Staging / specific environment details]
**Test data:** [Any accounts, identifiers, or data needed for testing]
**Prerequisites:** [Setup steps or conditions needed before testing]

**Scenarios to verify** (from ticket QA Scenarios):
- [ ] [Scenario 1 from the ticket, with any environment-specific notes]
- [ ] [Scenario 2]
- [ ] [Scenario 3]

**Areas of risk:** [What the developer is most concerned about; where to focus extra attention]

**Known limitations:** [Anything not yet working or deferred]
```

### QA Completion

When QA is complete, add a comment confirming what was tested and the outcome. This comment is the *evidence* that the QA scenarios from the ticket were executed and passed.

This comment is required for compliance. It provides evidence that the change was verified before deployment. Use this structure:

```
QA complete for [TICKET-XXX].

**Environment:** [UAT / Staging]
**Tested by:** [Name]
**Date:** [YYYY-MM-DD]

**Scenarios verified:**
- [x] [Scenario 1 — passed]
- [x] [Scenario 2 — passed]
- [x] [Scenario 3 — passed with note: ...]

**Regression checks:**
- [x] [Existing functionality X still works as expected]
- [x] [No visual regressions observed]

**Issues found:** [None / List any issues discovered and their tickets]

**Verdict:** Ready for deployment.
```

If QA fails, document the failures clearly and transition the ticket back to In Progress with a comment explaining what failed and why.

### Release/Deployment Notes

After deployment, add a comment with:

- Deployment date and environment
- Any post-deployment verification performed
- Issues encountered (if any)

This comment is required for compliance. It completes the audit trail for the change.

### Progress Updates

Use comments for time-sensitive updates:

- Blockers encountered
- Scope clarifications discovered during work
- Decisions made during implementation

Don't use comments for information that should be in the description. Update the description instead.

---

## Compliance and Audit Trail

Our Secure Development practice (ISO 27001) requires that changes tracked by tickets have a clear trail of what happened through the change process. your ticket system's History feature captures some of this automatically (e.g., status transitions), but we rely on **explicit comments** to document key decisions and approvals.

### Required Comments for Compliance

The following comments are required at specific points in the ticket lifecycle:

| Transition | Required Comment |
|------------|------------------|
| To Review | Link to pull request URL in GitHub (if code changes are involved) |
| To QA | QA prep: what changed, scenarios to verify (from ticket), environment, test data, areas of risk |
| QA Complete | QA completion: scenarios verified (checklist), regression checks, issues found, verdict |
| Deployed | Deployment confirmation with date, environment, and any post-deployment verification |

Example QA completion comment:

```
Testing complete. Verified on UAT environment:
- [x] Password reset email received within expected timeframe
- [x] Expired links show appropriate error message
- [x] New password works for subsequent login

Ready for deployment.
```

### Risk Fields are Critical

Risk assessment fields are not optional bureaucracy. They are a compliance requirement that:

- Documents the potential impact of changes before they are made
- Records the mitigation steps that will be taken
- Provides evidence that risks were considered and addressed
- Creates accountability for the risk/mitigation decisions

Every ticket that involves system changes must have:
1. **Risk Description**: What could go wrong
2. **Risk Level**: Honest assessment (not defaulting to "Low")
3. **Mitigation Plan**: What we are doing to address the risk
4. **Residual Risk Level**: What risk remains after mitigation
5. **Status** (risk): Updated as mitigation is completed

### What the Audit Trail Must Show

For any change, an auditor should be able to look at the ticket and see:

1. **What was requested**: Clear description, acceptance criteria, and QA scenarios
2. **What risks were identified**: Risk Description and Risk Level fields
3. **How risks were mitigated**: Mitigation Plan field and evidence in comments/PRs
4. **Who reviewed the change**: Code review comments, approvals
5. **What was tested**: QA scenarios in description (specification) and QA completion comment (evidence)
6. **When it was deployed**: Deployment comment with date and environment
7. **Final risk status**: Residual Risk Level and Status (risk) fields updated

### History vs Comments

your ticket system automatically records some changes in the ticket History:
- Status transitions (with timestamps and who made them)
- Field value changes
- Assignee changes

However, History does not capture **context or reasoning**. Use comments to explain:
- Why a status changed (not just that it changed)
- What was verified during QA
- Any issues encountered and how they were resolved
- Approvals and sign-offs

---

## Anti-patterns

### Empty Descriptions

**Problem:** Tickets with no description or only a title.

**Why it's harmful:** Forces synchronous communication to understand scope. No record of what was intended.

**Fix:** Always include at minimum: purpose, scope, and acceptance criteria.

### Links Without Context

**Problem:** Description contains only links to Slack, Datadog, or external tools.

**Why it's harmful:** Links break, access changes, context is lost.

**Fix:** Summarise the relevant information in the ticket. Link as supplementary reference.

### Informal Requests

**Problem:** Service desk requests written as casual messages without structure.

**Why it's harmful:** Requires triage effort to understand what's actually being asked.

**Fix:** Use templates. Ask clarifying questions before beginning work.

### Scope Creep via Comments

**Problem:** Additional requirements added in comments without updating description or acceptance criteria.

**Why it's harmful:** Creates ambiguity about what's actually in scope. Acceptance criteria don't reflect reality.

**Fix:** Update the description when scope changes. Add new acceptance criteria. Use Out of Scope for deferred work.

---

## Examples

### Exemplary Tickets

These tickets from existing projects demonstrate good practices:

- **TICKET-3125** (Sub-task): Excellent structure with To-do, Business Decisions, User Journey, Out of Scope, and Acceptance Criteria sections.

- **TICKET-3613** (Bug): Clear problem report with Summary, Background/Context, Problem Statement, Expected/Actual Behaviour, Impact, and Request sections.

- **TICKET-552** (Bug): Technical investigation with Facts (SQL query + result), Further Investigation (log excerpts), and Proposed Solution. Documents the debugging process.

- **TICKET-1797** (Task): Demonstrates risk field usage and QA documentation in comments.

### Anti-pattern Tickets

These tickets demonstrate patterns to avoid:

- **TICKET-3124, TICKET-1794**: No description at all. Rely entirely on summary and tribal knowledge.

- **Various service desk tickets**: Informal requests without structured format, requiring significant triage.

---

## Related Resources

- [Definition of Ready](definition-of-ready.md)
- [Epic Template](../../templates/tickets/epic.md)
- [Feature Template](../../templates/tickets/feature.md)
- [Task Template](../../templates/tickets/task.md)
- [Bug Template](../../templates/tickets/bug.md)
- [Sub-task Template](../../templates/tickets/subtask.md)
- [Service Request Template](../../templates/tickets/service-request.md)
