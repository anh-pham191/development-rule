# Requirement Types

A comprehensive requirements effort covers more than "what the system does." Each kind below has
a definition, what to capture, and the thing teams habitually forget. Work through the kinds that
apply — skip the ones that genuinely don't, but say so explicitly rather than silently dropping them.

For each type, the elicitation questions that expose its hidden complexity live in
[elicitation-playbook.md](elicitation-playbook.md).

---

## 1. Business / goal requirements

**Captures:** Why this exists, the outcome it serves, how success is measured.

- The problem in one sentence, and whose problem it is.
- The measurable signal that tells you it worked (a metric, a removed cost, a closed risk).
- The deadline or trigger forcing the work now.

**Commonly missed:** Building with no measurable outcome — "done" becomes "code merged" instead of
"the outcome moved." If you can't name the success signal, the scope isn't defined yet.

## 2. Functional requirements

**Captures:** The behaviours, rules, and computations the system performs. The observable
input → output contract.

- The core happy-path behaviour as input/output pairs.
- Every business rule and how it's applied.
- The decision branches and what determines each one.

**Commonly missed:** The non-happy paths. A functional requirement isn't complete until its edge
cases (empty, zero, max, boundary values) and error behaviours are specified. See the boundaries
and failure sections of the playbook.

## 3. Non-functional / quality requirements

**Captures:** How well the system must do its job — the quality attributes.

- **Performance:** latency budget (p50/p99), throughput, payload sizes.
- **Availability:** uptime target, acceptable downtime, degradation behaviour.
- **Scalability:** expected volume now and growth; what breaks first under load.
- **Security:** authn/authz, threat model, secrets handling.
- **Auditability / compliance:** what must be provable after the fact, retention of evidence.
- **Observability:** what must be measurable; how you know it's healthy.

**Commonly missed:** Treating these as "obvious" and unstated. A latency budget of "fast" is not a
requirement. For regulated/financial/compliance domains, auditability is often the *hardest*
requirement and the one most often left implicit.

## 4. Data requirements

**Captures:** The data the system owns, reads, produces, and retains.

- Entities and their relationships; which system is the source of truth for each.
- Accuracy/precision requirements (e.g. monetary values, rounding rules).
- Retention, deletion, and archival rules.
- PII / sensitivity classification and the handling each class demands.
- Data lineage / provenance — can you explain where a value came from?

**Commonly missed:** Retention and PII rules, and **precision** for money and time (rounding,
truncation, timezone, units). These rarely appear in the original request and almost always matter.

## 5. Interface / integration requirements

**Captures:** The contracts at the system's edges.

- APIs the system exposes (shape, versioning, error contract) and consumes.
- Events published/subscribed; ordering and delivery guarantees.
- Upstream dependencies and what happens when they're slow or down.
- Backward/forward compatibility commitments.

**Commonly missed:** What happens when an upstream is unavailable, slow, or returns bad data; and
how the contract evolves without breaking existing consumers.

## 6. Constraints (regulatory, technical, organisational)

**Captures:** The fixed conditions the solution must satisfy, not chosen but imposed.

- Regulatory / statutory rules and the deadlines attached to them.
- Technical constraints (must run on X, must use Y, must integrate with Z).
- Organisational constraints (budget, timeline, team, existing platform decisions).

**Commonly missed:** Compliance *deadlines*, and the fact that externally imposed rules often
change on fixed dates — which forces a temporal-versioning requirement (see §9 and the playbook).

## 7. User stories & acceptance criteria

**Captures:** The work framed from the user's perspective, with testable "done" conditions.

- "As a … I want … so that …" for each meaningful capability.
- Acceptance criteria phrased so a test could pass or fail against them
  (Given/When/Then works well).

**Commonly missed:** Acceptance criteria that are actually testable. "Works correctly" is not
acceptance; "Given a back-dated request, the result reflects the configuration in effect on that
date" is.

## 8. Non-goals / out of scope

**Captures:** What this work explicitly will *not* do.

- Capabilities a reader might reasonably assume are included but aren't.
- Adjacent concerns delegated to another service/team.
- "Later, not now" items, distinguished from "never."

**Commonly missed:** Writing them down at all. Explicit non-goals are the cheapest defence against
scope creep and the best signal that the boundary was actually thought through.

## 9. Temporal / change-over-time requirements

**Captures:** How the system behaves as the rules, rates, and data it depends on change.

- When a rule/rate changes, do past computations stay stable (effective-dating) or recompute?
- Back-dated and future-dated inputs: which version of the rules applies?
- Versioning of the rules themselves and how a change is rolled out.

**Commonly missed:** Almost always. For any system whose rules, configuration, or reference data
change over time — pricing tiers, eligibility policy, feature-flag rollouts, ranking/scoring
weights, document templates, permission rules — this is a first-class requirement, not an
afterthought. If the request doesn't mention time, that's the strongest signal you must raise it.

## 10. Assumptions & open questions

**Captures:** What is being taken as true without verification, and what remains undecided.

- Every assumption the design rests on, stated explicitly.
- Open questions with an owner and a "needed by" point.

**Commonly missed:** Surfacing assumptions at all. An unstated assumption is a silent bet; writing
it down converts it into something a reviewer can challenge.

## 11. Stakeholders & decision rights

**Captures:** Who is affected and who decides.

- Who must approve the scope.
- Who is impacted downstream and must be consulted.
- Who owns the requirement once it's live.

**Commonly missed:** Naming the decider. Requirements stall when "someone should decide" has no name.

---

## How the types map to documents

Many projects already segment requirement kinds into separate files (for example: a cross-cutting
"meta" doc, a "non-functional" doc, a "change-management" doc, and one functional doc per
feature/primitive). Prefer the project's existing segmentation over inventing your own — match the
house structure described in [documentation.md](documentation.md).
