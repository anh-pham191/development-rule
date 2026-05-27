---
name: aws-observability-cost
description: >
  AWS observability and cost-control philosophy and patterns, authored as code with
  Terraform/OpenTofu. Use when: (1) Configuring CloudWatch logs, metrics, or alarms,
  (2) Building dashboards as code, (3) Adding X-Ray or OpenTelemetry (ADOT) tracing,
  (4) Setting log retention or log archival, (5) Provisioning AWS Budgets, (6) Enabling
  Cost Anomaly Detection, (7) Defining SLOs or synthetic canaries. Provides tool-agnostic
  principles and short HCL snippets that apply to any AWS account.
---

# AWS Observability & Cost

Core principles and patterns for making AWS workloads observable, alertable, and cost-disciplined, provisioned as code.

## Philosophy

### 1. Observable by Default
Structured logs, metrics, and traces ship from day one, not after the first incident.
- DO: Provision telemetry in the same module as the workload it observes
- DON'T: Bolt observability on once you are already blind during an outage

### 2. Actionable Alarms
Alarms fire on SLO-relevant signals and route to on-call.
- DO: Alarm on error rate, latency, and saturation with an SNS `alarm_actions` target
- DON'T: Alarm only on raw CPU, or leave alarms with no notification target

### 3. Cost-Visible
Spend is visible continuously through tags, Budgets, and anomaly detection.
- DO: Provision Budgets and Cost Anomaly Detection per account and service as code
- DON'T: Discover overspend on the monthly bill

### 4. Retention-Disciplined
Every log group has an explicit retention; cold logs archive to S3.
- DO: Set `retention_in_days` explicitly and archive long-term logs
- DON'T: Leave log groups on "Never expire", silently accruing cost

### 5. Correlatable
Trace IDs flow through logs, metrics, and traces via X-Ray/OTel.
- DO: Propagate trace IDs and use consistent dimensions and tags
- DON'T: Keep siloed logs with no request correlation

### 6. Proactive
SLOs, composite alarms, anomaly detection, and canaries catch issues early.
- DO: Run synthetic canaries on key paths and group alarms into composites
- DON'T: Operate purely reactively

### 7. Dashboards as Code
Dashboards, alarms, and budgets live in Terraform and are reviewed in PRs.
- DO: Define dashboards and alarms in version-controlled HCL
- DON'T: Hand-build dashboards that vanish with their author

### 8. Right-Sized Spend
Capacity is reviewed, not assumed.
- DO: Use Cost Explorer and Savings Plans for baseline; kill idle resources
- DON'T: Set-and-forget capacity at on-demand rates

## Logs

Set explicit retention and encryption on every log group, then subscribe long-term logs to S3 or a SIEM.

```hcl
resource "aws_cloudwatch_log_group" "api" {
  name              = "/app/api"
  retention_in_days = 30          # explicit, never "Never expire"
  kms_key_id        = aws_kms_key.logs.arn
  tags              = local.cost_tags
}

resource "aws_cloudwatch_log_subscription_filter" "archive" {
  name            = "api-archive"
  log_group_name  = aws_cloudwatch_log_group.api.name
  filter_pattern  = ""            # ship everything to the archive/SIEM
  destination_arn = aws_kinesis_firehose_delivery_stream.to_s3.arn
  role_arn        = aws_iam_role.cwl_to_firehose.arn
}
```

## Metrics & Alarms

Alarm on symptoms users feel, and always attach an `alarm_actions` target.

```hcl
resource "aws_cloudwatch_metric_alarm" "api_5xx" {
  alarm_name          = "api-5xx-rate"
  namespace           = "AWS/ApplicationELB"
  metric_name         = "HTTPCode_Target_5XX_Count"
  statistic           = "Sum"
  period              = 60
  evaluation_periods  = 5
  threshold           = 10
  comparison_operator = "GreaterThanThreshold"
  alarm_actions       = [aws_sns_topic.oncall.arn]   # route to on-call
  tags                = local.cost_tags
}
```

Group related alarms into an `aws_cloudwatch_composite_alarm` so a single noisy signal does not page on-call alone, and so you can express "alarm only when errors AND latency are both bad".

## Dashboards as Code

Define dashboards in HCL so they are reviewed in PRs and outlive their author.

```hcl
resource "aws_cloudwatch_dashboard" "api" {
  dashboard_name = "api-overview"
  dashboard_body = jsonencode({
    widgets = [{
      type   = "metric"
      width  = 12
      height = 6
      properties = {
        title   = "API 5xx rate"
        region  = var.region
        metrics = [["AWS/ApplicationELB", "HTTPCode_Target_5XX_Count"]]
      }
    }]
  })
}
```

## Tracing

Enable distributed tracing so requests can be correlated end to end.

- Lambda: set `tracing_config { mode = "Active" }` on `aws_lambda_function` for X-Ray.
- ECS/Fargate: run the ADOT (AWS Distro for OpenTelemetry) collector as a sidecar container and export spans to X-Ray; propagate the trace ID into your structured logs so logs, metrics, and traces share one identifier.

## Budgets & Cost

Provision Budgets with notifications and Cost Anomaly Detection as code. Cost-allocation tags (see aws-foundations) make per-service attribution possible.

```hcl
resource "aws_budgets_budget" "monthly" {
  name         = "account-monthly"
  budget_type  = "COST"
  limit_amount = "1000"
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  notification {
    comparison_operator       = "GREATER_THAN"
    threshold                 = 80          # alert at 80% of budget
    threshold_type            = "PERCENTAGE"
    notification_type         = "ACTUAL"
    subscriber_sns_topic_arns = [aws_sns_topic.cost_alerts.arn]
  }
}

resource "aws_ce_anomaly_monitor" "services" {
  name              = "per-service"
  monitor_type      = "DIMENSIONAL"
  monitor_dimension = "SERVICE"
}

resource "aws_ce_anomaly_subscription" "alerts" {
  name             = "anomaly-alerts"
  frequency        = "DAILY"
  monitor_arn_list = [aws_ce_anomaly_monitor.services.arn]
  subscriber {
    type    = "EMAIL"
    address = "finops@example.com"
  }
}
```

## SLOs & Canaries

Map each user-facing SLO to a CloudWatch alarm so reliability targets are measured, not assumed, and run synthetic canaries against critical journeys to catch breakage before customers do.

```hcl
resource "aws_synthetics_canary" "checkout" {
  name                 = "checkout-path"
  artifact_s3_location = "s3://${aws_s3_bucket.canary.id}/checkout"
  execution_role_arn   = aws_iam_role.canary.arn
  runtime_version      = "syn-nodejs-puppeteer-15.0" # use the latest supported runtime — AWS deprecates old ones
  handler              = "checkout.handler"
  schedule { expression = "rate(5 minutes)" }   # probe the key path continuously
}
```

A failing canary or a breached latency alarm is the operational expression of an SLO: define the target once (e.g. 99.9% of checkout requests under 800ms), then alarm on the metric that proves it.
