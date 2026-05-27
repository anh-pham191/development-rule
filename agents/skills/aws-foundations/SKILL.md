---
name: aws-foundations
description: >
  AWS account, Organizations, and IAM foundation guidelines for building a secure,
  multi-account estate with Terraform/OpenTofu. Use when: (1) Setting up AWS accounts
  or Organizations and OU structure, (2) Writing IAM roles and policies, (3) Authoring
  Service Control Policies (SCPs), (4) Defining a tagging strategy, (5) Deciding region
  strategy, (6) Establishing an audit baseline (CloudTrail/Config), (7) Federating CI
  to AWS via OIDC. Provides least-privilege, role-based patterns that apply to any AWS
  estate.
---

# AWS Foundations

Core principles for building secure, predictable, and reproducible AWS infrastructure with Terraform/OpenTofu.

## Philosophy

### 1. Least-Privilege
Grant only the actions and resources a principal needs; bound it with a permission boundary.
- DO: Scope statements to specific actions and resource ARNs
- DON'T: Write `Action: "*"` on `Resource: "*"`

### 2. Role-Based, No Long-Lived Keys
Assume roles for short-lived STS credentials; federate CI via OIDC and people via Identity Center.
- DO: Use OIDC web-identity roles and Identity Center permission sets
- DON'T: Mint IAM-user access keys for pipelines

### 3. Multi-Account by Default
Use account boundaries for isolation, arranged under OUs.
- DO: Separate prod, non-prod, security, and shared-services accounts
- DON'T: Run everything in one account

### 4. Guardrailed
Encode constraints centrally with SCPs so teams can't forget them.
- DO: Deny unapproved regions, block disabling CloudTrail, require encryption
- DON'T: Rely on per-team memory to enforce the rules

### 5. Consistently Tagged
Tag from the provider so ownership and cost metadata are the default.
- DO: Set `default_tags` for owner/env/cost-centre/managed-by and enforce a tag policy
- DON'T: Leave untagged resources with no owner

### 6. Explicit Trust
Make assume-role trust narrow and conditional.
- DO: Gate trust with `aws:PrincipalOrgID`, repo claims, or external IDs
- DON'T: Trust `"AWS": "*"`

### 7. Region-Deliberate
Pin regions on purpose, with data-residency in mind.
- DO: Pin the provider region per stack and SCP-deny unused regions
- DON'T: Let resources land in whatever region a default picks

### 8. Auditable
Centralise an immutable, tamper-resistant audit trail.
- DO: Send org CloudTrail and Config to a locked logging account
- DON'T: Keep trails per-account where an operator can stop them

### 9. Reproducible Baselines
Treat an account's secure starting state as code.
- DO: Apply a baseline module (logging, guardrails, break-glass) on every account vend
- DON'T: Configure new accounts by hand

### 10. No Self-Escalation (Bootstrap Separation)
Manage the deploy role's IAM in a separate bootstrap root the deploy role can't modify.
- DO: Keep deploy-role policy in bootstrap state applied by root/a human
- DON'T: Let the automation role manage its own IAM policy

## Account & Org Structure

Group accounts under OUs by purpose, not by team.

```hcl
resource "aws_organizations_organization" "this" {
  feature_set                   = "ALL"
  enabled_policy_types          = ["SERVICE_CONTROL_POLICY", "TAG_POLICY"]
  aws_service_access_principals = ["cloudtrail.amazonaws.com", "config.amazonaws.com"]
}

# OU layout: Security, Infrastructure, Workloads-Prod, Workloads-NonProd, Sandbox
resource "aws_organizations_organizational_unit" "security" {
  name      = "Security"
  parent_id = aws_organizations_organization.this.roots[0].id
}

resource "aws_organizations_account" "log_archive" {
  name      = "log-archive"
  email     = "aws+log-archive@example.com"
  parent_id = aws_organizations_organizational_unit.security.id
}
```

## SCPs

Deny dangerous actions centrally; SCPs cap what any IAM in the account can do.

```hcl
data "aws_iam_policy_document" "guardrails" {
  # Deny everything outside approved regions
  statement {
    sid       = "DenyUnapprovedRegions"
    effect    = "Deny"
    actions   = ["*"]
    resources = ["*"]
    condition {
      test     = "StringNotEquals"
      variable = "aws:RequestedRegion"
      values   = ["ap-southeast-2", "us-east-1"]
    }
  }

  # Deny disabling the audit trail
  statement {
    sid       = "DenyCloudTrailTampering"
    effect    = "Deny"
    actions   = ["cloudtrail:StopLogging", "cloudtrail:DeleteTrail"]
    resources = ["*"]
  }

  # Require encryption on new S3 objects
  statement {
    sid       = "DenyUnencryptedUploads"
    effect    = "Deny"
    actions   = ["s3:PutObject"]
    resources = ["*"]
    condition {
      test     = "Null"
      variable = "s3:x-amz-server-side-encryption"
      values   = ["true"]
    }
  }
}
```

## IAM Least-Privilege

Scope actions and resources, add condition keys, and bound the role.

```hcl
data "aws_iam_policy_document" "app_read" {
  statement {
    sid       = "ReadAppBucket"
    actions   = ["s3:GetObject"]
    resources = ["arn:aws:s3:::app-data-prod/*"]
    condition {
      test     = "StringEquals"
      variable = "aws:PrincipalOrgID"
      values   = [aws_organizations_organization.this.id]
    }
  }
}

# Permission boundary caps the role no matter what is later attached.
resource "aws_iam_role" "app" {
  name                 = "app-prod"
  permissions_boundary = aws_iam_policy.boundary.arn
  assume_role_policy   = data.aws_iam_policy_document.app_trust.json
}
```

## Roles over Keys

Federate CI through OIDC; no static keys.

```hcl
data "aws_iam_policy_document" "ci_trust" {
  statement {
    effect  = "Allow"
    actions = ["sts:AssumeRoleWithWebIdentity"]
    principals {
      type        = "Federated"
      identifiers = [aws_iam_openid_connect_provider.github.arn]
    }
    condition {
      test     = "StringLike"
      variable = "token.actions.githubusercontent.com:sub"
      values   = ["repo:my-org/my-repo:ref:refs/heads/main"]
    }
  }
}

resource "aws_iam_role" "ci_deploy" {
  name               = "ci-deploy"
  assume_role_policy = data.aws_iam_policy_document.ci_trust.json
}

# Any KMS Decrypt grant should be scoped to the service that uses the key.
data "aws_iam_policy_document" "ci_kms" {
  statement {
    actions   = ["kms:Decrypt"]
    resources = [aws_kms_key.app.arn]
    condition {
      test     = "StringEquals"
      variable = "kms:ViaService"
      values   = ["s3.ap-southeast-2.amazonaws.com"]
    }
  }
}
```

## Bootstrap & Self-Escalation Separation

The deploy role's own IAM lives in a bootstrap root that the deploy role cannot modify.

```hcl
# bootstrap/ — applied by the account root / a human, with its OWN state backend.
# The ci-deploy role is granted permissions here, NOT by the ci-deploy role itself.
#
# Why this is separate: if the deploy role managed its own policy, it could append
# a statement widening its permissions on the next apply — a self-escalation loophole.
# Keeping these grants in a state the deploy role cannot touch closes that path.
resource "aws_iam_role_policy_attachment" "ci_deploy_grants" {
  role       = "ci-deploy"
  policy_arn = aws_iam_policy.ci_deploy_permissions.arn
}
```

## Tagging & default_tags

Make tagging the default and enforce it org-wide.

```hcl
provider "aws" {
  region = "ap-southeast-2"
  default_tags {
    tags = {
      owner       = "platform-team"
      env         = "prod"
      cost-centre = "cc-1042"
      managed-by  = "opentofu"
    }
  }
}

# Required tags: owner, env, cost-centre, managed-by — enforce every one.
resource "aws_organizations_policy" "required_tags" {
  name = "required-tags"
  type = "TAG_POLICY"
  content = jsonencode({
    tags = {
      owner       = { tag_key = { "@@assign" = "owner" } }
      env         = { tag_key = { "@@assign" = "env" } }
      cost-centre = { tag_key = { "@@assign" = "cost-centre" } }
      managed-by  = { tag_key = { "@@assign" = "managed-by" } }
    }
  })
}
```

## Region Strategy

Pin the region per stack; go multi-region only when a real requirement justifies it.

```hcl
provider "aws" {
  region = "ap-southeast-2" # primary; data-residency aware
}

# Add a second, aliased provider ONLY when DR or a documented need requires it.
provider "aws" {
  alias  = "dr"
  region = "us-east-1"
}
```

## Audit Baseline

Org-wide CloudTrail and Config into a centralised, locked log-archive account.

```hcl
resource "aws_cloudtrail" "org" {
  name                          = "org-trail"
  s3_bucket_name                = aws_s3_bucket.log_archive.id # in log-archive account
  is_organization_trail         = true
  is_multi_region_trail         = true
  include_global_service_events = true
}

# A configuration recorder is per-account — deploy this baseline in every account
# (e.g. via the account-baseline module). Aggregate org-wide findings separately
# with an aws_config_configuration_aggregator in the audit/log-archive account.
resource "aws_config_configuration_recorder" "this" {
  name     = "account-config"
  role_arn = aws_iam_role.config.arn
  recording_group {
    all_supported = true
  }
}

# Immutable: versioning + Object Lock on the log-archive bucket.
resource "aws_s3_bucket_object_lock_configuration" "log_archive" {
  bucket = aws_s3_bucket.log_archive.id
  rule {
    default_retention {
      mode = "COMPLIANCE"
      days = 365
    }
  }
}
```
