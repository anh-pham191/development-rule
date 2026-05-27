## Terraform Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and predictable Terraform/OpenTofu infrastructure code.

**Assumptions:** You are running Terraform 1.10+ or OpenTofu 1.8+, with remote state backed by a versioned, encrypted backend. Plan and apply are driven through CI rather than from individual laptops, and a cloud provider (AWS, Azure, GCP, or similar) is already configured and authenticated elsewhere in your toolchain. The principles below are provider-agnostic; examples lean on AWS purely for illustration.

### 1. Declarative

Describe the desired end-state and let the engine reconcile it. Terraform is not a scripting language — resist encoding step-by-step procedures into it.

- ✓ DO: Express infrastructure as resources whose final shape you want; let the graph decide ordering via references
- ✗ DON'T: Reach for `null_resource` + `local-exec` to imperatively "run a step" when a first-class resource already models that thing

### 2. Modular

Build small, single-purpose modules with typed variables and explicit outputs, and release them under semver tags. Follow the rule of three — abstract a module on the third repetition, not the first; premature abstraction calcifies the wrong shape.

- ✓ DO: Give each module one responsibility, type and document every variable, expose a deliberate output surface, and tag releases (`v1.2.0`)
- ✗ DON'T: Create a "god module" that provisions a whole environment, or extract a module the first time you copy a block

### 3. Explicit & Pinned

Make versions and dependencies unambiguous. Set `required_version`, constrain providers with `~>`, and commit the `.terraform.lock.hcl` so every run resolves identically. The root module controls exact versions; child modules state lower bounds but do not pin tightly.

- ✓ DO: Pin the tool and provider versions at the root, commit the lock file, and let child modules express only minimum compatible constraints
- ✗ DON'T: Float providers unpinned, gitignore the lock file, or pin exact versions inside reusable child modules

### 4. Readable

Optimise for human understanding. Configuration should read top-to-bottom like prose, with names that convey intent.

- ✓ DO: Use descriptive resource and variable names, `for_each` with meaningful keys, and consistent `terraform fmt` layout
- ✗ DON'T: Use opaque count indices for stable sets, cryptic abbreviations, or deeply nested ternaries that hide the real value

### 5. State-Disciplined

Treat state as the source of truth and protect it accordingly. Use a remote backend; on S3 use native S3 locking via `use_lockfile = true` — this replaces the legacy DynamoDB lock table as of Terraform 1.10 / OpenTofu. Enable bucket versioning and SSE-KMS, and segment state along blast-radius boundaries.

- ✓ DO: Store state remotely with versioning, encryption, and `use_lockfile = true`; split state so a network change can't corrupt a database state
- ✗ DON'T: Commit `terraform.tfstate` to git, stand up a fresh DynamoDB lock table for new projects, or keep one monolithic state for the whole estate

### 6. Environment-Isolated

Keep long-lived environments physically separate. Each gets its own directory, its own backend configuration, and its own `tfvars`. Reserve CLI workspaces for ephemeral or feature stacks where the only delta is a name.

- ✓ DO: Lay out `environments/dev/` and `environments/prod/` as separate roots with distinct backends; use workspaces only for short-lived per-branch stacks
- ✗ DON'T: Switch between `dev` and `prod` with `terraform workspace select` — the failure mode is a forgotten workspace applying a dev change straight into production

### 7. Plan-First

A plan is a reviewable artefact, not a formality. Generate `plan -out=tfplan`, review it in the PR, then apply exactly that saved plan so what was reviewed is what runs.

- ✓ DO: Run `plan -out=tfplan` in CI, surface the diff for review, and `apply tfplan`; gate prod behind an explicit approval
- ✗ DON'T: Run `apply -auto-approve` against production from a laptop, or apply a plan different from the one that was reviewed

### 8. Secret-Safe

Secrets never live in HCL, tfvars, or state in plaintext. Source them at apply time from a secrets manager. Be clear-eyed about `sensitive = true`: it redacts values from CLI and log output but does **not** encrypt them in state.

- ✓ DO: Pull credentials from SSM Parameter Store or Secrets Manager via data sources at apply time, and encrypt state at rest
- ✗ DON'T: Hardcode passwords in `.tf`/`.tfvars`, or assume `sensitive = true` keeps a secret out of the state file — it does not

### 9. Idempotent

A second apply with no code change must be a no-op. Constant drift after apply is a bug in your configuration, not a quirk of the tool.

- ✓ DO: Use resources and data sources that converge; verify a clean re-plan shows zero changes
- ✗ DON'T: Use provisioners or scripts that mutate something on every run and force perpetual diffs

### 10. Tested & Validated

Validate before you apply. Run `fmt`, `validate`, and `tflint` on every change, and write behavioural tests with native `terraform test` / `.tftest.hcl` (GA since Terraform 1.6 / OpenTofu 1.10). Reserve heavyweight Terratest for genuine end-to-end integration.

- ✓ DO: Gate CI on `fmt -check`, `validate`, `tflint`, and `.tftest.hcl` assertions; use Terratest only for real provisioned integration tests
- ✗ DON'T: Treat a green `validate` as proof of correctness, or reach for Terratest when a plan-time assertion would do

### 11. Policy-Governed

Enforce standards in layers rather than relying on review alone: tflint for hygiene, a security scanner (Trivy or Checkov) in pre-commit, and a policy engine (OPA/Conftest or Sentinel) evaluated against the JSON plan. Note that tfsec is being consolidated into Trivy.

- ✓ DO: Layer tflint → Trivy/Checkov → OPA/Conftest against `terraform show -json` so each stage catches what the previous can't
- ✗ DON'T: Depend on a human reviewer to spot a public S3 bucket or an unencrypted volume that a scanner would flag in seconds

### 12. Blast-Radius-Aware

Design so a mistake stays contained. Segment state and modules by failure domain, and put guardrails on the resources you cannot afford to lose.

- ✓ DO: Set `lifecycle { prevent_destroy = true }` on production databases and stateful stores, and keep their state separate from churny resources
- ✗ DON'T: Place a prod database in the same state as frequently-replaced compute where a `taint` or refactor can cascade into a destroy

### 13. Tool-Aware (Terraform vs OpenTofu)

Terraform and OpenTofu share the same HCL but diverge on licence and features. Choose deliberately and pin the tool version.

- ✓ DO: Pick OpenTofu for guaranteed-OSS (MPL 2.0, Linux Foundation) plus native state encryption; pick Terraform if you're committed to HCP/TFC; pin whichever you choose
- ✗ DON'T: Assume feature parity — native state encryption, early variable evaluation, and provider `for_each` are OpenTofu-only
