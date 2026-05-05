# Workflow Loop Prompts

Ready-to-use prompts for orchestrating AI agents through each step of a ticket-driven Workflow Loop.

## When to Use This Playbook

Use these prompts when working through a ticket or task with an AI coding assistant. Each prompt corresponds to a step in the Workflow Loop, derived from the [Agentic Development Philosophy](../philosophy/agentic-development.md).

Adapt the prompts to your context:
- Replace `TICKET-XX` with your actual ticket reference
- Replace `<container-name>` with your Docker container (if any)
- Add project-specific context as needed

---

## Quick Reference

| Step | Purpose | Key Question | Approval Gate |
|------|---------|--------------|---------------|
| **0** | Context Foundation | "Do we understand the system?" | Human confirms context is sufficient |
| **1** | Specification | "What does done look like?" | Human confirms spec is clear |
| **1a** | Readiness | "Is the ticket ready to implement?" | Definition of Ready confirmed |
| **2** | Implementation Plan | "How will we build it safely?" | Human confirms plan + hazards |
| **3** | Build | "Does it work?" | Human approves each checkpoint |
| **3a** | Review | "Is it right?" | Critical / High findings resolved |
| **4** | Documentation | "What did we learn?" | Human confirms docs are complete |
| **5** | Completion | "Did we build what we specified?" | Human approves commit |

---

## Step 0: Context Foundation

*Use at project start or when context is stale.*

```
I'm starting work on project [name / repo].

Before we begin any tasks, help me establish the context foundation:

1. Read these files: [list key files: README, architecture docs, etc.]
2. Summarise:
   - What this system does
   - Key architectural patterns
   - Technologies and frameworks used
   - Any known constraints or hazards

3. Identify gaps:
   - What documentation is missing?
   - What would a new developer need to know that isn't written down?

Do not propose solutions yet. I want to understand the current state first.
```

---

## Step 1: Specification

*Use when starting a new ticket/task.*

```
Ticket: TICKET-XX
Title: [ticket title]

Review this ticket and produce a specification:

1. Restate the goal in your own words
2. List acceptance criteria (testable conditions for "done")
3. Define scope boundaries:
   - What IS included
   - What is explicitly OUT of scope
4. Identify dependencies or prerequisites
5. Flag any ambiguities or assumptions that need clarification

Do not plan implementation yet. I want to confirm the specification is correct first.
```

---

## Step 2: Implementation Plan

*Use after specification is approved.*

```
The specification for TICKET-XX is approved.

Create an implementation plan:

1. **Approach**: How will you build this? What's the technical strategy?
2. **Sequence**: What order will you work in? What depends on what?
3. **Hazard Analysis** (mandatory):
   - What could go wrong?
   - What mistakes are common in this type of work?
   - What safeguards will we use? (tests, validation, rollback)
4. **Validation Strategy**: How will we verify each part works?

List all assumptions explicitly. I will validate them before we proceed to build.
```

---

## Step 3: Build

*Use after implementation plan is approved.*

```
The implementation plan for TICKET-XX is approved.

Begin building. Follow this process:

1. **TDD where practical**:
   - Write failing test first
   - Implement minimum code to pass
   - Refactor if needed

2. **Work incrementally**:
   - Complete one logical unit at a time
   - Run tests after each change: `docker exec <container-name> <test command>`
   - Pause for review at natural checkpoints

3. **Flag issues immediately**:
   - If you discover a gap in the spec, stop and ask
   - If an assumption proves wrong, stop and ask
   - If you encounter an unexpected hazard, stop and ask

Start with [first item from implementation plan]. Explain what you're about to do before writing code.
```

---

## Step 4: Documentation

*Use after build is complete and tests pass.*

```
Implementation for TICKET-XX is complete and tests pass.

Now capture the knowledge from this work:

1. **What should be documented?**
   - New patterns or approaches used
   - Decisions made during implementation (candidates for ADRs)
   - Hazards discovered that weren't in the original plan
   - Anything a future developer would need to know

2. **What context assets need updating?**
   - Architecture docs
   - README or setup instructions
   - Known hazards documentation

3. **What should go in the ticket?**
   - Summary of what was built
   - Any deviations from the original plan
   - Follow-up items discovered

Draft the documentation updates. I will review before we finalise.
```

---

## Step 5: Completion

*Use after documentation is done.*

```
Documentation for TICKET-XX is complete.

Final validation before we close:

1. **Acceptance criteria check**:
   - Walk through each criterion from Step 1
   - Confirm: does the implementation meet it? Yes/No

2. **Specification alignment**:
   - Does what we built match what we set out to build?
   - Any gaps between intent and outcome?

3. **Loose ends**:
   - Any follow-up tickets needed?
   - Any out-of-scope items to track?

4. **Ready to commit?**
   - Show me what will be committed (staged files, diff summary)
   - Draft commit message: `TICKET-XX: [description]`
   - Wait for my explicit approval before committing

Summarise the final state. I will confirm completion.
```

---

## Tips for Effective Use

1. **Don't skip steps.** Each step builds on the previous. Skipping specification leads to rework.

2. **Wait for approval at each gate.** The prompts explicitly pause for human confirmation. Respect those checkpoints.

3. **Adapt, don't follow blindly.** These are templates. Adjust for your project's conventions, tools, and context.

4. **Use separation of concerns.** Consider validating plans with a second AI model (different tab/session) before approving.

5. **Flag assumptions early.** The earlier an incorrect assumption is caught, the less rework required.
