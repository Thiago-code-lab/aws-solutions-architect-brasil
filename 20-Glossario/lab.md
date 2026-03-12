# Lab Prático — Glossário AWS: IAM Avançado e Investigação de Policies (Módulo 20)

> **Região:** us-east-1 | **Custo estimado:** ~$0 (IAM é gratuito)  
> **Pré-requisitos:** AWS CLI configurado, permissão de IAM na conta

---

## Objetivo

Explorar os tipos de policies IAM na prática, simular avaliações de permissão, criar roles cross-account, e entender o modelo de avaliação de políticas AWS.

---

## Parte 1 — Tipos de Policies IAM

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# 1. Identity-based Policy (inline em usuário)
aws iam create-user --user-name "lab-glossario-dev"

# Attach managed policy
aws iam attach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"

# Criar customer managed policy
POLICY_ARN=$(aws iam create-policy \
  --policy-name "lab-s3-bucket-especifico" \
  --policy-document "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Sid\": \"LeituraEscritaBucketLab\",
      \"Effect\": \"Allow\",
      \"Action\": [\"s3:GetObject\", \"s3:PutObject\", \"s3:ListBucket\"],
      \"Resource\": [
        \"arn:aws:s3:::lab-glossario-${ACCOUNT_ID}\",
        \"arn:aws:s3:::lab-glossario-${ACCOUNT_ID}/*\"
      ]
    }]
  }" \
  --query 'Policy.Arn' --output text)

echo "Policy ARN: $POLICY_ARN"

# Attach ao usuário
aws iam attach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "$POLICY_ARN"

# Listar políticas do usuário
echo "Managed policies do usuário:"
aws iam list-attached-user-policies \
  --user-name "lab-glossario-dev" \
  --query 'AttachedPolicies[*].[PolicyName, PolicyArn]' \
  --output table
```

---

## Parte 2 — Permissions Boundary

```bash
# Criar Permissions Boundary — define o MÁXIMO que a role pode ter
BOUNDARY_ARN=$(aws iam create-policy \
  --policy-name "lab-dev-boundary" \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "PermitirS3eDynamoDB",
        "Effect": "Allow",
        "Action": ["s3:*", "dynamodb:*", "cloudwatch:*", "logs:*"],
        "Resource": "*"
      },
      {
        "Sid": "DenyIAM",
        "Effect": "Deny",
        "Action": "iam:*",
        "Resource": "*"
      },
      {
        "Sid": "DenyEC2Caros",
        "Effect": "Deny",
        "Action": ["ec2:RunInstances"],
        "Resource": "arn:aws:ec2:*:*:instance/*",
        "Condition": {
          "StringNotLike": {
            "ec2:InstanceType": ["t3.*", "t4g.*"]
          }
        }
      }
    ]
  }' \
  --query 'Policy.Arn' --output text)

# Criar role COM permissions boundary
ROLE_ARN=$(aws iam create-role \
  --role-name "lab-dev-role-com-boundary" \
  --assume-role-policy-document "{
    \"Version\":\"2012-10-17\",
    \"Statement\":[{
      \"Action\":\"sts:AssumeRole\",
      \"Effect\":\"Allow\",
      \"Principal\":{\"AWS\":\"arn:aws:iam::${ACCOUNT_ID}:root\"}
    }]
  }" \
  --permissions-boundary "$BOUNDARY_ARN" \
  --query 'Role.Arn' --output text)

# Dar ao usuário permissão de AdministratorAccess (mas bucket boundary limita)
aws iam attach-role-policy \
  --role-name "lab-dev-role-com-boundary" \
  --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"

echo "Role com boundary criada: $ROLE_ARN"
echo ""
echo "Mesmo com AdministratorAccess, a role NÃO pode:"
echo "  - Executar ações IAM"
echo "  - Lançar EC2 de tipos diferentes de t3.* e t4g.*"
```

---

## Parte 3 — simulate-principal-policy

```bash
# Simular se usuário pode fazer PutObject em S3
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:user/lab-glossario-dev" \
  --action-names "s3:PutObject" \
  --resource-arns "arn:aws:s3:::lab-glossario-${ACCOUNT_ID}/teste.txt" \
  --query 'EvaluationResults[0].[EvalActionName, EvalDecision]' \
  --output table

# Simular se pode criar EC2
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:user/lab-glossario-dev" \
  --action-names "ec2:RunInstances" \
  --resource-arns "arn:aws:ec2:us-east-1::instance/*" \
  --query 'EvaluationResults[0].[EvalActionName, EvalDecision]' \
  --output table

# Simular ações múltiplas de uma vez
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:user/lab-glossario-dev" \
  --action-names \
    "s3:GetObject" \
    "s3:DeleteObject" \
    "dynamodb:PutItem" \
    "iam:CreateRole" \
    "lambda:InvokeFunction" \
  --resource-arns "arn:aws:s3:::lab-glossario-${ACCOUNT_ID}/*" \
  --query 'EvaluationResults[*].[EvalActionName, EvalDecision]' \
  --output table
```

---

## Parte 4 — Role Cross-Account

```bash
# Role para ser assumida por OUTRA conta (ou pela própria para demo)
CROSS_ROLE_ARN=$(aws iam create-role \
  --role-name "lab-cross-account-readonly" \
  --assume-role-policy-document "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Effect\": \"Allow\",
      \"Principal\": {\"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\"},
      \"Action\": \"sts:AssumeRole\",
      \"Condition\": {
        \"StringEquals\": {\"sts:ExternalId\": \"segredo-compartilhado-123\"}
      }
    }]
  }" \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-cross-account-readonly" \
  --policy-arn "arn:aws:iam::aws:policy/ReadOnlyAccess"

echo "Role Cross-Account: $CROSS_ROLE_ARN"

# Assumir a role (AssumeRole com ExternalId)
CREDS=$(aws sts assume-role \
  --role-arn "$CROSS_ROLE_ARN" \
  --role-session-name "lab-cross-session" \
  --external-id "segredo-compartilhado-123" \
  --duration-seconds 3600 \
  --query 'Credentials' --output json)

echo "Credenciais temporárias recebidas:"
echo "$CREDS" | python3 -c "
import json, sys
c = json.load(sys.stdin)
print(f'  AccessKeyId: {c[\"AccessKeyId\"][:10]}...')
print(f'  Expiration: {c[\"Expiration\"]}')
"

# Usar credenciais assumidas
AWS_ACCESS_KEY_ID=$(echo "$CREDS" | python3 -c "import json,sys; print(json.load(sys.stdin)['AccessKeyId'])")
AWS_SECRET_ACCESS_KEY=$(echo "$CREDS" | python3 -c "import json,sys; print(json.load(sys.stdin)['SecretAccessKey'])")
AWS_SESSION_TOKEN=$(echo "$CREDS" | python3 -c "import json,sys; print(json.load(sys.stdin)['SessionToken'])")

# Verificar identidade com as credenciais assumidas
AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY_ID" \
AWS_SECRET_ACCESS_KEY="$AWS_SECRET_ACCESS_KEY" \
AWS_SESSION_TOKEN="$AWS_SESSION_TOKEN" \
aws sts get-caller-identity
```

---

## Parte 5 — Revisar Políticas AWS Managed

```bash
# Listar policies AWS managed mais usadas
echo "=== AWS Managed Policies Comuns ==="
for POLICY in \
  "AmazonS3FullAccess" \
  "AmazonEC2ReadOnlyAccess" \
  "ReadOnlyAccess" \
  "AdministratorAccess" \
  "AWSLambdaBasicExecutionRole" \
  "AmazonDynamoDBFullAccess"; do

  PARN="arn:aws:iam::aws:policy/${POLICY}"
  VERSION=$(aws iam get-policy \
    --policy-arn "$PARN" \
    --query 'Policy.DefaultVersionId' --output text 2>/dev/null)
  
  if [ -n "$VERSION" ]; then
    echo ""
    echo "--- $POLICY ---"
    aws iam get-policy-version \
      --policy-arn "$PARN" \
      --version-id "$VERSION" \
      --query 'PolicyVersion.Document.Statement[*].[Effect, Action[0:3], Resource[0]]' \
      --output table 2>/dev/null | head -10
  fi
done
```

---

## Limpeza

```bash
# Desanexar e deletar policies/roles/usuário
aws iam detach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"

aws iam detach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "$POLICY_ARN"

aws iam delete-policy --policy-arn "$POLICY_ARN"
aws iam delete-user --user-name "lab-glossario-dev"

aws iam detach-role-policy \
  --role-name "lab-dev-role-com-boundary" \
  --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"
aws iam delete-role --role-name "lab-dev-role-com-boundary"
aws iam delete-policy --policy-arn "$BOUNDARY_ARN"

aws iam detach-role-policy \
  --role-name "lab-cross-account-readonly" \
  --policy-arn "arn:aws:iam::aws:policy/ReadOnlyAccess"
aws iam delete-role --role-name "lab-cross-account-readonly"
```

---

## O Que Você Aprendeu

- **Identity-based policy:** attached a user/role/group — define o que a identidade PODE fazer
- **Resource-based policy:** attached ao recurso (S3, SQS) — define quem PODE acessar o recurso
- **Permissions Boundary:** não concede acesso — somente reduz o teto; avaliação: `identidade AND boundary`
- **`simulate-principal-policy`:** ferramenta essencial para depurar permissões sem fazer mudanças reais
- **AssumeRole:** troca credenciais de longa duração por temporárias; ExternalId previne confused deputy
- **Regra de avaliação:** Deny explícito > tudo. Sem Allow explícito = Deny implícito = acesso negado
