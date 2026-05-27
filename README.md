# development_rule

A personal collection of universal rules and philosophy for high-quality agentic software development.

This repository extracts the **tool-agnostic, domain-agnostic core** from a larger team toolbox: principles, conventions, and skills that apply to any project, in any organisation, regardless of language, stack, or ticket system.

## What's Inside

This repository separates **Knowledge** (for humans) from **Agents** (for AI).

### Knowledge — for humans (and agents that read it)

| Directory | Answers |
|-----------|---------|
| [`knowledge/philosophy/`](./knowledge/philosophy/) | "What do we value?" — agentic development, code review, API design, technology selection, service-oriented architecture, per-language principles (Go, PHP, Python, Rust, R, C, JavaScript, TypeScript, Node.js, React, Vue), and infrastructure-as-code (Terraform/OpenTofu, AWS foundations, networking, compute, data, observability & cost) |
| [`knowledge/process/`](./knowledge/process/) | "How do we do this activity?" — git commit conventions, the Definition of Ready |
| [`knowledge/playbooks/`](./knowledge/playbooks/) | "How do we handle this situation?" — workflow-loop prompts |

### Agents — operational configuration for AI

| Directory | Purpose |
|-----------|---------|
| [`agents/skills/`](./agents/skills/) | Task-specific instructions: `agent-safety`, `code-quality`, `skill-creator`, `adr-documentation`, `requirements-elicitation`, `milestones`, per-language skills (`go-development`, `php-development`, `python-development`, `rust-development`, `r-development`, `c-development`, `typescript-development`, `node-development`, `react-development`, `vue-development`), and infrastructure-as-code skills (`terraform-development`, `aws-foundations`, `aws-networking`, `aws-compute`, `aws-data`, `aws-observability-cost`) |

### Templates

| File | Use |
|------|-----|
| [`templates/commit-message.txt`](./templates/commit-message.txt) | Commit-message scaffold |

### Reference

- [`glossary.md`](./glossary.md) — terminology used across this repository
- [`AGENTS.md`](./AGENTS.md) — operational rules for AI agents working in this repo

## Guiding Principles

**Composable over monolithic.** Small, focused pieces that combine well. A skill should do one thing clearly.

**Tool-agnostic at the core.** Knowledge and skills are plain markdown. Tool-specific bindings (slash commands, MCP integrations, ticket-system templates) belong in the consuming project, not here.

**Evolved from use.** Nothing here is theoretical. Resources earned a place by solving a real problem.

## Using This Repository

Browse and copy what fits the project you're working on. Treat the contents as a starting point — adapt the wording, the conventions, and the examples to match your team's reality.

## Provenance

The content here is derived from a wider team toolbox. Domain-specific knowledge (a particular tax system, product area, ticketing project, MR workflow, etc.) and project-planning artefacts have been deliberately left out so the repository stays universal.

## Licence

[MIT](./LICENSE).
