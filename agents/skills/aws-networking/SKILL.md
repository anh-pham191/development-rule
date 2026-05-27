---
name: aws-networking
description: >
  AWS networking patterns authored with Terraform/OpenTofu for multi-AZ, VPC-based
  workloads on AWS provider 6.x. Use when: (1) Designing a VPC or planning CIDR
  ranges, (2) Laying out subnets across Availability Zones, (3) Writing security
  groups or NACLs, (4) Configuring PrivateLink / VPC endpoints, (5) Setting up
  routing, NAT, or Transit Gateway connectivity, (6) Enforcing in-transit
  encryption, (7) Reviewing network exposure or auditing boundaries. Covers
  secure-by-default, segmented, reproducible network design.
---

# AWS Networking

Secure-by-default, segmented, reproducible AWS networks defined in Terraform/OpenTofu.

## Philosophy

### 1. Least-Exposure
Default-deny inbound; open only specific source/port pairs.
- DO: Scope ingress to a source SG ID or CIDR plus one port (443)
- DON'T: Allow `0.0.0.0/0` on SSH/RDP/DB ports

### 2. Private by Default
App and data tiers live in private subnets; reach AWS services over endpoints.
- DO: Use VPC endpoints; keep public subnets for LBs and NAT only
- DON'T: Assign public IPs to app or database instances

### 3. Segmented
Public/private/data subnet tiers across AZs, each with its own route table and NACL.
- DO: Define per-tier subnets, route tables, and NACLs
- DON'T: Build flat single-subnet VPCs

### 4. Encrypted In Transit
TLS on endpoints; VPC encryption controls for in-Region traffic. PrivateLink does NOT encrypt by default.
- DO: Require TLS; roll out VPC encryption controls monitor → enforce
- DON'T: Assume PrivateLink or internal traffic is encrypted

### 5. Highly Available
Subnets and NAT per AZ; multi-AZ load balancers.
- DO: Provision NAT per AZ in production
- DON'T: Route a whole region through a single NAT gateway

### 6. Explicit Routing
Named route tables, documented TGW attachments, non-overlapping CIDRs.
- DO: Plan non-overlapping CIDRs and name routes by intent
- DON'T: Choose overlapping CIDRs that block future peering

### 7. Least-Privilege Endpoints
Endpoint policies restrict principals and resources.
- DO: Constrain endpoint policies to specific principals/ARNs
- DON'T: Ship default allow-all endpoint policies

### 8. Auditable Boundaries
Flow logs centralised; rule changes via PRs.
- DO: Send VPC Flow Logs to the logging account; change SGs/NACLs in code
- DON'T: Edit SGs/NACLs in the console and cause drift

## VPC & Subnet Layout

Lay out tiered subnets across the AZ list with `for_each`. Plan a non-overlapping CIDR block per VPC (e.g. a `/16`) and carve predictable per-tier `/20`s so the ranges never collide with peers.

```hcl
locals {
  azs = ["ap-southeast-2a", "ap-southeast-2b", "ap-southeast-2c"]
  # Non-overlapping per-tier offsets within the VPC /16.
  tiers = {
    public  = 0   # 10.0.0.0/20, 10.0.16.0/20, ...
    private = 48  # 10.0.48.0/20, ...
    data    = 96  # 10.0.96.0/20, ...
  }
}

resource "aws_subnet" "private" {
  for_each          = { for i, az in local.azs : az => i }
  vpc_id            = aws_vpc.main.id
  availability_zone = each.key
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 4, local.tiers.private + each.value)
  tags              = { Name = "private-${each.key}", Tier = "private" }
}
```

## Security Groups

On AWS provider 6.x, declare the security group, then attach each rule as its own resource — do NOT use inline `ingress`/`egress` blocks (they fight with rule resources and obscure intent). Reference other security groups by ID for internal flows.

```hcl
resource "aws_security_group" "app" {
  name_prefix = "app-"
  vpc_id      = aws_vpc.main.id
}

resource "aws_security_group" "lb" {
  name_prefix = "lb-"
  vpc_id      = aws_vpc.main.id
}

# HTTPS only, and only from the load balancer's SG (SG referencing SG).
resource "aws_vpc_security_group_ingress_rule" "app_https_from_lb" {
  security_group_id            = aws_security_group.app.id
  referenced_security_group_id = aws_security_group.lb.id
  ip_protocol                  = "tcp"
  from_port                    = 443
  to_port                      = 443
}

resource "aws_vpc_security_group_egress_rule" "app_all_out" {
  security_group_id = aws_security_group.app.id
  ip_protocol       = "-1"
  cidr_ipv4         = "0.0.0.0/0"
}
```

Never open management or database ports to the world. Trivy (`AVD-AWS-0107`) and tflint (`aws_security_group_*`) flag `0.0.0.0/0` on sensitive ports — keep these checks in CI:

```hcl
# DON'T — flagged by Trivy/tflint, will fail the pipeline:
resource "aws_vpc_security_group_ingress_rule" "ssh_world" {
  security_group_id = aws_security_group.app.id
  cidr_ipv4         = "0.0.0.0/0"   # SSH from anywhere
  ip_protocol       = "tcp"
  from_port         = 22
  to_port           = 22
}
```

## NACLs vs Security Groups

Use both — they operate at different layers:

- **Security groups** are *stateful* and attach to ENIs/instances. They are the primary, fine-grained firewall: return traffic is automatically allowed. Reach for these first.
- **NACLs** are *stateless* and attach to subnets. Use them as a coarse subnet boundary (e.g. deny a known-bad CIDR, block one tier from reaching another) and remember to allow ephemeral return ports explicitly.

```hcl
resource "aws_network_acl_rule" "data_deny_public" {
  network_acl_id = aws_network_acl.data.id
  rule_number    = 100
  egress         = false
  protocol       = "-1"
  rule_action    = "deny"
  cidr_block     = "0.0.0.0/0"   # data tier accepts nothing from the public range
}
```

## PrivateLink / VPC Endpoints

Two kinds: **Gateway endpoints** (S3 and DynamoDB only — free, attached to route tables) and **Interface endpoints** (most other services — ENIs in your subnets, billed hourly). Constrain both with an endpoint policy.

```hcl
# Gateway endpoint for S3 — wired into the private route tables.
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]
  policy            = data.aws_iam_policy_document.s3_endpoint.json
}

# Least-privilege endpoint policy.
data "aws_iam_policy_document" "s3_endpoint" {
  statement {
    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::${var.account_id}:root"]
    }
    actions   = ["s3:GetObject", "s3:PutObject"]
    resources = ["${aws_s3_bucket.app.arn}/*"]
  }
}

# Interface endpoint for an AWS service.
resource "aws_vpc_endpoint" "ssm" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ssm"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = [for s in aws_subnet.private : s.id]
  security_group_ids  = [aws_security_group.endpoints.id]
  private_dns_enabled = true
  policy              = data.aws_iam_policy_document.ssm_endpoint.json
}
```

Reminder: PrivateLink keeps traffic off the public internet but is **not encryption** — keep TLS on the application layer.

## Encryption in Transit

Adopt AWS VPC encryption controls for in-Region traffic and roll out safely: start in **monitor** mode to observe what would be affected, review, then move to **enforce**. Supported resources currently include ALB/NLB, Fargate, Transit Gateway, and Nitro-based EC2 instances.

```hcl
resource "aws_vpc_encryption_control" "main" {
  vpc_id = aws_vpc.main.id
  mode   = "monitor" # observe first; switch to "enforce" once clean
}
```

## Connectivity & HA

NAT per AZ avoids a region-wide single point of failure. At scale, prefer a Transit Gateway hub over a mesh of VPC peerings, and keep all CIDRs non-overlapping so routes stay unambiguous.

```hcl
resource "aws_nat_gateway" "this" {
  for_each      = aws_subnet.public
  allocation_id = aws_eip.nat[each.key].id
  subnet_id     = each.value.id   # one NAT per AZ
}

# Transit Gateway attachment — scales better than N×N peering.
resource "aws_ec2_transit_gateway_vpc_attachment" "main" {
  transit_gateway_id = var.transit_gateway_id
  vpc_id             = aws_vpc.main.id
  subnet_ids         = [for s in aws_subnet.private : s.id]
  tags               = { Name = "tgw-attach-${var.vpc_name}" }
}
```

## Flow Logs

Ship VPC Flow Logs to a central destination (the logging account's bucket or log group) so boundaries are auditable independent of the workload account.

```hcl
resource "aws_flow_log" "main" {
  vpc_id                   = aws_vpc.main.id
  traffic_type             = "ALL"
  log_destination_type     = "s3"
  log_destination          = "arn:aws:s3:::central-flow-logs/${var.vpc_name}/"
  max_aggregation_interval = 60
}
```
