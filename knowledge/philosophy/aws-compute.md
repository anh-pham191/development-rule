## AWS Compute Philosophy

This codebase follows these core principles to ensure compute infrastructure on AWS is reliable, secure, cost-conscious, and reproducible.

**Assumptions:** Containerised and serverless workloads on AWS (ECS/Fargate, EKS, Lambda, EC2/ASG). Infrastructure is defined with Terraform/OpenTofu, deployed as immutable artefacts (versioned AMIs and container images pinned by digest), and spread across multiple Availability Zones for resilience.

### 1. Immutable

Replace, don't mutate. Roll out new versioned artefacts rather than patching what is already running.

Every deploy ships a fresh, versioned AMI or container image. Capacity is rolled with blue-green or rolling strategies and `create_before_destroy` so the old version stays healthy until the new one is proven.

- ✓ DO: Pin images by digest, version AMIs, use `create_before_destroy`, roll forward with blue-green/rolling deploys
- ✗ DON'T: SSH into a running fleet to patch it; mutate live instances in place; deploy `:latest`

### 2. Right-Sized & Elastic

Match capacity to real demand. Scale on observed metrics, and scale down (or to zero) when load falls.

Prefer managed and serverless capacity (Fargate, Lambda) so you are not babysitting always-on nodes. Where you do run a fleet, autoscale on metrics that reflect actual work, not guesses.

- ✓ DO: Autoscale on real metrics (CPU/memory/queue depth/RPS), use Fargate/managed capacity, scale to zero where viable
- ✗ DON'T: Over-provision always-on fleets "to be safe"; leave idle capacity running overnight

### 3. Least-Privilege Runtime

Each workload gets its own narrowly scoped identity. Compute should be able to do exactly what it needs and nothing more.

Use ECS task roles, EKS IRSA or Pod Identity, and per-function Lambda execution roles — one identity per service, scoped to the resources that service touches.

- ✓ DO: Scope IAM per service (task role / IRSA / Pod Identity / execution role); grant only the actions and resources needed
- ✗ DON'T: Share one broad instance profile across the whole cluster; bake credentials or secrets into images

### 4. Stateless

Treat compute as cattle, not pets. Any instance or task can be terminated and replaced without losing data.

Externalise all durable state to managed services — RDS, DynamoDB, S3, ElastiCache. Nothing important should live on a local disk that dies with the node.

- ✓ DO: Push state to RDS/DynamoDB/S3/ElastiCache; design tasks to be disposable; store sessions/caches externally
- ✗ DON'T: Keep local-disk state that is lost on replacement; rely on instance affinity or sticky local files

### 5. Resilient

Survive the loss of an instance, a task, or an Availability Zone without an outage.

Place capacity across multiple AZs, wire up health checks and graceful connection draining, and set sensible desired/min/max. Let deployment circuit breakers roll back a bad release automatically.

- ✓ DO: Spread across multiple AZs, configure health checks and graceful drain, set sane desired/min/max, enable deployment circuit breakers
- ✗ DON'T: Run single-AZ services; deploy without rollback; let a failed task take the service down

### 6. Declarative Capacity

Capacity, launch configuration, and deploy strategy live in code — never the console.

Launch templates, capacity providers, and deployment strategy are all expressed in Terraform/OpenTofu so the desired state is reviewable and reproducible. Console tweaks drift and cannot be replayed.

- ✓ DO: Define launch templates, capacity providers, and deploy strategy in HCL; review changes via plan
- ✗ DON'T: Hand-tune ASGs or services in the console; let live infrastructure drift from the code

### 7. Observable

A service you cannot see into is a service you cannot operate. Ship the signals before you need them.

Emit structured logs to CloudWatch, enable Container Insights, and propagate distributed traces with X-Ray or OpenTelemetry. Debugging should never require shelling into a box.

- ✓ DO: Structured logs to CloudWatch, Container Insights, X-Ray/OTel tracing, useful metrics and alarms
- ✗ DON'T: Treat services as black boxes debugged only via SSH; rely on `printf` to stdout with no aggregation

### 8. Explicit

Prefer clarity over implicit behaviour. Make capacity, dependencies, and IAM scope obvious in the code.

- ✓ DO: Name resources clearly, reference ARNs and digests explicitly, pass dependencies through variables and data sources
- ✗ DON'T: Lean on implicit defaults for capacity or security; hide IAM grants behind wildcards

### 9. Cost-Conscious

Pay for what you need. Choose the cheapest capacity that meets the resilience and latency requirements.

Reach for Spot, Graviton, and Fargate where the workload tolerates them, and cover steady baseline load with Savings Plans. Idle clusters are pure waste.

- ✓ DO: Use Spot/Graviton/Fargate where appropriate, buy Savings Plans for steady baseline, kill idle clusters
- ✗ DON'T: Run on-demand x86 for everything; leave dead environments billing; size for peak and forget

### 10. Secret-Injected-Safely

Secrets reach the workload at runtime from a secrets store — never from plaintext or an image layer.

Inject secrets via the task-definition `secrets` block referencing Secrets Manager (or SSM Parameter Store) ARNs, so the value is resolved at launch and never committed or logged.

- ✓ DO: Reference Secrets Manager/SSM ARNs in the task definition `secrets` block; let the agent inject at runtime
- ✗ DON'T: Put secrets in plaintext `environment` variables, image layers, or Terraform state in cleartext

### 11. Pragmatic

Solve the problem at hand with the simplest compute model that fits. We value boring, operable infrastructure over clever topology.

- ✓ DO: Start with the managed option (Fargate/Lambda), add complexity only when a real requirement demands it
- ✗ DON'T: Stand up Kubernetes for three containers; over-engineer for hypothetical scale

When a deviation from the obvious choice is warranted, accompany it with a comment explaining why the trade-off was made.

### 12. Predictable

Infrastructure should behave the same way every time it is applied.

- ✓ DO: Pin versions and digests, keep plans clean, make applies idempotent and repeatable
- ✗ DON'T: Depend on mutable `:latest` tags or unpinned modules that change under you between applies

### 13. Well-Commented

Well-written HCL is self-evident about *what* it provisions. Comments add value when they explain *why*: the constraint, the trade-off, or the reasoning that isn't obvious from the resource.

- ✓ DO: Explain why a workload runs single-tenant, why Spot is excluded, or why a concurrency limit was chosen
- ✗ DON'T: Narrate what each resource block does; leave comments that restate the resource type
