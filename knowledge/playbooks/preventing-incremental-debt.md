# Preventing Incremental Debt

How to keep a long-running, agent-built project clean when it grows one small, approved change at a time.

## When to Use This Playbook

Use it on any project where agents ship many small changes in sequence — a feature, a tweak, a follow-up, each with its own plan, tests and review. Read it when you start such a project, when you write the plan for each change, and whenever the change count passes a checkpoint (see [Debt Passes](#debt-passes)).

## Why It Exists

A strict per-change process (plan, test first, independent review, verification) stops bugs and security regressions. It does **not** stop structural debt, because every step only asks "is this change correct?", never "is the whole still simple?". Each change is added on top of the last; nothing in the loop removes anything. After dozens of correct changes the project can still end up with:

- **Dead code from replaced features.** A later change supersedes an earlier one, the new path is built and tested, and the old command, handler or permission stays registered because no step asked for it to go. Unused entry points are also unused attack surface.
- **Very large files.** The entry point, the main view, the API layer and the validation module each grow past several thousand lines because every change adds "just one more" case to the file that already handles the area.
- **Comments that read like a changelog.** "Change 12: … change 19: … change 23: …" piles up above code that now does something else. The history lives in the plan and version control; the comment goes stale.
- **Several components for one concept.** A status indicator, then an activity indicator, then a badge variant, then another for a new screen — each added by a different change, each with its own spacing rules.
- **Layered style overrides.** Each design round adds rules on top of the previous round's instead of replacing them, so specificity fights and compensating margins appear.
- **Tests coupled to structure.** Tests that assert exact markup, class names or ordering break on every visual change, so existing tests are edited in almost every change and stop protecting behaviour.
- **Ever-growing agent instructions.** Per-change approvals appended to the instructions file every agent reads at start-up make every task slower and more expensive, for no benefit once the change has shipped.

None of these is a bug, and none is caught by a reviewer looking only at the diff. They are caught only by asking the questions below on purpose.

## Rules for Every Change

Add these to each change's plan and to its review.

### 1. Name what the change replaces, and remove it

The plan states what existing behaviour, code path, setting, command or permission the change supersedes. Removing the superseded pieces is part of the same change, with its own tests (e.g. "the old command is no longer registered", "the old setting is read once and migrated"). If removal must wait, record it as an explicit follow-up with the reason — never leave it implied.

### 2. Put new code where it belongs, not where it is easiest

Before adding to a file, check its size against the project's budget (see [Budgets](#budgets)). A file over budget does not take new feature code unless a reason for the exception is recorded (see [Budgets](#budgets)); otherwise extract the area being touched into its own module first (a pure refactor with tests unchanged), then add to the new module.

### 3. Extend a concept; don't add a parallel one

Before creating a component, helper or style for something that looks like an existing one (a mark, a badge, a dialog, a row), extend the existing one with a variant. Two things that look the same on screen should come from one piece of code with one set of spacing rules.

### 4. Replace styles, don't stack them

A design change edits or deletes the rules it changes. It does not add a more specific selector to win over the old rule. Shared values (colours, gaps, sizes) are tokens defined once.

### 5. Comments describe the present

A comment says what the code does now and why it is shaped that way. It does not say which change introduced or altered it. Change identifiers belong in the plan, the commit message and the pull request. When a change alters behaviour, it rewrites the comment rather than appending to it. (See [code-quality](../../agents/skills/code-quality/SKILL.md) §5.)

### 6. Test behaviour, not markup

Assert what a user or caller can observe — the text, the accessible role and name, the effect of an action, the value saved — not class names, element order or exact structure, unless that structure *is* the requirement. A visual change that keeps behaviour should leave behaviour tests untouched.

### 7. Keep standing rules apart from per-change approvals

The instructions every agent reads (e.g. `AGENTS.md`) hold only the standing rules: safety, the threat model, conventions, quality gates. A per-change approval lives with that change (its plan entry, ticket or pull request). Once the change has shipped, its approval is history and moves out of the start-up file.

## Budgets

Set these at project start in the project's instructions and check them in review. Starting points:

| What | Budget | Over budget means |
|------|--------|-------------------|
| Source file | ~800 lines | Extract before adding more |
| Component or function | Fits on one screen | Split by responsibility |
| Root application component | Wiring only: state, routing, composition | Move feature logic into feature modules or hooks |
| Agent start-up instructions | A few pages | Move history and finished approvals out |
| Components per visual concept | One, with variants | Merge |

Budgets are prompts to act, not hard failures. A file may exceed one for a stated reason; the reason is written down.

## Debt Passes

Every 10–15 changes, or before starting a large feature, run a debt pass. Keep a "changes since the last debt pass" count in the project's status or milestones file so the checkpoint is not missed. The pass is read-only first:

1. **Audit** — independent agents list, with file and line references:
   - code, commands, permissions, settings and styles nothing reaches any more;
   - files and components over budget;
   - parallel implementations of one concept;
   - history-style comments;
   - tests that assert structure rather than behaviour;
   - the size of the agent start-up instructions.
2. **Rank** — the human picks what to pay down, by risk and by how often the area changes. Dead entry points and permissions go first: they are both debt and attack surface.
3. **Pay down** — each item is its own change through the normal flow. A pure refactor keeps every existing test passing **without editing it**; that is what makes the refactor safe. Removing dead code may remove the tests that covered it, and says so. Rewriting a structure-coupled test into a behaviour test is its own change, kept separate from any refactor.

## Review Checklist

Reviewers add these questions to every change:

- Does the plan name what this change replaces, and is it removed (or an explicit follow-up recorded)?
- Did any file cross its budget? If so, was the area extracted first?
- Does the change add a second implementation of an existing concept?
- Do new styles replace old rules, or override them?
- Do new or edited comments describe the present only?
- Do new tests assert behaviour? Were existing tests edited for a change that should not alter behaviour?
- Did the change add to the agent start-up instructions? Does it belong there permanently?

## Anti-Patterns

- **"The tests pass, so it's clean."** Passing tests prove behaviour, not structure.
- **"We'll clean it up later"** with no recorded follow-up. Later never has a ticket.
- **Appending to the file that already handles the area** because it is the quickest place to make the test pass.
- **Recording every approval in the start-up instructions forever.**
- **Editing existing tests in nearly every change.** It is a signal the tests are coupled to structure.

## Related Documents

- [Request-to-Implementation Flow](../process/request-to-implementation-flow.md) — the per-change flow these rules plug into
- [Code Quality](../../agents/skills/code-quality/SKILL.md) — readability principles at the function level
- [Code Review Philosophy](../philosophy/code-review.md) — severity model for the review checklist
- [Agentic Development Philosophy](../philosophy/agentic-development.md) — Living Documentation and Iterative Refinement
- [Glossary](../../glossary.md) — Structural Debt, Debt Pass
