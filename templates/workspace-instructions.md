# Personal projects workspace

These rules apply to every project in this workspace folder. A project's own instructions file adds to them; where the two conflict, the project file wins for that project.

The full text of every rule lives in `~/Documents/Projects/script/development_rule/`. The paths below are relative to that folder.

The three documents every change depends on are imported in full, so they are always in context:

@~/Documents/Projects/script/development_rule/knowledge/process/request-to-implementation-flow.md
@~/Documents/Projects/script/development_rule/knowledge/playbooks/modular-project-layout.md
@~/Documents/Projects/script/development_rule/knowledge/playbooks/preventing-incremental-debt.md

Read any other linked file when a step calls for it.

## Every change

- Follow `knowledge/process/request-to-implementation-flow.md` for any non-trivial change: plan → interrogate → plan → review → implement. No implementation from a vague spec.
- Every plan names what the change replaces and removes it in the same change, or records an explicit follow-up. See `knowledge/playbooks/preventing-incremental-debt.md`.
- New code goes in the feature folder that owns it, not in a technical-layer folder (`components/`, `utils/`, `logic/`). See `knowledge/playbooks/modular-project-layout.md`.
- Before editing a folder, read its instructions file (`AGENTS.md`) if it has one.
- Review against `knowledge/philosophy/code-review.md` and the Review Checklists in both playbooks above.

## New project

- Before scaffolding, lay the project out by the two imported playbooks above.
- Organise code by feature from the first change.
- Create the root `AGENTS.md`, plus a `CLAUDE.md` whose only line is `@AGENTS.md`, so every agent tool loads the same file.
- Add a folder `AGENTS.md` only when that folder has rules of its own (invariants, contracts, traps); list it in the root index in the same change.
- The root `AGENTS.md` records the project's size budgets (start from the table in the debt playbook), an index of folder instructions files, and the line "Read the folder's instructions file before editing there."
- Keep a "changes since the last debt pass" count in the project's status or milestones file; run a debt pass every 10–15 changes.

## Writing

- Write prose, comments, commit messages and reports in New Zealand English.
- Commit messages follow `knowledge/process/git-commits.md`.
