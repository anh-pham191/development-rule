# Request-to-Implementation Flow

The standard flow for handling any user request in projects that consume these rules. The goal is to reach implementation only after the request is fully understood, comprehensively planned, and reviewed — so that the agents doing the implementation produce close to zero bugs or unexpected behaviour.

This flow is mandatory unless the user explicitly asks for a one-shot, throwaway, or trivial change.

## Why It Exists

Most agent-introduced bugs are not coding errors — they are scoping errors. The agent implements what it *assumed* the user wanted, not what the user actually wanted. This flow front-loads understanding and planning so that implementation becomes a mechanical exercise against a well-specified target.

## The Flow

```
user request
    │
    ▼
1. Draft a comprehensive plan
    │
    ▼
2. Interrogate the user (loop until aligned)
    │
    ▼
3. Build the plan using superpowers skills
    │
    ▼
4. Review the plan (loop until robust)
    │
    ▼
5. Implement via subagent-driven development
```

### 1. Draft a Comprehensive Plan

Before asking the user anything, draft a plan that covers what you currently believe the request entails:

- The intended outcome and success criteria
- The surfaces touched (files, modules, services, data)
- Assumptions you are making
- Open questions and unknowns
- Risks, edge cases, and failure modes

This first pass is a *strawman* — it exists to surface what you don't know, not to be correct.

### 2. Interrogate the User

Treat the strawman as a list of things to verify. Ask the user targeted questions until the plan is unambiguous. Acceptable reasons to ask:

- An assumption could change the implementation if wrong
- Scope is ambiguous (in vs. out)
- Multiple reasonable interpretations exist
- A constraint (performance, compatibility, security) is not stated
- Acceptance criteria are not testable as written

Keep asking until the answer to "could a competent implementer build the wrong thing from this?" is **no**. Use the [`superpowers:brainstorming`](../../agents/skills/) skill where exploration of intent and design is needed.

### 3. Build the Plan Using Superpowers Skills

Once aligned, produce the implementation plan using the relevant superpowers skills:

- [`superpowers:writing-plans`](../../agents/skills/) — structure the plan itself
- [`superpowers:test-driven-development`](../../agents/skills/) — encode the spec as tests first
- [`superpowers:using-git-worktrees`](../../agents/skills/) — isolate the workspace before execution
- [`superpowers:dispatching-parallel-agents`](../../agents/skills/) — identify independent units of work
- [`superpowers:subagent-driven-development`](../../agents/skills/) — decompose the plan into subagent-executable tasks

The plan must:

- Decompose the work into discrete, independently verifiable tasks
- Specify inputs, outputs, and acceptance for each task
- Identify which tasks are parallelisable and which are sequential
- Reference the tests that prove correctness
- Name the verification commands that prove completion

### 4. Review the Plan (Loop)

A plan is not done when written — it is done when reviewed. Iterate on the plan until reviewers cannot find a way it could go wrong:

- Walk each task as if you were the subagent executing it. Is the brief self-contained? Could it be misread?
- For each acceptance criterion, name the exact command that proves it
- For each assumption, name the failure mode if the assumption is wrong
- For each risk, name the mitigation in the plan or accept it explicitly
- Use [`superpowers:requesting-code-review`](../../agents/skills/) or a review subagent to get an independent read

Repeat until the plan is robust. The cost of a planning round is small; the cost of a buggy implementation is large.

### 5. Implement via Subagent-Driven Development

Only now does code get written. Dispatch the plan via [`superpowers:subagent-driven-development`](../../agents/skills/) and [`superpowers:executing-plans`](../../agents/skills/). The orchestrator:

- Hands each subagent a self-contained brief from the plan
- Verifies subagent output against the named acceptance commands
- Does not declare completion without running [`superpowers:verification-before-completion`](../../agents/skills/)

## When to Short-Circuit

The flow is not negotiable for non-trivial work, but the following are acceptable short-circuits:

- **Pure questions** (no code change requested) — answer directly
- **Trivial edits the user has fully specified** (rename, typo, one-line fix) — implement directly
- **User explicitly requests a one-shot** ("just do X, don't overthink it") — implement directly, but note the skipped steps

In every other case, follow the flow.

## Anti-Patterns

- **Jumping to implementation** after a single user message. The first message is rarely a complete spec.
- **Asking one question, getting one answer, then implementing.** Interrogation is a loop, not a single round trip.
- **Treating the plan as a deliverable** rather than a working document. Plans get revised; that is the point.
- **Skipping the review loop** because the plan "feels right". Feelings are not verification.
- **Dispatching subagents against a vague plan.** Vague briefs produce vague work, which produces bugs.

## Related Documents

- [Definition of Ready](./definition-of-ready.md) — readiness gate that this flow feeds into
- [Agentic Development Philosophy](../philosophy/agentic-development.md) — the framing this flow operationalises
