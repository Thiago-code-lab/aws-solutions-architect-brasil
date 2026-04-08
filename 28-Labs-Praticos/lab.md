# Lab Prático — Diagnóstico e Debugging: Ambientes Quebrados (Módulo 18)

> **Região:** us-east-1 | **Custo estimado:** ~$0.02  
> **Pré-requisitos:** AWS CLI configurado, Docker (opcional), Python 3.8+

---

## Objetivo

Praticar diagnóstico de problemas comuns em ambientes AWS: Lambda com timeout em VPC, S3 com erro 403, DynamoDB com throttling, e ECS com task que para imediatamente.

---

## Parte 1 — Lambda Timeout em VPC (Problema Clássico)

```bash
# Criar VPC sem NAT Gateway (causa comum de timeout Lambda → internet)
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.20.0.0/16 \
  --query 'Vpc.VpcId' --output text)
aws ec2 modify-vpc-attribute --vpc-id "$VPC_ID" --enable-dns-hostnames

SUBNET_PRIV=$(aws ec2 create-subnet \
  --vpc-id "$VPC_ID" \
  --cidr-block 10.20.1.0/24 \
  --availability-zone us-east-1a \
  --query 'Subnet.SubnetId' --output text)

SG_ID=$(aws ec2 create-security-group \
  --group-name "lab-debug-sg" \
  --description "Lab debugging" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)

# IAM Role para Lambda
LAMBDA_ROLE=$(aws iam create-role \
  --role-name "lab-debug-lambda-role" \
  --assume-role-policy-document '{
    "Version":"2012-10-17",
    "Statement":[{"Action":"sts:AssumeRole","Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"}}]
  }' \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-debug-lambda-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"

sleep 10  # propagar IAM

# Criar Lambda DENTRO da VPC sem NAT — vai dar timeout ao tentar chamar APIs externas
zip -j /tmp/lambda_debug.zip - << 'PYTHON'
import urllib.request
import json

def handler(event, context):
    print("Tentando acessar API externa...")
    try:
        # Isso vai dar timeout em Lambda na VPC sem NAT Gateway
        req = urllib.request.urlopen("https://httpbin.org/get", timeout=5)
        data = json.loads(req.read())
        print(f"Sucesso: {data}")
    except Exception as e:
        print(f"ERRO: {type(e).__name__}: {e}")
        raise e
    return {"status": "ok"}
PYTHON

aws lambda create-function \
  --function-name "lab-debug-vpc-timeout" \
  --runtime python3.12 \
  --role "$LAMBDA_ROLE" \
  --handler lambda_function.handler \
  --zip-file fileb:///tmp/lambda_debug.zip \
  --timeout 15 \
  --vpc-config "SubnetIds=${SUBNET_PRIV},SecurityGroupIds=${SG_ID}"

# Invocar para reproduzir o problema
echo "Invocando Lambda (vai dar timeout)..."
aws lambda invoke \
  --function-name "lab-debug-vpc-timeout" \
  --payload '{}' \
  --cli-binary-format raw-in-base64-out \
  /tmp/debug_output.json
cat /tmp/debug_output.json

# DIAGNÓSTICO: Verificar logs para confirmar o problema
sleep 5
aws logs filter-log-events \
  --log-group-name "/aws/lambda/lab-debug-vpc-timeout" \
  --filter-pattern "ERRO" \
  --query 'events[*].message' \
  --output text

echo ""
echo "=== SOLUÇÃO ==="
echo "Lambda em VPC privada sem NAT Gateway não consegue acessar a internet."
echo "Soluções:"
echo "  1. Adicionar NAT Gateway na subnet pública + route table"
echo "  2. Usar VPC Endpoints para serviços AWS (S3, DynamoDB, SQS)"
echo "  3. Remover Lambda da VPC se não precisar de recursos privados"
```

---

## Parte 2 — S3 403 Forbidden: Diagnóstico

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_403="lab-debug-403-${ACCOUNT_ID}"

# Criar bucket com Block Public Access habilitado
aws s3api create-bucket --bucket "$BUCKET_403"
aws s3api put-public-access-block \
  --bucket "$BUCKET_403" \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Subir objeto de teste
echo "Conteúdo secreto de diagnóstico" > /tmp/teste_403.txt
aws s3 cp /tmp/teste_403.txt "s3://${BUCKET_403}/teste_403.txt"

# Bucket Policy RESTRITIVA — simula acesso negado
aws s3api put-bucket-policy \
  --bucket "$BUCKET_403" \
  --policy "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Sid\": \"DenyEveryone\",
      \"Effect\": \"Deny\",
      \"Principal\": \"*\",
      \"Action\": \"s3:GetObject\",
      \"Resource\": \"arn:aws:s3:::${BUCKET_403}/*\",
      \"Condition\": {
        \"StringNotEquals\": {
          \"aws:PrincipalAccount\": \"999999999999\"
        }
      }
    }]
  }"

# Tentar acessar — vai dar 403
echo "Tentando acessar objeto (esperado: 403)..."
aws s3 cp "s3://${BUCKET_403}/teste_403.txt" /tmp/download_403.txt 2>&1 || true

echo ""
echo "=== DIAGNÓSTICO 403 S3 ==="
echo "Checklist:"

# 1. Block Public Access
echo "1. Block Public Access:"
aws s3api get-public-access-block --bucket "$BUCKET_403" 2>/dev/null

# 2. Bucket Policy
echo "2. Bucket Policy:"
aws s3api get-bucket-policy --bucket "$BUCKET_403" 2>/dev/null | python3 -m json.tool 2>/dev/null

# 3. ACL do objeto
echo "3. ACL do objeto:"
aws s3api get-object-acl --bucket "$BUCKET_403" --key "teste_403.txt" 2>/dev/null

# 4. Encryption (necessária pemissão kms:Decrypt?)
echo "4. Encryption do objeto:"
aws s3api head-object --bucket "$BUCKET_403" --key "teste_403.txt" \
  --query '[ServerSideEncryption, SSEKMSKeyId]' 2>/dev/null

# CORREÇÃO: ajustar policy para permitir conta atual
aws s3api put-bucket-policy \
  --bucket "$BUCKET_403" \
  --policy "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Sid\": \"PermitirContaAtual\",
      \"Effect\": \"Allow\",
      \"Principal\": {\"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\"},
      \"Action\": \"s3:GetObject\",
      \"Resource\": \"arn:aws:s3:::${BUCKET_403}/*\"
    }]
  }"

echo "Policy corrigida. Tentando acessar novamente..."
aws s3 cp "s3://${BUCKET_403}/teste_403.txt" /tmp/download_403_ok.txt && cat /tmp/download_403_ok.txt
```

---

## Parte 3 — DynamoDB Throttling

```bash
# Criar tabela com capacidade mínima
TABELA="lab-debug-throttle"
aws dynamodb create-table \
  --table-name "$TABELA" \
  --attribute-definitions "AttributeName=pk,AttributeType=S" \
  --key-schema "AttributeName=pk,KeyType=HASH" \
  --billing-mode PROVISIONED \
  --provisioned-throughput "ReadCapacityUnits=1,WriteCapacityUnits=1"

aws dynamodb wait table-exists --table-name "$TABELA"

# Script Python de carga (provoca throttle)
cat > /tmp/carga_dynamo.py << 'PYTHON'
import boto3
from botocore.exceptions import ClientError
import time
import json

dynamodb = boto3.client('dynamodb', region_name='us-east-1')
TABELA = 'lab-debug-throttle'

erros_throttle = 0
sucesso = 0

print("Enviando 50 escritas rápidas (WCU=1, vai throttle)...")
for i in range(50):
    try:
        dynamodb.put_item(
            TableName=TABELA,
            Item={'pk': {'S': f'item-{i}'}, 'valor': {'N': str(i)}}
        )
        sucesso += 1
    except ClientError as e:
        if e.response['Error']['Code'] in ['ProvisionedThroughputExceededException', 'RequestLimitExceeded']:
            erros_throttle += 1
        else:
            raise

print(f"Resultados: {sucesso} sucessos, {erros_throttle} throttling")

# Verificar métricas de throttle
print("\nPara ver throttle no CloudWatch:")
print("aws cloudwatch get-metric-statistics \\")
print("  --namespace AWS/DynamoDB \\")
print("  --metric-name WriteThrottleEvents \\")
print("  --dimensions Name=TableName,Value=lab-debug-throttle \\")
print("  --start-time $(date -d '-5 minutes' --utc +%FT%TZ) \\")
print("  --end-time $(date --utc +%FT%TZ) \\")
print("  --period 60 --statistics Sum")
PYTHON

python3 /tmp/carga_dynamo.py

echo ""
echo "=== SOLUÇÕES PARA THROTTLE DYNAMODB ==="
echo "1. Aumentar WCU/RCU provisionado"
echo "2. Habilitar Auto Scaling (recomendado)"
echo "3. Mudar para PAY_PER_REQUEST (on-demand)"
echo "4. Implementar exponential backoff no cliente"
echo "5. Usar DAX para reads repetidos"

# Demonstrar: mudar para on-demand
aws dynamodb update-table \
  --table-name "$TABELA" \
  --billing-mode PAY_PER_REQUEST
echo "Tabela migrada para PAY_PER_REQUEST."
```

---

## Parte 4 — ECS Task Stopped: Diagnóstico

```bash
# Criar cluster ECS
aws ecs create-cluster --cluster-name "lab-debug-cluster"

# IAM Role para ECS Task Execution
EXEC_ROLE=$(aws iam create-role \
  --role-name "lab-ecs-exec-role" \
  --assume-role-policy-document '{
    "Version":"2012-10-17",
    "Statement":[{"Action":"sts:AssumeRole","Effect":"Allow","Principal":{"Service":"ecs-tasks.amazonaws.com"}}]
  }' \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-ecs-exec-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"

# Registrar task com imagem inexistente (vai falhar com EssentialTaskExited)
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
aws ecs register-task-definition \
  --family "lab-debug-task" \
  --network-mode awsvpc \
  --requires-compatibilities FARGATE \
  --cpu "256" \
  --memory "512" \
  --execution-role-arn "$EXEC_ROLE" \
  --container-definitions "[{
    \"name\": \"app\",
    \"image\": \"${ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/nao-existe:latest\",
    \"essential\": true,
    \"logConfiguration\": {
      \"logDriver\": \"awslogs\",
      \"options\": {
        \"awslogs-group\": \"/ecs/lab-debug\",
        \"awslogs-region\": \"us-east-1\",
        \"awslogs-stream-prefix\": \"ecs\"
      }
    }
  }]"

echo ""
echo "=== DIAGNÓSTICO ECS TASK STOPPED ==="
echo ""
echo "Causas comuns de stoppedReason:"
echo ""
echo "  CannotPullContainerError:"
echo "    - Imagem não existe no ECR"
echo "    - Sem acesso ao ECR (IAM ou VPC Endpoint)"
echo "    - Task na subnet privada sem NAT"
echo ""
echo "  EssentialTaskExited:"
echo "    - Container saiu com código não-zero"
echo "    - Comando entrypoint inválido"
echo "    - Out of Memory (subir memory limit)"
echo ""
echo "  ResourceInitializationError:"
echo "    - Secrets Manager/Parameter Store inacessível"
echo "    - VPC Endpoint ausente para secrets"
echo ""
echo "Comandos de diagnóstico:"
echo "aws ecs describe-tasks --cluster <cluster> --tasks <task-arn>"
echo "  → olhar: stoppedReason, containers[0].reason, exitCode"
echo ""
echo "aws logs get-log-events --log-group-name /ecs/<task> --log-stream-name <stream>"
echo "  → logs do container antes de morrer"
echo ""
echo "aws ecs describe-task-definition --task-definition <name>"
echo "  → verificar image, environment, secrets"
```

---

## Limpeza

```bash
# Deletar recursos DynamoDB
aws dynamodb delete-table --table-name "$TABELA"

# Deletar recursos ECS
aws ecs delete-cluster --cluster "lab-debug-cluster"
aws iam detach-role-policy \
  --role-name "lab-ecs-exec-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
aws iam delete-role --role-name "lab-ecs-exec-role"

# Deletar Lambda e VPC
aws lambda delete-function --function-name "lab-debug-vpc-timeout"
aws iam detach-role-policy \
  --role-name "lab-debug-lambda-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"
aws iam delete-role --role-name "lab-debug-lambda-role"

# Deletar Security Group, Subnet e VPC
aws ec2 delete-security-group --group-id "$SG_ID"
aws ec2 delete-subnet --subnet-id "$SUBNET_PRIV"
aws ec2 delete-vpc --vpc-id "$VPC_ID"

# Deletar resources S3
aws s3 rm "s3://${BUCKET_403}" --recursive
aws s3api delete-bucket --bucket "$BUCKET_403"

# Arquivos temporários
rm -f /tmp/lambda_debug.zip /tmp/teste_403.txt /tmp/download_403.txt \
      /tmp/download_403_ok.txt /tmp/carga_dynamo.py /tmp/debug_output.json
```

---

## O Que Você Aprendeu

- **Lambda + VPC timeout:** subnet privada sem NAT = sem internet; usar VPC Endpoints para AWS APIs
- **S3 403 checklist:** Block Public Access → Bucket Policy → Object ACL → KMS permissions → IAM Policy
- **DynamoDB throttle:** WCU/RCU excedido → logs CloudWatch `WriteThrottleEvents`; solução: on-demand ou Auto Scaling
- **ECS stoppedReason:** fonte primária de diagnóstico; sempre checar `exitCode` e CloudWatch Logs do container
- **Debugging mindset:** reproduzir → confirmar nos logs → identificar causa raiz → aplicar menor mudança → validar
