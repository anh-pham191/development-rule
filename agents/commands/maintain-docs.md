# Maintain Documentation

Perform a documentation-maintenance pass across the repository to keep the durable artefacts consistent with the current state of this repository and to keep cross-links sound. Run this between tickets, after completing a piece of work, or periodically during long implementation stretches.

## Context

This repository is the canonical source for cross-cutting AI-agent configuration, skills, knowledge, and templates shared across the organisation projects. When a skill, playbook, or convention changes, the surrounding references must change with it — and the consumer projects that copy this repository content need to know a forward-port may be warranted on their side. Stale documentation is worse than no documentation; it actively misleads.

This command sweeps the repository for drift, contradictions, and broken references, and flags items that consumers (consumer projects, and any other downstream repo) should consider forward-porting.

It is deliberately scoped to a **review and report** pass — it identifies drift, proposes fixes, and on user approval applies the trivial ones. Non-trivial restructures are still separate tickets.

## Scope

The sweep covers:

### Core knowledge and process

- [`knowledge/philosophy/`](../../knowledge/philosophy/) — the durable "why" documents (agentic development, API design, code review, service-oriented architecture, technology selection, and the language-specific philosophies).
- [`knowledge/process/`](../../knowledge/process/) — Definition of Ready, git commits, tickets.
- [`knowledge/playbooks/`](../../knowledge/playbooks/) — operational playbooks.

### Agent configuration

- [`agents/skills/`](../../agents/skills/) — every skill's `SKILL.md` should have clear invocation triggers and reference its `references/` files correctly.
- [`agents/commands/`](../../agents/commands/) — this directory and its commands.
- [`agents/subagents/`](../../agents/subagents/) — subagent personas.
- [`agents/mcp/`](../../agents/mcp/) — MCP server notes.
- [`agents/hooks/`](../../agents/hooks/) — hook configuration, if any.

### Templates and scaffolding

- [`templates/tickets/`](../../templates/tickets/) — should match the your ticket system field definitions in [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md).
- [`templates/containerising-go-services/`](../../templates/containerising-go-services/) — should match [`knowledge/conventions/containerising-go-services.md`](../../knowledge/conventions/containerising-go-services.md) if that document exists.
- [`templates/commit-message.txt`](../../templates/commit-message.txt) — should match the commit convention in [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md).

### Top-level artefacts

- [`README.md`](../../README.md) — repository structure overview, getting-started pointers.
- [`AGENTS.md`](../../AGENTS.md) — operational rules for agents.
- [`glossary.md`](../../glossary.md) — the terminology index; should reflect terms introduced or retired by recent changes.

## Process

### 1. Identify the recent change surface

Before sweeping everything, narrow to what has changed recently. Look at:

- The last few merged branches (`git log main --oneline -30`).
- The current working branch's diff, if relevant.
- Any ticket just completed (comments, linked tickets).

Changes to these files indicate a likely documentation-sweep target:

| If this changed… | …check these downstream effects |
|------------------|---------------------------------|
| A skill's `SKILL.md` | Its invocation description still correct; consumers that auto-load may need a refresh |
| A `knowledge/process/` document | `templates/tickets/` consistency; `agents/commands/` references; `AGENTS.md` references |
| A `knowledge/philosophy/` document | Skills that reference it; `glossary.md` entries |
| `glossary.md` | Terminology usage across `knowledge/`, `agents/`, `AGENTS.md`, `README.md` |
| `agents/skills/skill-creator/` | Every other skill's structural conformance |
| A template under `templates/` | The process document the template serves |

### 2. Sweep passes

Run each pass and record findings:

#### Pass A — Cross-link integrity

For every markdown document in the repository:

- Internal links resolve (no 404s to moved or deleted files).
- Anchors exist (no `#section-that-was-renamed`).
- Relative paths are correct after any directory restructure.

For documents that moved: ensure back-links updated.

#### Pass B — Terminology consistency

- Every specialised term used in multiple documents matches its [`glossary.md`](../../glossary.md) entry.
- Any term introduced recently has a glossary entry.
- Retired terminology (if any) has been replaced everywhere.

#### Pass C — Skill and command coherence

- Every skill's invocation description matches the situations it is actually useful in.
- Every command's *See also* section resolves.
- Every command's *Rules* section doesn't contradict another command's rules (e.g. no two commands both claiming to own "commit").
- The `agents/commands/` set matches what is documented in `AGENTS.md` and `README.md`.

#### Pass D — Template-to-process alignment

- Each your ticket system template ([`templates/tickets/<type>.md`](../../templates/tickets/)) reflects the current field definitions in [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md).
- [`templates/commit-message.txt`](../../templates/commit-message.txt) matches [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md).

#### Pass E — Structural consistency

- `AGENTS.md` references every top-level area under its "Where to put new content" section (or equivalent).
- `README.md` structure section matches the actual repository tree.
- No dead directories (directory exists but `README.md` not referenced anywhere).

#### Pass F — NZ English and style

- Spot-check recently added documents for NZ English spellings.
- Flag obvious style drift.

### 3. Consumer-project forward-port awareness

This repository changes propagate to consumer projects (consumer projects) **by forward-port, not by live dependency**. For each material change identified in the sweep, note:

- **Is this change worth forward-porting?** (A trivial typo fix usually isn't; a skill behaviour change is.)
- **Which consumers are likely affected?** (Platform copies skills, commands, and subagents into `.cursor/`. Taxpay copies a subset.)
- **Has a forward-port ticket been raised downstream?** If the change is significant and no ticket exists, flag it — the consumer-side agents should pick it up in their own `maintain-docs` passes.

Do **not** change consumer repositories from within a this repository session. Forward-port is deliberately pull-based on the consumer side.

### 4. Triage findings

Group findings into:

- **Trivial** (broken link, typo, stale path) — can be fixed in this pass with user approval.
- **Consequential** (contradicting instructions, outdated process description, missing cross-references for a new concept) — propose as its own ticket.
- **Structural** (documents that need to move, merge, or split) — propose as its own ticket with explicit scope.

### 5. Present findings

Output:

```
Documentation maintenance pass — [date] — branch [branch-or-main]

Trivial fixes (proposed for in-pass application):
  - [file:line] [one-line description]
  - ...

Consequential findings (propose as new tickets):
  - [area] [description] — recommended ticket type: [bug/task/improvement]
  - ...

Structural findings (propose as new tickets):
  - [area] [description] — recommended approach: [split/merge/rename]
  - ...

Consumer-project forward-port candidates:
  - [change] affects [consumer(s)] — recommend flagging for their maintain-docs pass
  - ...
```

### 6. Apply trivial fixes (on approval)

For the Trivial category only, on explicit user approval:

- Apply the fixes in small atomic change-sets.
- Each fix is its own commit if substantive; trivial link repairs can be grouped.
- Follow `/commit` discipline for all commit operations.

## Rules

- Do NOT make non-trivial changes in this pass. Propose them as tickets.
- Do NOT touch consumer repositories from within a this repository session.
- Do NOT restructure a document opportunistically — that is always a new ticket.
- Do NOT commit without explicit user approval for each commit.
- DO report thoroughly; silent drift-repair hides information.
- DO flag findings for consumer projects in the report, so their maintain-docs passes can pick them up.
- All written outputs use NZ English.

## See also

- [`agents/commands/commit.md`](./commit.md) — for committing trivial fixes.
- [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md) — commit discipline, atomicity, the "good commit is a small, coherent change" rule.
- [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md) — ticket conventions for any consequential or structural follow-up tickets raised.
- [`AGENTS.md`](../../AGENTS.md) — repository-level conventions this sweep enforces.
- [`glossary.md`](../../glossary.md) — the terminology index used in Pass B.
