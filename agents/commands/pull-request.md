# Pull Request

Push the branch and raise a pull request on GitHub. Transition the ticket to **In Review**.

## Context

This is the final agent-driven step in the workflow. All implementation, review, fix passes, documentation maintenance, and commits should be complete; the work is ready for human peer review on the PR.

This repository is hosted on GitHub. `gh` is the canonical CLI. If `gh` is unavailable, fall back to asking the user to create the PR manually and provide the URL; still do the your ticket system transition once the URL is confirmed.

## Pre-flight

Before pushing or raising the PR:

1. **Working tree clean.** Run `git status`; there must be no uncommitted changes.
2. **All commits belong to the ticket.** Walk `git log main...HEAD`; each commit should reference the ticket and be atomic per [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md).
3. **Branch convention.** Branch name follows this repository's `TICKET-XX-kebab-case-description` pattern.
4. **Review gate cleared.** At least one `/review-code` pass has been completed and either no significant findings remain or the user has explicitly decided to carry them through human review.
5. **Ticket status.** Ticket is *In Progress*. This command transitions it to *In Review*.

If any pre-flight item fails, stop and surface the issue to the user before pushing.

## Process

### 1. Push the branch

```
git push -u origin $(git branch --show-current)
```

If the branch already tracks origin, `git push` suffices. The `-u` is a first-push convenience.

### 2. Create the PR via `gh`

Draft the PR title and description **before** calling `gh`. Show them to the user; apply any edits; then create.

**Title convention:**

```
TICKET-XX: Imperative-mood summary of the change
```

Same convention as commit subjects — imperative mood, capitalised, under ~72 characters, no trailing period.

**Description structure** (in order):

1. **Summary** — 1–3 sentences naming what changed and why.
2. **What's new / changed** — bullet list of files and their purpose. Group by area (e.g. *new commands*, *updated philosophy*, *template refresh*).
3. **Decisions and trade-offs** — any choice worth flagging to the reviewer.
4. **Out of scope** — things the reviewer might reasonably expect and that are deliberately deferred, with pointers (follow-up ticket reference or explanatory note).
5. **Consumer-project forward-port notes** — if the change is material and consumers (consumer projects) are likely to forward-port, name them.
6. **Review checklist** — the specific things you want the human reviewer to focus on. (Not a comprehensive restatement of `/review-code`'s findings — a targeted list.)
7. **Closes / relates** — `Closes TICKET-XX` for the primary ticket; `Relates to TICKET-YY` for linked tickets.

Example shape:

```markdown
## Summary

One to three sentences.

## What's changed

- `path/to/file.md` — purpose
- ...

## Decisions

- Chose X over Y because ...

## Out of scope

- Z is deferred to TICKET-NN

## Forward-port

- platform: worth considering (skill-X is copied there)
- taxpay: no impact

## Reviewer focus

- `path/to/file.md` — the [specific concern]

Closes TICKET-XX
```

Then:

```
gh mr create --title "TICKET-XX: Subject" --description "$(cat <<'EOF'
<description from above>
EOF
)" --remove-source-branch --squash-before-merge=false
```

Adjust flags as appropriate:

- `--remove-source-branch` — delete the branch on merge (the default for this repository).
- `--squash-before-merge=false` — preserve commit history for atomic commits. If a single-commit ticket, a squash default is fine.
- `--assignee`, `--reviewer`, `--draft` — use as directed by the user.

### 3. Transition the ticket

Once the PR is created and the URL is known:

- Transition the ticket to **In Review** via the ticket-system transition tool.
- Post a PR-link comment on the ticket:

  > **Pull Request:** `[URL]`
  > **Branch:** `[branch-name]`
  > **Commits:** `[N]` — `[one-line summary of each if small number]`
  > **Awaiting:** human review.

### 4. Report back

In-chat, confirm:

- The PR URL.
- The your ticket system transition.
- Any outstanding items the user should know about (e.g. a non-blocking deferred finding, a consumer-project forward-port note).

## Rules

- Do NOT push without a clean working tree and all commits on the ticket.
- Do NOT force-push unless the user explicitly asks for it, and not without a pause-for-confirmation.
- Do NOT merge the PR. Merging is a human action on the GitHub UI.
- Do NOT transition the ticket to *Done*. That is the human's action after the PR merges.
- DO draft the PR description before creating; show it to the user.
- DO use `gh` as the primary CLI; fall back to manual creation if unavailable.
- All written outputs use NZ English.

## See also

- [`agents/commands/commit.md`](./commit.md) — the step before this one.
- [`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md) — commit-message and branch conventions.
- [`knowledge/process/ticket-structure.md`](../../knowledge/process/ticket-structure.md) — ticket-comment conventions.
- [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md) — rules around destructive operations (force-push, branch deletion).
