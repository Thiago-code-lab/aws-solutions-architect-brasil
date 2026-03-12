# Links e Recursos — IAM e Segurança (Módulo 09)

## Documentação Oficial AWS

- [AWS IAM — Documentação Oficial](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM — Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [IAM — Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)
- [IAM — Service Control Policies (SCP)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [IAM — ABAC com Tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_attribute-based-access-control.html)
- [AWS STS — AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html)
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
- [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
- [Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)
- [AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)
- [AWS Shield](https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html)

## FAQs

- [AWS IAM FAQ](https://aws.amazon.com/iam/faqs/)
- [Amazon GuardDuty FAQ](https://aws.amazon.com/guardduty/faqs/)

## Whitepapers

- [AWS Security Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-security-best-practices/aws-security-best-practices.html)
- [Security Pillar — Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

## Artigos de Blog AWS

- [IAM least privilege: patterns and practices](https://aws.amazon.com/blogs/security/techniques-for-writing-least-privilege-iam-policies/)
- [ABAC with AWS IAM and resource tags](https://aws.amazon.com/blogs/security/attribute-based-access-control-with-aws-services/)
- [How to use STS AssumeRole with ExternalId](https://aws.amazon.com/blogs/security/how-to-use-external-id-when-granting-access-to-your-aws-resources/)

## Ferramentas de Estudo

- [Tutorials Dojo — AWS IAM Cheat Sheet](https://tutorialsdojo.com/aws-identity-and-access-management-iam/)
- [AWS IAM Policy Simulator](https://policysim.aws.amazon.com/)

## Comparativo Serviços de Segurança

| Serviço | O que detecta / protege | Tipo |
|---------|------------------------|------|
| GuardDuty | Comportamentos suspeitos (accounts, workloads, data) | Detecção de ameaças |
| Macie | Dados sensíveis (PII) em S3 | Descoberta de dados |
| Inspector | Vulnerabilidades em EC2/ECR/Lambda | Avaliação de vulnerabilidades |
| WAF | Requests HTTP maliciosas (SQL inj, XSS) | Proteção de aplicação |
| Shield Standard | DDoS L3/L4 automático | Proteção DDoS |
| Shield Advanced | DDoS avançado + suporte 24/7 | Proteção DDoS premium |
| Config | Conformidade de configurações | Auditoria de configuração |
| Security Hub | Agregador de findings de segurança | Visão consolidada |
