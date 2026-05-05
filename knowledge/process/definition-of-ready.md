# Definition of Ready

The Definition of Ready (DoR) is a universal standard that every ticket must satisfy before implementation begins. It answers the question: **is this ticket ready to be implemented?**

The DoR is the entry gate. The Definition of Done is the exit gate. Together, they bookend the implementation lifecycle — DoR ensures we start well, DoD ensures we finish well.

## Philosophy

### Why It Exists

Starting implementation on a ticket that isn't ready is one of the most common sources of wasted effort. Unclear scope leads to rework. Missing context leads to incorrect assumptions. Absent risk assessment leads to uncontrolled change. The DoR prevents these failures by requiring that a ticket be adequately specified, assessed, and scoped before anyone writes a line of code.

### Same Standard for All Work

The DoR applies equally to all ticket types — features, bugs, tasks, sub-tasks, and service requests. The criteria are the same. What varies is the depth and effort required to satisfy them: a simple configuration change may take minutes to assess; a complex feature touching financial calculations may take significantly longer.

This universality is important. It means everyone on the team shares a common understanding of "ready", regardless of the type of work.

### Conceptually Distinct from Acceptance Criteria

Acceptance criteria define what the ticket must deliver — they are task-specific. The DoR defines whether the ticket is adequately prepared for implementation — it is universal. One of the DoR criteria is that acceptance criteria exist and are testable, but the DoR itself is not a list of what to build. It is a checklist of preparedness.

---

## The Criteria

A ticket is **Ready** when all of the following are satisfied. 

### 1. Summary

The summary starts with an action verb, includes the affected component or area, and is specific enough to distinguish from similar tickets

### 2. Description

The ticket description includes purpose, scope, and out-of-scope sections. A reader unfamiliar with the context can understand what is being asked and why

### 3. Acceptance criteria

All acceptance criteria are specific, testable, and complete. Each criterion can be verified as pass/fail. Together, they fully cover the requirements stated in the description

### 4. Template compliance

The ticket follows the type-specific template structure. Bugs include expected vs. actual behaviour. Features include requirements and business decisions. Sub-tasks reference their parent for context

### 5. Code alignment

All code references in the ticket — file paths, method names, class names — match the current codebase. Stale references point implementers in the wrong direction and waste time.

This is a best-effort check. Line numbers shift with every commit and are not expected to be exact, but file paths and symbol names should be verified against the current state of the code.

This criterion does not apply to tickets that do not reference specific code.

### 6. Risk fields

All seven risk fields are populated and honestly assessed.

### 7. Hazard assessment

Domain-specific risks and failure modes have been identified. This goes beyond the structured risk fields to consider: what could go wrong that is specific to the domain this ticket touches?

Examples of domain-specific hazards:
- Financial calculations: rounding errors, currency handling, tax rate application
- Third-party integrations: API behaviour changes, rate limiting, data format assumptions
- Authentication: access control gaps, session handling, privilege escalation
- Data migrations: irreversibility, referential integrity, performance under load
- Shared services: blast radius across consumers

### 8. Codebase readiness

The code being changed has tests that document its current behaviour. Tests are specifications — they encode knowledge about how the system behaves today. Code without tests is code without a verifiable contract: if you change it, you cannot know what you broke.

If the existing code lacks adequate test coverage, preparatory work to establish that coverage should be completed before the main implementation begins. This ensures the implementation has a safety net.

This criterion does not apply to purely additive work (new files, new endpoints) where no existing code is being modified, or to non-code changes.

### 9. Dependencies

All linked blockers are resolved (Done or Closed). No unresolved blocking relationships prevent implementation from proceeding.

This criterion checks the current state of linked issues. It does not require scanning for undiscovered dependencies — if a dependency is not linked, it is not assessed here.

### 10. Actionability

The ticket provides sufficient context for an implementer to begin work without needing to ask clarifying questions. This is a holistic judgement across all other criteria: given the description, acceptance criteria, scope, code references, risk assessment, and domain context — can someone pick this up and start implementing?

If the answer is "they would need to ask about X", the ticket is not actionable.

---

## Applicability

Not every criterion applies to every ticket type. When a criterion does not apply, it is marked as not applicable rather than unmet.

| Criterion | Code changes | Configuration | Documentation | Data updates |
|---|:---:|:---:|:---:|:---:|
| Summary | Yes | Yes | Yes | Yes |
| Description | Yes | Yes | Yes | Yes |
| Acceptance criteria | Yes | Yes | Yes | Yes |
| Template compliance | Yes | Yes | Yes | Yes |
| Code alignment | Yes | If referenced | No | No |
| Risk fields | Yes | Yes | Yes | Yes |
| Hazard assessment | If applicable | If applicable | No | No |
| Codebase readiness | Yes | No | No | No |
| Dependencies | Yes | Yes | Yes | Yes |
| Actionability | Yes | Yes | Yes | Yes |

Risk fields are required for all ticket types. For non-code tickets, the Risk Description may be "No system change required" and the Risk Level may be Low — but the fields must still be populated.

---

## Assessment

### How readiness is evaluated

Readiness can be assessed manually by a team member or automated through tooling. The criteria are the same either way. The method of assessment does not change the standard.

Each criterion is scored as:
- **Met** — the criterion is satisfied
- **Not met** — the criterion is not satisfied, with a specific note on what is outstanding
- **Not applicable** — the criterion does not apply to this ticket type

The **readiness score** (e.g., "8/10 criteria met") gives an at-a-glance assessment. Outstanding items are listed so the assessor and the team know exactly what remains.

### Verdicts

After assessment, the ticket falls into one of these categories:

- **Ready** — all applicable criteria are met. Implementation can begin.
- **Ready with refinements** — criteria are met after applying fixes that can be resolved immediately (e.g., correcting a stale file reference, rewording an ambiguous acceptance criterion). Refinements are applied during assessment and the ticket is confirmed as ready.
- **Partially ready** — some criteria are outstanding but require human action that cannot be resolved during assessment (e.g., an unresolved blocker owned by another team). The ticket is not blocked, but the outstanding items should be resolved before implementation begins.
- **Preparatory work needed** — the codebase lacks adequate test coverage for the code being changed. Preparatory work (typically a sub-task to establish test coverage) must be completed before the main implementation. The ticket is blocked until this work is done.
- **Not ready** — fundamental deficiencies exist that cannot be resolved without significant rework (e.g., no description, no acceptance criteria, unclear scope). The ticket must be revised and reassessed.

---

## Relationship to Definition of Done

The DoR and DoD are complementary gates:

| | Definition of Ready | Definition of Done |
|---|---|---|
| **When** | Before implementation begins | After implementation is complete |
| **Question** | Is this ticket ready to be implemented? | Is this ticket finished? |
| **Applies to** | The ticket as a specification | The work product delivered |
| **Owner** | The person or process assessing readiness | The implementer and reviewer |
| **Universal** | Same criteria for all tickets | Same criteria for all tickets |

One of the DoD criteria is "All acceptance criteria are met" — which only makes sense if acceptance criteria exist and are testable (DoR criterion 3). Similarly, "Risk mitigation steps documented in the ticket have been completed" (DoD) requires that a mitigation plan exists (DoR criterion 6). The DoR establishes the foundations that the DoD builds on.

---

## Related Documents

- [Code Review Philosophy](../philosophy/code-review.md)
- [Git Commit Philosophy and Conventions](git-commits.md)
