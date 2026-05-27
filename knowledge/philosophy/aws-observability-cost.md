## AWS Observability & Cost Philosophy

This guide follows these core principles to ensure AWS workloads are observable, alertable, and cost-disciplined from the first commit.

**Assumptions:** This guide assumes telemetry and cost controls are provisioned as code with Terraform or OpenTofu (never hand-clicked in the console), that CloudWatch is the baseline for logs, metrics, alarms, and dashboards, and that cost-allocation tags are already applied consistently across accounts and resources (this ties to the aws-foundations guide). Observability and cost are treated as first-class deliverables of every service, not afterthoughts bolted on once something has already broken or the bill has already arrived.

### 1. Observable by Default

Structured logs, metrics, and traces ship from day one, wired up in the same Terraform module that provisions the workload. Observability is part of "done", not a follow-up ticket.

- ✓ DO: Emit structured JSON logs, custom metrics, and trace segments from the first deploy; provision log groups and alarms alongside the resource they observe
- ✗ DON'T: Bolt telemetry on after the first incident, when you are blind and under pressure

### 2. Actionable Alarms

Alarms fire on SLO-relevant signals — error rate, latency, saturation — and every alarm routes to an on-call destination that a human will actually see. An alarm with no notification target is just a coloured square on a dashboard.

- ✓ DO: Alarm on the symptoms users feel (errors, p99 latency, queue depth) and attach an `alarm_actions` SNS topic that pages on-call
- ✗ DON'T: Alarm only on raw CPU, or create noisy alarms with no `alarm_actions` and no owner

### 3. Cost-Visible

Spend is visible continuously through cost-allocation tags, AWS Budgets with alerts, and Cost Anomaly Detection per account and service. You should learn about overspend within hours, not at the end of the billing cycle.

- ✓ DO: Provision Budgets with notification thresholds and anomaly monitors as code; tag every resource for cost attribution
- ✗ DON'T: Discover overspend on the monthly bill, with no idea which service or change caused it

### 4. Retention-Disciplined

Every CloudWatch log group sets an explicit `retention_in_days`. Cold logs that must be kept are archived to cheaper S3 storage; nothing is retained forever by accident.

- ✓ DO: Set `retention_in_days` explicitly on every log group and subscribe long-term logs to S3 archival
- ✗ DON'T: Leave log groups on "Never expire", silently accruing storage cost for data nobody will ever read

### 5. Correlatable

A single request can be followed across logs, metrics, and traces through a shared trace ID, propagated via X-Ray or OpenTelemetry. Dimensions and tags are named consistently so signals can be joined.

- ✓ DO: Propagate trace IDs into structured logs and use consistent metric dimensions and resource tags across services
- ✗ DON'T: Keep siloed logs with no request correlation, forcing engineers to guess which lines belong to which request

### 6. Proactive

Reliability is defended ahead of failures with SLOs, composite alarms, anomaly detection, and synthetic canaries on key user paths. The goal is to catch degradation before customers report it.

- ✓ DO: Define SLOs, group related alarms into composite alarms, and run Synthetics canaries against critical journeys
- ✗ DON'T: Operate purely reactively, learning about outages from customer complaints

### 7. Dashboards as Code

Dashboards, alarms, and budgets live in Terraform and are reviewed in pull requests like any other change. Observability artefacts survive the person who created them.

- ✓ DO: Define `aws_cloudwatch_dashboard`, alarms, and budgets in version-controlled HCL, reviewed in PRs
- ✗ DON'T: Hand-build dashboards in the console that vanish, undocumented, when their author leaves

### 8. Right-Sized Spend

Capacity is reviewed, not assumed. Cost Explorer and anomaly findings inform sizing, Savings Plans cover steady baseline load, and idle resources are removed.

- ✓ DO: Review Cost Explorer and anomalies regularly, buy Savings Plans for predictable baseline, and kill idle resources
- ✗ DON'T: Set-and-forget capacity, paying on-demand rates for load you could have committed or for resources nobody uses

### 9. Explicit

Telemetry and cost configuration are stated outright in code, never left to implicit defaults. Retention, thresholds, and alarm targets are visible at a glance.

- ✓ DO: Spell out retention, alarm thresholds, budget limits, and notification targets in HCL
- ✗ DON'T: Rely on hidden console defaults or undocumented account-level settings

### 10. Pragmatic

Instrument what matters and resist gold-plating. A handful of signals tied to real user impact beats a wall of metrics nobody reads. We value simplicity over cleverness.

- ✓ DO: Start with the few signals that map to SLOs, then add depth where incidents prove it is needed
- ✗ DON'T: Over-engineer observability for hypothetical failure modes, or alarm on every metric AWS exposes

### 11. Well-Commented

Telemetry and cost code is self-evident about *what* it does. Comments add value when they explain *why*: why a threshold was chosen, why a log group is exempt from short retention, or why a budget sits where it does.

- ✓ DO: Comment the reasoning behind alarm thresholds, retention exceptions, and budget limits
- ✗ DON'T: Narrate what each resource block obviously creates, duplicating the resource name
