## Agentic Development Philosophy

These principles guide human-AI collaboration in software development, ensuring outputs we can trust.

**Confidence = Integrity + Quality + Provenance**

Trust in AI-assisted outputs requires all three pillars. Integrity without quality means reliably doing the wrong thing. Quality without provenance means correctness cannot be explained. Provenance without integrity means documentation describes a system that does not match reality.

---

## Integrity: The System Does What It's Supposed To

Integrity means the system behaves exactly as specified: no hidden assumptions, no silent drift, no undocumented behaviours.

### 1. Human Sovereignty

Humans direct, orchestrate, validate, and decide. Agents implement within defined boundaries.

- ✓ DO: Maintain human control at decision points; define clear boundaries for agent work
- ✗ DON'T: Cede judgment to agents; allow agents to operate outside their defined scope

### 2. Explicit Standards

Behaviour is governed by documented standards, not autonomous judgment or implicit understanding.

- ✓ DO: Document expectations before work begins; provide agents with explicit guidance
- ✗ DON'T: Rely on agents inferring expectations; assume shared understanding without documentation

### 3. Accountability

Humans bear responsibility for AI-generated outputs. The organisation is accountable regardless of who or what produced the work.

- ✓ DO: Ensure clear ownership of agent-assisted work; accept responsibility for outputs you direct
- ✗ DON'T: Treat AI involvement as absolution; assume "the agent did it" transfers liability

---

## Quality: Outputs Are Demonstrably Correct

Quality means outputs can be proven correct through evidence, not merely asserted to be correct.

### 4. Iterative Refinement

Quality emerges through cycles of creation, review, and refinement, not single-pass execution. Invest in verification over speed.

- ✓ DO: Expect multiple iterations; verify outputs against acceptance criteria before proceeding
- ✗ DON'T: Accept first-pass output without scrutiny; rush to completion under time pressure

### 5. Independent Verification

The creator of work cannot be its approver. Trust requires external validation, not self-assessment.

- ✓ DO: Use different agents or humans for production and review; require independent sign-off
- ✗ DON'T: Allow self-approval; collapse creation and review into one actor

### 6. Hazard Awareness

Proactively analyse what could go wrong before building. Anticipate failure modes.

- ✓ DO: Identify domain risks, known pitfalls, and potential failure modes before implementation
- ✗ DON'T: Assume success; skip hazard analysis because the task seems straightforward

---

## Provenance: Every Decision Is Traceable

Provenance means every output and decision can be traced back to its source and rationale.

### 7. Traceable Decisions

Every significant decision is documented with context and rationale, not just outcome.

- ✓ DO: Record decisions with the reasoning behind them; capture options considered and trade-offs accepted
- ✗ DON'T: Make significant decisions without documentation; assume future readers will understand implicitly

### 8. Living Documentation

Work is incomplete until knowledge is captured. Documentation must evolve with the systems it describes.

- ✓ DO: Treat documentation as acceptance criteria; update context assets when reality changes
- ✗ DON'T: Defer documentation to "later"; let documentation drift from reality

---

### Related Documents

- [Code Review Philosophy](code-review.md): Principles for human and agent review
