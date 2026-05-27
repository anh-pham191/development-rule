## AWS Networking Philosophy

This codebase builds AWS networks that are secure by default, segmented, and reproducible from code.

**Assumptions:** Workloads run inside one or more VPCs and are provisioned with Terraform/OpenTofu (HCL). Networks span multiple Availability Zones for resilience. We target the AWS provider 6.x, where security group rules are declared as separate `aws_vpc_security_group_ingress_rule` / `aws_vpc_security_group_egress_rule` resources rather than inline `ingress`/`egress` blocks. Examples assume IPv4 dual-stack-capable layouts but the principles apply equally to IPv6.

### 1. Least-Exposure

Default to denying inbound traffic and open only the specific paths a workload needs. Every allowed flow should name a precise source and port, never a wildcard.

- ✓ DO: Scope ingress to a specific source security group ID or CIDR plus a single port (e.g. 443); reference internal services by security group ID, not IP
- ✗ DON'T: Allow `0.0.0.0/0` on SSH (22), RDP (3389), or database ports (5432/3306); leave a blanket "allow all" rule "to unblock for now"

### 2. Private by Default

Application and data workloads belong in private subnets with no route to the public internet for inbound traffic. Reach AWS services over PrivateLink/VPC endpoints rather than the public internet.

- ✓ DO: Place app and database tiers in private subnets; use Gateway/Interface endpoints for AWS service access; keep public subnets for load balancers and NAT only
- ✗ DON'T: Assign public IPs to app or database instances; route private workloads to an internet gateway for convenience

### 3. Segmented

Divide the VPC into public, private, and data subnet tiers, each replicated across AZs, with its own route table and NACL. Segmentation contains blast radius and makes intent legible.

- ✓ DO: Define per-tier subnets across AZs with tier-specific route tables and NACLs
- ✗ DON'T: Build flat single-subnet VPCs where every workload shares one routing and access boundary

### 4. Encrypted In Transit

Terminate TLS on public endpoints and adopt AWS VPC encryption controls for in-Region traffic, rolling out in monitor mode before enforcing. Note that PrivateLink does NOT encrypt traffic by default — the application layer must still use TLS.

- ✓ DO: Require TLS on load balancers and endpoints; enable VPC encryption controls in monitor mode, review, then enforce; keep app-layer TLS even over PrivateLink
- ✗ DON'T: Assume PrivateLink or "internal-only" traffic is encrypted; expose plaintext listeners on shared paths

### 5. Highly Available

Spread subnets and NAT gateways across AZs and front workloads with multi-AZ load balancers so a single AZ failure is survivable.

- ✓ DO: Provision a NAT gateway per AZ in production; use multi-AZ load balancers and subnets
- ✗ DON'T: Route an entire production region through a single NAT gateway, creating a region-wide single point of failure

### 6. Explicit Routing

Make routing legible: name route tables, document Transit Gateway attachments, and plan non-overlapping CIDRs. Routing should be obvious from reading the code, not inferred.

- ✓ DO: Give route tables descriptive names, document TGW attachments, and allocate non-overlapping CIDR ranges
- ✗ DON'T: Choose overlapping VPC CIDRs that block future peering or Transit Gateway routing; leave default-named, unexplained routes

### 7. Least-Privilege Endpoints

VPC endpoints are a security boundary, not just a connectivity shortcut. Constrain them with endpoint policies that limit which principals and resources may be reached.

- ✓ DO: Attach endpoint policies that restrict principals, actions, and resource ARNs
- ✗ DON'T: Ship the default allow-all endpoint policy on Interface or Gateway endpoints

### 8. Auditable Boundaries

Network boundaries must be observable and change-controlled. Capture flow data centrally and make every rule change reviewable.

- ✓ DO: Send VPC Flow Logs to the central logging account; change security groups and NACLs through pull requests
- ✗ DON'T: Edit security groups or NACLs in the console, causing drift away from the Terraform state

### 9. Explicit

Prefer clarity over implicit behaviour. Make every allowed flow and route visible in code rather than relying on defaults.

- ✓ DO: Declare each ingress/egress rule and route as a named resource with intent in tags or comments
- ✗ DON'T: Lean on implicit default security groups, default route tables, or provider defaults that hide what is actually permitted

### 10. Predictable

Network behaviour should match what the code says without surprises. The plan output is the source of truth.

- ✓ DO: Reference resources by attribute (IDs, ARNs) so dependencies are deterministic; review `plan` before every apply
- ✗ DON'T: Hard-code IPs that drift, or rely on apply ordering side effects that differ between environments

### 11. Pragmatic & Well-Commented

Solve the actual exposure and connectivity requirements at hand without over-engineering, and comment the *why* behind non-obvious choices.

- ✓ DO: Start with the simplest layout that meets the segmentation and HA requirement; comment why a specific CIDR, peering, or exception exists
- ✗ DON'T: Build speculative Transit Gateway meshes for a single VPC; leave a surprising rule (a wide CIDR, a cross-account principal) unexplained
