---
name: requirements-elicitation
description: >
  Drive a structured, boundary-probing discussion to define and document software requirements,
  then write them into the project's requirements convention. Use when: (1) Starting a new
  feature, service, or primitive and the scope is not yet pinned down, (2) A ticket says "build X"
  but the edge cases, failure modes, and decisions are unstated, (3) Turning a vague request into
  a reviewable requirements doc, (4) Capturing functional, non-functional, data, integration, or
  regulatory requirements, (5) Surfacing the architecture decisions hidden inside a request.
  Runs an interactive elicitation with the user and writes the result to the repo's docs.
---

# Requirements Elicitation

Turn a fuzzy request into a comprehensive, reviewable requirements document by running an
aggressive, boundary-hunting discussion — then documenting the outcome where the project keeps
its requirements.

This skill orchestrates a conversation. Your value is not transcribing what the user already
knows; it is **exposing what they haven't thought about** — the boundaries, edge cases, failure
modes, and architecture decisions buried inside "build X" — and pinning each one down.

## The one rule

**Never ask an obvious question.** If the answer is in the code, the ticket, or the existing
docs, go read it. If a competent engineer would answer it the same way every time, don't ask it.
Spend every question on a genuine fork — a place where the answer materially changes what gets
built. Vague chore questions ("What should it do?", "Is performance important?", "Any other
requirements?") are banned. See [elicitation-playbook.md](references/elicitation-playbook.md).

## Workflow

```
1. Frame → 2. Investigate → 3. Classify → 4. Elicit (probe, iterate per area)
  → 5. Synthesise & challenge → 6. Document → 7. Decisions → ADR → 8. Review & close
```

### 1. Frame
Establish: which service/area, what problem, who is asking, what ticket (if any). State the scope
boundary back to the user in one sentence and get agreement before going deeper.

### 2. Investigate before asking
Read the relevant code, existing requirement docs, ADRs, and the ticket. Build a model of what is
already decided. Every question you can answer yourself is a question you must not ask the user.

### 3. Classify the requirement types in play
Decide which *kinds* of requirement this work touches (functional, non-functional, data,
integration, constraints/regulatory, …). Each kind has its own things-people-forget.
See [requirement-types.md](references/requirement-types.md).

### 4. Elicit — probe each area aggressively
Work one area at a time. Lead with the sharpest unknown. Use `AskUserQuestion` for decisions
(always with concrete options and their trade-offs) and open conversation for narrative. Hunt:
temporal change, limits/boundaries, partial failure, concurrency & ordering, source-of-truth,
trust boundaries, data lifecycle, and scope edges. The full question bank lives in
[elicitation-playbook.md](references/elicitation-playbook.md).

### 5. Synthesise & challenge
Play back a structured draft. Actively surface contradictions, unstated assumptions, and gaps.
Mark every unresolved item in an **Open Questions** register rather than papering over it.

### 6. Document
Write the requirements into the project's convention — correct folder, file name, and header
style — and update the docs index. Link to existing docs rather than duplicating them.
See [documentation.md](references/documentation.md) and
[templates.md](references/templates.md).

### 7. Decisions → ADR
If the discussion produced a genuine architectural decision (a hard-to-reverse choice between
real alternatives), capture it as an ADR alongside the requirements doc, and cross-link.

### 8. Review & close
Iterate the document until the user signs off. Then handle the project's close-out (commit,
ticket comment) per its conventions.

## Questioning stance

- **One fork at a time.** Each question targets a decision that changes the design.
- **Options, not interrogation.** Frame choices with their trade-offs so the user is *deciding*,
  not being quizzed.
- **Push on the boundary.** Zero, null, empty, max, overflow, first, last, concurrent, retried,
  superseded, back-dated. The interesting requirements live at the edges.
- **Name the assumption.** When the user is silent on something, state the assumption explicitly
  and ask them to confirm or correct it.
- **Time is a dimension.** Especially for rules and configuration that change over time: what
  happens when the rule changes? Do past results stay stable? See the temporal section of the playbook.

## References

- **[requirement-types.md](references/requirement-types.md)** — the taxonomy of requirement kinds and what each one must capture.
- **[elicitation-playbook.md](references/elicitation-playbook.md)** — the aggressive question bank (by category) and how to use `AskUserQuestion` well.
- **[documentation.md](references/documentation.md)** — where/how to write the doc, house-style header, index update, ADR spin-off, naming and commit conventions.
- **[templates.md](references/templates.md)** — ready-to-fill markdown templates for a requirements doc and the open-questions register.
