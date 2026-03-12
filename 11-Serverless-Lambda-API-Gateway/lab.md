# Lab Prático — Serverless: Lambda + API Gateway + DynamoDB (Módulo 11)

> **Região:** us-east-1 | **Custo estimado:** < $0.10 (Free Tier cobre toda a parte Lambda/DynamoDB)  
> **Pré-requisitos:** AWS CLI configurado, SAM CLI instalado, Python 3.12

---

## Objetivo

Criar uma API REST serverless completa com autenticação, que permite **criar** e **consultar pedidos** usando Lambda + API Gateway + DynamoDB + Cognito.

---

## Parte 1 — Infraestrutura com Terraform

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

# DynamoDB — tabela de pedidos
resource "aws_dynamodb_table" "pedidos" {
  name         = "lab-pedidos"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "pedidoId"

  attribute {
    name = "pedidoId"
    type = "S"
  }

  ttl {
    attribute_name = "expiracao"
    enabled        = true
  }

  tags = {
    Environment = "lab"
    Module      = "11-serverless"
  }
}

# IAM Role para Lambda
resource "aws_iam_role" "lambda_role" {
  name = "lab-lambda-pedidos-role"

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
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_iam_role_policy" "lambda_dynamo" {
  name = "lab-lambda-dynamo-policy"
  role = aws_iam_role.lambda_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ]
      Resource = aws_dynamodb_table.pedidos.arn
    }]
  })
}

# Lambda — criar pedido
resource "aws_lambda_function" "criar_pedido" {
  function_name = "lab-criar-pedido"
  role          = aws_iam_role.lambda_role.arn
  handler       = "criar_pedido.lambda_handler"
  runtime       = "python3.12"
  timeout       = 30
  memory_size   = 256
  filename      = "lambda_package.zip"

  environment {
    variables = {
      TABLE_NAME = aws_dynamodb_table.pedidos.name
    }
  }
}

# Lambda — consultar pedido
resource "aws_lambda_function" "consultar_pedido" {
  function_name = "lab-consultar-pedido"
  role          = aws_iam_role.lambda_role.arn
  handler       = "consultar_pedido.lambda_handler"
  runtime       = "python3.12"
  timeout       = 30
  memory_size   = 128
  filename      = "lambda_package.zip"

  environment {
    variables = {
      TABLE_NAME = aws_dynamodb_table.pedidos.name
    }
  }
}

# API Gateway
resource "aws_apigatewayv2_api" "pedidos_api" {
  name          = "lab-pedidos-api"
  protocol_type = "HTTP"

  cors_configuration {
    allow_origins = ["*"]
    allow_methods = ["GET", "POST"]
    allow_headers = ["Content-Type", "Authorization"]
  }
}

resource "aws_apigatewayv2_stage" "default" {
  api_id      = aws_apigatewayv2_api.pedidos_api.id
  name        = "$default"
  auto_deploy = true
}

# Integrações Lambda ↔ API GW
resource "aws_apigatewayv2_integration" "criar" {
  api_id             = aws_apigatewayv2_api.pedidos_api.id
  integration_type   = "AWS_PROXY"
  integration_uri    = aws_lambda_function.criar_pedido.invoke_arn
  integration_method = "POST"
}

resource "aws_apigatewayv2_integration" "consultar" {
  api_id             = aws_apigatewayv2_api.pedidos_api.id
  integration_type   = "AWS_PROXY"
  integration_uri    = aws_lambda_function.consultar_pedido.invoke_arn
  integration_method = "POST"
}

resource "aws_apigatewayv2_route" "criar_route" {
  api_id    = aws_apigatewayv2_api.pedidos_api.id
  route_key = "POST /pedidos"
  target    = "integrations/${aws_apigatewayv2_integration.criar.id}"
}

resource "aws_apigatewayv2_route" "consultar_route" {
  api_id    = aws_apigatewayv2_api.pedidos_api.id
  route_key = "GET /pedidos/{pedidoId}"
  target    = "integrations/${aws_apigatewayv2_integration.consultar.id}"
}

# Permissão Lambda para API GW invocar
resource "aws_lambda_permission" "criar_apigw" {
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.criar_pedido.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.pedidos_api.execution_arn}/*/*"
}

resource "aws_lambda_permission" "consultar_apigw" {
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.consultar_pedido.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.pedidos_api.execution_arn}/*/*"
}

output "api_url" {
  value = aws_apigatewayv2_api.pedidos_api.api_endpoint
}
```

---

## Parte 2 — Código Lambda

```python
# criar_pedido.py
import json
import boto3
import uuid
import os
from datetime import datetime, timezone, timedelta

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def lambda_handler(event, context):
    body = json.loads(event.get('body', '{}'))
    
    if not body.get('produto') or not body.get('quantidade'):
        return {
            'statusCode': 400,
            'body': json.dumps({'erro': 'produto e quantidade são obrigatórios'})
        }
    
    pedido_id = str(uuid.uuid4())
    expiracao = int((datetime.now(timezone.utc) + timedelta(days=30)).timestamp())
    
    item = {
        'pedidoId': pedido_id,
        'produto': body['produto'],
        'quantidade': int(body['quantidade']),
        'status': 'pendente',
        'criadoEm': datetime.now(timezone.utc).isoformat(),
        'expiracao': expiracao
    }
    
    table.put_item(Item=item)
    
    return {
        'statusCode': 201,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({'pedidoId': pedido_id, 'status': 'pendente'})
    }
```

```python
# consultar_pedido.py
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def lambda_handler(event, context):
    pedido_id = event['pathParameters']['pedidoId']
    
    response = table.get_item(Key={'pedidoId': pedido_id})
    
    if 'Item' not in response:
        return {
            'statusCode': 404,
            'body': json.dumps({'erro': 'pedido não encontrado'})
        }
    
    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps(response['Item'], default=str)
    }
```

---

## Parte 3 — Deploy e Teste

```bash
# 1. Empacotar Lambda
zip -j lambda_package.zip criar_pedido.py consultar_pedido.py

# 2. Deploy infraestrutura
terraform init
terraform plan
terraform apply -auto-approve

# 3. Capturar URL da API
API_URL=$(terraform output -raw api_url)

# 4. Criar pedido
curl -X POST "${API_URL}/pedidos" \
  -H "Content-Type: application/json" \
  -d '{"produto": "Notebook", "quantidade": 2}'

# Esperado: {"pedidoId": "uuid-xxx", "status": "pendente"}

# 5. Consultar pedido
PEDIDO_ID="uuid-do-passo-anterior"
curl "${API_URL}/pedidos/${PEDIDO_ID}"

# 6. Verificar logs
aws logs tail /aws/lambda/lab-criar-pedido --since 5m

# 7. Verificar DynamoDB
aws dynamodb scan --table-name lab-pedidos \
  --query 'Items[*].[pedidoId.S, produto.S, status.S]' \
  --output table
```

---

## Parte 4 — Teste de Concorrência e Escalonamento

```bash
# Enviar 100 pedidos simultâneos (requer 'ab' ou wrk)
# Com curl em loop:
for i in $(seq 1 20); do
  curl -s -X POST "${API_URL}/pedidos" \
    -H "Content-Type: application/json" \
    -d "{\"produto\": \"Item-${i}\", \"quantidade\": ${i}}" &
done
wait

# Ver métricas de concorrência Lambda no CloudWatch
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name ConcurrentExecutions \
  --dimensions Name=FunctionName,Value=lab-criar-pedido \
  --start-time $(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 60 \
  --statistics Maximum
```

---

## Limpeza

```bash
# Destruir TODOS os recursos (evita custos)
terraform destroy -auto-approve

# Verificar que DynamoDB foi removido
aws dynamodb list-tables --query 'TableNames[?contains(@, `lab`)]'

# Verificar que Lambdas foram removidas
aws lambda list-functions --query 'Functions[?contains(FunctionName, `lab`)].FunctionName'
```

---

## O Que Você Aprendeu

- Lambda executa código sem servidor, cobrando apenas por invocação
- API Gateway HTTP API é mais barato e simples que REST API para Lambda
- DynamoDB PAY_PER_REQUEST: sem capacidade provisionada, paga por requisição
- IAM Role de Lambda precisa ser explicitamente autorizada (princípio menor privilégio)
- TTL no DynamoDB expira itens automaticamente (gratuito, sem cobrança extra)
- `terraform destroy` remove tudo — sempre fazer ao final do lab
