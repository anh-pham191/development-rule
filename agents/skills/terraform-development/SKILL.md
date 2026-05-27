---
name: terraform-development
description: >
  Terraform/OpenTofu development philosophy, idioms, and patterns for writing clean,
  maintainable, and predictable infrastructure as code. Use when: (1) Working on Terraform
  or OpenTofu projects, (2) Writing or reviewing HCL, (3) Designing reusable modules,
  (4) Configuring state and backends, (5) Setting up plan/apply CI pipelines,
  (6) Implementing policy-as-code, (7) Choosing between Terraform and OpenTofu.
  Provides idiomatic patterns and principles that apply to any Terraform/OpenTofu codebase.
---

# Terraform Development

Core principles for writing clean, maintainable, and predictable Terraform/OpenTofu infrastructure code.

## Philosophy

### 1. Declarative
Describe the desired end-state, not the steps to reach it.
- DO: Model the thing you want as a first-class resource and let the graph order it
- DON'T: Use `null_resource` + `local-exec` for steps a real resource already models

### 2. Modular
Small single-purpose modules; abstract on the third repetition, not the first.
- DO: Type and document variables, expose explicit outputs, tag releases with semver
- DON'T: Build a god module, or extract one the first time you copy a block

### 3. Explicit & Pinned
Make versions unambiguous; the root controls exact versions, not child modules.
- DO: Set `required_version`, constrain providers with `~>`, commit `.terraform.lock.hcl`
- DON'T: Float providers unpinned or pin exact versions inside child modules

### 4. Readable
Optimise for human understanding; configuration should read like prose.
- DO: Use descriptive names and `for_each` with meaningful keys
- DON'T: Use opaque count indices for stable sets or cryptic abbreviations

### 5. State-Disciplined
Treat state as protected source of truth, segmented by blast radius.
- DO: Remote backend with versioning, SSE-KMS, and `use_lockfile = true`
- DON'T: Commit tfstate, or stand up a new DynamoDB lock table

### 6. Environment-Isolated
Long-lived environments are separate directories; workspaces are for ephemeral stacks.
- DO: Give each env its own directory, backend, and tfvars
- DON'T: Switch dev/prod with `workspace select` — a forgotten workspace hits prod

### 7. Plan-First
A plan is a reviewable artefact; apply exactly what was reviewed.
- DO: `plan -out=tfplan`, review in CI, then `apply tfplan` behind prod approval
- DON'T: Run `apply -auto-approve` against prod from a laptop

### 8. Secret-Safe
Secrets never sit in HCL, tfvars, or state plaintext.
- DO: Source secrets from SSM/Secrets Manager at apply time
- DON'T: Assume `sensitive = true` encrypts state — it only redacts logs

### 9. Idempotent
A second apply with no code change must be a no-op.
- DO: Use converging resources; verify a clean re-plan shows zero changes
- DON'T: Use provisioners that mutate on every run and force perpetual diffs

### 10. Tested & Validated
Validate before applying; native testing is GA.
- DO: Gate CI on `fmt`, `validate`, `tflint`, and `.tftest.hcl` assertions
- DON'T: Reach for Terratest when a plan-time assertion would do

### 11. Policy-Governed
Enforce standards in layers, not by review alone.
- DO: Layer tflint → Trivy/Checkov → OPA/Conftest on the JSON plan
- DON'T: Rely on a human to catch a public bucket a scanner flags instantly

### 12. Blast-Radius-Aware
Design so a mistake stays contained.
- DO: Set `lifecycle { prevent_destroy = true }` on critical stateful resources
- DON'T: Put a prod database in the same state as churny compute

### 13. Tool-Aware
Same HCL, divergent licence and features; choose deliberately.
- DO: Pick OpenTofu for guaranteed-OSS + native state encryption, Terraform for HCP/TFC; pin the tool
- DON'T: Assume parity — state encryption, early var eval, provider `for_each` are OpenTofu-only

## Project layout

Prefer a directory per environment over workspaces. Each environment owns its backend and tfvars; shared logic lives in `modules/`.

```
.
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── backend.tf      # dev state bucket/key
│   │   └── dev.tfvars
│   └── prod/
│       ├── main.tf
│       ├── backend.tf      # prod state bucket/key
│       └── prod.tfvars
└── modules/
    └── network/
        ├── variables.tf
        ├── main.tf
        ├── outputs.tf
        └── versions.tf
```

## Module design

A module is a unit with a typed input surface, an explicit output surface, and its own version constraints. Validate inputs at the boundary; a child module declares `required_providers` but does not pin exact versions.

```hcl
# modules/network/variables.tf
variable "cidr_block" {
  description = "CIDR range for the VPC."
  type        = string
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "cidr_block must be a valid CIDR."
  }
}

# modules/network/versions.tf
terraform {
  required_version = ">= 1.10"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0" # lower bound only — the root pins exactly
    }
  }
}

# modules/network/outputs.tf
output "vpc_id" {
  description = "ID of the created VPC."
  value       = aws_vpc.this.id
}
```

## Version & provider pinning

Pin the tool and constrain providers with `~>` at the root, commit `.terraform.lock.hcl`, and bump deliberately with `init -upgrade`.

```hcl
terraform {
  required_version = "~> 1.10"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0" # allow 6.x, block 7.0
    }
  }
}
# commit .terraform.lock.hcl; upgrade via: terraform init -upgrade
```

## State backend & locking

Use a remote backend with versioning and encryption. On S3, lock natively with `use_lockfile` — no DynamoDB table required.

```hcl
terraform {
  backend "s3" {
    bucket       = "acme-tfstate-prod"
    key          = "network/terraform.tfstate"
    region       = "ap-southeast-2"
    encrypt      = true              # encrypt state at rest
    kms_key_id   = "alias/tfstate"   # SSE-KMS
    use_lockfile = true              # native S3 locking — no DynamoDB needed
  }
}
# bucket must have versioning enabled
```

## Workspaces vs directories

Use a separate directory (with its own backend and tfvars) for every long-lived environment; reserve CLI workspaces for ephemeral, identically-shaped stacks such as per-feature-branch sandboxes. The failure mode of using workspaces for prod is the prod typo: you run `terraform workspace select dev` once, forget, and your next `apply` pushes a dev change straight into production with no other signal that anything is wrong.

## Secrets

Pull secrets from a secrets manager via data sources at apply time. `sensitive = true` redacts CLI and log output only — it does not encrypt the value in state.

```hcl
data "aws_secretsmanager_secret_version" "db" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "this" {
  # value redacted from logs, but still plaintext in state — rely on backend encryption
  password = data.aws_secretsmanager_secret_version.db.secret_string
}
```

## Plan discipline & CI

Validate, lint, then plan to a saved file, review it, and apply exactly that. Gate the prod apply behind a manual approval.

```bash
terraform fmt -check
terraform validate
tflint
terraform plan -out=tfplan        # review this diff in the PR
terraform apply tfplan            # apply the reviewed plan; prod step requires approval
```

## Testing

Use native `terraform test` with `.tftest.hcl` files — GA since Terraform 1.6 / OpenTofu 1.10. Use `command = plan` for cheap assertions and `command = apply` for real provisioning; reserve Terratest for heavyweight integration.

```hcl
# tests/network.tftest.hcl
run "valid_cidr_creates_vpc" {
  command = plan

  variables {
    cidr_block = "10.0.0.0/16"
  }

  assert {
    condition     = aws_vpc.this.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR did not match the input variable"
  }
}
```

## Policy as code

Enforce standards in layers: tflint for hygiene, a security scanner in pre-commit, and a policy engine against the machine-readable plan. (tfsec is being consolidated into Trivy.)

```bash
tflint                                        # 1. hygiene / lint
trivy config .                                # 2. security scan (or: checkov -d .)
terraform show -json tfplan > plan.json
conftest test plan.json                       # 3. OPA/Conftest (or Sentinel) on the plan
```

## Terraform vs OpenTofu

Both consume the same HCL, but differ on licence and capabilities. Choose deliberately and pin the tool version.

Decision: choose **OpenTofu** when you want guaranteed open source (MPL 2.0, stewarded by the Linux Foundation) and native state encryption; choose **Terraform** when you're committed to HCP/Terraform Cloud and its ecosystem.

| Aspect              | Terraform (BSL)        | OpenTofu (MPL 2.0)        |
|---------------------|------------------------|---------------------------|
| Licence             | Business Source Licence | MPL 2.0, Linux Foundation |
| State encryption    | Not native             | Native (`encryption {}`)  |
| Early variable eval | No                     | Yes                       |
| Provider `for_each` | No                     | Yes                       |

```hcl
# OpenTofu-only: native client-side state encryption
terraform {
  encryption {
    key_provider "aws_kms" "this" {
      kms_key_id = "alias/tfstate"
    }
    method "aes_gcm" "this" {
      keys = key_provider.aws_kms.this
    }
    state {
      method = method.aes_gcm.this
    }
  }
}
```
