# Review

Review completed work against the ticket's acceptance criteria, the repository's standards, and the load-bearing conventions. The user will provide a ticket key (e.g. `/review-code TICKET-105`).

This command **presents findings**. It does not give an *approved* or *not approved* verdict — that decision belongs to the user.

## Delegation (preferred)

**Prefer delegating the review to the [`peer-reviewer`](../subagents/peer-reviewer.md) subagent.** The subagent runs in an isolated context with no knowledge of the implementation conversation — this eliminates self-review bias. It is configured `readonly: true`, so it cannot modify files.

When delegating, pass the subagent:

- The ticket key and a pointer to the ticket on your ticket system ().
- The branch name.
- A summary of what was changed (file list, one-line each).
- The acceptance criteria, extracted from the ticket.
- Any non-obvious context the reviewer should know (e.g. *"the ticket's AC 3 is partially deferred to a follow-up — see your ticket system comment X"*).

If delegation is not practical (e.g. the user has asked for a specific, quick review in the current chat), perform the review in-line using the same checklist below.

## Calibrate review dimensions to work type

This repository is primarily markdown. Review emphasis shifts with the artefact type:

- **Skill or playbook** — invocation triggers clear? structure per [`skill-creator`](../skills/skill-creator/SKILL.md)? cross-links intact? no contradiction with adjacent skills?
- **Process or philosophy document** — consistent with other process/philosophy docs? terminology matches [`glossary.md`](../../glossary.md)? examples concrete and correct?
- **Agent configuration** (`agents/commands/`, `agents/subagents/`, `agents/mcp/`, `agents/hooks/`) — instructions unambiguous? delegation and tool-use rules explicit? safety rules intact?
- **Template** (under [`templates/`](../../templates/)) — placeholders clear? consistent with the your ticket system field definitions and the DoR?
- **`AGENTS.md` / `README.md` / `glossary.md`** — structural changes? new sections well-placed? index entries complete?
- **Code** (rare — scripts or configuration that executes) — apply the relevant language skill (go, php, vue, js) and [`code-quality`](../skills/code-quality/SKILL.md).

## Inputs to read

1. **The ticket** in full — description, acceptance criteria, risk fields, all comments (implementation plan, decisions, review passes).
2. **The diff** — `git diff main...HEAD` or the branch-to-base diff.
3. **Every changed file, read in full** — not just the diffed sections. Context matters.
4. **[`AGENTS.md`](../../AGENTS.md)** — for the repository-level conventions against which the change is assessed.
5. **[`knowledge/philosophy/code-review.md`](../../knowledge/philosophy/code-review.md)** — the severity model and the definition of what each level means.
6. **Relevant philosophy and skills** — if the change is in a Go skill's scope, load [`knowledge/philosophy/go-development.md`](../../knowledge/philosophy/go-development.md); if it's a ticket-template change, load [`agents/skills/ticket-management/SKILL.md`](../skills/ticket-management/SKILL.md); etc.
7. **[`glossary.md`](../../glossary.md)** — for terminology consistency.

## Checklist

For every review, walk these categories. Skip categories genuinely irrelevant to the change, but note that you skipped them.

### 1. Acceptance criteria

For each AC in the ticket:

- **Met** — pointer to the specific artefact that satisfies it.
- **Partially met** — what is and isn't covered; is the gap intentional (with a follow-up) or accidental?
- **Not met** — why; what's missing.

### 2. Correctness

- Documentation accuracy — does what the document says actually hold?
- Example correctness — every example runs / compiles / resolves where applicable.
- Cross-link integrity — internal links resolve.
- Glossary alignment — terms used consistently with [`glossary.md`](../../glossary.md).

### 3. Conventions

- NZ English throughout.
- Commit-message convention followed if a commit message is in scope ([`knowledge/process/git-commits.md`](../../knowledge/process/git-commits.md)).
- Skill structure per [`skill-creator`](../skills/skill-creator/SKILL.md) if a skill was added or modified.
- Template structure per the existing templates if a template changed.

### 4. Safety-relevant changes

- Any change to [`agents/skills/agent-safety/`](../skills/agent-safety/) is critical. Review with extreme care; a weakening of a safety rule is a Critical finding.
- Any change to a command under `agents/commands/` that removes or softens a "do NOT" rule is Critical unless explicitly justified.

### 5. Documentation impact

- If the change introduces a new concept: is it in the glossary and linked from the right places?
- If the change changes a process: is the process document updated? Are the templates updated?
- If the change affects a skill: is the skill's invocation description still accurate? Does the SKILL.md still lead to the right references?

### 6. Consumer-project impact

This repository content is copied or referenced by consumer projects (consumer projects, and others). For each changed file, assess:

- Is this something a consumer project has copied? If yes, note that a forward-port may be required downstream (not in this ticket — flag for the consumer side's `maintain-docs` pass).
- Is this a *tool-agnostic* change, or has a tool-specific assumption crept in? This repository aims to stay tool-agnostic at the core; anything tool-specific belongs in the consumer's `.cursor/` or equivalent.

### 7. Scope

- Every changed file serves the acceptance criteria.
- No opportunistic changes unrelated to the ticket.
- No orphaned files (created but not referenced anywhere).

## Present findings

Categorise every finding by the severity model defined in [`knowledge/philosophy/code-review.md`](../../knowledge/philosophy/code-review.md):

- **Critical** — blocks commit. Safety rule weakened, factual error, broken cross-link to a load-bearing document, violated acceptance criterion without explicit deferral.
- **High** — significant issue. Bug in an example, missing test coverage where a test is warranted, violation of an established pattern or convention.
- **Medium** — improvement recommended. Maintainability concern, suboptimal structure, minor inconsistency.
- **Low** — minor suggestion. Style preference, optional enhancement, food for thought.

For each Critical and High finding:

- Quote the specific line or section.
- State the rule or expectation it violates (with a link where possible).
- Recommend a concrete change.

For Medium and Low findings, brevity is fine.

## Output

1. **Post the review as a comment** on the ticket. Use the categorisation above; lead with the count of Critical / High / Medium / Low findings.
2. **Summarise in-chat** the verdict shape: *Critical findings present → must be addressed*; *High findings present → should be addressed*; *Medium/Low only → ready at user's discretion*.
3. **Do NOT transition the ticket.** The human decides whether the work proceeds to `/fix-findings` or `/commit`.

## Rules

- Do NOT modify any file.
- Do NOT approve or decline — present findings only.
- Do NOT skip the safety-relevant category on any change.
- Do NOT review work where you have made substantive implementation decisions unless explicitly instructed — prefer delegating to the `peer-reviewer` subagent.
- DO read every changed file in full.
- DO point at specific lines for Critical and High findings.
- All written outputs use NZ English.

## See also

- [`agents/subagents/peer-reviewer.md`](../subagents/peer-reviewer.md) — the preferred reviewer.
- [`agents/commands/fix-findings.md`](./fix-findings.md) — the next step if findings exist.
- [`agents/commands/commit.md`](./commit.md) — the next step if findings are trivial or none.
- [`knowledge/philosophy/code-review.md`](../../knowledge/philosophy/code-review.md) — the review philosophy and severity model.
- [`agents/skills/code-quality/SKILL.md`](../skills/code-quality/SKILL.md) — readability principles the reviewer applies.
