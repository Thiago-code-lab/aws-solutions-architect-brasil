# Lab Prático — Introdução SAA-C03

## 🎯 Objetivo

Preparar um ambiente de estudos seguro e econômico para os próximos módulos, validando acesso à conta, região padrão, identificação da conta AWS e criação de uma trilha mínima de governança com orçamento e bucket de apoio. O lab ajuda o aluno a começar os estudos com disciplina de custo e visibilidade operacional.

## 📋 Pré-requisitos

- AWS CLI configurado com um perfil válido
- Terraform >= 1.5
- Permissões IAM necessárias: sts:GetCallerIdentity, s3:CreateBucket, s3:PutBucketTagging, budgets:CreateBudget, sns:CreateTopic, sns:Subscribe, iam:PassRole quando aplicável
- Região padrão: us-east-1

## 🏗️ Arquitetura do Lab

```text
Usuário
  |
  +-> AWS CLI profile
          |
          +-> STS GetCallerIdentity
          +-> S3 Bucket de estudos
          +-> AWS Budget mensal -> SNS Email
```

## 🔧 Passo a Passo

### Parte 1 — Via AWS CLI

```bash
# Confirmar a identidade ativa e evitar executar labs na conta errada
aws sts get-caller-identity --profile default --region us-east-1

# Confirmar a região efetiva usada pelo perfil
aws configure get region --profile default

# Criar bucket único para artefatos de estudo
aws s3api create-bucket \
  --bucket saa-c03-study-artifacts-123456789012 \
  --region us-east-1

# Marcar o bucket para facilitar limpeza e análise de custo
aws s3api put-bucket-tagging \
  --bucket saa-c03-study-artifacts-123456789012 \
  --tagging 'TagSet=[{Key=Project,Value=SAA-C03},{Key=Owner,Value=Study},{Key=Environment,Value=Lab}]'

# Criar um tópico SNS para receber alerta de orçamento
aws sns create-topic \
  --name saa-c03-budget-alerts \
  --region us-east-1

# Substitua pelo ARN retornado no comando anterior
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:saa-c03-budget-alerts \
  --protocol email \
  --notification-endpoint seu-email@exemplo.com \
  --region us-east-1
```

### Parte 2 — Via Terraform

```hcl
# main.tf
terraform {
  required_version = ">= 1.5.0"

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

variable "alert_email" {
  type        = string
  description = "Email que receberá alertas de budget"
}

data "aws_caller_identity" "current" {}

resource "aws_s3_bucket" "study_artifacts" {
  bucket = "saa-c03-study-artifacts-${data.aws_caller_identity.current.account_id}"

  tags = {
    Project     = "SAA-C03"
    Environment = "Lab"
    Owner       = "Study"
  }
}

resource "aws_s3_bucket_versioning" "study_artifacts" {
  bucket = aws_s3_bucket.study_artifacts.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_sns_topic" "budget_alerts" {
  name = "saa-c03-budget-alerts"
}

resource "aws_sns_topic_subscription" "email" {
  topic_arn = aws_sns_topic.budget_alerts.arn
  protocol  = "email"
  endpoint  = var.alert_email
}

resource "aws_budgets_budget" "monthly" {
  name         = "saa-c03-monthly-budget"
  budget_type  = "COST"
  limit_amount = "10"
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  cost_filter {
    name   = "TagKeyValue"
    values = ["Project$SAA-C03"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_sns_topic_arns  = [aws_sns_topic.budget_alerts.arn]
  }
}
```

## ✅ Validação

```bash
# Validar bucket
aws s3api get-bucket-tagging --bucket saa-c03-study-artifacts-123456789012 --region us-east-1

# Validar assinatura SNS
aws sns list-subscriptions-by-topic \
  --topic-arn arn:aws:sns:us-east-1:123456789012:saa-c03-budget-alerts \
  --region us-east-1

# Validar identidade da conta
aws sts get-caller-identity --region us-east-1
```

## 🧹 Limpeza de Recursos

```bash
# Remover conteúdo e bucket
aws s3 rm s3://saa-c03-study-artifacts-123456789012 --recursive --region us-east-1
aws s3api delete-bucket --bucket saa-c03-study-artifacts-123456789012 --region us-east-1

# Remover tópico SNS
aws sns delete-topic \
  --topic-arn arn:aws:sns:us-east-1:123456789012:saa-c03-budget-alerts \
  --region us-east-1

# Se usou Terraform
terraform destroy -var="alert_email=seu-email@exemplo.com"
```

## 💰 Custo Estimado

Se o bucket estiver praticamente vazio e o tópico SNS tiver baixo volume, o custo por 1 hora tende a ser muito próximo de zero. O componente mais sensível costuma ser o AWS Budgets, dependendo da conta e do tipo de uso. Em termos práticos, é um lab de custo muito baixo, mas ainda assim deve ser limpo ao final.
