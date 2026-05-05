# Implement

Implement a change against a ticket that has been cleared by `/preflight-ticket`. The user will provide a ticket key (e.g. `/implement TICKET-105`).

## Context

This command takes a *Ready* ticket into actual changes. It enforces scope discipline, plan-before-work, and the repository's load-bearing conventions while the work is being done.

This repository is a knowledge and agent-configuration repository — "implementation" is almost always a **change-set to documentation, skills, playbooks, or agent configuration**, not source code. The discipline is the same; the artefacts are different.

This command is written for this repository's own work (your project). Consumer projects that adopt the command replace the project key and Cloud ID with their own; the process carries across.

## Pre-flight

Before touching any file:

1. **Confirm readiness.** Has `/preflight-ticket` been run against this ticket and a *DoR confirmed* comment posted? If not, stop and run `/preflight-ticket` first.
2. **Confirm ticket status.** The ticket should be in **To Do** (transitioning to In Progress is the first thing this command does) — or already in **In Progress** if you are resuming work.
3. **Confirm the branch.** Branch name should follow the repository convention documented in [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md) and the historical pattern `TICKET-XX-kebab-case-description`. If the current branch is `main` or otherwise wrong, stop and ask the user.
4. **Read any existing your ticket system comments.** Prior agents or humans may have posted a plan, a partial implementation, or findings. Do not overwrite their work.

## Process

### 1. Transition to In Progress

**Immediately** transition the ticket to *In Progress* via the ticket-system transition tool. Do this *before* any file changes, so the board accurately reflects active work.

### 2. Post the implementation plan as a comment

Before editing files, post a comment on the ticket describing what you intend to do. This creates an audit trail and gives other agents context if work is interrupted.

The comment should cover:

- **Approach** — the main steps in order.
- **Files in scope** — full paths; mark each as *new*, *modified*, or *deleted*.
- **Trade-offs and decisions** — anything non-obvious; justify the choice.
- **Out of scope** — things an implementer might reasonably expect to be included but are deliberately deferred.
- **Open questions** — if any remain after `/preflight-ticket`; flag them for the user to resolve.

Use the content format `markdown` when posting.

If you discover mid-implementation that the plan needs to change materially, **post a revised plan as a follow-up comment** rather than silently diverging. A mid-flight pivot is a decision the ticket should record.

### 3. Implementation

Work through the plan with the following discipline:

#### Scope discipline

- **Stay inside the ticket.** If you find unrelated problems (typos elsewhere, an outdated cross-link in an adjacent file), note them as a follow-up ticket rather than opportunistically fixing them here.
- **Atomic commits.** Each commit must represent one logical change. If your plan has three distinct change-sets, plan for three commits.
- **No sneaking changes in.** Configuration changes, dependency upgrades, or refactors not in the plan need their own ticket.

#### Documentation, skill, and configuration discipline

This is most of what this repository does:

- **Cross-link hygiene** — any new document or skill should link to its related documents; any document referenced elsewhere should back-reference where appropriate.
- **NZ English** — "behaviour", "organisation", "specialised", "colour", "centre". See [`AGENTS.md`](../../AGENTS.md).
- **Glossary consistency** — if you introduce a specialised term, add it to [`glossary.md`](../../glossary.md) and use the glossary form consistently.
- **Skill structure** — new or modified skills under [`agents/skills/`](../../agents/skills/) follow the [`skill-creator`](../skills/skill-creator/SKILL.md) conventions: concise SKILL.md with clear invocation triggers; detail lives in `references/`.
- **Template fidelity** — if a ticket is of a templated type (bug, feature, task, etc.), the corresponding [`templates/tickets/<type>.md`](../../templates/tickets/) template is the source of truth for ticket structure. Templates themselves should only change with explicit approval.

#### Code discipline (if the change touches code)

Most this repository work is markdown. When a change touches an actual script, template file, or configuration that executes, the standard code-quality conventions apply:

- Follow the relevant skill: [`agents/skills/go-development/SKILL.md`](../skills/go-development/SKILL.md), [`agents/skills/php-development/SKILL.md`](../skills/php-development/SKILL.md), [`agents/skills/vue-development/SKILL.md`](../skills/vue-development/SKILL.md), etc.
- Apply the [`code-quality`](../skills/code-quality/SKILL.md) principles.
- Prefer small, readable changes; comments explain *why*, not *what*.

#### Tests are specifications

If the ticket's deliverable is a test, the ticket is the specification. Do not alter a test to make unrelated code pass. If an existing test fails and you believe the test is wrong, **stop and raise it with the user** — test modification requires explicit approval. See [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md).

### 4. Verification

Before handing off to `/review-code`:

- **Re-read the acceptance criteria.** For each AC, point to the specific artefact that satisfies it.
- **Walk the cross-links.** Every internal link you added or touched should resolve.
- **Spot-check consumer impact.** If you changed a skill or a process document that is copied or referenced in the consumer project, note the consumer-impact explicitly in a ticket comment. Do not forward-port in this ticket unless that is the ticket's deliverable.
- **Re-read your diff.** Does each hunk serve the acceptance criteria? Is anything lingering that should be a separate ticket?

### 5. Handoff

When implementation is complete:

1. **Post an *implementation complete* comment** on the ticket summarising what was done — a bullet list of the file changes and the AC each one satisfies.
2. **Recommend the next command** — usually `/review-code` (unless the user has already indicated a shortcut).
3. **Do NOT commit or merge.** Staging and commit belong to `/commit`. The branch remains local until `/pull-request` pushes it.

## Rules

- Do NOT modify tests or test expectations without explicit user approval.
- Do NOT perform destructive operations (`rm -rf`, force-push, branch deletion, mass file moves) without pausing for explicit approval.
- Do NOT change templates under [`templates/`](../../templates/) opportunistically — templates are load-bearing; their change should be its own ticket.
- Do NOT transition the ticket past *In Progress*. `/review-code` → *In Review* (conceptually) and `/pull-request` does the actual transition.
- Do NOT commit. That is `/commit`'s job.
- DO keep the ticket updated in real time with plan, decisions, and completion.
- DO follow the repository's load-bearing rules listed in [`AGENTS.md`](../../AGENTS.md).
- All written outputs use NZ English.

## See also

- [`agents/commands/preflight-ticket.md`](./preflight-ticket.md) — runs before this command.
- [`agents/commands/review-code.md`](./review-code.md) — runs after this command.
- [`agents/commands/fix-findings.md`](./fix-findings.md) — runs between review passes.
- [`agents/commands/commit.md`](./commit.md) — when ready to commit.
- [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md) — load-bearing safety rules.
- [`agents/skills/code-quality/SKILL.md`](../skills/code-quality/SKILL.md) — readability principles.
- [`knowledge/process/definition-of-ready.md`](../../knowledge/process/definition-of-ready.md) — the gate this command trusts is already passed.
- [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md) — comment conventions.
- [`AGENTS.md`](../../AGENTS.md) — repository-level conventions.
