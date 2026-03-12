# Lab Prático — S3: Versionamento, Lifecycle, Replicação e Eventos (Módulo 08)

> **Região:** us-east-1 | **Custo estimado:** ~$0.01 (armazenamento mínimo, < 1h)  
> **Pré-requisitos:** AWS CLI configurado, Python 3.8+

---

## Objetivo

Praticar S3 avançado: versionamento de objetos, lifecycle policies, CRR, presigned URLs, e notificações de eventos com Lambda.

---

## Parte 1 — Versionamento e Lifecycle

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_MAIN="lab-s3-lab-${ACCOUNT_ID}"
BUCKET_REPLICA="lab-s3-replica-${ACCOUNT_ID}"

# Criar buckets
aws s3api create-bucket --bucket "$BUCKET_MAIN"

# Habilitar versionamento
aws s3api put-bucket-versioning \
  --bucket "$BUCKET_MAIN" \
  --versioning-configuration Status=Enabled

# Habilitar criptografia padrão
aws s3api put-bucket-encryption \
  --bucket "$BUCKET_MAIN" \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      },
      "BucketKeyEnabled": true
    }]
  }'

# Criar múltiplas versões do mesmo objeto
for i in 1 2 3; do
  echo "Versão ${i} do arquivo — $(date)" > /tmp/config.txt
  aws s3 cp /tmp/config.txt "s3://${BUCKET_MAIN}/config.txt"
  sleep 1
done

# Listar versões
aws s3api list-object-versions \
  --bucket "$BUCKET_MAIN" \
  --prefix "config.txt" \
  --query 'Versions[*].[VersionId, LastModified, IsLatest]' \
  --output table

# Recuperar versão específica
FIRST_VERSION=$(aws s3api list-object-versions \
  --bucket "$BUCKET_MAIN" \
  --prefix "config.txt" \
  --query 'Versions[-1].VersionId' --output text)

aws s3api get-object \
  --bucket "$BUCKET_MAIN" \
  --key "config.txt" \
  --version-id "$FIRST_VERSION" \
  /tmp/config_v1.txt

echo "Conteúdo da versão mais antiga:"
cat /tmp/config_v1.txt

# Lifecycle Policy
aws s3api put-bucket-lifecycle-configuration \
  --bucket "$BUCKET_MAIN" \
  --lifecycle-configuration '{
    "Rules": [
      {
        "ID": "mover-para-ia-30-dias",
        "Status": "Enabled",
        "Filter": {"Prefix": "logs/"},
        "Transitions": [
          {"Days": 30, "StorageClass": "STANDARD_IA"},
          {"Days": 90, "StorageClass": "GLACIER"}
        ],
        "Expiration": {"Days": 365}
      },
      {
        "ID": "limpar-versoes-antigas",
        "Status": "Enabled",
        "Filter": {"Prefix": ""},
        "NoncurrentVersionTransitions": [
          {"NoncurrentDays": 30, "StorageClass": "STANDARD_IA"}
        ],
        "NoncurrentVersionExpiration": {"NoncurrentDays": 90},
        "AbortIncompleteMultipartUpload": {"DaysAfterInitiation": 7}
      }
    ]
  }'

echo "Lifecycle configurado!"
```

---

## Parte 2 — Presigned URLs

```bash
# Criar arquivo privado para demo de presigned URL
echo "Documento confidencial de acesso temporário" > /tmp/documento_privado.txt
aws s3 cp /tmp/documento_privado.txt "s3://${BUCKET_MAIN}/privado/documento.txt"

# Verificar que o objeto é privado
aws s3api get-object-acl \
  --bucket "$BUCKET_MAIN" \
  --key "privado/documento.txt" \
  --query 'Grants[*].[Permission, Grantee.Type]' \
  --output table

# Gerar presigned URL (válida por 60 segundos)
PRESIGNED_URL=$(aws s3 presign \
  "s3://${BUCKET_MAIN}/privado/documento.txt" \
  --expires-in 60)

echo "Presigned URL (60s):"
echo "$PRESIGNED_URL" | head -c 100
echo "..."

# Usar a URL para download
curl -s "$PRESIGNED_URL" -o /tmp/download_presigned.txt
echo "Download via presigned URL:"
cat /tmp/download_presigned.txt

# Presigned URL para UPLOAD (PUT)
PRESIGNED_PUT=$(aws s3 presign \
  "s3://${BUCKET_MAIN}/uploads/arquivo_usuario.txt" \
  --expires-in 300)

echo "Presigned PUT URL gerada (simulando upload de cliente)"
# curl -X PUT "$PRESIGNED_PUT" -T /tmp/config.txt  # descomentar para testar
```

---

## Parte 3 — S3 Event Notification com Lambda

```bash
# IAM Role para Lambda
LAMBDA_ROLE_ARN=$(aws iam create-role \
  --role-name "lab-s3-event-lambda-role" \
  --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Action":"sts:AssumeRole","Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"}}]}' \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-s3-event-lambda-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"

sleep 10

# Lambda que processa novos objetos
zip -j /tmp/s3_event_handler.zip - << 'PYTHON'
import json

def handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key    = record['s3']['object']['key']
        size   = record['s3']['object']['size']
        event_name = record['eventName']
        print(f"[S3 EVENT] {event_name}: s3://{bucket}/{key} ({size} bytes)")
    return {'statusCode': 200}
PYTHON

FUNC_ARN=$(aws lambda create-function \
  --function-name "lab-s3-event-processor" \
  --runtime python3.12 \
  --role "$LAMBDA_ROLE_ARN" \
  --handler s3_event_handler.handler \
  --zip-file fileb:///tmp/s3_event_handler.zip \
  --query 'FunctionArn' --output text)

# Permissão para S3 invocar Lambda
aws lambda add-permission \
  --function-name "lab-s3-event-processor" \
  --principal s3.amazonaws.com \
  --action lambda:InvokeFunction \
  --statement-id "s3-invoke" \
  --source-arn "arn:aws:s3:::${BUCKET_MAIN}" \
  --source-account "$ACCOUNT_ID"

# Configurar notificação no S3
aws s3api put-bucket-notification-configuration \
  --bucket "$BUCKET_MAIN" \
  --notification-configuration "{
    \"LambdaFunctionConfigurations\": [{
      \"LambdaFunctionArn\": \"${FUNC_ARN}\",
      \"Events\": [\"s3:ObjectCreated:*\"],
      \"Filter\": {
        \"Key\": {
          \"FilterRules\": [{\"Name\": \"prefix\", \"Value\": \"uploads/\"}]
        }
      }
    }]
  }"

# Fazer upload para disparar evento
echo "Arquivo de teste $(date)" > /tmp/novo_upload.txt
aws s3 cp /tmp/novo_upload.txt "s3://${BUCKET_MAIN}/uploads/novo_arquivo.txt"

sleep 15

# Ver logs da Lambda
aws logs filter-log-events \
  --log-group-name "/aws/lambda/lab-s3-event-processor" \
  --filter-pattern "S3 EVENT" \
  --query 'events[*].message' \
  --output text
```

---

## Limpeza

```bash
# Deletar Lambda
aws lambda delete-function --function-name "lab-s3-event-processor"
aws iam detach-role-policy \
  --role-name "lab-s3-event-lambda-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
aws iam delete-role --role-name "lab-s3-event-lambda-role"

# Deletar todas as versões dos objetos S3
python3 << PYTHON
import boto3
s3 = boto3.client('s3', region_name='us-east-1')
bucket = "$BUCKET_MAIN"
paginator = s3.get_paginator('list_object_versions')
for page in paginator.paginate(Bucket=bucket):
    for obj_type in ['Versions', 'DeleteMarkers']:
        for item in page.get(obj_type, []):
            s3.delete_object(Bucket=bucket, Key=item['Key'], VersionId=item['VersionId'])
print("Todos os objetos e versões deletados")
PYTHON

aws s3api delete-bucket --bucket "$BUCKET_MAIN"

# Remover arquivos locais
rm -f /tmp/config.txt /tmp/config_v1.txt /tmp/documento_privado.txt \
      /tmp/download_presigned.txt /tmp/novo_upload.txt /tmp/s3_event_handler.zip
```

---

## O Que Você Aprendeu

- **Versionamento:** uma vez habilitado, nunca "desabilitado" — só "suspenso"; toda versão armazenada é cobrada
- **Delete marker:** deletar objeto versionado cria Delete Marker (objeto não some de fato)
- **Lifecycle transitions:** STANDARD → STANDARD_IA (mín 30 dias) → GLACIER (mín 90 dias)
- **Presigned URL:** permite acesso temporário sem credenciais; expira no tempo definido; auditorado via CloudTrail
- **S3 Events:** pattern push (S3 → Lambda/SNS/SQS); ideal para pipelines de processamento de arquivos
- **CRR:** replication cross-region requer versionamento nos dois buckets e IAM role com s3:ReplicateObject
