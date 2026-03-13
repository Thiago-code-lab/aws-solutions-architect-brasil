# Lab Prático — Casos de Uso Reais: SNS Fan-out + Step Functions + EventBridge (Módulo 17)

> **Região:** us-east-1 | **Custo estimado:** ~$0.01 (Step Functions Express, SNS, SQS — volumes de lab)  
> **Pré-requisitos:** AWS CLI configurado, Terraform >= 1.5

---

## Objetivo

Implementar padrões arquiteturais comuns no mercado: fan-out assíncrono com SNS+SQS, orquestração de processos com Step Functions Express Workflows, e event-driven com EventBridge.

---

## Parte 1 — Terraform: Fan-out SNS + SQS + Lambda

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

provider "aws" { region = "us-east-1" }

data "aws_caller_identity" "current" {}

locals {
  prefix = "lab-fanout"
  region = "us-east-1"
  account_id = data.aws_caller_identity.current.account_id
}

# IAM Role para Lambdas
resource "aws_iam_role" "lambda_exec" {
  name = "${local.prefix}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "lambda.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_basic" {
  role       = aws_iam_role.lambda_exec.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_iam_role_policy" "lambda_sqs" {
  name = "${local.prefix}-sqs-policy"
  role = aws_iam_role.lambda_exec.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["sqs:ReceiveMessage", "sqs:DeleteMessage", "sqs:GetQueueAttributes"]
      Resource = "*"
    }]
  })
}

# SNS Topic principal
resource "aws_sns_topic" "pedidos" {
  name = "${local.prefix}-pedidos"
}

# SQS fila para processamento de estoque
resource "aws_sqs_queue" "estoque_dlq" {
  name                       = "${local.prefix}-estoque-dlq"
  message_retention_seconds  = 1209600
}

resource "aws_sqs_queue" "estoque" {
  name                       = "${local.prefix}-estoque"
  visibility_timeout_seconds = 60
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.estoque_dlq.arn
    maxReceiveCount     = 3
  })
}

# SQS fila para envio de e-mail
resource "aws_sqs_queue" "email_dlq" {
  name = "${local.prefix}-email-dlq"
}

resource "aws_sqs_queue" "email" {
  name                       = "${local.prefix}-email"
  visibility_timeout_seconds = 60
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.email_dlq.arn
    maxReceiveCount     = 3
  })
}

# Políticas de SQS para receber do SNS
data "aws_iam_policy_document" "sqs_estoque" {
  statement {
    principals {
      type        = "Service"
      identifiers = ["sns.amazonaws.com"]
    }
    actions   = ["sqs:SendMessage"]
    resources = [aws_sqs_queue.estoque.arn]
    condition {
      test     = "ArnEquals"
      variable = "aws:SourceArn"
      values   = [aws_sns_topic.pedidos.arn]
    }
  }
}

resource "aws_sqs_queue_policy" "estoque" {
  queue_url = aws_sqs_queue.estoque.id
  policy    = data.aws_iam_policy_document.sqs_estoque.json
}

data "aws_iam_policy_document" "sqs_email" {
  statement {
    principals {
      type        = "Service"
      identifiers = ["sns.amazonaws.com"]
    }
    actions   = ["sqs:SendMessage"]
    resources = [aws_sqs_queue.email.arn]
    condition {
      test     = "ArnEquals"
      variable = "aws:SourceArn"
      values   = [aws_sns_topic.pedidos.arn]
    }
  }
}

resource "aws_sqs_queue_policy" "email" {
  queue_url = aws_sqs_queue.email.id
  policy    = data.aws_iam_policy_document.sqs_email.json
}

# Subscrições SNS → SQS
resource "aws_sns_topic_subscription" "para_estoque" {
  topic_arn = aws_sns_topic.pedidos.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.estoque.arn
  filter_policy = jsonencode({
    tipo = ["pedido_criado"]
  })
}

resource "aws_sns_topic_subscription" "para_email" {
  topic_arn = aws_sns_topic.pedidos.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.email.arn
}

# Lambda processadora de estoque
data "archive_file" "lambda_estoque" {
  type        = "zip"
  output_path = "/tmp/lambda_estoque.zip"
  source {
    content  = <<PYTHON
import json

def handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        msg = json.loads(body.get('Message', '{}'))
        print(f"[ESTOQUE] Baixando estoque para produto: {msg.get('produto_id', 'DESCONHECIDO')}")
        print(f"[ESTOQUE] Quantidade: {msg.get('quantidade', 0)}")
    return {'statusCode': 200}
PYTHON
    filename = "handler.py"
  }
}

resource "aws_lambda_function" "estoque" {
  filename         = data.archive_file.lambda_estoque.output_path
  source_code_hash = data.archive_file.lambda_estoque.output_base64sha256
  function_name    = "${local.prefix}-estoque"
  role             = aws_iam_role.lambda_exec.arn
  handler          = "handler.handler"
  runtime          = "python3.12"
  timeout          = 30
}

resource "aws_lambda_event_source_mapping" "estoque" {
  event_source_arn = aws_sqs_queue.estoque.arn
  function_name    = aws_lambda_function.estoque.arn
  batch_size       = 5
}

# Lambda processadora de email
data "archive_file" "lambda_email" {
  type        = "zip"
  output_path = "/tmp/lambda_email.zip"
  source {
    content  = <<PYTHON
import json

def handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        msg = json.loads(body.get('Message', '{}'))
        print(f"[EMAIL] Enviando confirmação para: {msg.get('email_cliente', 'sem-email')}")
        print(f"[EMAIL] Pedido ID: {msg.get('pedido_id', 'SEM-ID')}")
    return {'statusCode': 200}
PYTHON
    filename = "handler.py"
  }
}

resource "aws_lambda_function" "email" {
  filename         = data.archive_file.lambda_email.output_path
  source_code_hash = data.archive_file.lambda_email.output_base64sha256
  function_name    = "${local.prefix}-email"
  role             = aws_iam_role.lambda_exec.arn
  handler          = "handler.handler"
  runtime          = "python3.12"
  timeout          = 30
}

resource "aws_lambda_event_source_mapping" "email" {
  event_source_arn = aws_sqs_queue.email.arn
  function_name    = aws_lambda_function.email.arn
  batch_size       = 5
}

output "sns_topic_arn" { value = aws_sns_topic.pedidos.arn }
output "fila_estoque_url" { value = aws_sqs_queue.estoque.id }
output "fila_email_url" { value = aws_sqs_queue.email.id }
```

---

## Parte 2 — Publicar Mensagens e Observar Fan-out

```bash
terraform init && terraform apply -auto-approve

SNS_ARN=$(terraform output -raw sns_topic_arn)

# Publicar mensagem — vai para AMBAS as filas
aws sns publish \
  --topic-arn "$SNS_ARN" \
  --subject "Novo Pedido" \
  --message '{
    "pedido_id": "PED-001",
    "produto_id": "PROD-42",
    "quantidade": 2,
    "email_cliente": "joao@exemplo.com",
    "tipo": "pedido_criado"
  }' \
  --message-attributes '{
    "tipo": {
      "DataType": "String",
      "StringValue": "pedido_criado"
    }
  }'

# Publicar sem atributo — vai apenas para fila email (sem filter)
aws sns publish \
  --topic-arn "$SNS_ARN" \
  --message '{
    "pedido_id": "PED-002",
    "produto_id": "PROD-10",
    "quantidade": 1,
    "email_cliente": "maria@exemplo.com"
  }'

# Verificar logs das Lambdas
sleep 15  # aguardar processamento

LOG_GROUP_ESTOQUE="/aws/lambda/lab-fanout-estoque"
LOG_GROUP_EMAIL="/aws/lambda/lab-fanout-email"

aws logs filter-log-events \
  --log-group-name "$LOG_GROUP_ESTOQUE" \
  --filter-pattern "[ESTOQUE]" \
  --query 'events[*].message' \
  --output text

aws logs filter-log-events \
  --log-group-name "$LOG_GROUP_EMAIL" \
  --filter-pattern "[EMAIL]" \
  --query 'events[*].message' \
  --output text
```

---

## Parte 3 — Step Functions Express Workflow

```bash
# IAM Role para Step Functions
SF_ROLE_ARN=$(aws iam create-role \
  --role-name "lab-sf-express-role" \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Action": "sts:AssumeRole",
      "Effect": "Allow",
      "Principal": {"Service": "states.amazonaws.com"}
    }]
  }' \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-sf-express-role" \
  --policy-arn "arn:aws:iam::aws:policy/CloudWatchLogsFullAccess"

# Criar State Machine (Express — baixa latência, mais barato)
SF_ARN=$(aws stepfunctions create-state-machine \
  --name "lab-workflow-pedido" \
  --type EXPRESS \
  --role-arn "$SF_ROLE_ARN" \
  --definition '{
    "Comment": "Workflow de processamento de pedido",
    "StartAt": "ValidarPedido",
    "States": {
      "ValidarPedido": {
        "Type": "Task",
        "Resource": "arn:aws:states:::lambda:invoke",
        "Parameters": {
          "FunctionName": "lab-fanout-estoque",
          "Payload.$": "$"
        },
        "Retry": [{
          "ErrorEquals": ["Lambda.ServiceException", "Lambda.AWSLambdaException"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }],
        "Catch": [{
          "ErrorEquals": ["States.ALL"],
          "Next": "PedidoRejeitado",
          "ResultPath": "$.erro"
        }],
        "Next": "VerificarEstoque"
      },
      "VerificarEstoque": {
        "Type": "Choice",
        "Choices": [{
          "Variable": "$.quantidade",
          "NumericLessThanEquals": 0,
          "Next": "EstoqueInsuficiente"
        }],
        "Default": "ProcessarPagamento"
      },
      "ProcessarPagamento": {
        "Type": "Pass",
        "Parameters": {
          "pedido_id.$": "$.pedido_id",
          "status": "PAGAMENTO_APROVADO",
          "timestamp.$": "$$.Execution.StartTime"
        },
        "Next": "NotificarCliente"
      },
      "NotificarCliente": {
        "Type": "Task",
        "Resource": "arn:aws:states:::lambda:invoke",
        "Parameters": {
          "FunctionName": "lab-fanout-email",
          "Payload.$": "$"
        },
        "End": true
      },
      "EstoqueInsuficiente": {
        "Type": "Fail",
        "Error": "EstoqueInsuficiente",
        "Cause": "Produto sem estoque disponível"
      },
      "PedidoRejeitado": {
        "Type": "Fail",
        "Error": "ValidacaoFalhou",
        "Cause": "Pedido não passou na validação"
      }
    }
  }' \
  --query 'stateMachineArn' --output text)

echo "State Machine: $SF_ARN"

# Executar workflow
EXEC_ARN=$(aws stepfunctions start-execution \
  --state-machine-arn "$SF_ARN" \
  --input '{
    "pedido_id": "PED-100",
    "produto_id": "PROD-42",
    "quantidade": 2,
    "email_cliente": "joao@exemplo.com"
  }' \
  --query 'executionArn' --output text)

# Aguardar e checar resultado
sleep 10
aws stepfunctions describe-execution \
  --execution-arn "$EXEC_ARN" \
  --query '[status, stopDate, output]' \
  --output text
```

---

## Limpeza

```bash
# Deletar Step Functions
aws stepfunctions delete-state-machine --state-machine-arn "$SF_ARN"
aws iam detach-role-policy \
  --role-name "lab-sf-express-role" \
  --policy-arn "arn:aws:iam::aws:policy/CloudWatchLogsFullAccess"
aws iam delete-role --role-name "lab-sf-express-role"

# Destruir Terraform (SNS, SQS, Lambdas)
terraform destroy -auto-approve

rm -f /tmp/lambda_estoque.zip /tmp/lambda_email.zip
```

---

## O Que Você Aprendeu

- **SNS Fan-out:** uma mensagem → múltiplos consumidores; filter policies filtram por atributo
- **SQS como buffer:** desacopla SNS da Lambda, garante entrega mesmo se Lambda falhar
- **DLQ:** após `maxReceiveCount` tentativas, mensagem vai para a DLQ — alertar via CloudWatch
- **Step Functions Express:** ideal para alta frequência, < 5min; Standard: long-running, exatamente uma vez
- **Retry + Catch:** Step Functions gerencia resiliência sem código de retry na Lambda
- **Choice State:** equivalente a `if/else` no workflow — ramifica baseado em dados da execução
