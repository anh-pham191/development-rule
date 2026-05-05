# Glossary

This glossary defines key terms used across toolbox documentation. Consistent terminology supports clear communication between humans and agents.

---

## Index of Terms

- [Acceptance Criteria (AC)](#acceptance-criteria-ac)
- [ADR (Architecture Decision Record)](#adr-architecture-decision-record)
- [Agent](#agent)
- [Agentic Workflow Loop](#agentic-workflow-loop)
- [Context Asset](#context-asset)
- [Definition of Done (DoD)](#definition-of-done-dod)
- [Hazard Analysis](#hazard-analysis)
- [Workflow Loop](#workflow-loop)

---

## A

### Acceptance Criteria (AC)

Specific, testable conditions that define when work is complete and correct. Acceptance criteria are established before implementation begins and serve as the authoritative measure of success.

Acceptance criteria should be:
- **Specific**: Unambiguous about what "done" looks like
- **Testable**: Possible to objectively verify whether they are met
- **Complete**: Cover the full scope of expected behaviour

In agentic development, acceptance criteria provide agents with clear targets and humans with objective validation points.

**Related**: [Agentic Development Philosophy](knowledge/philosophy/agentic-development.md), principle 7 (Documentation as Acceptance)

---

### ADR (Architecture Decision Record)

A document that captures a significant technical decision along with its context, options considered, and rationale. ADRs create an auditable trail explaining why a system exists in its current form.

An ADR typically includes:
- The decision made
- The context and constraints at the time
- Options that were considered
- The reasoning behind the choice
- Consequences and trade-offs accepted

ADRs support the principle of Traceable Decisions, ensuring future readers understand not just what was decided, but why.

**External references**:
- [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) (Michael Nygard, 2011): The definitive origin. Short, readable, and explains the "why" clearly.
- [adr.github.io](https://adr.github.io/): The community hub with templates, tooling, and further reading.

**Related**: [Agentic Development Philosophy](knowledge/philosophy/agentic-development.md), principle 6 (Traceable Decisions)

---

### Agent

An AI system that performs tasks autonomously within defined boundaries. Unlike traditional AI that answers questions, agents take action (writing code, reviewing documents, running tests) under human direction.

Agents operate according to context assets and within constraints defined by human orchestrators. They do not make decisions outside their defined scope, override documented standards, or approve their own work.

**External references**:
- [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) (Anthropic, 2024): Distinguishes workflows (predefined code paths) from agents (LLMs dynamically directing their own processes). Emphasises guardrails, trust boundaries, and appropriate use cases.
- [Practices for Governing Agentic AI Systems](https://cdn.openai.com/papers/practices-for-governing-agentic-ai-systems.pdf) (OpenAI, 2024): Defines agentic AI as "systems that can pursue complex goals with limited direct supervision." Focuses on accountability, safety practices, and responsible parties in the agent lifecycle.
- [What are AI Agents?](https://www.ibm.com/think/topics/ai-agents) (IBM): An accessible enterprise-oriented overview for non-technical readers.

**Related**: [Agentic Development Philosophy](knowledge/philosophy/agentic-development.md), principles 2 (Human Sovereignty) and 5 (Separation of Concerns)

---

### Agentic Workflow Loop

A set of guiding heuristics for agentic development, organised as steps where each step includes an iterative review loop. The agentic workflow loop provides structure for progressing from requirements through to delivery while maintaining quality at each stage.

The human orchestrator decides when to exit each review loop and proceed to the next step. Quality emerges through iteration, not single-pass execution.

**Related**: [Agentic Development Philosophy](knowledge/philosophy/agentic-development.md), principle 4 (Iterative Quality)

---

## C

### Context Asset

Documented standards, guidelines, validated examples, and decision records that define how work should be done. Context assets encode institutional knowledge in a form that persists across sessions and team members.

Context assets serve as the "operating instructions" for agents and the institutional memory for teams. Because agents have no persistent memory, context assets provide the knowledge that enables consistent, high-quality outputs.

Examples of context assets:
- Philosophy documents
- Coding standards and conventions
- Architecture Decision Records (ADRs)
- Domain-specific hazard documentation
- Validated examples and templates

Context assets are living documents that evolve with the systems they describe.

**Related**: [Agentic Development Philosophy](knowledge/philosophy/agentic-development.md), principles 3 (Explicit Standards) and 9 (Living Knowledge)

---

## D

### Definition of Done (DoD)

A shared understanding of what it means for work to be complete. Unlike acceptance criteria (which are specific to a task), the Definition of Done is a team-wide standard that applies to all work.

A Definition of Done typically includes:
- Code complete and reviewed
- Tests written and passing
- Documentation updated
- Acceptance criteria met
- Ready for deployment

In agentic development, the Definition of Done explicitly includes documentation: work is not done until knowledge is captured in the appropriate context assets.

**External references**:
- [The Scrum Guide](https://scrumguides.org/scrum-guide.html): The canonical source for Definition of Done in Scrum.
- [What is a Definition of Done?](https://www.scrum.org/resources/what-definition-done) (Scrum.org): Practical explanation and examples.

**Related**: [Agentic Development Philosophy](knowledge/philosophy/agentic-development.md), principle 7 (Documentation as Acceptance); see also [Acceptance Criteria (AC)](#acceptance-criteria-ac)

---

## H

### Hazard Analysis

A mandatory component of implementation planning that explicitly identifies what could go wrong. Hazard analysis surfaces domain-specific risks, known pitfalls, and potential failure modes before implementation begins.

Hazard analysis addresses:
- What are the domain-specific risks for this work?
- What mistakes have been made in similar work before?
- What could go wrong during implementation?
- What safeguards will prevent or catch these issues?

**Related**: [Agentic Development Philosophy](knowledge/philosophy/agentic-development.md), principle 8 (Hazard Awareness)

---

## W

### Workflow Loop

See [Agentic Workflow Loop](#agentic-workflow-loop).
