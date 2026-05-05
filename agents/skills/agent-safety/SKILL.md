---
name: agent-safety
description: >
  Safe agent behavior patterns for commits, test changes, deployments, and destructive operations.
  Use when: (1) About to commit code, (2) Modifying tests, (3) Running deployments,
  (4) Performing destructive operations like deletions or overwrites, (5) Any action
  that could have irreversible consequences. Provides guardrails to prevent costly mistakes.
---

# Agent Safety

Critical guardrails for safe agent behavior. These rules prevent costly, irreversible mistakes.

## Human Approval Requirements

**NEVER commit without explicit human approval. This applies to ALL situations.**

| Prohibited | Required |
|------------|----------|
| Automatic commits after reviews | Display review results for human decision |
| Commits without human confirmation | Wait for explicit "yes, commit this" |
| Assuming approval means commit | Present findings and ASK for approval |

## Tests Are Specifications

Once written and approved, **NEVER modify tests without explicit permission**.

- Tests define expected behavior—they are the contract
- If a test fails, fix the implementation, not the test
- Changing tests changes the specification of what the code should do
- Always ask: "Should I update the test to match this new behavior?" before modifying

## Safe Behavior Checklist

Before any potentially destructive action:

1. **Pause and verify**: Is this action reversible?
2. **Show the plan**: Present what you intend to do
3. **Wait for approval**: Get explicit "yes, do this" confirmation
4. **Execute carefully**: Perform the action
5. **Verify results**: Show what was done

## Destructive Operations

These actions require explicit human approval:

- Git commits and pushes
- File deletions
- Database modifications
- Configuration changes
- Deployment actions
- Test modifications
- Any bulk/batch operations

## Recovery Mindset

Always consider:
- What could go wrong?
- How would we recover if this fails?
- Is there a backup or undo path?
- Should we do a dry-run first?
