# Bug Template

Use this template for Bugs: defects where system behaviour doesn't match expectations or requirements.

---

## Template

```markdown
## Summary

[One paragraph overview of the bug. What's broken, where, and what's the impact?]

## Bug type

[Regression / Feature bug]

## Steps to Reproduce

[How to trigger the bug. Be specific enough that someone unfamiliar can reproduce it.]

1. [Step 1]
2. [Step 2]
3. [Step 3]

**Environment:** [Production / UAT / Development]
**Browser/Client:** [If relevant]
**User/Account:** [If relevant, with identifiers]

**Intermittent?** [Yes / No]. If Yes: [e.g. "Seen in 3 of 10 attempts" or "Conditions that increase likelihood: …"]

## Expected Behaviour

[What should happen according to requirements or reasonable user expectations.]

## Actual Behaviour

[What actually happens. Include error messages. Attach screenshots, log excerpts, or screen recordings to this ticket so developers have evidence in one place.]

## Impact

[Business consequences. Who is affected and how? This helps prioritise correctly.]

- **Users affected:** [Number or scope]
- **Frequency:** [How often does this occur?]
- **Workaround:** [Is there a workaround? What is it?]

## Investigation

[If debugging has been done, document the findings. Include queries, log excerpts, or observations.]

### Facts

[What we know with certainty, backed by evidence.]

### Analysis

[Interpretation of the facts. What do we think is happening?]

## Proposed Solution

[If the fix is known, describe the approach. If not, remove this section.]

## Risk Assessment

**Risk Description:** [What could go wrong with the fix? NOT the bug itself — describe the risk of deploying the code change. e.g., "Fix introduces an unexpected regression in the payment flow"]

**Risk Likelihood:** [Rare / Unlikely / Possible / Likely / Almost Certain]
**Impact:** [What is the impact if the fix introduces a problem?]
**Risk Level:** [Low / Medium / High / Critical — Low means no mitigation required; use at least Medium if code review or UAT testing is needed]

**Mitigation Plan:** [What are we doing to mitigate the risk of the fix? e.g., "Code review + automated test coverage + regression testing on UAT"]

**Residual Risk Level:** [Low / Medium / High / Critical — usually Low after mitigation]
**Status:** [Open / Under Mitigation / Mitigated / Accepted / Closed]

## Acceptance Criteria

[How we verify the bug is fixed.]

- [ ] [Scenario that previously failed now works correctly]
- [ ] [No regression in related functionality]
- [ ] [Appropriate test coverage added]
```

---

## Guidance

### Required Sections

At minimum, every Bug should have:
- **Summary**: What's broken
- **Bug type**: Regression or Feature bug (own section before Steps to Reproduce)
- **Expected Behaviour**: What should happen
- **Actual Behaviour**: What actually happens
- **Risk Assessment**: All risk fields must be populated (describes risk of the fix, not severity of the bug)
- **Acceptance Criteria**: How we verify the fix

### Providing Context

Good bug reports include:
- **Specific examples**: Real identifiers (redacted if sensitive), timestamps, request IDs
- **Evidence**: Attach screenshots, log excerpts, and screen recordings to the ticket so developers have everything in one place
- **Reproduction steps**: Even if intermittent, describe conditions

### Investigation Section

Use the Investigation section to document debugging work:
- Keeps knowledge in the ticket, not just in developers' heads
- Helps if work is handed off
- Creates an audit trail for complex issues

### Distinguishing Expected vs Actual

Be precise about the gap:

**Good:**
- Expected: "Password reset email arrives within 5 minutes"
- Actual: "No email received after 30 minutes; no error shown to user"

**Bad:**
- Expected: "Email should work"
- Actual: "Email doesn't work"

### Intermittent Bugs

When a bug does not reproduce every time, add:
- **Reproducibility**: e.g. "Seen in 3 of 10 attempts" or "Roughly 1 in 5"
- **Conditions that increase likelihood**: e.g. "More likely under load", "Only when session expires mid-flow", "First request after deploy"

This helps developers prioritise and narrow down root cause.

### Impact Justifies Priority

The Impact section helps prioritisation. A bug affecting one user occasionally is different from one blocking all users continuously. Be honest about impact to get the right priority.
