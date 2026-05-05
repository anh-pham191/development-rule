# Code Review Philosophy

Code review is a critical quality assurance practice that spans all development work. This document codifies our principles for when, why, and how we review code.

## Audience

- **Primary:** Human developers and AI agents performing code review
- **Secondary:** Non-developers participating in agentic development

## Scope

This document covers:

- Principles that guide effective code review
- Our two-tier review model (agent and human review)
- What reviewers should look for
- How to give and receive feedback
- Practical guidelines for our team

## How to Read This Document

The **Philosophy** section describes principles and values. These use DO/DON'T examples to illustrate intent.

The **Practical Guidelines** section describes process requirements. These use RFC 2119 keywords:

- **MUST** / **MUST NOT**: Absolute requirement or prohibition
- **SHOULD** / **SHOULD NOT**: Recommended, but exceptions may exist with justification
- **MAY**: Optional; implementers can choose

---

## Philosophy

These principles answer: *"What do we value in code review?"*

### 1. Review as Collaboration

Reviews are collaborative discussions, not gatekeeping exercises. The goal is to improve the code together.

- ✓ DO: Work with authors to find solutions; ask clarifying questions; assume good intent
- ✗ DON'T: Use review as a power exercise; focus on "catching" mistakes; make it adversarial

### 2. Shared Understanding

Reviews spread knowledge across the team. No single points of failure; no knowledge silos.

- ✓ DO: Review code outside your usual area; explain context in PR descriptions; learn from reviewing
- ✗ DON'T: Only review code you wrote; hoard knowledge; skip review because "only I understand this"

### 3. Quality Over Speed

Thorough review is more valuable than fast approval. A rubber-stamp approval provides false confidence.

- ✓ DO: Take time to understand what you're approving; say when you can't give proper attention
- ✗ DON'T: Approve without reading; skim large changes; approve to unblock without understanding

### 4. Constructive Feedback

Critique the code, not the coder. Suggest improvements rather than just pointing out problems.

- ✓ DO: Explain why something is an issue; offer solutions; frame as questions when appropriate
- ✗ DON'T: Just say "this is wrong"; make it personal; leave problems without suggested fixes

### 5. Appropriate Scope

Smaller, focused changes receive better review and surface problems earlier.

- ✓ DO: Keep changes focused on one purpose; split large changes; make review manageable
- ✗ DON'T: Bundle unrelated changes; submit changes too large to review in one sitting

### 6. Context Matters

Understand the *why* before critiquing the *what*. A decision that seems wrong in isolation may make sense given constraints you're not aware of.

- ✓ DO: Read the ticket and PR description first; ask clarifying questions; consider constraints
- ✗ DON'T: Assume you know better without context; critique before understanding intent

### 7. Tests Are Code

Review tests with the same rigour as production code. Tests are specifications that define what the system should do.

- ✓ DO: Review test logic carefully; check edge case coverage; treat test changes as code changes
- ✗ DON'T: Skim tests; accept weak assertions; approve code changes without reviewing their tests

### 8. Security Awareness

Always consider security implications during review. Security issues caught in review are far cheaper than those found in production.

- ✓ DO: Check for exposed secrets, injection vulnerabilities, auth gaps; flag uncertain cases for additional review
- ✗ DON'T: Assume security is someone else's job; approve when uncertain about security implications

### 9. Consistency Over Preference

Follow established patterns. Personal preferences don't override team conventions.

- ✓ DO: Reference documented conventions; maintain codebase consistency; propose convention changes through proper channels
- ✗ DON'T: Impose personal style; request changes based on preference alone; treat unclear conventions as license to do whatever

### 10. Timely Response

Reviews should not block progress unnecessarily. Respect for others' time is respect for the team.

- ✓ DO: Respond within one business day; communicate if you can't review in time; pick up unassigned reviews
- ✗ DON'T: Leave review requests unanswered; block others while you context-switch to other work

---

## Two-Tier Review Model

Our development process uses two complementary layers of review, each serving a distinct purpose.

### Agent Review

In agentic development, AI agents MUST review staged changes before commits are made. This review provides feedback to the author (whether human or agent) that informs the implementing agent's work.

**Purpose:**

- Catch issues early, before they enter version control
- Provide rapid feedback during development
- Surface problems before human review

**Characteristics:**

- Occurs on staged changes, before commit
- Compulsory in the agentic development process
- Feedback is provided to the author to address
- Applies to all changes: code, documentation, configuration

**Agent reviewers should:**

- Act as experienced software developers and system architects
- Be critical: do not rubber-stamp; actively look for problems
- Evaluate against established standards and patterns
- Consider automated check results (linting, static analysis, tests)
- Identify potential security concerns
- Flag deviations from conventions
- Provide specific, actionable feedback

**Example: Good vs. Poor Agent Feedback**

Poor feedback:
> "This could be improved."

Good feedback:
> "[High] This function modifies the input slice in place, which could cause unexpected side effects for callers. Consider returning a new slice instead, or document the mutation clearly in the function signature."

Good feedback is specific, explains the issue, indicates severity, and suggests a solution.

**Feedback severity levels:**

Agent reviewers should categorise findings by severity:

| Level | Meaning | Author Response |
|-------|---------|-----------------|
| **Critical** | Blocks commit. Security vulnerability, data loss risk, or fundamental correctness issue. | Must address before proceeding |
| **High** | Significant issue. Bug, missing test coverage, or violation of established patterns. | Should address; explain if not |
| **Medium** | Improvement recommended. Maintainability concern, suboptimal approach, or minor inconsistency. | Consider and respond |
| **Low** | Minor suggestion. Style preference, optional enhancement, or food for thought. | Acknowledge; address if easy |

### Human Review (Formal Gate)

Human review of GitHub Pull Requests is the formal approval gate. No code merges to protected branches without human approval.

**Purpose:**

- Provide human judgment and accountability
- Ensure changes meet quality and security standards
- Spread knowledge across the team
- Create an auditable approval trail

**Characteristics:**

- Occurs on GitHub Pull Requests
- Single approval is sufficient; multiple approvals are welcome
- All automated checks must pass before approval
- Approval signifies the reviewer accepts responsibility for the change

**Human reviewers should:**

- Verify automated checks have passed
- Review agent feedback if available (helpful context, not authoritative)
- Apply their own judgment and expertise
- Consider aspects that automated tools cannot assess
- Approve only when genuinely confident in the change

### Separation of Author and Reviewer

A fundamental principle: **the reviewer must not be the author.**

- Authors (human or agent) may self-check their work before requesting review
- Code review, as defined in this document, requires a different person or agent
- No one approves their own work

This separation ensures independent evaluation and prevents blind spots from propagating.

### Relationship Between Tiers

Agent review and human review are complementary, not redundant:

| Aspect | Agent Review | Human Review |
|--------|--------------|--------------|
| Timing | Before commit | Before merge |
| Authority | Compulsory feedback | Approval gate |
| Speed | Immediate | Within 1 business day |
| Judgment | Pattern-based | Contextual, nuanced |
| Accountability | Informs implementation | Reviewer accountable |

Agent review findings MAY be visible to human reviewers as additional context. Human reviewers are not bound by agent conclusions. The human reviewer's decision is authoritative.

### When Agent and Human Reviewers Disagree

Agent review acts as feedback to a human, who then has discretion about applying this to the implementing agent. If an agent reviewer raises an issue that a human reviewer considers acceptable, the human reviewer's decision is authoritative.

---

## What Reviewers Look For

Whether you're an agent reviewing staged changes or a human reviewing a pull request, consider these areas:

### Correctness

- Does the code do what it's supposed to do?
- Does it handle edge cases appropriately?
- Are there logic errors or off-by-one mistakes?
- Does it match the specification or ticket requirements?

### Security

- Are there exposed secrets or credentials?
- Is user input validated and sanitised?
- Are there injection vulnerabilities (SQL, command, etc.)?
- Is authentication and authorisation properly enforced?
- Is sensitive data protected appropriately?

See also: Data Security Philosophy

### Maintainability

- Is the code readable and understandable?
- Are names clear and descriptive?
- Is complexity appropriate, or could it be simplified?
- Will future developers understand this code?
- Are there appropriate comments where needed?

### Tests

- Are there tests for the new or changed functionality?
- Do tests cover the important cases, including edge cases?
- Are tests themselves well-written and maintainable?
- Do tests actually verify behaviour, or just exercise code?

See also: Testing Philosophy (when available)

### Consistency

- Does the code follow established patterns in the codebase?
- Does it adhere to style conventions?
- Are similar problems solved in similar ways?
- If deviating from patterns, is there good reason?

### Performance

- Are there obvious performance problems?
- Are database queries efficient?
- Is there unnecessary work in hot paths?
- Are appropriate data structures used?

### Documentation

- Are public APIs documented?
- Are complex algorithms explained?
- Is the commit message clear and meaningful?
- Are relevant documentation files updated?

---

## Giving and Receiving Feedback

### For Reviewers: Giving Feedback

**Be specific.** Instead of "this is confusing," explain what's confusing and suggest an improvement.

**Be kind.** Remember there's a person (or an agent learning from your feedback) on the other end. Assume good intent.

**Distinguish severity.** Make clear whether something is a blocking issue, a suggestion, or just a thought. Consider using prefixes:
- "Blocker:" must be addressed before approval
- "Suggestion:" would improve the code but not required
- "Nit:" minor style preference, take it or leave it
- "Question:" seeking to understand, not necessarily requesting change

**Explain why.** Don't just say what's wrong; explain why it matters. This helps authors learn and helps them push back if you've misunderstood.

**Offer solutions.** When pointing out a problem, suggest how to fix it when you can. This is more helpful than just identifying issues.

### For Authors: Receiving Feedback

**Assume good intent.** Reviewers are trying to help, not attack you. Read feedback charitably.

**Respond to everything.** Every comment deserves a response, whether that's a code change, a reply, or an emoji acknowledgment. Don't leave reviewers wondering if you saw their feedback.

**Push back when appropriate.** You don't have to accept every suggestion. If you disagree, explain why. Discussion is valuable.

**Learn from feedback.** Recurring feedback is a signal. If multiple reviewers point out similar issues, consider it a learning opportunity.

### Resolving Comments

Comments on pull requests should be raised, addressed, and resolved as part of the review process:

- **Either party may resolve.** The commenter can resolve after seeing the fix. The author can resolve after addressing the feedback.
- **Not all comments require code changes.** If a comment is a thought, a potential improvement for later, or not applicable to this change, the author may respond (with an explanation, a message, or an emoji) and resolve.
- **Unresolved comments signal incomplete review.** Before approving, check that comments have been addressed. Before merging, ensure all threads are resolved.

---

## Review in Agentic Development

Our agentic development process creates some unique considerations for code review.

### Agent-Authored Code

When code is authored by an AI agent and reviewed by a human:

- **Apply the same standards.** Agent code should meet the same quality bar as human code.
- **The author (human orchestrator) is accountable.** The human who directed the agent work is responsible for the output.
- **Agent capabilities differ from human capabilities.** Agents may produce verbose but correct code, or may miss context that humans would catch. Review accordingly.

### Agent-Reviewed Code

When an agent reviews code (whether human-authored or agent-authored):

- **Agent feedback is advisory.** Authors consider and respond to feedback but make their own decisions.
- **Agents should provide specific, actionable feedback.** Vague feedback like "consider improving this" is unhelpful.
- **Agents should reference established standards.** Feedback should tie to documented conventions, not arbitrary preferences.

### Non-Developer Participants

Our agentic development process enables non-developers to contribute code through AI assistance. These contributions go through the same review process:

- **Same quality bar applies.** Non-developer contributions are held to the same standards.
- **Reviewers should be constructive.** Remember that non-developers are learning; provide educational feedback.
- **The process works.** Agent-assisted development with proper review enables contribution from people who couldn't contribute otherwise.

### Agent-to-Human Handoff

When work moves from agent review (staged changes) to human review (pull request):

- Agent review findings may be included for human reviewer context
- Humans review independently; agent conclusions don't bind human judgment
- Humans bring contextual understanding that agents may lack
- The human approval is what counts

---

## Practical Guidelines

### When Review Is Required

Code review is mandatory for all changes. All changes MUST be reviewed before merge:

- All code changes
- Configuration changes
- Documentation changes
- Infrastructure-as-code changes

**Hotfix Exception:** In urgent situations, changes MAY be committed and merged with review occurring after the fact. This exception MUST be authorised by a senior team member or chapter lead. Post-merge review MUST occur promptly.

### Response Time

Reviewers MUST respond to review requests within **one business day**.

"Respond" means one of:
- Complete the review
- Indicate you'll review by a specific time
- Decline so they can find another reviewer

Leaving a review request unanswered is not acceptable.

### Reviewer Selection

Any team member MAY review and approve a change. We do not require specific reviewers or enforce CODEOWNERS.

However:
- Authors SHOULD consider requesting review from someone with relevant expertise
- Requests SHOULD be open for anyone to pick up
- Waiting for a specific person SHOULD NOT delay getting any review

Reviewers MAY request a second opinion from another team member if they are unsure about the change or would like to draw on known expertise.

### Approval Criteria

For a human to approve a pull request:

1. **The reviewer MUST NOT be the author.** Authors MAY self-check before requesting review, but approval MUST come from someone else. No one approves their own work.
2. All automated checks MUST pass (linting, static analysis, tests, security scans)
3. The reviewer MUST have genuinely reviewed the changes
4. The reviewer MUST be confident the change is correct and appropriate
5. All review comments MUST be addressed (resolved or acknowledged)

Single approval is sufficient. Multiple approvals MAY be requested for significant changes.

### Pull Request Size

Merge requests SHOULD be small enough to review thoroughly. The quality of review degrades as size increases: large changes receive superficial review, small changes receive thorough review.

Each pull request SHOULD do one thing. Signs a change should be split:

- Unrelated files are changing together
- A reviewer would need to context-switch between different concerns
- It cannot be reviewed in a single sitting
- You find yourself thinking "while I'm here, I'll also..."

Use judgment rather than rigid line counts. Generated code (migrations, protobuf), tests, and configuration changes have different review characteristics than hand-written production code.

---

## Connection to Workflow Loop

Code review occurs at multiple points in the agentic development workflow loop:

- **Step 3 (Build):** Agent review occurs during implementation, providing feedback on staged changes before commits
- **Step 5 (Completion):** Human review of the pull request validates that work meets its specification before delivery

Both review tiers contribute to the quality gates that ensure outputs meet standards before proceeding.

---

## Areas for Experimentation

The following areas require further experience to establish firm guidance:

- **Reviewing AI-assisted refactoring:** Large mechanical changes from agents present unique review challenges. How should reviewers efficiently verify correctness across many files?
- **Agent review calibration:** What severity thresholds lead to the best outcomes? How critical should agent reviewers be?
- **Dual-agent code/review loops:** The dynamics of agent-to-agent review cycles before human involvement need further documentation.

As we gain experience, this document will evolve to capture what works.

---

## Related Documents

- Go Development Philosophy (`knowledge/philosophy/go-development.md`)
- PHP Development Philosophy (`knowledge/philosophy/php-development.md`)
- JavaScript Development Philosophy (`knowledge/philosophy/javascript-development.md`)
- API Design Philosophy (`knowledge/philosophy/api-design.md`)
- Agentic Development Philosophy (`knowledge/philosophy/agentic-development.md`)
