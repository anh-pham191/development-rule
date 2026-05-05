# Git Commit Philosophy and Conventions

## 1. Philosophy

### 1.1 Why We Commit This Way

Our commit practices are shaped by a few core beliefs:

**Commits are permanent.** Once pushed, a commit becomes part of the project's immutable history. Unlike code that can be refactored or comments that can be updated, commit history is forever. This permanence demands intentionality.

**Every change needs a reason.** If work is worth doing, it's worth tracking. Commits without context (orphaned changes with no ticket, no rationale, no trace) create debt for future readers. We link commits to tickets not as bureaucracy, but as a discipline that forces clarity about *why* we're making changes.

**Change management is traceability.** Linking commits to tickets provides the audit trail teams (and compliance regimes such as ISO 27001) rely on: every change can be traced to an authorised request, reviewed through a defined process, and attributed to responsible parties. This isn't overhead; it's how control over the codebase is demonstrated.

**Commits should be atomic.** Each commit should represent a complete, coherent unit of work that can stand on its own. A ticket may require multiple commits to implement, and that's fine, provided each commit is atomic and follows our conventions. What matters is that every commit makes sense in isolation: it has a clear purpose, doesn't leave the codebase in a broken state, and can be understood (or reverted) independently.

**History serves the future.** We write commit messages and structure our history for the people (and agents) who will read it later, whether debugging a regression, understanding a decision, or onboarding to the codebase. The few minutes spent crafting a clear commit pay dividends across the project's lifetime.

### 1.2 The Role of Commit History

Git history serves multiple functions. Understanding these helps us make intentional choices about what we commit and how we describe it.

#### History as Documentation

Commit history tells the story of how the codebase evolved. A well-crafted history allows anyone (human or agent) to understand not just *what* the code does, but *how it came to be this way*. Each commit message is a chance to document intent, providing context that may not be obvious from the code alone.

Unlike inline comments that can drift out of sync, commit history is immutable. It captures the state of understanding at the moment the change was made. This makes it particularly valuable for understanding decisions made under constraints that no longer apply.

#### History as Debugging Tool

When something breaks, `git bisect` becomes invaluable, but only if commits are meaningful units. A history of atomic, well-scoped commits lets you pinpoint exactly when a bug was introduced. Conversely, large "work in progress" commits that bundle unrelated changes make bisection useless.

The goal: any commit can be checked out and should represent a coherent, working state (or at minimum, a coherent change set).

#### History as Audit Trail

In agentic development, provenance matters. We need to trace outputs back to their origins: *which ticket drove this change? What was the intent? Who (or what) approved it?*

Commit history, combined with ticket references, creates an auditable chain from requirement to implementation. This supports:
- **Compliance**: demonstrating that changes were authorised and tracked
- **Accountability**: understanding who made decisions and why
- **Recovery**: knowing what changed when investigating issues

The principle: *every commit should be traceable to a reason for existing*.

---

## 2. Commit Scope

### 2.1 Atomic Commits

A commit is atomic when it represents a single, complete logical change. This means:

- **Self-contained**: The commit makes sense on its own without requiring other commits to understand it
- **Coherent**: All changes in the commit serve the same purpose
- **Intentional state**: The codebase should be in a deliberate, understood state after the commit

Atomic doesn't mean small. A commit that adds a new feature with its tests, types, and documentation is atomic if all those pieces serve the same logical change. Conversely, a five-line commit that fixes a bug *and* tweaks an unrelated comment is not atomic.

**Intentionality over green builds**: A commit that introduces failing tests as part of TDD is valid because the failing state is intentional and understood. What we avoid is *accidental* breakage: commits that leave the codebase in a broken state through oversight rather than design. If tests fail after your commit, you should be able to explain why that's expected.

**The test**: Could you explain this commit's purpose in one sentence? Could it be reverted independently without breaking other work? Is the resulting state intentional? If yes to all three, it's likely atomic.

### 2.2 Multiple Commits Per Ticket

A single ticket often requires multiple commits. This is expected and encouraged when the work naturally decomposes into distinct logical steps. Common patterns:

- **Refactor-then-implement**: One commit refactors existing code to prepare for the change; a second commit implements the new behaviour
- **Infrastructure-then-feature**: One commit adds supporting infrastructure (types, utilities, configuration); a second uses it
- **Test-first (TDD)**: One commit adds failing tests that specify the expected behaviour; a subsequent commit implements the functionality to make them pass
- **Incremental progress**: Complex features built in stages, each commit representing a coherent milestone

Each commit in the sequence should still reference the ticket and be atomic in its own right.

### 2.3 Boundaries and Edge Cases

**Discoveries mid-work**: You're implementing TICKET-35 and discover an unrelated bug. Don't bundle the fix into your current commit. Options:
1. Create a new ticket for the bug and fix it in a separate commit
2. Note it for later and stay focused on the current ticket
3. If truly trivial (typo, obvious one-liner), a separate `fix:` commit may be acceptable

**Oversized tickets**: If a ticket requires so many commits that the relationship between them becomes unclear, the ticket is probably too large. Consider splitting the ticket rather than continuing with a sprawling commit history.

**Undersized tickets**: If multiple tickets are so small they'd each result in trivial commits, consider whether they should be consolidated. However, err toward keeping tickets separate. Small commits are rarely a problem; unclear ones are.

---

## 3. Commit Messages

### 3.1 Format Structure

Commit messages follow a structured format:

```
TICKET-123: Subject line in imperative mood

Optional body providing context, rationale, or details that don't
fit in the subject line. Wrap at 72 characters.

Optional footer for metadata like breaking changes or co-authors.
```

#### Subject Line

The subject line is the most important part. It's what appears in logs, blame output, and PR summaries.

- **Start with the ticket reference**: `TICKET-35: ` prefix links the commit to its authorising ticket
- **Use imperative mood**: Write as if completing the sentence "This commit will..." (e.g., "Add validation" not "Added validation" or "Adds validation")
- **Keep it under 72 characters**: The ticket prefix counts toward this limit
- **Capitalise the first word after the prefix**: `TICKET-35: Add feature` not `TICKET-35: add feature`
- **No trailing period**: The subject is a title, not a sentence

#### Body

The body is optional but encouraged for non-trivial changes. Use it to explain:

- **Why** the change was made (the ticket provides the "what", the body provides the "why")
- **Context** that future readers will need
- **Trade-offs** or alternatives that were considered
- **Anything surprising** about the implementation

Separate the body from the subject with a blank line. Wrap lines at 72 characters to ensure readability in terminals and git tools.

#### Footer

The footer is optional and used for structured metadata:

- **Breaking changes**: `BREAKING CHANGE: API endpoint renamed from /users to /accounts`
- **Co-authors**: `Co-authored-by: Name <email@example.com>`
- **References**: `Refs: TICKET-36, TICKET-37` (for related but not primary tickets)

Separate the footer from the body with a blank line.

### 3.2 Ticket References

#### The Convention

Every commit message begins with a ticket reference as a prefix:

```
TICKET-35: Add git commit philosophy documentation
```

The format is: `PROJECT-NUMBER: Description in imperative mood`

#### Why We Link Commits to Tickets

**Traceability in both directions**: From a commit, you can find the ticket that authorised and scoped the work. From a ticket, you can find all commits that implemented it. This bidirectional link is fundamental to our audit trail.

**Scope enforcement**: The ticket reference is a constant reminder that every commit should serve a defined purpose. If you can't name the ticket, question whether the work should be happening, or whether a ticket should be created first.

**Agentic context**: When an agent reads commit history, ticket references provide anchors for understanding *why* changes were made. The agent can retrieve the ticket for fuller context, including acceptance criteria, related discussions, and decision rationale.

**Automated tooling**: Ticket prefixes enable automation such as linking commits to tickets in your tracker, generating release notes, and filtering history by ticket. Consistent formatting makes this reliable.

#### When There's No Ticket

In general, if work is worth committing, it's worth tracking. However, genuinely trivial fixes (typos in comments, formatting) may not warrant a ticket. In these cases:

- Consider whether the fix can be bundled with related ticketed work
- If standalone, use a conventional prefix like `chore:` or `fix:` to indicate untracked minor work
- The bar for "too trivial to track" should be high. Err toward creating tickets.

### 3.3 Examples

#### Good Examples

**Simple change with clear subject:**
```
TICKET-35: Add git commit philosophy documentation
```
*Why it works*: Ticket reference, imperative mood, concise, self-explanatory.

**Subject with body for context:**
```
TICKET-42: Replace custom date parsing with date-fns

The hand-rolled date parser had edge cases around timezone handling
that were causing inconsistent behaviour in reports. date-fns is
already a dependency (used in the frontend) so this adds no new
weight to the bundle.
```
*Why it works*: Subject states what changed; body explains why and notes a relevant consideration.

**TDD workflow commit:**
```
TICKET-18: Add unit tests for payment validation

Tests cover edge cases identified in the ticket:
- Expired cards
- Invalid CVV formats  
- Mismatched billing addresses

Implementation to follow in subsequent commit.
```
*Why it works*: Clearly signals intentional failing tests; documents what's covered.

**Breaking change:**
```
TICKET-51: Rename user endpoints for consistency

Standardise on /accounts to match our domain language.

BREAKING CHANGE: /users/* endpoints renamed to /accounts/*
Clients must update their API calls before upgrading.
```
*Why it works*: Subject describes the change; footer clearly flags the breaking impact.

#### Poor Examples

**Missing ticket reference:**
```
Fix bug in login flow
```
*Problem*: No traceability. Which ticket authorised this work? What was the bug?

**Past tense, vague:**
```
TICKET-22: Fixed stuff
```
*Problem*: "Fixed" instead of "Fix"; "stuff" tells the reader nothing.

**Bundled unrelated changes:**
```
TICKET-30: Add search feature and fix typo in README and update deps
```
*Problem*: Three unrelated changes in one commit. Should be three separate commits.

**Too long, includes implementation detail:**
```
TICKET-15: Update the UserService class to add a new method called validateEmail that uses a regex pattern to check if emails are valid before saving to database
```
*Problem*: Subject line is too long and describes implementation rather than intent. Better: `TICKET-15: Add email validation before user creation`

**No context for non-obvious change:**
```
TICKET-44: Increase timeout to 30s
```
*Problem*: What timeout? Why 30 seconds? A body explaining the reasoning would help future readers.

---

## 4. Branch Conventions

### 4.1 Branch Naming

Feature branches follow a structured naming convention:

```
<type>/<TICKET>_<description>
```

Where:
- **type**: The category of work (`feature`, `fix`, `chore`, `document`, `task`)
- **TICKET**: The ticket reference (e.g., `TICKET-24`)
- **description**: A brief description in lowercase kebab-case

**Examples:**
```
feature/TICKET-24_skill-creator
fix/TICKET-42_date-parsing-timezone
chore/TICKET-50_update-dependencies
document/TICKET-35_git-commit-philosophy
task/TICKET-36_ticket-integration
```

**Type definitions:**
| Type | Use for |
|------|---------|
| `feature` | New functionality or capabilities |
| `fix` | Bug fixes |
| `chore` | Maintenance work (dependencies, configs, cleanup) |
| `document` | Documentation additions or updates |
| `task` | General work items that don't fit other categories |

For epic-level work, the epic ticket ID can serve as the branch ticket reference. Multiple feature branches may exist under the same epic, all sharing the ticket for traceability.

**Hierarchy example:**
```
Epic: TICKET-24 (Agent Skills)
├── Branch: feature/TICKET-24_skill-creator
│   └── Commit: TICKET-27: Create skill-creator skill
├── Branch: document/TICKET-24_skill
│   └── Commit: TICKET-36: Document ticket philosophy
```

This creates clear traceability: branch type and ticket → epic or task, commit → specific work item.

### 4.2 Branch Lifecycle

#### Creation

Create a feature branch from the latest integration branch (`develop`, or `main` if the project has no `develop`):

```bash
git checkout develop
git pull origin develop
git checkout -b feature/TICKET-35_git-commit-docs
```

Branch immediately when starting work on a ticket. This ensures:
- Your work is isolated from other in-progress changes
- The ticket can be transitioned to "In Progress" with a corresponding branch
- Others can see active work in the repository

#### During Development

- **Keep branches short-lived**: Long-running branches accumulate merge conflicts and drift from the integration branch. Aim to merge within days, not weeks.
- **Stay up to date by merging**: Keep your branch current by merging the integration branch into your feature branch: `git merge develop`. This preserves history and is safe for branches that have been pushed.
- **Rebasing**: Only rebase if your branch has not been shared (pushed). Once a branch is pushed, prefer merging to avoid rewriting shared history.
- **Push frequently**: Push your branch to the remote for backup and visibility, even if not ready for review

#### Merging

We use pull requests for all merges to the integration branch. The PR process provides:
- A review checkpoint before changes enter the primary branch
- A record of discussion and approval
- An opportunity for CI to validate the changes

**Merge strategy**: Prefer regular merge commits to preserve the commit history from your branch. Squash merging is available but kept rare. Use it when a branch contains messy checkpoint commits that don't individually add value. In general, we prefer well-structured commits during development over cleaning up with squash at merge time.

#### Deletion

GitHub is configured to automatically delete branches when pull requests are merged. Rely on this default rather than manually deleting branches.

**Exception**: Long-lived branches like `main`, `develop`, or release branches are never deleted.

---

## 5. Commit Content

### 5.1 What Belongs Together

A commit should contain changes that serve a single purpose. Use these guidelines to decide what belongs together:

#### Include Together

- **Feature code and its tests**: If you're adding a function, include the tests for that function in the same commit (unless following TDD, where tests come first)
- **Type definitions and their usage**: New types or interfaces belong with the code that uses them
- **Refactoring in service of a change**: If you rename a variable to prepare for a new feature, that rename can go in the same commit as the feature, provided the connection is clear
- **Documentation for new behaviour**: Updates to comments, READMEs, or docstrings that describe the change you're making

#### Separate Into Different Commits

- **Unrelated bug fixes**: Discovered a bug while working on a feature? Commit it separately with its own ticket reference
- **Opportunistic refactoring**: Cleaning up code that bothers you but isn't required for your current work should be a separate commit
- **Formatting or linting changes**: Bulk reformatting belongs in its own commit, not mixed with logic changes
- **Dependency updates**: Package upgrades should be isolated unless directly required for the feature (and even then, consider separating)

#### The Diff Test

Before committing, review your staged changes and ask:
- Does every changed file relate to the commit message?
- If someone reverted this commit, would anything unrelated be lost?
- Could any of these changes be described with a different commit message?

If changes don't all fit under one purpose, split them.

### 5.2 Special Cases

#### Generated Files

Files generated by build tools, compilers, or code generators require careful handling:

- **Committed generated files**: If generated files are checked into the repository (e.g., compiled assets, OpenAPI client code), include them in the same commit as the source changes that caused the regeneration. This keeps the source and output in sync.
- **Avoid committing if possible**: Prefer generating files at build time rather than committing them. This eliminates sync issues and reduces noise in diffs.
- **Large regenerations**: If a small source change causes a large regeneration (e.g., updating a schema regenerates thousands of lines), consider whether the generated files should be in `.gitignore` instead.

#### Dependency Changes

- **Direct dependencies**: When adding a dependency required for your feature, you may include it in the same commit as the feature, but ensure the commit message mentions the addition.
- **Dependency upgrades**: Routine upgrades (security patches, version bumps) should be separate commits with their own ticket reference. These changes have their own risk profile and may need independent review or rollback.
- **Lock files**: Always commit lock files (`package-lock.json`, `composer.lock`, `go.sum`, etc.) alongside dependency changes. Never commit dependency changes without the corresponding lock file update.

#### Test Inclusion

- **New functionality**: Include tests in the same commit as the code they test, unless following TDD (where tests come first in their own commit).
- **Bug fixes**: Include a regression test that proves the fix works. The test should fail without the fix and pass with it.
- **Refactoring**: If refactoring doesn't change behaviour, existing tests should continue to pass. No new tests required unless coverage was previously missing.
- **Test-only changes**: Fixes to flaky tests, test refactoring, or coverage improvements can be standalone commits with their own ticket.

#### Configuration Changes

- **Environment-specific config**: Changes to environment configuration (CI pipelines, deployment configs) should generally be separate commits unless directly coupled to a code change.
- **Linter/formatter config**: Rule changes should be separate from the code changes they enable or require. First commit the config change, then commit the code that follows the new rules.

---

## 6. Process

### 6.1 Pre-Commit

Before committing, verify:

1. **Review the diff**: `git diff --staged` shows exactly what you're about to commit. Read it. Look for:
   - Debug statements or temporary code
   - Commented-out code that should be removed
   - Unintended changes (files you didn't mean to include)
   - Secrets or credentials

2. **Run linting and static analysis**: Ensure your changes pass the project's linting rules and static analysis checks. Fix any issues before committing rather than leaving them for review.

3. **Check the scope**: Does everything in the diff serve the same purpose? If not, unstage and split.

4. **Verify the state**: If this commit should leave tests passing, run them. If tests are intentionally failing (TDD), confirm that's the case.

5. **Draft the message**: Can you write a clear, single-purpose commit message? Difficulty here often signals a scope problem.

### 6.2 Review and Amendment

#### When to Amend

Use `git commit --amend` to fix the most recent commit when:

- **You forgot to stage a file**: The change logically belongs with the commit you just made
- **The commit message has a typo or error**: Fix it before it's merged
- **You spotted a small mistake immediately**: A missing semicolon, a debug statement you left in

**Critical rule**: Only amend commits when you can be sure no other developer has checked out the affected branch. If others are working from your branch, amending rewrites history and causes problems for them.

#### When to Make a New Commit

Create a new commit instead of amending when:

- **Others may have checked out the branch**: Don't rewrite history others depend on
- **The fix is logically separate**: If addressing review feedback that changes the approach, a new commit documents that evolution
- **You want to preserve the progression**: During review, separate commits can show how you responded to feedback

#### Responding to Review Feedback

When a pull request receives feedback:

1. **Make fixes in new commits**: This allows reviewers to see what changed since their last review
2. **Reference the feedback**: Commit messages like `TICKET-35: Address review feedback - extract helper function` provide context
3. **Consider squashing at merge**: If the fix-up commits don't add value to history, use squash merge to consolidate (though we keep this rare)

#### Interactive Rebase

Use `git rebase -i` to clean up history before merging:

- Squash work-in-progress commits into coherent units
- Reorder commits for logical flow
- Edit commit messages for clarity

**Only rebase when no one else has checked out the branch.** If others are working from your branch, rebasing will cause conflicts for them.

#### Squashing Policy

We keep squashing rare. Our preference is:

1. **Make good commits as you go**: Follow our conventions during development, not just at merge time
2. **Preserve meaningful history**: Well-structured commits tell the story of how the change was built
3. **Squash only when necessary**: Use squash merge for branches with genuinely messy checkpoint commits that don't add value

Squashing throws away information. It's appropriate for cleaning up noise, not for hiding the development process.

### 6.3 Integration

#### Pull Request Workflow

All changes reach the integration branch (`develop` or `main`) through pull requests:

1. **Push your branch**: Ensure all commits are pushed to the remote
2. **Create the pull request**: Link it to the ticket in the description
3. **Request review**: Assign appropriate reviewers based on the changes
4. **Address feedback**: Make additional commits as needed (see Section 6.2)
5. **Merge when ready**: Merge once all project workflow requirements are satisfied: CI passes, reviewers approve, appropriate testing has been performed, and any other project-specific criteria are met

#### CI/CD Expectations

Continuous integration validates every pull request. Before merging:

- **All pipeline stages must pass**: Linting, static analysis, tests, and any other configured checks
- **No unresolved discussions**: Review comments should be addressed or explicitly deferred to a follow-up ticket
- **Branch is up to date**: Merge the integration branch into your feature branch if it has fallen behind

If CI fails:
1. Investigate the failure. Don't assume it's flaky.
2. Fix the issue in a new commit
3. Push and wait for CI to run again

#### Merge Commit Messages

When merging, GitHub generates a merge commit. The default message is usually sufficient, but ensure:

- The pull request title is clear (it often becomes the merge commit subject)
- The PR description provides context (it's linked from the merge commit)

#### After Merging

1. **Verify the integration branch**: Confirm CI passes on the integration branch after your merge
2. **Transition the ticket**: Move the ticket to the appropriate status (e.g., "In Review" or "Done" depending on your workflow)
3. **Delete the branch**: GitHub handles this automatically if configured

#### Handling Merge Conflicts

If your branch conflicts with the integration branch:

1. Merge the integration branch into your feature branch: `git merge develop`
2. Resolve conflicts locally
3. Test that everything still works
4. Push the merge commit
5. Request re-review if the conflict resolution was non-trivial

---

## 7. Agentic Considerations

Git commits represent one of the most consequential actions an agent can take. Unlike reading files or analysing code, committing creates permanent entries in the project's history. This section addresses how our commit philosophy applies when AI agents are involved in the development workflow.

### The Cardinal Rule: Human Approval Required

**NEVER commit without explicit human approval. This applies to ALL situations.**

This is not a suggestion. It is a hard constraint encoded in our agent safety guidelines. An agent may:
- Prepare changes
- Stage files
- Draft commit messages
- Present a summary for review

But the actual commit requires a human to explicitly approve. "Yes, commit this" must come from a person, not be inferred or assumed.

*Rationale*: Commits are permanent. They enter the project's audit trail and may be pushed to shared branches. The cost of an erroneous commit (reverting, explaining, potential downstream effects) far exceeds the cost of pausing for confirmation.

### Separation of Concerns

Following our agentic development framework, the agent that produces work should not be the same agent (or session) that approves it. In practice for commits:

- **Producer role**: Agent implements changes, stages files, drafts commit message
- **Review role**: Human (or separate review process) validates the changes
- **Approval role**: Human explicitly confirms the commit should proceed

This separation prevents an agent from autonomously deciding its own work is "good enough" to commit.

### Commit Message Drafting

Agents should draft commit messages following our conventions, but with awareness that:

1. **The human may edit**: Draft messages are proposals, not final. Present them clearly for review.
2. **Context may be incomplete**: The agent may not know all relevant context. Flag uncertainty rather than fabricating confidence.
3. **Ticket references must be accurate**: Never invent or guess ticket numbers. If the ticket reference is unclear, ask.

### Pre-Commit Verification

Before presenting changes for commit approval, agents should:

1. **Verify the change set**: Show exactly what will be committed (staged files, diff summary)
2. **Confirm ticket alignment**: State which ticket this commit addresses and why
3. **Flag concerns**: Note any uncertainty, incomplete work, or potential issues
4. **Present the draft message**: Show the proposed commit message for approval

### What Agents Should Never Do

- Commit without explicit human approval
- Push commits to remote branches without explicit approval
- Amend commits that have been pushed
- Assume approval from a previous interaction carries forward
- Combine unrelated changes to "save time"

### Recovery Mindset

Agents should approach commits with a recovery mindset:
- What if this commit introduces a bug?
- Can this change be easily reverted?
- Is the commit message clear enough to understand the intent later?
- Have we preserved enough context to debug issues?

This aligns with our broader principle: *confidence must be engineered, not assumed*. A careful approach to commits is part of engineering that confidence.
