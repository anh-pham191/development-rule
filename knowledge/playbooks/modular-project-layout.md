# Modular Project Layout

How to lay out a codebase so that it stays easy to navigate as it grows one approved change at a time, with short per-folder agent instructions where a folder has rules of its own.

## When to Use This Playbook

Use it when you start a project, when you write the plan for a change that adds a new area, and in every [debt pass](./preventing-incremental-debt.md#debt-passes). It pairs with [Preventing Incremental Debt](./preventing-incremental-debt.md): that playbook keeps files small; this one keeps the folders and the agent instructions navigable.

### Scope

This playbook organises code **inside one codebase, app or service**. It is not about splitting a system into microservices or separate deployables. A large system that needs service boundaries decides them through its own architecture decisions (see [ADR Documentation](../../agents/skills/adr-documentation/SKILL.md) and [Service-Oriented Architecture](../philosophy/service-oriented-architecture.md)); the guidance here then applies inside each service.

## Why It Exists

Splitting large files fixes file size but not navigation. A project built through many small changes can end with a flat technical-layer layout — a `components/` folder of over a hundred files, a `logic/` folder of nearly as many — and one root agent instructions file that has grown to tens of kilobytes because every rule and approval was appended to it. An agent then cannot tell which files belong to the feature it is changing, reads far more than it needs, and misses the one rule that applies to the folder it is editing because it is buried among hundreds that do not.

The fix is structural: feature folders from the first change, and small instruction files that sit next to the code they govern.

## 1. Feature Folders from Day One

Group code by **feature or domain**, not by technical layer.

```
Layer-first (avoid)              Feature-first (prefer)
src/                             src/
├── components/  (100+ files)    ├── billing/
├── logic/       (90+ files)     │   ├── InvoiceList.tsx
├── hooks/                       │   ├── useInvoices.ts
└── utils/                       │   ├── tax.ts
                                 │   └── tax.test.ts
                                 ├── accounts/
                                 │   └── …
                                 └── shared/
                                     └── …
```

- **A feature folder holds everything for its feature together:** components, logic, hooks, types and tests. Changing a feature means working in one folder.
- **`shared/` is small and earned.** Only code that two or more features genuinely use goes there. Code used by one feature stays in that feature, even if it looks reusable.
- **Split by budget.** When a folder passes its [folder budget](./preventing-incremental-debt.md#budgets), or stops having one clear responsibility you can name in a sentence, split it into sub-features. The budget is a prompt to act, not a hard limit.
- **Layer folders inside a feature are fine** when the feature is large (`billing/ui/`, `billing/logic/`); the top level is still by feature.
- **Follow the framework's required layout** where one exists (routing folders, generated code, language packaging rules), and organise by feature beneath it.

### Moving files is a pure refactor

Reorganising into feature folders changes no behaviour, so it follows the [pure-refactor rule](./preventing-incremental-debt.md#debt-passes): test bodies stay unchanged, and the only test changes are **import-path-only test edits**, named as such in the plan and the report so reviewers can confirm no assertion changed. Keep a move in its own change, separate from any behaviour change.

## 2. Per-Folder Agent Instructions, Only Where They Fit

A folder gets its own short agent instructions file (such as `AGENTS.md`) **only when it has rules of its own** that an agent must know before editing there: security invariants, protocol or contract rules, data-format compatibility, known traps. Folders with nothing special get no file — every file is noise that every agent pays for.

A folder instructions file stays within its [budget](./preventing-incremental-debt.md#budgets) and says:

- **What the folder owns** — its responsibility in a sentence or two, and what it deliberately does not do.
- **The invariants that apply here** — the rules a change in this folder must keep, each stated once.
- **The traps** — what has gone wrong before or is easy to get wrong.

Example:

```markdown
# billing/

Owns invoices, tax calculation and payment records. Does not send email; it emits events that `notifications/` handles.

## Invariants
- Money is stored as integer minor units, never floating point.
- An issued invoice is never edited; corrections are credit notes.

## Traps
- Rounding happens once, on the invoice total, not per line.
```

### The root file

The root instructions file keeps what is project-wide:

- standing rules (safety, approval policy — what needs human approval, no-commit rules);
- the threat model;
- quality gates;
- code shape (budgets, comments, layout);
- **an index of folder instructions files**, with one line per folder saying what it covers;
- the instruction: **"Read the folder's instructions file before editing there."**

The explicit index and instruction make the layout work whether or not the agent tool loads nested instruction files on its own.

### One home per rule

Each rule lives in **exactly one file**: the folder file if it applies only there, the root file if it applies project-wide. Never copy a rule into a second file — copies drift, and an agent cannot tell which copy is current. Link to the home instead.

### Reviewers follow the index

Moving invariants out of the root file must not mean they stop being checked. A reviewer reads the root file, follows the index to the instructions file of every folder the change touches, and checks each invariant there. When an invariant moves from the root to a folder, the root index entry is updated in the same change.

## Rules for Every Change

Add these to each change's plan and review:

1. **New code goes in the owning feature folder.** If no folder owns it, the plan names the new folder; `shared/` only for code two or more features use.
2. **New folder-specific rules go in that folder's instructions file**; project-wide rules go in the root file. Never both. A new folder instructions file is added to the root index in the same change.
3. **Watch folder budgets.** A change that pushes a folder past budget splits it first (a pure refactor), or records why not.
4. **Shipped approvals are history**, not rules — they leave the instructions files (see [Preventing Incremental Debt](./preventing-incremental-debt.md#7-keep-standing-rules-apart-from-per-change-approvals)).

## Review Checklist

- Is new code in the feature folder that owns it, not in a technical-layer folder or `shared/` by default?
- Does a new folder-specific rule live in that folder's instructions file (or the root file if project-wide), and only there?
- Is every new folder instructions file listed in the root index? Are any rules duplicated across files?
- Did any flat technical-layer folder (`components/`, `utils/`, `logic/`, …) grow, or any folder pass its budget?
- For a file move: are the only test edits import-path-only, and named as such?
- Were the invariants in the instructions file of every folder the change touches checked?

## Anti-Patterns

- **Layer-first top level** (`components/`, `logic/`, `utils/`) that grows a file per change until no one can find anything.
- **`shared/` as a dumping ground** for code only one feature uses.
- **An instructions file in every folder** "for consistency", most of them empty restatements of the root.
- **The same rule in the root and a folder file**, slowly diverging.
- **Moving files and changing behaviour in one change**, so reviewers cannot tell a moved line from an edited one.

## Related Documents

- [Preventing Incremental Debt](./preventing-incremental-debt.md) — file budgets, removal of superseded code, debt passes
- [Code Quality](../../agents/skills/code-quality/SKILL.md) — readability principles
- [Request-to-Implementation Flow](../process/request-to-implementation-flow.md) — where layout decisions enter the plan
- [Service-Oriented Architecture](../philosophy/service-oriented-architecture.md) — boundaries between services, out of scope here
- [Glossary](../../glossary.md) — Feature Folder, Folder Instructions File
