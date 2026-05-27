---
name: aws-data
description: >
  AWS data and storage patterns provisioned with Terraform/OpenTofu. Use when:
  (1) Provisioning RDS or Aurora databases, (2) Provisioning DynamoDB tables,
  (3) Creating S3 buckets, (4) Provisioning ElastiCache, (5) Defining KMS keys,
  (6) Configuring backups, PITR, or AWS Backup, (7) Adding deletion protection or
  final snapshots, (8) Choosing the right data store for a workload. Secure,
  durable, and recoverable by default.
---

# AWS Data

Core principles for provisioning secure, durable, and recoverable AWS data stores with Terraform/OpenTofu.

## Philosophy

### 1. Encrypted at Rest
Encrypt every store with KMS; customer-managed CMKs with rotation for sensitive data.
- DO: Set `kms_key_id`/`storage_encrypted` on RDS, DynamoDB, EBS, S3
- DON'T: Provision unencrypted volumes, buckets, or databases

### 2. Durable & Recoverable
Automated backups, PITR, cross-region/cross-account copies, tested restores.
- DO: Enable PITR and copy prod backups cross-account/region; drill restores
- DON'T: Trust untested or same-account-only backups

### 3. Deletion-Protected
Production data must be hard to destroy by accident.
- DO: Set `deletion_protection`, `prevent_destroy`, `skip_final_snapshot = false`
- DON'T: Leave prod databases freely destroyable

### 4. Least-Privilege Access
Reachable only by the workloads that need it.
- DO: Private subnets, scoped SG/IAM, S3 Block Public Access
- DON'T: Public RDS, public buckets, or broad `s3:*` on `*`

### 5. Right Store for the Job
Match the store to the access pattern.
- DO: Relational → RDS/Aurora, key-value → DynamoDB, objects → S3, cache → ElastiCache
- DON'T: Force every problem into one store

### 6. Lifecycle-Managed
Data has a lifespan; manage it explicitly.
- DO: S3 lifecycle rules, Glacier archival, log retention, DynamoDB TTL
- DON'T: Keep everything hot and unbounded forever

### 7. Highly Available
Production tolerates losing an AZ.
- DO: `multi_az = true` for prod RDS; DynamoDB global tables where needed
- DON'T: Run single-AZ production databases

### 8. Auditable & Versioned
Irreplaceable data must be recoverable and governed.
- DO: S3 versioning, object lock for compliance, audit logs, Config rules
- DON'T: Store irreplaceable data without versioning

### 9. Secret-Shell Pattern
Secret values never enter Terraform state.
- DO: Create the secret shell in IaC, populate the value by runbook, reference by ARN
- DON'T: Hardcode secrets in HCL or store secret material in an unencrypted state backend

## Choosing a data store

| Need | Store | Why |
|------|-------|-----|
| Relational, transactions, joins | RDS / Aurora | ACID, SQL, mature tooling |
| High-scale key-value, predictable latency | DynamoDB | Serverless scale, single-digit ms |
| Objects, blobs, static assets, backups | S3 | Cheap, durable (11 nines), lifecycle tiers |
| Hot ephemeral lookups, sessions | ElastiCache | Sub-ms in-memory, regenerable |

Rule of thumb: pick for the access pattern, not familiarity. If the data is regenerable, it belongs in a cache; if it is irreplaceable, version and back it up.

## RDS / Aurora

```hcl
resource "random_password" "master" {
  length  = 32
  special = false
}

resource "aws_db_subnet_group" "main" {
  name       = "app-private"
  subnet_ids = var.private_subnet_ids # private subnets only
}

resource "aws_db_parameter_group" "pg" {
  family = "postgres16"
  parameter {
    name  = "rds.force_ssl"
    value = "1" # reject non-TLS connections
  }
}

resource "aws_db_instance" "main" {
  identifier                = "app-prod"
  engine                    = "postgres"
  instance_class            = "db.r6g.large"
  multi_az                  = true
  storage_encrypted         = true
  kms_key_id                = aws_kms_key.data.arn
  db_subnet_group_name      = aws_db_subnet_group.main.name
  parameter_group_name      = aws_db_parameter_group.pg.name
  vpc_security_group_ids    = [aws_security_group.db.id]
  backup_retention_period   = 30
  deletion_protection       = true
  skip_final_snapshot       = false
  final_snapshot_identifier = "app-prod-final"

  username = "app"
  password = random_password.master.result # only ever in encrypted state

  lifecycle { prevent_destroy = true }
}

resource "aws_secretsmanager_secret" "db_master" {
  name       = "app/prod/db-master"
  kms_key_id = aws_kms_key.data.arn
}

resource "aws_secretsmanager_secret_version" "db_master" {
  secret_id     = aws_secretsmanager_secret.db_master.id
  secret_string = random_password.master.result
}
```

## DynamoDB

```hcl
resource "aws_dynamodb_table" "events" {
  name         = "events"
  billing_mode = "PAY_PER_REQUEST" # on-demand
  hash_key     = "pk"
  range_key    = "sk"

  attribute {
    name = "pk"
    type = "S"
  }
  attribute {
    name = "sk"
    type = "S"
  }
  attribute {
    name = "gsi1pk"
    type = "S"
  }

  point_in_time_recovery { enabled = true }

  server_side_encryption {
    enabled     = true
    kms_key_arn = aws_kms_key.data.arn
  }

  ttl {
    attribute_name = "expiresAt"
    enabled        = true
  }

  # GSIs are defined here; model them up front — they cannot be reshaped cheaply later
  global_secondary_index {
    name            = "gsi1"
    hash_key        = "gsi1pk"
    projection_type = "ALL"
  }
}
```

## S3

AWS provider v4+ splits bucket configuration into separate resources — do not use inline blocks.

```hcl
resource "aws_s3_bucket" "data" {
  bucket = "app-prod-data"
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket                  = aws_s3_bucket.data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.data.arn
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_lifecycle_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  rule {
    id     = "archive-and-expire"
    status = "Enabled"
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
    expiration { days = 2555 }
  }
}
```

## KMS

Use a customer-managed CMK when you need key rotation policy, cross-account grants, or an audit trail of `Decrypt` calls. An AWS-managed key is fine for low-sensitivity data where you do not need control over the policy.

```hcl
resource "aws_kms_key" "data" {
  description             = "App data encryption key"
  enable_key_rotation     = true
  deletion_window_in_days = 30
}

resource "aws_kms_alias" "data" {
  name          = "alias/app-data"
  target_key_id = aws_kms_key.data.key_id
}
```

Scope key usage to the services that need it with a `kms:ViaService` condition:

```hcl
data "aws_iam_policy_document" "key_usage" {
  statement {
    actions   = ["kms:Decrypt", "kms:GenerateDataKey"]
    resources = ["*"]
    principals {
      type        = "AWS"
      identifiers = [aws_iam_role.app.arn]
    }
    condition {
      test     = "StringEquals"
      variable = "kms:ViaService"
      values   = ["s3.${var.region}.amazonaws.com"]
    }
  }
}
```

## Secrets shell pattern

Create the secret *shell* in IaC; never let the value enter state. A runbook populates the value out-of-band (CLI/console), and workloads read it by ARN.

```hcl
resource "aws_secretsmanager_secret" "api_key" {
  name       = "app/prod/third-party-api-key"
  kms_key_id = aws_kms_key.data.arn
  # No aws_secretsmanager_secret_version here — value is set out-of-band
}
```

Runbook (out-of-band, not in Terraform):

```bash
aws secretsmanager put-secret-value \
  --secret-id app/prod/third-party-api-key \
  --secret-string "$VALUE"
```

Inject into the workload by ARN, never by literal value:

```hcl
secrets = [{
  name      = "API_KEY"
  valueFrom = aws_secretsmanager_secret.api_key.arn
}]
```

## Backups & DR

```hcl
resource "aws_backup_plan" "prod" {
  name = "prod-daily"
  rule {
    rule_name         = "daily-35d"
    target_vault_name = aws_backup_vault.prod.name
    schedule          = "cron(0 5 * * ? *)"
    lifecycle { delete_after = 35 }

    copy_action { # cross-region / cross-account copy for prod
      destination_vault_arn = var.dr_vault_arn
      lifecycle { delete_after = 90 }
    }
  }
}
```

A backup you have never restored is unproven. Schedule periodic restore drills and confirm the recovered data is usable.

## Lifecycle & retention

Transition cold S3 data to Glacier and expire what you no longer need (see the S3 lifecycle rule above). Set explicit retention on logs so they do not accumulate indefinitely.

```hcl
resource "aws_cloudwatch_log_group" "app" {
  name              = "/app/prod"
  retention_in_days = 90
  kms_key_id        = aws_kms_key.data.arn
}
```
