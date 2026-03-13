# Casos de Uso — IAM e Segurança

## Caso 1: Acesso Cross-Account Seguro com MFA para Operações Críticas

**Contexto:**  
Uma empresa possui uma conta de produção AWS (`conta-prod`) e uma conta de ferramentas de DevOps (`conta-devops`). Engenheiros trabalham na `conta-devops` e precisam realizar deploys na `conta-prod`. Operações de deletar recursos (ECR images, snapshots) devem exigir MFA.

**Requisitos:**
- Engenheiros na conta-devops assumem uma role na conta-prod
- Operações destrutivas exigem MFA
- Apenas a OU "Engineering" da organização pode assumir a role

**Arquitetura:**

```
conta-devops (Engenheiros)              conta-prod
┌─────────────────────┐                ┌──────────────────────────────┐
│ IAM User: eng-user  │                │ IAM Role: DevOpsDeployRole    │
│  └── Policy:        │  AssumeRole    │  ├── Trust Policy:            │
│       sts:AssumeRole│───────────────▶│  │  Allow: conta-devops       │
│       (com MFA)     │                │  │  Condition: MFA + OrgID    │
└─────────────────────┘                │  └── Permission Policy:       │
                                       │       - ecs:*                 │
                                       │       - ecr:*                 │
                                       │       - ec2:DescribeInstances │
                                       │       - Deny: DeleteX sem MFA │
                                       └──────────────────────────────┘
```

**Trust Policy da Role (conta-prod):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
    "Action": "sts:AssumeRole",
    "Condition": {
      "Bool": {"aws:MultiFactorAuthPresent": "true"},
      "StringEquals": {"aws:PrincipalOrgID": "o-xxxxxxxxxx"}
    }
  }]
}
```

**Permission Policy com Deny destrutivo sem MFA:**
```json
{
  "Statement": [{
    "Effect": "Deny",
    "Action": ["ecr:DeleteRepository", "ec2:DeleteSnapshot", "rds:DeleteDBInstance"],
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
    }
  }]
}
```

**Conceitos cobrados:** cross-account role assumption, Trust Policy, MFA condition, PrincipalOrgID, explicit deny

---

## Caso 2: KMS com Envelope Encryption para Dados Sensíveis no S3

**Contexto:**  
Uma fintech precisa encriptar arquivos de clientes no S3 com uma CMK gerenciada pelo cliente, garantir que apenas a aplicação específica possa decriptar (não todos os devs), e ter logs de auditoria de quem acessou a chave.

**Arquitetura:**

```
                    ┌──────────────────────────────────┐
EC2 (app-role)      │  KMS Customer-Managed CMK         │
  │                 │  ╔══════════════════════════════╗ │
  ├─GenerateDataKey▶│  ║ Key Policy:                  ║ │
  │◀── data key ────│  ║ - Root account: admin        ║ │
  │                 │  ║ - key-admin-role: manage      ║ │
  │  Encrypt data   │  ║ - app-role: kms:GenerateKey  ║ │
  │  with plaintext │  ║              kms:Decrypt      ║ │
  │  data key       │  ║ - DENY: kms:Decrypt           ║ │
  │                 │  ║   if NOT app-role             ║ │
  ├─Store to S3────▶│  ╚══════════════════════════════╝ │
  │  {              └──────────────────────────────────┘
  │   encrypted_data,                 │
  │   encrypted_data_key              │ CloudTrail
  │  }                                ▼
  └─────────────────────────  Logs de auditoria:
                              - Who called kms:Decrypt
                              - From which EC2 (via ARN)
                              - IP de origem
```

**Decisão de design:** Por que CMK e não SSE-S3?
- SSE-S3: AWS gerencia a chave → sem controle de quem decripta
- CMK: key policy define exatamente quais roles podem usar → princípio do menor privilégio
- Rotação anual automática pode ser habilitada
- CloudTrail registra cada uso da CMK (GenerateDataKey, Decrypt)

**Conceitos cobrados:** Customer-managed CMK, key policy, envelope encryption, auditoria via CloudTrail

---

## Caso 3: Secrets Manager para Rotação Automática de Credenciais do RDS

**Contexto:**  
Uma aplicação Node.js na EC2 acessa um banco RDS PostgreSQL. A política de segurança exige rotação de senhas a cada 30 dias sem downtime da aplicação.

**Arquitetura:**

```
EC2 (app-role)                                  Secrets Manager
   │                                           ┌──────────────────────┐
   │ GetSecretValue(SecretId)                  │  /app/prod/rds-master │
   ├─────────────────────────────────────────▶ │  {                   │
   │◀── {username, password, host, port}────── │   "username": "app", │
   │                                           │   "password": "xxxx" │
   │                                           │   "host": "db.xxx",  │
   │                                           │   "port": 5432       │
   │                                           │  }                   │
    (Aplicação NUNCA armazena senha localmente) └────────┬─────────────┘
                                                         │ Rotation (30d)
                                               ┌─────────▼─────────────┐
                                               │ Lambda (rotation fn)  │
                                               │  1. createSecret       │
                                               │  2. setSecret (new pw) │
                                               │  3. testSecret (verify)│
                                               │  4. finishSecret       │
                                               └───────────────────────┘
                                                         │
                                               ┌─────────▼─────────────┐
                                               │  RDS PostgreSQL        │
                                               │  ALTER USER app        │
                                               │  PASSWORD 'novo_pw'    │
                                               └───────────────────────┘
```

**IAM para a instância EC2:**
```json
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:us-east-1:123456789:secret:/app/prod/rds-master*",
  "Condition": {
    "StringEquals": {"aws:RequestedRegion": "us-east-1"}
  }
}
```

**Conceitos cobrados:** Secrets Manager, rotação automática, instância profile, GetSecretValue, ResourceARN com wildcard

---

## Caso 4: Permission Boundary para Delegar Criação de Roles

**Contexto:**  
Uma empresa quer permitir que o time de DevOps crie roles IAM para aplicações, mas sem que eles possam criar uma role com mais permissões do que eles mesmos têm (privilege escalation).

**Solução:**

```
Admin (SysOps)
  └── Cria Permission Boundary Policy (max-app-boundary)
        Permite apenas: ec2:Describe*, s3:Get*, s3:Put*, sqs:*, cloudwatch:*
        Proíbe: iam:*, kms:*, ec2:RunInstances

  └── Cria Policy para DevOps com condição:
        Allow: iam:CreateRole, iam:AttachRolePolicy
        Condition: iam:PermissionsBoundary = arn:...max-app-boundary

DevOps Team
  └── Pode criar roles APENAS SE:
        - Attach ao boundary max-app-boundary
        - A role final fica limitada ao boundary
        - Mesmo que o DevOps tente dar AdministratorAccess,
          o boundary limita o resultado efetivo
```

**Policy do time DevOps:**
```json
{
  "Effect": "Allow",
  "Action": ["iam:CreateRole", "iam:AttachRolePolicy", "iam:PutRolePolicy"],
  "Resource": "arn:aws:iam::*:role/app-*",
  "Condition": {
    "StringEquals": {
      "iam:PermissionsBoundary": "arn:aws:iam::123456789:policy/max-app-boundary"
    }
  }
}
```

**Conceitos cobrados:** Permission Boundary, privilege escalation prevention, delegated administration

---

## Caso 5: Acesso Restrito ao S3 Apenas via VPC Endpoint

**Contexto:**  
Uma empresa quer garantir que dados confidenciais no S3 só sejam acessados de dentro da VPC corporativa, bloqueando qualquer acesso externo (incluindo acesso direto com credenciais válidas pela internet).

**Bucket Policy:**
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": [
    "arn:aws:s3:::empresa-confidencial",
    "arn:aws:s3:::empresa-confidencial/*"
  ],
  "Condition": {
    "StringNotEquals": {
      "aws:SourceVpce": "vpce-0123456789abcdef"
    }
  }
}
```

```
Internet                   VPC Privada
   │                          │
   │ ❌ DENY (no SourceVpce)  │ ✅ ALLOW (SourceVpce match)
   │                          │
   ×                         EC2       S3 Gateway Endpoint
            (Acesso bloqueado)│         │
                              │◀────────┘
                         VPC Routing → S3 sem internet gateway
```

**Conceitos cobrados:** resource-based policy, aws:SourceVpce, VPC Gateway Endpoint para S3, zero-trust
