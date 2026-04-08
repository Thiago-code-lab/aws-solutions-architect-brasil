# Lab — IAM e Segurança

> **Região:** us-east-1  
> **Custo estimado:** ~$1–3/mês (KMS CMK) + tempo de EC2 se usado  
> **Pré-requisito:** AWS CLI configurado, Terraform >= 1.5

## Objetivos do Lab
1. Criar uma Customer-Managed CMK no KMS com key policy customizada
2. Criar uma role cross-account com condição de MFA e PrincipalOrgID
3. Armazenar e rotacionar uma credencial no Secrets Manager
4. Usar Parameter Store com SecureString encriptado com CMK própria

---

## Parte 1: CMK no KMS via AWS CLI

```bash
# Variáveis
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION="us-east-1"

# Criar CMK com descrição
KEY_ID=$(aws kms create-key \
  --description "CMK para lab IAM e Segurança" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS \
  --region $REGION \
  --query KeyMetadata.KeyId --output text)

echo "CMK criada: $KEY_ID"

# Criar alias amigável
aws kms create-alias \
  --alias-name alias/lab-iam-security \
  --target-key-id $KEY_ID \
  --region $REGION

# Habilitar rotação anual automática
aws kms enable-key-rotation \
  --key-id $KEY_ID \
  --region $REGION

# Testar encriptação
echo "dado sensível" | base64 | aws kms encrypt \
  --key-id alias/lab-iam-security \
  --plaintext fileb:///dev/stdin \
  --output text \
  --query CiphertextBlob \
  --region $REGION > /tmp/encrypted.b64

echo "Dado encriptado: $(cat /tmp/encrypted.b64)"

# Decriptar
aws kms decrypt \
  --ciphertext-blob fileb://<(base64 -d /tmp/encrypted.b64) \
  --output text \
  --query Plaintext \
  --region $REGION | base64 -d
```

---

## Parte 2: Terraform — CMK + Role Cross-Account + Secrets Manager

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

data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

# ──────────────────────────────────────────────────────
# 1. Customer-Managed KMS Key
# ──────────────────────────────────────────────────────

data "aws_iam_policy_document" "kms_key_policy" {
  # Root account: administração total (recover always)
  statement {
    sid    = "EnableRootAdmin"
    effect = "Allow"
    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"]
    }
    actions   = ["kms:*"]
    resources = ["*"]
  }

  # Serviços AWS (Secrets Manager, SSM) podem usar a chave
  statement {
    sid    = "AllowAWSServices"
    effect = "Allow"
    principals {
      type        = "Service"
      identifiers = ["secretsmanager.amazonaws.com", "ssm.amazonaws.com"]
    }
    actions = [
      "kms:GenerateDataKey*",
      "kms:Decrypt",
      "kms:Encrypt",
    ]
    resources = ["*"]
  }

  # App role: apenas encrypt/decrypt, sem administração de chave
  statement {
    sid    = "AllowAppRole"
    effect = "Allow"
    principals {
      type        = "AWS"
      identifiers = [aws_iam_role.app_role.arn]
    }
    actions = [
      "kms:Decrypt",
      "kms:GenerateDataKey",
      "kms:DescribeKey",
    ]
    resources = ["*"]
  }
}

resource "aws_kms_key" "lab_cmk" {
  description             = "CMK para lab IAM e Segurança SAA-C03"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  policy                  = data.aws_iam_policy_document.kms_key_policy.json
}

resource "aws_kms_alias" "lab_cmk_alias" {
  name          = "alias/saa-lab-iam-security"
  target_key_id = aws_kms_key.lab_cmk.key_id
}

# ──────────────────────────────────────────────────────
# 2. IAM Role para Aplicação (com permission boundary)
# ──────────────────────────────────────────────────────

resource "aws_iam_policy" "app_boundary" {
  name        = "AppPermissionBoundary"
  description = "Limite máximo de permissões para roles de aplicação"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["s3:Get*", "s3:Put*", "s3:List*"]
        Resource = "*"
      },
      {
        Effect   = "Allow"
        Action   = ["secretsmanager:GetSecretValue"]
        Resource = "arn:aws:secretsmanager:us-east-1:${data.aws_caller_identity.current.account_id}:secret:/app/*"
      },
      {
        Effect   = "Allow"
        Action   = ["kms:Decrypt", "kms:GenerateDataKey"]
        Resource = aws_kms_key.lab_cmk.arn
      }
    ]
  })
}

resource "aws_iam_role" "app_role" {
  name                 = "saa-lab-app-role"
  permissions_boundary = aws_iam_policy.app_boundary.arn

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
      Action    = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_instance_profile" "app_profile" {
  name = "saa-lab-app-profile"
  role = aws_iam_role.app_role.name
}

# ──────────────────────────────────────────────────────
# 3. Secrets Manager — Secret encriptado com CMK
# ──────────────────────────────────────────────────────

resource "aws_secretsmanager_secret" "db_secret" {
  name                    = "/app/prod/db-credentials"
  description             = "Credenciais do banco de dados de produção"
  kms_key_id              = aws_kms_key.lab_cmk.arn
  recovery_window_in_days = 7

  tags = {
    Environment = "lab"
    Module      = "02-IAM-e-Seguranca"
  }
}

resource "aws_secretsmanager_secret_version" "db_secret_v1" {
  secret_id = aws_secretsmanager_secret.db_secret.id
  secret_string = jsonencode({
    username = "app_user"
    password = "SenhaTemporaria123!"
    host     = "rds-endpoint.us-east-1.rds.amazonaws.com"
    port     = 5432
    dbname   = "appdb"
  })
}

# ──────────────────────────────────────────────────────
# 4. Parameter Store — SecureString com CMK própria
# ──────────────────────────────────────────────────────

resource "aws_ssm_parameter" "app_config" {
  name   = "/app/prod/api-key"
  type   = "SecureString"
  value  = "minha-api-key-temporaria-para-lab"
  key_id = aws_kms_key.lab_cmk.arn

  tags = {
    Environment = "lab"
    Module      = "02-IAM-e-Seguranca"
  }
}

# ──────────────────────────────────────────────────────
# Outputs
# ──────────────────────────────────────────────────────

output "cmk_key_id" {
  value       = aws_kms_key.lab_cmk.key_id
  description = "ID da CMK criada"
}

output "cmk_arn" {
  value       = aws_kms_key.lab_cmk.arn
  description = "ARN da CMK"
}

output "app_role_arn" {
  value       = aws_iam_role.app_role.arn
  description = "ARN da role da aplicação"
}

output "secret_arn" {
  value       = aws_secretsmanager_secret.db_secret.arn
  description = "ARN do secret no Secrets Manager"
}
```

---

## Parte 3: Teste e Validação

```bash
# Verificar key rotation status
aws kms get-key-rotation-status \
  --key-id alias/saa-lab-iam-security \
  --region us-east-1

# Listar versões do secret
aws secretsmanager describe-secret \
  --secret-id /app/prod/db-credentials \
  --region us-east-1

# Obter o secret (simula o que a aplicação faz)
aws secretsmanager get-secret-value \
  --secret-id /app/prod/db-credentials \
  --region us-east-1 \
  --query SecretString --output text | python3 -m json.tool

# Obter parâmetro do SSM
aws ssm get-parameter \
  --name /app/prod/api-key \
  --with-decryption \
  --region us-east-1 \
  --query Parameter.Value --output text

# Ver quem usou a CMK (últimas 24h via CloudTrail)
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=Decrypt \
  --start-time $(date -u -v-1d +%Y-%m-%dT%H:%M:%SZ) \
  --region us-east-1 \
  --query 'Events[].{User:Username,Time:EventTime,Source:EventSource}' \
  --output table
```

---

## Cleanup (importantes para evitar custos)

```bash
# Terraform destroy
terraform destroy -auto-approve

# Ou manualmente via CLI:

# Agenda deleção da CMK (mínimo 7 dias, não é imediato)
aws kms schedule-key-deletion \
  --key-id $(aws kms describe-key --key-id alias/saa-lab-iam-security \
    --query KeyMetadata.KeyId --output text) \
  --pending-window-in-days 7 \
  --region us-east-1

# Deletar secret (recovery_window_in_days=0 para deleção imediata)
aws secretsmanager delete-secret \
  --secret-id /app/prod/db-credentials \
  --force-delete-without-recovery \
  --region us-east-1

# Deletar parâmetro SSM
aws ssm delete-parameter \
  --name /app/prod/api-key \
  --region us-east-1

echo "Cleanup concluído. CMK será deletada em 7 dias."
```

---

## Pontos de Revisão

- [ ] Por que a key policy do KMS inclui o root account como administrador?
- [ ] O que acontece se você deletar a key policy e o root account não estiver listado?
- [ ] Qual é a diferença entre `kms:Encrypt` e `kms:GenerateDataKey*`?
- [ ] Por que o Permission Boundary não é suficiente sozinho? O que mais é necessário?
- [ ] Em qual cenário `aws:PrincipalOrgID` é mais eficiente do que listar account IDs?
