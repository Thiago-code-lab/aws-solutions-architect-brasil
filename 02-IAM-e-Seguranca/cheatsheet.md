# Cheatsheet — IAM e Segurança

## Tipos de Política IAM

| Tipo | Quem gerencia | Reutilizável | Coexiste com identity | Vínculo |
|---|---|---|---|---|
| AWS Managed Policy | AWS | Sim | Sim | 1:N |
| Customer Managed Policy | Você | Sim | Sim | 1:N |
| Inline Policy | Você | Não | Sim | 1:1 (deletada com identity) |
| Resource-Based Policy | Você | N/A | N/A | No recurso |
| Permission Boundary | Você | Sim | Limita identity | Max permissions |
| SCP | Org admin | Sim | Limita conta | Guardrail por OU/conta |

---

## Lógica de Avaliação de Permissões

```
1. Explicit DENY em qualquer política?          → DENY (fim)
2. SCP da Organization permite a ação?          → não? DENY
3. Permission Boundary permite a ação?          → não? DENY
4. Identity Policy permite?                     → sim? ALLOW (same account)
   └── Resource Policy permite?                 → sim? ALLOW (cross-account sem role assumption)
5. Nenhuma das opções permitiu?                 → DENY implícito
```

> **Cross-account sem Resource Policy:** AMBAS identity policy E resource policy devem permitir.  
> **Cross-account COM Resource Policy (ex: S3, KMS):** Resource policy pode permitir por si só (mas boundary e SCP ainda se aplicam).

---

## Condições IAM Mais Cobradas

| Condição | Exemplo de uso |
|---|---|
| `aws:RequestedRegion` | Restringir ações a `us-east-1` e `sa-east-1` |
| `aws:MultiFactorAuthPresent: true` | Exigir MFA para ações sensíveis |
| `aws:PrincipalOrgID` | Limitar acesso ao S3 bucket a toda a Org |
| `aws:SourceVpc` / `aws:SourceVpce` | Restringir acesso apenas vindo da VPC/endpoint |
| `aws:CurrentTime` | Janela de tempo para acesso |
| `s3:prefix` | Restringir ListBucket a determinado prefixo |
| `iam:PassedToService` | Controlar a quais serviços uma role pode ser delegada |

---

## KMS — Tipos de CMK

| Tipo | Gerenciada por | Custos | Rotação | Editável | Visível |
|---|---|---|---|---|---|
| AWS-owned | AWS (interna) | Sem custo | Automática pela AWS | Não | Não |
| AWS-managed | AWS (sua conta) | Sem custo por CMK | Anual automática | Não | Sim |
| Customer-managed | Você | $1/mês + $0.03/10K API | Opcional anual | Sim | Sim |

**Key Material Origin:** `AWS_KMS` (padrão) | `EXTERNAL` (import) | `AWS_CLOUDHSM`

---

## KMS — Envelope Encryption (Sequência)

```
Encriptar:
  1. GenerateDataKey(CMK ARN)
     → recebe: Plaintext data key + Encrypted data key
  2. Usar Plaintext key para encriptar dados (AES-256-GCM)
  3. Apaga Plaintext key da memória
  4. Armazena: Encrypted data + Encrypted data key

Decriptar:
  1. Decrypt(Encrypted data key, CMK ARN)
     → recebe: Plaintext data key
  2. Usa Plaintext key para decriptar dados
  3. Apaga Plaintext key da memória
```

---

## Secrets Manager vs Parameter Store

| Característica | Secrets Manager | Parameter Store |
|---|---|---|
| Rotação automática | ✅ Nativa (Lambda) | ❌ (trigger externo) |
| Integração RDS/Redshift/DocDB | ✅ Nativa | ❌ |
| Custo | $0,40/secret/mês + API | Standard: grátis; Advanced: $0,05/param/mês |
| Tamanho máximo | 64 KB | 4 KB (standard), 8 KB (advanced) |
| Versionamento | ✅ (staging labels) | ✅ |
| Cross-account | ✅ (resource policy) | Com customer CMK |
| Hierarchical paths | Limitado | ✅ (`/app/prod/db`) |
| Melhor para | DB credentials com rotação | Configurações de app, parâmetros |

---

## ACM — Resumo

| Característica | Public ACM | ACM Private CA |
|---|---|---|
| Custo | Grátis | ~$400/mês por CA + $0,75/cert |
| Exportável | ❌ Não | ✅ Sim |
| Renovação | Automática | Automática |
| Validação | DNS (preferido) ou Email | Sem validação externa |
| Uso | ALB, CloudFront, API GW | Microserviços internos, mTLS |
| Região CloudFront | ⚠️ Certificado deve estar em us-east-1 | — |

---

## Padrões de Acesso Cross-Account

| Cenário | Mecanismo |
|---|---|
| Acesso a S3 de outra conta | Bucket policy + (role ou user com identity policy) |
| Acesso a CMK KMS de outra conta | Key policy no CMK + identity policy no caller |
| Assumir role em outra conta | Trust policy na role + iam:AssumeRole no caller |
| Compartilhar RDS snapshot | Share snapshot com account ID destino |
| Acesso via PrivateLink | VPC Endpoint + NLB + VPC Endpoint Service policy |

---

## IAM Identity Center — Conceitos

```
AWS Organizations
  └── IAM Identity Center (SSO)
        ├── Identity Source: Built-in / AD / External IdP (SAML/SCIM)
        ├── Users & Groups
        ├── Permission Sets (= Collection of IAM policies)
        └── Account Assignments
              ├── Account A → User X → Permission Set Admin
              ├── Account B → Group Y → Permission Set ReadOnly
              └── Application assignments (SaaS via SAML)
```

---

## Dicas de Prova (IAM)

- **Explicit DENY sempre vence**, independente de qualquer Allow
- Permission Boundary ≠ SCP: boundary limita a *identidade*, SCP limita a *conta*
- Management Account **nunca** é afetada por SCPs
- Para cross-account S3/KMS sem role assumption → resource-based policy com o account ID destino
- Para exigir MFA: `Bool: aws:MultiFactorAuthPresent: true` (não `StringEquals`)
- CloudFront + ACM → certificado **deve** estar em `us-east-1`
- KMS Multi-Region Key: mesmo material mas ARNs diferentes por região
- Secrets Manager cobra por secret; Parameter Store Standard é gratuito
