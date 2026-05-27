---
name: aws-compute
description: >
  AWS compute patterns authored with Terraform/OpenTofu. Use when: (1) Choosing a compute
  model for a workload, (2) Defining ECS/Fargate services and task definitions, (3) Running
  workloads on EKS (managed node groups, Fargate profiles, IRSA/Pod Identity), (4) Managing
  EC2/ASG fleets with launch templates and autoscaling, (5) Writing Lambda functions and
  their execution roles, (6) Configuring autoscaling on real metrics, (7) Setting up immutable
  blue-green/rolling deploys, (8) Scoping task and execution roles to least privilege.
---

# AWS Compute

Patterns for reliable, secure, cost-conscious AWS compute defined as immutable, multi-AZ infrastructure in Terraform/OpenTofu.

## Philosophy

### 1. Immutable
Replace, don't mutate — ship versioned artefacts and roll forward.
- DO: Pin images by digest, version AMIs, use `create_before_destroy`, blue-green/rolling deploys
- DON'T: SSH in to patch a live fleet; deploy `:latest`

### 2. Right-Sized & Elastic
Match capacity to real demand; scale down when idle.
- DO: Autoscale on real metrics, prefer Fargate/managed capacity, scale to zero where viable
- DON'T: Over-provision always-on fleets; leave idle capacity billing

### 3. Least-Privilege Runtime
One narrowly scoped identity per workload.
- DO: Per-service task roles / IRSA / Pod Identity / execution roles
- DON'T: One broad instance profile for the whole cluster; secrets baked into images

### 4. Stateless
Treat nodes as cattle; externalise durable state.
- DO: Push state to RDS/DynamoDB/S3/ElastiCache; design tasks as disposable
- DON'T: Keep local-disk state that dies with the instance

### 5. Resilient
Survive the loss of an instance, task, or AZ.
- DO: Multi-AZ placement, health checks, graceful drain, deployment circuit breakers
- DON'T: Run single-AZ services; deploy without rollback

### 6. Declarative Capacity
Capacity and deploy strategy live in code.
- DO: Launch templates, capacity providers, deploy strategy in HCL
- DON'T: Console-tweaked ASGs that drift

### 7. Observable
Ship the signals before you need them.
- DO: Structured CloudWatch logs, Container Insights, X-Ray/OTel tracing
- DON'T: Black-box services debugged only via SSH

### 8. Cost-Conscious
Pay for what you need.
- DO: Spot/Graviton/Fargate where appropriate, Savings Plans for steady baseline, kill idle clusters
- DON'T: On-demand x86 for everything

### 9. Secret-Injected-Safely
Resolve secrets at runtime from a store.
- DO: Reference Secrets Manager/SSM ARNs in the task `secrets` block
- DON'T: Put secrets in plaintext `environment` variables or image layers

## Choosing a Compute Model

Pick the most managed option that meets the requirement, and only step down the ladder when a real constraint forces it:

- **Lambda** — event-driven, bursty, or short-lived work. No servers, scales to zero, pay per invocation.
- **Fargate** — long-running containers with no desire to manage nodes. The default for most services.
- **EKS** — you genuinely need Kubernetes (existing manifests, operators, ecosystem, multi-tenant scheduling).
- **EC2 / ASG** — full control over the host (custom kernels, GPUs, licensing, specialised networking).

Decision note: `Lambda → Fargate → EKS → EC2/ASG`, descending in how much you manage and ascending in control. Start at the top.

## ECS / Fargate

The centre of gravity. A task definition with separate task and execution roles, an image pinned by digest, and a service with a deployment circuit breaker.

```hcl
data "aws_ecr_image" "app" {
  repository_name = aws_ecr_repository.app.name
  image_tag       = "v1.4.2"
}

resource "aws_iam_role" "task" {
  name               = "app-task"
  assume_role_policy = data.aws_iam_policy_document.ecs_tasks_assume.json
}

resource "aws_iam_role" "execution" {
  name               = "app-execution"
  assume_role_policy = data.aws_iam_policy_document.ecs_tasks_assume.json
}

resource "aws_ecs_task_definition" "app" {
  family                   = "app"
  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  cpu                      = 512
  memory                   = 1024
  task_role_arn            = aws_iam_role.task.arn       # what the app may do
  execution_role_arn       = aws_iam_role.execution.arn  # pull image, write logs, read secrets

  container_definitions = jsonencode([{
    name      = "app"
    image     = "${aws_ecr_repository.app.repository_url}@${data.aws_ecr_image.app.image_digest}"
    essential = true
    portMappings = [{ containerPort = 8080 }]
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-group"         = aws_cloudwatch_log_group.app.name
        "awslogs-region"        = var.region
        "awslogs-stream-prefix" = "app"
      }
    }
  }])
}

resource "aws_ecs_service" "app" {
  name            = "app"
  cluster         = aws_ecs_cluster.this.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 3
  launch_type     = "FARGATE"

  network_configuration {
    subnets         = var.private_subnet_ids # spread across AZs
    security_groups = [aws_security_group.app.id]
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 8080
  }

  deployment_circuit_breaker {
    enable   = true
    rollback = true
  }
}
```

Service autoscaling on a real metric (target-tracking on CPU):

```hcl
resource "aws_appautoscaling_target" "app" {
  service_namespace  = "ecs"
  resource_id        = "service/${aws_ecs_cluster.this.name}/${aws_ecs_service.app.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  min_capacity       = 2
  max_capacity       = 20
}

resource "aws_appautoscaling_policy" "cpu" {
  name               = "app-cpu-target"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.app.resource_id
  scalable_dimension = aws_appautoscaling_target.app.scalable_dimension
  service_namespace  = aws_appautoscaling_target.app.service_namespace

  target_tracking_scaling_policy_configuration {
    target_value = 60
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
  }
}
```

## ECR

Immutable tags so a digest can never be reassigned, plus a lifecycle policy to evict old images.

```hcl
resource "aws_ecr_repository" "app" {
  name                 = "app"
  image_tag_mutability = "IMMUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }
}

resource "aws_ecr_lifecycle_policy" "app" {
  repository = aws_ecr_repository.app.name
  policy = jsonencode({
    rules = [{
      rulePriority = 1
      description  = "Keep last 20 images"
      selection = {
        tagStatus   = "any"
        countType   = "imageCountMoreThan"
        countNumber = 20
      }
      action = { type = "expire" }
    }]
  })
}
```

## ALB

Load balancer, target group, and listener with a health check that gates traffic.

```hcl
resource "aws_lb" "app" {
  name               = "app"
  load_balancer_type = "application"
  subnets            = var.public_subnet_ids # multiple AZs
  security_groups    = [aws_security_group.alb.id]
}

resource "aws_lb_target_group" "app" {
  name        = "app"
  port        = 8080
  protocol    = "HTTP"
  target_type = "ip" # Fargate awsvpc
  vpc_id      = var.vpc_id

  health_check {
    path                = "/healthz"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    interval            = 15
    matcher             = "200"
  }
}

resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.app.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = var.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

## EKS

Reach for EKS only when you genuinely need Kubernetes. Use a managed node group (or a Fargate profile) for capacity, and scope pod-level IAM with IRSA or Pod Identity rather than the node role.

```hcl
resource "aws_eks_cluster" "this" {
  name     = "platform"
  role_arn = aws_iam_role.cluster.arn
  version  = "1.30"

  vpc_config {
    subnet_ids = var.private_subnet_ids # multi-AZ
  }
}

resource "aws_eks_node_group" "default" {
  cluster_name    = aws_eks_cluster.this.name
  node_group_name = "default"
  node_role_arn   = aws_iam_role.node.arn
  subnet_ids      = var.private_subnet_ids

  scaling_config {
    desired_size = 3
    min_size     = 2
    max_size     = 10
  }
}

# Pod-scoped IAM (Pod Identity): the app pod's service account gets its own role,
# not the broad node role.
resource "aws_eks_pod_identity_association" "app" {
  cluster_name    = aws_eks_cluster.this.name
  namespace       = "app"
  service_account = "app"
  role_arn        = aws_iam_role.app_pod.arn
}
```

## EC2 / ASG

Drop to EC2 only when you need the host. Launch template carries the immutable AMI; the ASG spans AZs; target-tracking handles scaling; an instance refresh rolls new AMIs immutably.

```hcl
resource "aws_launch_template" "app" {
  name_prefix   = "app-"
  image_id      = var.app_ami_id # versioned, immutable
  instance_type = "m7g.large"    # Graviton

  iam_instance_profile { arn = aws_iam_instance_profile.app.arn }
}

resource "aws_autoscaling_group" "app" {
  name                = "app"
  vpc_zone_identifier = var.private_subnet_ids # multiple AZs
  min_size            = 2
  max_size            = 12
  desired_capacity    = 3
  target_group_arns   = [aws_lb_target_group.app.arn]
  health_check_type   = "ELB"

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }

  # Roll new AMIs immutably rather than mutating live instances.
  instance_refresh {
    strategy = "Rolling"
    preferences { min_healthy_percentage = 90 }
  }
}

resource "aws_autoscaling_policy" "cpu" {
  name                   = "app-cpu-target"
  autoscaling_group_name = aws_autoscaling_group.app.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 55
  }
}
```

## Lambda

Function plus a tightly scoped execution role; configuration comes from SSM, not hard-coded values. Use reserved concurrency to cap blast radius and provisioned concurrency to kill cold starts on latency-critical paths.

```hcl
resource "aws_iam_role" "fn" {
  name               = "processor-exec"
  assume_role_policy = data.aws_iam_policy_document.lambda_assume.json
}

resource "aws_lambda_function" "processor" {
  function_name = "processor"
  role          = aws_iam_role.fn.arn # scoped to exactly this function's needs
  package_type  = "Image"
  image_uri     = "${aws_ecr_repository.processor.repository_url}@${data.aws_ecr_image.processor.image_digest}"

  environment {
    variables = {
      QUEUE_URL = data.aws_ssm_parameter.queue_url.value # config from SSM
    }
  }

  reserved_concurrent_executions = 50 # cap blast radius
}

# Provisioned concurrency for latency-sensitive functions (no cold starts).
resource "aws_lambda_provisioned_concurrency_config" "processor" {
  function_name                     = aws_lambda_function.processor.function_name
  qualifier                         = aws_lambda_function.processor.version
  provisioned_concurrent_executions = 5
}
```

## Secrets Injection

Never put secrets in plaintext `environment` variables. Reference a Secrets Manager ARN from the task definition `secrets` block — the ECS agent resolves it at launch, and the value stays out of the task definition and out of state.

```hcl
container_definitions = jsonencode([{
  name  = "app"
  image = "${aws_ecr_repository.app.repository_url}@${data.aws_ecr_image.app.image_digest}"

  # Resolved at runtime from Secrets Manager — never committed, never logged.
  secrets = [{
    name      = "DATABASE_URL"
    valueFrom = aws_secretsmanager_secret.db_url.arn
  }]
}])
```

The execution role must be allowed to read that secret (`secretsmanager:GetSecretValue` on the specific ARN, plus `kms:Decrypt` if it is encrypted with a customer-managed key) — and nothing broader.
