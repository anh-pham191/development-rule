## AWS Foundations Philosophy

This codebase follows these core principles to build secure, predictable, and reproducible AWS infrastructure.

**Assumptions:** A multi-account AWS estate managed through AWS Organizations, with all infrastructure provisioned declaratively via Terraform/OpenTofu and access driven entirely by IAM (roles, policies, and federated identity rather than long-lived users). Snippets below are HCL.

### 1. Least-Privilege

Grant only the actions and resources a principal genuinely needs. Build policies with `aws_iam_policy_document` and constrain them with permission boundaries so a role can never exceed its intended ceiling.

- ✓ DO: Scope statements to specific actions and resource ARNs, and attach a permission boundary to every assumable role
- ✗ DON'T: Write `Action: "*"` on `Resource: "*"`, or hand out `AdministratorAccess` because scoping felt fiddly

### 2. Role-Based, No Long-Lived Keys

Humans and machines should assume roles and receive short-lived STS credentials. Federate CI via OIDC and give people access through IAM Identity Center; there is no reason for static keys to exist.

- ✓ DO: Use OIDC web-identity roles for pipelines and Identity Center permission sets for people, with short session durations
- ✗ DON'T: Create an IAM user, mint an access key, and paste it into a CI secret

### 3. Multi-Account by Default

Use account boundaries as your strongest isolation. Separate accounts per environment, plus dedicated security and shared-services accounts, all arranged under Organizational Units.

- ✓ DO: Run prod, non-prod, security, and shared-services in distinct accounts grouped by OU
- ✗ DON'T: Pile every environment and workload into a single account and rely on tags or naming for separation

### 4. Guardrailed

Encode org-wide constraints centrally with Service Control Policies so that no team has to remember the rules. Guardrails deny dangerous actions regardless of an account's own IAM.

- ✓ DO: Attach SCPs that deny unapproved regions, block disabling CloudTrail, and require encryption on creation
- ✗ DON'T: Rely on each team to "remember" to enable encryption or avoid a region, with no enforcing boundary

### 5. Consistently Tagged

Every resource carries ownership and cost metadata so the estate stays explicit and accountable. Drive this from the provider so tagging is the default, not a manual step.

- ✓ DO: Set provider `default_tags` for owner, env, cost-centre, and managed-by, and enforce them with an Organizations tag policy
- ✗ DON'T: Leave untagged resources floating with no owner or cost allocation

### 6. Explicit Trust

A role's trust policy is a security boundary — make it narrow and obvious. State exactly which principals may assume a role and gate it with conditions.

- ✓ DO: Constrain assume-role trust with conditions such as `aws:PrincipalOrgID`, repository claims, or external IDs
- ✗ DON'T: Trust `"AWS": "*"` or any account that happens to ask

### 7. Region-Deliberate

Choose regions on purpose, with data-residency in mind, and make that choice predictable. Pin each stack's region and deny the regions you don't use.

- ✓ DO: Pin the provider region per stack and SCP-deny unused regions to shrink the blast radius
- ✗ DON'T: Let resources land in whatever region a default or a console click happened to pick

### 8. Auditable

You can only investigate what you recorded. Centralise an immutable audit trail that account operators cannot tamper with.

- ✓ DO: Send org-wide CloudTrail and AWS Config to a locked logging account with immutable, versioned buckets
- ✗ DON'T: Keep trails per-account where a compromised operator can stop logging or delete the evidence

### 9. Reproducible Baselines

Treat the secure starting state of an account as code. When a new account is vended it should arrive already wired with logging, guardrails, and break-glass.

- ✓ DO: Codify a baseline module (logging, Config, guardrails, break-glass role) and apply it on every account vend
- ✗ DON'T: Click new accounts together by hand so no two are configured the same way

### 10. No Self-Escalation (Bootstrap Separation)

The IAM grants for your IaC deploy/automation role must live in a separate bootstrap root owned by the account root — not in state the deploy role itself manages. Otherwise the automation can rewrite its own policy and widen its permissions at will, defeating every other control.

- ✓ DO: Keep the deploy-role policy in a bootstrap state the deploy role cannot touch, applied by a human/root identity
- ✗ DON'T: Let the automation role manage its own IAM policy — that is a self-escalation loophole

### 11. Pragmatic

Reach for the simplest arrangement that satisfies the security and isolation requirements. Multi-account, SCPs, and centralised logging are non-negotiable; everything beyond that should earn its complexity.

- ✓ DO: Start with the standard OU layout and a small baseline, and add structure when a real need appears
- ✗ DON'T: Build a sprawling account-factory abstraction before you have accounts to factory
