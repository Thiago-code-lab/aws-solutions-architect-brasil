# Links de Referência — IAM e Segurança

## Documentação Oficial AWS

### IAM
- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/) — Guia completo do Identity and Access Management
- [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) — Gramática completa de políticas JSON
- [IAM Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) — Como o IAM avalia permissões
- [Permission Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) — Guia de Permission Boundaries
- [IAM Condition Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) — Lista completa de chaves de condição globais

### IAM Identity Center
- [IAM Identity Center User Guide](https://docs.aws.amazon.com/singlesignon/latest/userguide/) — AWS SSO / Identity Center
- [Permission Sets Reference](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html) — Conceito de Permission Sets

### AWS Organizations + SCPs
- [AWS Organizations User Guide](https://docs.aws.amazon.com/organizations/latest/userguide/) — Organizations e SCPs
- [Service Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) — SCPs em detalhe
- [SCP Examples](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html) — Exemplos práticos de SCPs

### KMS
- [KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/) — Guia completo do KMS
- [KMS Key Policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) — Como funcionam key policies
- [Envelope Encryption](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#enveloping) — Conceito de envelope encryption
- [KMS Grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) — Grants vs Key Policies
- [Multi-Region Keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html) — Chaves multi-região

### Secrets Manager
- [Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/) — Guia completo
- [Rotating Secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html) — Como funciona a rotação
- [Secrets Manager vs Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/integration-ps-secretsmanager.html) — Comparativo oficial

### Parameter Store
- [Parameter Store User Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [Working with SecureString](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-securestring.html)

### ACM
- [ACM User Guide](https://docs.aws.amazon.com/acm/latest/userguide/) — Certificate Manager
- [ACM Private CA](https://docs.aws.amazon.com/privateca/latest/userguide/) — CA privada gerenciada

---

## Whitepapers e Best Practices

- [Security Pillar — AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/) — Pilar de segurança do WAFR
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) — Boas práticas oficiais da AWS
- [Security Best Practices in IAM](https://aws.amazon.com/architecture/security-identity-compliance/) — Arquitetura de segurança
- [Organizing Your AWS Environment Using Multiple Accounts](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/) — Whitepaper sobre multi-account

---

## AWS re:Invent e Talks Técnicos

- [IAM Policy Evaluation Deep Dive (re:Invent)](https://www.youtube.com/results?search_query=aws+reinvent+iam+policy+evaluation) — Buscar no YouTube: "AWS re:Invent IAM policy evaluation"
- [Delegating Access to KMS Keys](https://aws.amazon.com/blogs/security/how-to-restrict-amazon-sns-topic-access-to-only-allow-specific-aws-services/) — Blog AWS Security

---

## FAQs Importantes para o Exame

- [IAM FAQ](https://aws.amazon.com/iam/faqs/)
- [KMS FAQ](https://aws.amazon.com/kms/faqs/)
- [Secrets Manager FAQ](https://aws.amazon.com/secrets-manager/faqs/)

---

## Ferramentas Úteis

- [AWS Policy Simulator](https://policysim.aws.amazon.com/) — Testar políticas IAM sem efeito real
- [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html) — Gerar políticas visualmente
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html) — Detectar acesso externo não intencional

---

## Conceitos SAA-C03 Chave Neste Módulo

| Conceito | Frequência no Exame |
|---|---|
| Policy evaluation order (explicit deny priority) | Alta |
| Permission Boundary para delegar criação de roles | Alta |
| SCPs não afetam management account | Alta |
| KMS envelope encryption | Alta |
| Secrets Manager vs Parameter Store | Média |
| ACM + CloudFront requer us-east-1 | Alta |
| aws:PrincipalOrgID para bucket policies | Média |
| KMS Multi-Region Keys | Média |
