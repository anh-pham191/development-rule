# Commit

Stage and commit work on a ticket. The user will typically invoke this after `/review-code` and any `/fix-findings` passes have completed.

## Context

Commits are the durable record of changes. In this repository, each commit must be **atomic** (one logical change), carry a **ticket reference** in its subject, and follow the convention in [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md). The commit message is the audit trail.

This command **stages and drafts**; it does not finalise the commit until the user explicitly approves. That gating is a cardinal safety rule — see [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md).

## Pre-flight

1. **Ticket reference.** You need a ticket key. If the user didn't provide one, ask. The ticket should be in *In Progress*.
2. **Branch check.** Confirm the branch is not `main` and matches the ticket (`TICKET-XX-kebab-case-description`). If committing to `main`, stop.
3. **Working tree check.** Run `git status` and `git diff` to understand everything in play. Surface anything that looks out of scope.
4. **Scope check.** Every hunk staged should serve the ticket. Anything unrelated is a separate ticket, a separate commit, or both. Ask before bundling.
5. **Review gate.** Has a `/review-code` pass been run? If findings remain, confirm they are acknowledged and either fixed or explicitly deferred. Do not commit over unaddressed Critical findings.

## Process

### 1. Summarise the change

In-chat, before any git action:

- List files to be staged, grouped by logical concern.
- If there are multiple logical concerns, propose splitting into multiple commits with a proposed subject for each.
- Name anything deliberately *not* being staged (with the reasoning).

Wait for the user to confirm the grouping.

### 2. Stage explicitly

Use explicit `git add <path>` for each file. **Do not** use `git add -A` or `git add .` without explicit user approval for that specific invocation. This protects against staging files the user didn't intend (credentials, scratch files, work-in-progress in other directories).

### 3. Draft the commit message

Format:

```
TICKET-XX: Subject in imperative mood

Optional body explaining the why, wrapped at 72 characters.
Reference related tickets and decisions in the body where useful.

Closes TICKET-XX
```

Rules:

- **Subject ≤ 72 characters** including the `TICKET-XX:` prefix.
- **Imperative mood** — "Add", "Fix", "Refactor"; not "Added", "Fixes", "Refactoring".
- **Capitalise** the first word after the prefix.
- **No trailing period** on the subject.
- **NZ English** in subject and body.
- **Body explains the why**, not the what. The diff shows the what.
- **Footer** for `Closes TICKET-XX` on the primary ticket the commit finishes; `Relates to TICKET-YY` for linked tickets.

Present the drafted message to the user. Offer to revise. Do **not** commit yet.

### 4. Commit (on explicit approval only)

On user approval, run:

```
git commit -m "$(cat <<'EOF'
TICKET-XX: Subject

Body paragraph...

Closes TICKET-XX
EOF
)"
```

Always use the heredoc form to preserve formatting.

### 5. Post-commit your ticket system update

After the commit succeeds:

- Capture the commit hash (`git log -1 --format=%H`).
- Post a comment on the ticket:

  > **Commit:** `[hash]`
  > **Branch:** `[branch-name]`
  > **Message:** `[subject line]`
  >
  > `[optional: one-line note of any remaining work on this ticket]`

### 6. Report back

In-chat:

- Confirm the commit succeeded; show the hash and the subject line.
- Name any files not staged (with the reason).
- Recommend the next step — usually `/pull-request` if the ticket is done, or returning to implementation if further work remains.

## Rules (cardinal)

- **Never commit without explicit human approval in the current turn.** Approval from a previous turn does not carry forward. This is a cardinal safety rule; it does not bend. See [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md).
- **Never use `git add -A` or `git add .`** without explicit approval for that specific invocation.
- **Never commit to `main`.**
- **Never use `--amend`** unless the user explicitly requests it and the commit has not been pushed.
- **Never use `--no-verify`** to bypass hooks.
- **Never modify tests without permission** — tests are specifications.
- DO stage explicitly and draft the message for review.
- DO reference the ticket in the subject.
- DO keep commits atomic; split if needed.
- All written outputs use NZ English.

## A note on forward-port awareness

This repository content is copied into consumer projects (consumer projects). When that forward-port should happen is a separate concern — it is handled by [`/maintain-docs`](./maintain-docs.md), which sweeps for forward-port candidates across changes. This command deliberately does not block on forward-port considerations — the commit is local; forward-port is downstream.

## See also

- [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md) — the commit-message convention and atomicity philosophy.
- [`templates/commit-message.txt`](../../templates/commit-message.txt) — the commit-message template.
- [`agents/commands/pull-request.md`](./pull-request.md) — the next step after commits are in place.
- [`agents/commands/maintain-docs.md`](./maintain-docs.md) — where forward-port candidates are surfaced.
- [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md) — the cardinal safety rules that govern this command.
