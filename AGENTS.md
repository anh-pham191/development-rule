# AGENTS.md

Context for AI agents working in this repository.

## Purpose

`development_rule` is a personal, universal collection of rules and philosophy for agentic software development. The content is **tool-agnostic** and **domain-agnostic** — it does not bind to a particular language stack, ticket system, or organisation.

When an agent uses content from this repository in a consuming project, the agent (or a human) is expected to adapt the wording for that project's specifics — its ticket prefix, its CI provider, its review tool, its review terminology.

## Repository Structure

```
development_rule/
├── knowledge/
│   ├── philosophy/        # "What do we value?"
│   ├── process/           # "How do we do this activity?"
│   └── playbooks/         # "How do we handle this situation?"
│
├── agents/
│   └── skills/            # Task-specific instructions for AI agents
│
├── templates/             # Scaffolding (commit messages, etc.)
├── glossary.md            # Shared terminology
├── AGENTS.md              # This file
└── README.md              # Project overview
```

## Where to Put New Content

Ask: **"What question does this answer?"**

- Values and principles → `knowledge/philosophy/`
- Activity procedures → `knowledge/process/`
- Situational guidance → `knowledge/playbooks/`
- Task instructions for agents → `agents/skills/`
- Scaffolding to copy and adapt → `templates/`

If the content is **specific to a particular project, product, ticketing project, or business domain**, it does not belong here — keep it in that project's own repository.

## Working in This Repository

### Before Making Changes

1. Read [`README.md`](./README.md) for the guiding principles.
2. Check existing resources to avoid duplication.
3. Follow the structural patterns of nearby files.

### Skills

Skills live in [`agents/skills/`](./agents/skills/) and follow the [`skill-creator`](./agents/skills/skill-creator/SKILL.md) convention:

```
skill-name/
├── SKILL.md           # Required: frontmatter + instructions
├── scripts/           # Optional: executable helpers
├── references/        # Optional: detailed documentation
└── assets/            # Optional: templates, images, etc.
```

Keep `SKILL.md` concise; move detail into `references/`.

### Safety Requirements

Read and follow [`agents/skills/agent-safety/SKILL.md`](./agents/skills/agent-safety/SKILL.md). Key rules:

- **Never commit without explicit human approval.**
- **Never modify tests without permission** — tests are specifications.
- **Pause before destructive operations** — show the plan, wait for approval.

### Quality Standards

When contributing content here:

- **Composable over monolithic** — small, focused pieces.
- **Tool-agnostic at the core** — plain markdown; no binding to a particular stack.
- **Evolved from use** — add things that solve a real problem.

### Writing Conventions

- The author writes in **New Zealand English** (e.g. "behaviour", "organisation", "specialised", "colour", "centre"). Match it when editing existing files; new files in your fork can follow whatever convention you prefer.

### Commit Messages

This repository follows the convention documented in [`knowledge/process/git-commits.md`](./knowledge/process/git-commits.md). The short version:

- Subject in imperative mood, ≤ 72 characters.
- Body explains *why*, not *what*. Wrap at 72.
- Reference a ticket prefix when the work is tracked (`TICKET-XX: …`); plain `chore:`/`fix:` prefixes are fine for untracked trivial work.

## Key Files

- [`README.md`](./README.md) — repository overview and principles.
- [`glossary.md`](./glossary.md) — shared terminology.
- [`agents/skills/skill-creator/SKILL.md`](./agents/skills/skill-creator/SKILL.md) — how to create skills.
- [`agents/skills/agent-safety/SKILL.md`](./agents/skills/agent-safety/SKILL.md) — safety guardrails (read this).
- [`agents/skills/code-quality/SKILL.md`](./agents/skills/code-quality/SKILL.md) — code-quality principles.
- [`knowledge/process/git-commits.md`](./knowledge/process/git-commits.md) — commit conventions.
- [`knowledge/process/definition-of-ready.md`](./knowledge/process/definition-of-ready.md) — readiness gate before implementation.
- [`knowledge/process/request-to-implementation-flow.md`](./knowledge/process/request-to-implementation-flow.md) — mandatory flow from user request to implementation (plan → interrogate → plan with superpowers → review → subagent execution).
- [`knowledge/philosophy/agentic-development.md`](./knowledge/philosophy/agentic-development.md) — the framing principles for agent-assisted work.
- [`knowledge/philosophy/code-review.md`](./knowledge/philosophy/code-review.md) — review philosophy and the four-level severity model.
