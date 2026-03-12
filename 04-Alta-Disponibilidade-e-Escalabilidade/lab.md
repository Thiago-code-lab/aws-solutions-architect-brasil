# Lab — Alta Disponibilidade e Escalabilidade

> **Região:** us-east-1  
> **Custo estimado:** ~$0,02/hora ALB + EC2 t3.micro durante o lab  
> **Pré-requisito:** AWS CLI configurado, Terraform >= 1.5

## Objetivos do Lab
1. Criar um ALB com path-based routing para dois target groups
2. Configurar ASG com Target Tracking vinculado ao ALB
3. Adicionar Lifecycle Hook com notificação via EventBridge
4. Testar scale-out e scale-in com stress test

---

## Parte 1: ALB com Path-Based Routing via AWS CLI

```bash
REGION="us-east-1"
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" --output text --region $REGION)
SUBNETS=$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query "Subnets[*].SubnetId" --output text --region $REGION)
SUBNET_1=$(echo $SUBNETS | cut -d' ' -f1)
SUBNET_2=$(echo $SUBNETS | cut -d' ' -f2)

echo "VPC: $VPC_ID | Subnets: $SUBNET_1 $SUBNET_2"

# Security Group para ALB
ALB_SG=$(aws ec2 create-security-group \
  --group-name lab-alb-sg --description "ALB Lab" \
  --vpc-id $VPC_ID --region $REGION \
  --query GroupId --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $ALB_SG --protocol tcp --port 80 \
  --cidr 0.0.0.0/0 --region $REGION

# Security Group para instâncias (aceita do ALB apenas)
EC2_SG=$(aws ec2 create-security-group \
  --group-name lab-ec2-sg --description "EC2 Lab" \
  --vpc-id $VPC_ID --region $REGION \
  --query GroupId --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $EC2_SG --protocol tcp --port 80 \
  --source-group $ALB_SG --region $REGION

# Criar ALB
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name lab-alb \
  --subnets $SUBNET_1 $SUBNET_2 \
  --security-groups $ALB_SG \
  --region $REGION \
  --query "LoadBalancers[0].LoadBalancerArn" --output text)

echo "ALB criado: $ALB_ARN"

# Criar dois Target Groups
TG_API=$(aws elbv2 create-target-group \
  --name lab-tg-api \
  --protocol HTTP --port 80 \
  --vpc-id $VPC_ID \
  --health-check-path /api/health \
  --health-check-interval-seconds 15 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --region $REGION \
  --query "TargetGroups[0].TargetGroupArn" --output text)

TG_WEB=$(aws elbv2 create-target-group \
  --name lab-tg-web \
  --protocol HTTP --port 80 \
  --vpc-id $VPC_ID \
  --health-check-path /health \
  --health-check-interval-seconds 15 \
  --region $REGION \
  --query "TargetGroups[0].TargetGroupArn" --output text)

echo "TG API: $TG_API | TG WEB: $TG_WEB"

# Criar Listener HTTP:80 com regras de roteamento
LISTENER_ARN=$(aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TG_WEB \
  --region $REGION \
  --query "Listeners[0].ListenerArn" --output text)

# Adicionar regra de path-based para /api/*
aws elbv2 create-rule \
  --listener-arn $LISTENER_ARN \
  --priority 10 \
  --conditions '[{"Field":"path-pattern","Values":["/api/*"]}]' \
  --actions "[{\"Type\":\"forward\",\"TargetGroupArn\":\"$TG_API\"}]" \
  --region $REGION
```

---

## Parte 2: Terraform — ASG completo com Target Tracking e Lifecycle Hook

```hcl
# main.tf

terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "available" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

data "aws_ssm_parameter" "al2023" {
  name = "/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64"
}

# ──────────────────────────────────────────────────────
# 1. Security Groups
# ──────────────────────────────────────────────────────

resource "aws_security_group" "alb_sg" {
  name        = "lab-ha-alb-sg"
  description = "ALB Lab HA"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "ec2_sg" {
  name        = "lab-ha-ec2-sg"
  description = "EC2 Lab HA"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ──────────────────────────────────────────────────────
# 2. Application Load Balancer
# ──────────────────────────────────────────────────────

resource "aws_lb" "main" {
  name               = "lab-ha-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = data.aws_subnets.available.ids

  enable_deletion_protection = false # lab: permite destroy fácil

  tags = {
    Name   = "lab-ha-alb"
    Module = "04-Alta-Disponibilidade"
  }
}

resource "aws_lb_target_group" "web" {
  name        = "lab-ha-tg-web"
  port        = 80
  protocol    = "HTTP"
  vpc_id      = data.aws_vpc.default.id
  target_type = "instance"

  health_check {
    path                = "/health"
    interval            = 15
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 3
    matcher             = "200"
  }

  deregistration_delay = 30 # lab: reduzido para 30s (padrão 300s)
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# ──────────────────────────────────────────────────────
# 3. Launch Template
# ──────────────────────────────────────────────────────

resource "aws_launch_template" "web" {
  name_prefix            = "lab-ha-web-"
  image_id               = data.aws_ssm_parameter.al2023.value
  instance_type          = "t3.micro"
  vpc_security_group_ids = [aws_security_group.ec2_sg.id]

  metadata_options {
    http_tokens                 = "required"
    http_endpoint               = "enabled"
    http_put_response_hop_limit = 1
  }

  user_data = base64encode(<<-EOF
    #!/bin/bash
    yum install -y httpd stress
    systemctl enable --now httpd

    # Endpoint de health check
    cat > /var/www/html/health <<'HEALTH'
    OK
    HEALTH

    # Endpoint principal
    INSTANCE_ID=$(curl -s -X PUT http://169.254.169.254/latest/api/token \
      -H "X-aws-ec2-metadata-token-ttl-seconds: 60" | \
      xargs -I {} curl -s http://169.254.169.254/latest/meta-data/instance-id \
      -H "X-aws-ec2-metadata-token: {}")
    echo "<h1>Hello from $INSTANCE_ID</h1>" > /var/www/html/index.html
  EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name   = "lab-ha-web-instance"
      Module = "04-Alta-Disponibilidade"
    }
  }
}

# ──────────────────────────────────────────────────────
# 4. Auto Scaling Group
# ──────────────────────────────────────────────────────

resource "aws_autoscaling_group" "web" {
  name                = "lab-ha-asg"
  vpc_zone_identifier = data.aws_subnets.available.ids
  min_size            = 1
  max_size            = 4
  desired_capacity    = 2

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  target_group_arns = [aws_lb_target_group.web.arn]

  # Importante: usar ELB health check em produção
  health_check_type         = "ELB"
  health_check_grace_period = 60

  # Termination policy
  termination_policies = ["OldestLaunchTemplate", "Default"]

  tag {
    key                 = "Name"
    value               = "lab-ha-web"
    propagate_at_launch = true
  }
}

# ──────────────────────────────────────────────────────
# 5. Target Tracking Policy (baseada em ALB requests)
# ──────────────────────────────────────────────────────

resource "aws_autoscaling_policy" "target_tracking" {
  name                   = "lab-ha-target-tracking"
  autoscaling_group_name = aws_autoscaling_group.web.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    # Escalar para manter 100 requests/target por minuto
    predefined_metric_specification {
      predefined_metric_type = "ALBRequestCountPerTarget"
      resource_label = "${aws_lb.main.arn_suffix}/${aws_lb_target_group.web.arn_suffix}"
    }
    target_value       = 100.0
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

# ──────────────────────────────────────────────────────
# 6. Lifecycle Hook via EventBridge
# ──────────────────────────────────────────────────────

resource "aws_sns_topic" "lifecycle" {
  name = "lab-ha-lifecycle-notifications"
}

resource "aws_autoscaling_lifecycle_hook" "launch_hook" {
  name                   = "lab-launch-hook"
  autoscaling_group_name = aws_autoscaling_group.web.name
  lifecycle_transition   = "autoscaling:EC2_INSTANCE_LAUNCHING"
  default_result         = "CONTINUE"
  heartbeat_timeout      = 120 # lab: 2 minutos

  notification_target_arn = aws_sns_topic.lifecycle.arn
  role_arn                = aws_iam_role.asg_lifecycle_role.arn
}

resource "aws_iam_role" "asg_lifecycle_role" {
  name = "lab-asg-lifecycle-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { Service = "autoscaling.amazonaws.com" }
      Action    = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy" "asg_lifecycle_policy" {
  role = aws_iam_role.asg_lifecycle_role.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["sns:Publish"]
      Resource = aws_sns_topic.lifecycle.arn
    }]
  })
}

# ──────────────────────────────────────────────────────
# 7. CloudWatch Alarm para Step Scaling adicional
# ──────────────────────────────────────────────────────

resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "lab-ha-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 60
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "Trigger scale-out quando CPU > 80% por 2 minutos"

  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web.name
  }

  alarm_actions = [aws_autoscaling_policy.step_scale_out.arn]
}

resource "aws_autoscaling_policy" "step_scale_out" {
  name                   = "lab-ha-step-scale-out"
  autoscaling_group_name = aws_autoscaling_group.web.name
  policy_type            = "StepScaling"
  adjustment_type        = "ChangeInCapacity"

  step_adjustment {
    scaling_adjustment          = 1
    metric_interval_lower_bound = 0
    metric_interval_upper_bound = 20
  }

  step_adjustment {
    scaling_adjustment          = 2
    metric_interval_lower_bound = 20
  }
}

# ──────────────────────────────────────────────────────
# Outputs
# ──────────────────────────────────────────────────────

output "alb_dns" {
  value       = aws_lb.main.dns_name
  description = "DNS do ALB para teste"
}

output "asg_name" {
  value = aws_autoscaling_group.web.name
}
```

---

## Parte 3: Stress Test e Monitoramento

```bash
# Obter DNS do ALB
ALB_DNS=$(terraform output -raw alb_dns)

# Testar routing
curl http://$ALB_DNS/         # → TG-web
curl http://$ALB_DNS/health   # → health check resposta

# Gerar carga para disparar scale-out (emula via ab - Apache Benchmark)
# (instalar: yum install -y httpd-tools)
ab -n 10000 -c 50 http://$ALB_DNS/ &

# Monitorar ASG em tempo real
watch -n5 "aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names lab-ha-asg \
  --query 'AutoScalingGroups[0].{Min:MinSize,Max:MaxSize,Desired:DesiredCapacity,Instances:Instances[*].LifecycleState}' \
  --output json --region us-east-1"

# Ver atividades de scaling
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name lab-ha-asg \
  --region us-east-1 \
  --query "Activities[0:5].{Cause:Cause,Status:StatusCode,Time:StartTime}" \
  --output table
```

---

## Cleanup

```bash
# Terraform destroy (remove tudo)
terraform destroy -auto-approve

# Ou CLI para recursos criados manualmente:
# Deletar Listener e ALB
aws elbv2 delete-listener --listener-arn $LISTENER_ARN --region us-east-1
aws elbv2 delete-load-balancer --load-balancer-arn $ALB_ARN --region us-east-1

# Deletar Target Groups
aws elbv2 delete-target-group --target-group-arn $TG_API --region us-east-1
aws elbv2 delete-target-group --target-group-arn $TG_WEB --region us-east-1

# Deletar Security Groups
aws ec2 delete-security-group --group-id $EC2_SG --region us-east-1
aws ec2 delete-security-group --group-id $ALB_SG --region us-east-1

echo "Cleanup concluído."
```

---

## Pontos de Revisão

- [ ] Por que o `deregistration_delay` foi reduzido para 30s no lab?
- [ ] O que acontece se o health check na `/health` retornar 404?
- [ ] Com Target Tracking configurado para ALBRequestCountPerTarget=100, quantas instâncias são necessárias para 500 req/min?
- [ ] O Lifecycle Hook em `EC2_INSTANCE_LAUNCHING` afeta o scale-in?
- [ ] Por que o Step Scaling adiciona 1 instância para CPU 80-100% mas 2 para >100%?
