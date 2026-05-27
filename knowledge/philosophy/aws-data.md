## AWS Data Philosophy

This codebase follows these core principles to ensure AWS data and storage are secure, durable, and recoverable by default.

**Assumptions:** This guide assumes stateful AWS data services (RDS/Aurora, DynamoDB, S3, ElastiCache) are provisioned with Terraform or OpenTofu. Data is encrypted at rest with KMS, backups are automated, and production workloads run multi-AZ. State is stored in a remote, versioned, KMS-encrypted backend with locking, and infrastructure changes flow through plan-and-review.

### 1. Encrypted at Rest

All data stores must be encrypted at rest with KMS. Use customer-managed CMKs with automatic rotation for sensitive data so you control the key lifecycle and access policy; AWS-managed keys are acceptable only for low-sensitivity, non-regulated data.

- ✓ DO: Set `kms_key_id`/`kms_key_arn` on RDS, DynamoDB, EBS, and S3; enable `storage_encrypted = true`; rotate customer-managed CMKs
- ✗ DON'T: Provision unencrypted volumes, buckets, or databases; reuse a single CMK across unrelated trust boundaries

### 2. Durable & Recoverable

Data must survive deletion, corruption, and regional failure. Configure automated backups and point-in-time recovery, copy backups cross-region and cross-account for production, and actually test restores — an untested backup is a hope, not a recovery plan.

- ✓ DO: Enable automated backups, PITR, and cross-region/cross-account copies for prod; schedule periodic restore drills
- ✗ DON'T: Rely on untested backups, single-region copies, or same-account-only retention that a compromised account could wipe

### 3. Deletion-Protected

Production data stores must be hard to destroy by accident. Combine provider-level deletion protection, Terraform lifecycle guards, and final snapshots so a stray `destroy` or `terraform apply` cannot vaporise a database.

- ✓ DO: Set `deletion_protection = true`, `lifecycle { prevent_destroy = true }`, and `skip_final_snapshot = false` on prod data resources
- ✗ DON'T: Leave production databases freely destroyable, or skip the final snapshot to "save time"

### 4. Least-Privilege Access

Data stores must be reachable only by the workloads that need them. Place databases in private subnets, scope access through security groups and IAM, and keep S3 private with Block Public Access plus tight bucket policies.

- ✓ DO: Use private subnet groups, narrow security groups, scoped IAM policies, and `aws_s3_bucket_public_access_block`
- ✗ DON'T: Make RDS publicly accessible, expose public S3 buckets, or grant broad `s3:*`/`dynamodb:*` actions on `*`

### 5. Right Store for the Job

Choose the storage primitive that fits the access pattern rather than forcing everything into one engine. Relational and transactional workloads belong in RDS/Aurora, high-scale key-value in DynamoDB, objects and blobs in S3, and hot ephemeral lookups in ElastiCache.

- ✓ DO: Match the store to the data shape and query pattern; combine stores when a domain genuinely needs it
- ✗ DON'T: Bend one store to every problem (e.g. large blobs in a relational column, or relational joins faked in DynamoDB)

### 6. Lifecycle-Managed

Data has a lifespan; manage it explicitly. Transition cold objects to cheaper tiers, expire what you no longer need, set log retention, and use TTLs so storage cost and risk surface do not grow without bound.

- ✓ DO: Define S3 lifecycle rules, Glacier archival, CloudWatch log retention, and DynamoDB TTL
- ✗ DON'T: Keep everything in the hot tier forever or let logs and temporary data accumulate indefinitely

### 7. Highly Available

Production data must tolerate the loss of an Availability Zone. Run managed databases multi-AZ and reach for global tables or cross-region replicas when the availability requirement crosses regional boundaries.

- ✓ DO: Set `multi_az = true` for prod RDS; use DynamoDB global tables where multi-region availability is required
- ✗ DON'T: Run single-AZ production databases or treat a single region as inherently durable

### 8. Auditable & Versioned

Irreplaceable data must be recoverable from mistakes and provably governed. Enable versioning (and object lock where compliance demands immutability), turn on audit logs, and let Config rules detect drift from these guarantees.

- ✓ DO: Enable S3 versioning, object lock for compliance data, RDS/audit logs, and AWS Config rules
- ✗ DON'T: Store irreplaceable data in buckets without versioning, or run databases with auditing disabled

### 9. Secret-Shell Pattern

Secret values must never enter Terraform state. Create the Secrets Manager secret *shell* in IaC, then populate its value out-of-band via a documented runbook. Where a value genuinely must be generated in state (a `random_password` master credential), accept that it lives only inside a KMS-encrypted, access-controlled state bucket.

- ✓ DO: Define the secret shell in IaC, inject values out-of-band by runbook, and reference secrets by ARN at runtime
- ✗ DON'T: Hardcode secret values in HCL or commit them; never store secret material in a state backend that is unencrypted or broadly readable

### 10. Explicit

Make configuration obvious; never lean on provider defaults for anything that matters. Encryption, retention, backup windows, and access scope should be stated in code so reviewers can see the guarantees rather than infer them.

- ✓ DO: Set encryption, backup, retention, and protection attributes explicitly even when a default exists
- ✗ DON'T: Rely on implicit defaults for security- or durability-critical settings, or hide intent behind clever module indirection

### 11. Predictable & Pragmatic

Infrastructure should behave the same way every time, and complexity should be earned. Drive changes through reviewed plans, keep modules simple, and add multi-region, global tables, or bespoke replication only when a concrete requirement justifies the operational cost.

- ✓ DO: Review `plan` output before `apply`, keep state authoritative, and start with the simplest store that meets the need
- ✗ DON'T: Make out-of-band console changes that cause drift, or over-engineer for hypothetical scale before it exists
