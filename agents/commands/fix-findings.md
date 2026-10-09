# Fix Review Findings

Address findings from a `/review-code` pass. The user will point you at the review — typically the most recent `/review-code` output in conversation, or a finding-comment on the ticket.

## Context

This command runs between `/review-code` and the next `/review-code` pass (or `/commit`, if no further review is required). The ticket stays in *In Progress* throughout.

The severity model that findings use (**Critical / High / Medium / Low**) is defined in [`knowledge/philosophy/code-review.md`](../../knowledge/philosophy/code-review.md). That is the source of truth for what each level means and how the author should respond.

## Inputs to read

1. **The findings themselves.** Read them carefully before touching anything. Group them by severity and by file.
2. **The original ticket** — for acceptance criteria and scope. .
3. **Any file the findings point at** — re-read in full; don't fix blind.
4. **Relevant knowledge or skills** — especially if a finding cites a rule in [`AGENTS.md`](../../AGENTS.md), a skill under [`agents/skills/`](../../agents/skills/), or a philosophy under [`knowledge/philosophy/`](../../knowledge/philosophy/).

## Process

### 1. Classify each finding

For every finding, decide one of:

- **Fix** — agree with the finding; apply the change.
- **Push back** — disagree with the finding; document the reasoning and ask the user to adjudicate before proceeding.
- **Defer** — finding is valid but out of scope for this ticket; capture as a follow-up.
- **Clarify** — finding is unclear; ask the reviewer (or user) for specifics before acting.

Present the classification to the user **before** making any file changes for the disagree/defer/clarify cases. For Fix cases you may proceed, but surface your fix plan first if the fix is non-trivial.

### 2. Apply fixes by severity

Work through Critical findings first, then High, Medium, Low — and fix **all** of them. Review is capped at two cycles (see [Review Cycle Limit](../../knowledge/philosophy/code-review.md#review-cycle-limit)), so every pass must leave nothing behind; minor findings left over are what force a third cycle.

- **Critical, High, Medium, Low** — fix each one. Severity sets the order, not whether it gets fixed.
- **Push back or defer** only with the reasoning recorded and the user's agreement (step 1) — never as a way to skip a minor fix.

Each fix should be **minimal and targeted**:

- Make the smallest change that addresses the finding.
- Don't refactor adjacent code while you're there; that becomes its own ticket.
- Don't introduce new cross-cutting concerns — a finding about a missing cross-link in document A is not an invitation to audit cross-links across the whole repo.

### 3. Add a regression test per fix

Every fix that changes behaviour gets a **new** regression test: one that fails without the fix and passes with it. Write the test first where practical, watch it fail, then apply the fix. This proves the fix works and stops it regressing in the next cycle.

Where a finding has no executable behaviour to test (a typo in prose, a comment, a documentation link), skip the test and say so in the fix report.

### 4. Test-modification discipline

Adding new regression tests is expected. *Modifying* an existing test is not: if addressing a finding would require changing an existing test, stop. Test changes need explicit user approval — see [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md). Surface the tension to the user:

- State the finding.
- State what test would need to change.
- Explain why the code-side fix alone is insufficient.
- Wait for direction.

### 5. Verify

After each fix, re-read the affected section and confirm the fix does what it says. For documentation changes, walk the cross-links. For code changes, mentally trace through the logic.

If the work is in a directory that has automated checks (lint, format, tests), run them. Most of this repository is plain markdown — the verification is reading and cross-link walking.

### 6. Report back

Post a **fix-report comment** on the ticket summarising the pass:

> **Review-pass response — [pass number]**
>
> | Severity | Raised | Fixed | Regression tests added | Pushed back | Deferred | Clarifying |
> |----------|--------|-------|------------------------|-------------|----------|------------|
> | Critical | n      | n     | n                      | n           | n        | n          |
> | High     | n      | n     | n                      | n           | n        | n          |
> | Medium   | n      | n     | n                      | n           | n        | n          |
> | Low      | n      | n     | n                      | n           | n        | n          |
>
> **Notes:**
> - `[For each fix without a regression test: which, and why no test applies]`
> - `[For each pushed-back finding: which, why, awaiting decision]`
> - `[For each deferred finding: which, follow-up ticket reference if created]`
> - `[For each clarifying finding: which, question posed]`
>
> **Next step:** `[/review-code for cycle 2 / ready for /commit / extra cycle requested because … / awaiting user decision on Xs]`

Then recommend the next command to the user:

- **After cycle 1** — `/review-code` for the second and final cycle.
- **After cycle 2** — `/commit`. Recommend a third cycle only when the issue is genuinely big (e.g. cycle 2 raised a Critical finding, or a fix needed a substantial redesign), and state the reason.

## Rules

- Do NOT commit. This command only fixes; commit is separate.
- Do NOT modify tests without explicit user approval.
- Do NOT silently disagree with a finding — either fix it or document the pushback on your ticket system.
- Do NOT bundle unrelated improvements with fixes — scope discipline persists.
- DO fix every finding, whatever its severity, unless the user has agreed to a pushback or deferral.
- DO add a regression test for every fix that changes behaviour.
- DO apply the minimal change that resolves each finding.
- DO re-run cross-link checks and any automated verification after changes.
- All written outputs use NZ English.

## See also

- [`agents/commands/review-code.md`](./review-code.md) — produces the findings this command consumes.
- [`agents/commands/commit.md`](./commit.md) — runs once findings are resolved.
- [`knowledge/philosophy/code-review.md`](../../knowledge/philosophy/code-review.md) — the severity model and author-response table.
- [`agents/skills/agent-safety/SKILL.md`](../skills/agent-safety/SKILL.md) — especially the no-test-modification rule.
- [`agents/skills/code-quality/SKILL.md`](../skills/code-quality/SKILL.md) — readability principles for minimal, targeted fixes.
