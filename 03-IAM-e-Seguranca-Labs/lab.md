# Lab Prático — IAM Avançado: Roles, SCPs e STS (Módulo 09)

> **Região:** us-east-1 | **Custo estimado:** ~$0 (IAM e STS são gratuitos)  
> **Pré-requisitos:** AWS CLI configurado

---

## Objetivo

Praticar IAM avançado: criar roles com trust policies, SCPs em Organizations, AssumeRole cross-account, e usar Policy Simulator para validar permissões.

---

## Parte 1 — Roles IAM: Trust Policy e Permission Policy

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# 1. Role para EC2 acessar S3 e DynamoDB
EC2_ROLE_ARN=$(aws iam create-role \
  --role-name "lab-ec2-app-role" \
  --description "Role para aplicação EC2 — acesso S3 e DynamoDB" \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }' \
  --query 'Role.Arn' --output text)

# Criar customer managed policy com least privilege
APP_POLICY_ARN=$(aws iam create-policy \
  --policy-name "lab-app-policy" \
  --description "Acesso mínimo necessário para app de e-commerce" \
  --policy-document "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [
      {
        \"Sid\": \"S3LeituraEscrita\",
        \"Effect\": \"Allow\",
        \"Action\": [\"s3:GetObject\", \"s3:PutObject\", \"s3:DeleteObject\", \"s3:ListBucket\"],
        \"Resource\": [
          \"arn:aws:s3:::app-dados-${ACCOUNT_ID}\",
          \"arn:aws:s3:::app-dados-${ACCOUNT_ID}/*\"
        ]
      },
      {
        \"Sid\": \"DynamoDBCRUD\",
        \"Effect\": \"Allow\",
        \"Action\": [
          \"dynamodb:GetItem\", \"dynamodb:PutItem\", \"dynamodb:UpdateItem\",
          \"dynamodb:DeleteItem\", \"dynamodb:Query\", \"dynamodb:BatchWriteItem\"
        ],
        \"Resource\": \"arn:aws:dynamodb:us-east-1:${ACCOUNT_ID}:table/pedidos\"
      },
      {
        \"Sid\": \"LogsEscrita\",
        \"Effect\": \"Allow\",
        \"Action\": [\"logs:CreateLogGroup\", \"logs:CreateLogStream\", \"logs:PutLogEvents\"],
        \"Resource\": \"arn:aws:logs:us-east-1:${ACCOUNT_ID}:log-group:/app/*\"
      }
    ]
  }" \
  --query 'Policy.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-ec2-app-role" \
  --policy-arn "$APP_POLICY_ARN"

# Criar Instance Profile para EC2
aws iam create-instance-profile --instance-profile-name "lab-ec2-app-profile"
aws iam add-role-to-instance-profile \
  --instance-profile-name "lab-ec2-app-profile" \
  --role-name "lab-ec2-app-role"

echo "Instance Profile criado: lab-ec2-app-profile"

# 2. Role cross-account (para CI/CD em outra conta assumir)
CICD_ROLE_ARN=$(aws iam create-role \
  --role-name "lab-cicd-deploy-role" \
  --description "Role assumida pelo pipeline CI/CD para deploy" \
  --assume-role-policy-document "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Effect\": \"Allow\",
      \"Principal\": {
        \"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\"
      },
      \"Action\": \"sts:AssumeRole\",
      \"Condition\": {
        \"StringEquals\": {\"sts:ExternalId\": \"pipeline-secret-xyz\"},
        \"Bool\": {\"aws:MultiFactorAuthPresent\": \"false\"}
      }
    }]
  }" \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-cicd-deploy-role" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonECS_FullAccess"

echo "CICD Deploy Role: $CICD_ROLE_ARN"
```

---

## Parte 2 — STS AssumeRole

```bash
# Assumir a role CI/CD com ExternalId
CREDS=$(aws sts assume-role \
  --role-arn "$CICD_ROLE_ARN" \
  --role-session-name "pipeline-deploy-session" \
  --external-id "pipeline-secret-xyz" \
  --duration-seconds 900 \
  --query 'Credentials' --output json)

AK=$(echo "$CREDS" | python3 -c "import json,sys; c=json.load(sys.stdin); print(c['AccessKeyId'])")
SK=$(echo "$CREDS" | python3 -c "import json,sys; c=json.load(sys.stdin); print(c['SecretAccessKey'])")
ST=$(echo "$CREDS" | python3 -c "import json,sys; c=json.load(sys.stdin); print(c['SessionToken'])")
EXP=$(echo "$CREDS" | python3 -c "import json,sys; c=json.load(sys.stdin); print(c['Expiration'])")

echo "Credenciais temporárias (expire em: $EXP)"

# Verificar que a identidade mudou
AWS_ACCESS_KEY_ID="$AK" \
AWS_SECRET_ACCESS_KEY="$SK" \
AWS_SESSION_TOKEN="$ST" \
aws sts get-caller-identity

# STS GetCallerIdentity sem assumir role (quem sou eu agora)
aws sts get-caller-identity

# STS com session tags (para ABAC)
CREDS_TAGGED=$(aws sts assume-role \
  --role-arn "$CICD_ROLE_ARN" \
  --role-session-name "session-com-tags" \
  --external-id "pipeline-secret-xyz" \
  --tags '[
    {"Key": "Ambiente", "Value": "staging"},
    {"Key": "Equipe", "Value": "backend"},
    {"Key": "Projeto", "Value": "ecommerce"}
  ]' \
  --query 'Credentials.AccessKeyId' --output text)

echo "Session com tags: $CREDS_TAGGED"
```

---

## Parte 3 — Policy Simulator

```bash
# Simular permissões da role EC2
echo "=== Simulação de permissões da role EC2 ==="

aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:role/lab-ec2-app-role" \
  --action-names \
    "s3:GetObject" \
    "s3:PutObject" \
    "s3:DeleteBucket" \
    "dynamodb:PutItem" \
    "dynamodb:CreateTable" \
    "ec2:RunInstances" \
    "iam:CreateRole" \
    "logs:PutLogEvents" \
  --resource-arns "arn:aws:s3:::app-dados-${ACCOUNT_ID}/*" \
  --query 'EvaluationResults[*].[EvalActionName, EvalDecision]' \
  --output table

# Simular com condições de contexto
echo ""
echo "=== Simulação com contexto MFA ==="
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:role/lab-ec2-app-role" \
  --action-names "iam:DeleteRole" \
  --resource-arns "*" \
  --context-entries '[{
    "ContextKeyName": "aws:MultiFactorAuthPresent",
    "ContextKeyValues": ["true"],
    "ContextKeyType": "boolean"
  }]' \
  --query 'EvaluationResults[0].[EvalActionName, EvalDecision]' \
  --output table
```

---

## Parte 4 — IAM Conditions Comuns

```bash
# Policy com condições avançadas para estudo
cat << 'JSON'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyFora-Horario",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": ["203.0.113.0/24", "198.51.100.0/24"]
        }
      }
    },
    {
      "Sid": "RequireMFAParaIAM",
      "Effect": "Deny",
      "Action": ["iam:*", "sts:*"],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    },
    {
      "Sid": "ForcarHTTPS",
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": "*",
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
JSON

echo ""
echo "=== Conditions mais usadas no exame ==="
echo "aws:SourceIp          — origem por IP"
echo "aws:SourceVpc         — origem por VPC"
echo "aws:RequestedRegion   — região da requisição"
echo "aws:SecureTransport   — forçar HTTPS"
echo "aws:MultiFactorAuthPresent — exige MFA"
echo "sts:ExternalId        — parceiros cross-account (confused deputy)"
echo "aws:PrincipalTag/X    — ABAC: tag da identidade"
echo "aws:ResourceTag/X     — ABAC: tag do recurso"
echo "s3:prefix             — prefix de objetos S3"
echo "ec2:InstanceType      — limitar tipos de instância"
```

---

## Limpeza

```bash
# Remover instance profile
aws iam remove-role-from-instance-profile \
  --instance-profile-name "lab-ec2-app-profile" \
  --role-name "lab-ec2-app-role"
aws iam delete-instance-profile --instance-profile-name "lab-ec2-app-profile"

# Desanexar políticas e deletar roles
aws iam detach-role-policy \
  --role-name "lab-ec2-app-role" \
  --policy-arn "$APP_POLICY_ARN"
aws iam delete-policy --policy-arn "$APP_POLICY_ARN"
aws iam delete-role --role-name "lab-ec2-app-role"

aws iam detach-role-policy \
  --role-name "lab-cicd-deploy-role" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonECS_FullAccess"
aws iam delete-role --role-name "lab-cicd-deploy-role"
```

---

## O Que Você Aprendeu

- **Trust Policy:** quem PODE assumir a role (Principal); **Permission Policy:** o que a role PODE fazer
- **ExternalId:** proteção contra confused deputy attack em roles cross-account
- **Instance Profile:** wrapper obrigatório para anexar IAM Role a EC2
- **Session Tags:** passadas no AssumeRole; usadas em políticas ABAC com `aws:PrincipalTag`
- **STS duration:** máximo 12h para roles humanas; padrão 1h para roles de serviço
- **simulate-principal-policy:** único jeito seguro de testar permissões sem fazer chamadas reais
