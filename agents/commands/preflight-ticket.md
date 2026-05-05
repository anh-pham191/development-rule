# Preflight Ticket Readiness

Validate that a ticket is ready for implementation. The user will provide a ticket key (e.g. `/preflight-ticket TICKET-105`).

## Context

This command is the entry gate to implementation. No work should begin on a ticket until it has been validated as ready. It operationalises the [Definition of Ready](../../knowledge/process/definition-of-ready.md) — the universal standard every ticket must satisfy before implementation begins.

The ticket should be in **To Do** when this command runs. It stays in To Do until `/implement` transitions it to **In Progress**.

This command is written for this repository's own work (your project). Consumer projects that adopt the command replace the project key and Cloud ID with their own; the structure and the DoR it operationalises carry across unchanged.

## Inputs to read

Read all of the following before starting the assessment:

1. **The ticket** — via the your ticket-system MCP (this repository). Full description, acceptance criteria, risk fields, labels, all comments.
2. **The [Definition of Ready](../../knowledge/process/definition-of-ready.md)** — the ten criteria you are assessing against.
3. **[`AGENTS.md`](../../AGENTS.md)** (repo root) — for repository-level conventions and the your ticket system-integration rules.
4. **Relevant knowledge** — under [`knowledge/`](../../knowledge/). In particular, [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md) for ticket structure and comment conventions, and any [`knowledge/philosophy/`](../../knowledge/philosophy/) document the ticket's area touches.
5. **Any referenced files or skills** — if the ticket references a skill under [`agents/skills/`](../../agents/skills/), a playbook under [`knowledge/playbooks/`](../../knowledge/playbooks/), or a template under [`templates/`](../../templates/), read it. Verify that referenced paths and symbols still match the current tree (DoR criterion 5).
6. **Glossary** — [`glossary.md`](../../glossary.md) — if the ticket introduces or uses specialised terminology.

## Process

### 1. Readiness assessment against the Definition of Ready

Work through each of the ten DoR criteria. For each, mark **Met / Not met / Not applicable** with a one-line justification:

1. **Summary** — starts with an action verb; includes the affected area; specific enough to distinguish from similar tickets.
2. **Description** — purpose, scope, and out-of-scope sections present; a reader unfamiliar with the context can understand what is being asked and why.
3. **Acceptance criteria** — specific, testable, complete. Each verifiable as pass/fail.
4. **Template compliance** — matches the type-specific template ([`templates/tickets/<type>.md`](../../templates/tickets/)).
5. **Code alignment** — file paths and symbol names match the current tree. For this repository, this generally means *document and skill paths*, not source code.
6. **Risk fields** — Risk Description populated; Likelihood, Impact, Risk Level, Mitigation Plan, Residual Risk, Status honestly assessed.
7. **Hazard assessment** — area-specific failure modes identified. For this repository (a knowledge and agent-configuration repository), typical hazards are: documentation drift, cross-link rot, consumer-project breakage when a skill changes, terminology confusion, agent-safety violations if a command or rule is weakened.
8. **Codebase readiness** — for this repository this criterion usually reads as *documentation readiness*: does the content being changed already have enough shape that the change is traceable (e.g. a skill with SKILL.md, a playbook with a table of contents)? For purely additive work, this is often *Not applicable*.
9. **Dependencies** — all linked blockers are resolved (Done or Closed).
10. **Actionability** — sufficient context for an implementer to begin without needing to ask clarifying questions.

### 2. This repository-specific checks

Beyond the DoR, verify:

- **Knowledge vs Agents placement** — does the ticket put its deliverable in the correct top-level area? ([`knowledge/`](../../knowledge/) for human-readable documentation, [`agents/`](../../agents/) for AI-loaded configuration, [`templates/`](../../templates/) for scaffolding, [`project-planning/`](../../project-planning/) for in-flight project planning.)
- **Consumer-project impact** — if the ticket changes content that consumer projects (consumer projects) already copy or reference, is that impact named? Changes to skills under `agents/skills/` or process documents under `knowledge/process/` typically have consumer impact; pure additions usually don't.
- **Terminology consistency** — if the ticket introduces new terms, are they added to [`glossary.md`](../../glossary.md), or is the existing glossary entry consistent with how the ticket uses them?
- **Skill discipline** — if the ticket modifies a skill under [`agents/skills/<skill>/SKILL.md`](../../agents/skills/), does it follow the [`skill-creator`](../skills/skill-creator/SKILL.md) conventions? Keep SKILL.md concise; use `references/` for detail.

### 3. Hazard assessment (lightweight but mandatory)

Even if the DoR hazard-assessment field is populated, form your own view:

- What is the blast radius of this change? Which consumer projects could be affected?
- What are the most likely failure modes (broken cross-links, stale examples, contradictory guidance)?
- If the change is to a safety-relevant skill or rule ([`agent-safety`](../skills/agent-safety/SKILL.md)), is there any chance it weakens rather than strengthens the guardrail?
- What safeguards exist (PR review, peer reviewer, consumer-project retest)?

### 4. Present findings

Present a structured assessment to the user — do **not** update your ticket system until the user approves.

1. **Ticket readiness** — one of: *Ready / Ready with refinements / Partially ready / Preparatory work needed / Not ready* (the verdicts defined in the DoR).
2. **DoR score** — per-criterion *Met / Not met / Not applicable* with one-line notes; the headline figure like "8/10 criteria met, 1 not applicable".
3. **This repository-specific checks** — placement, consumer-project impact, terminology, skill discipline.
4. **Hazard assessment** — what could go wrong; what safeguards are in place.
5. **Proposed refinements** — specific, actionable changes to description, acceptance criteria, scope, or risk fields.

## Approval and your ticket system update

On user approval:

1. **Apply any agreed refinements** to the ticket description or acceptance criteria via the ticket-system edit-issue tool.
2. **Post a Definition of Ready comment** on the ticket:

   > **Definition of Ready — confirmed**
   >
   > **Verdict:** `[Ready / Ready with refinements / Partially ready / Preparatory work needed]`
   > **Score:** `[N] of [N-NA] applicable criteria met` (`[NA]` not applicable)
   >
   > **This repository-specific checks:**
   > - Placement: `[OK / note]`
   > - Consumer-project impact: `[none / platform / taxpay / other — with note]`
   > - Terminology: `[OK / glossary update required]`
   > - Skill discipline: `[OK / n/a]`
   >
   > **Hazard summary:** `[one paragraph]`
   >
   > **Outstanding items (if any):** `[bullet list; owner; how resolved]`

   This comment is the gate that `/implement` checks for before any file changes.

## Rules

- Do NOT make any file changes. This is an assessment-only command.
- Do NOT transition the ticket. Status stays *To Do* until `/implement` runs.
- Do NOT stage or commit. Git operations are handled exclusively by `/commit`.
- Do NOT skip the hazard assessment, regardless of how simple the change appears.
- DO flag any acceptance criterion that is ambiguous or untestable.
- DO flag any referenced path that has drifted from the current tree.
- DO propose specific, actionable refinements rather than vague suggestions.
- All written outputs use NZ English.

## See also

- [`knowledge/process/definition-of-ready.md`](../../knowledge/process/definition-of-ready.md) — the ten DoR criteria and verdicts.
- [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md) — ticket structure, AC format, risk-field definitions, comment conventions.
- [`agents/skills/ticket-management/SKILL.md`](../skills/ticket-management/SKILL.md) — ticket-skill summary.
- [`AGENTS.md`](../../AGENTS.md) — repository-level conventions.
- [`agents/commands/implement.md`](./implement.md) — the next step in the loop once the ticket is Ready.
