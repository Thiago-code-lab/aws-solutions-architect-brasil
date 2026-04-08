# Casos de Uso - AWS Organizations, Governanca e Custos

## Caso 1: Empresa em crescimento rapido

Cenario:
- 40 contas AWS em 6 meses
- times independentes por produto
- necessidade de guardrails comuns

Arquitetura:
- AWS Control Tower para landing zone
- OUs por ambiente (Sandbox, Dev, Prod)
- SCP bloqueando desativacao de CloudTrail e uso fora de regioes permitidas
- Identity Center para acesso federado

## Caso 2: FinOps com chargeback

Cenario:
- diretoria quer custo por produto e por squad

Arquitetura:
- Tags obrigatorias (Owner, CostCenter, Product, Environment)
- CUR no S3
- Athena + QuickSight para dashboards de chargeback
- Budgets por conta e por tag

## Caso 3: Reducao de custo compute

Cenario:
- gasto elevado em EC2 e Fargate

Arquitetura:
- Savings Plans para base estavel
- Spot para batch e workers stateless
- autoscaling com limites
- rightsizing com Compute Optimizer

## Caso 4: Governanca de seguranca

Cenario:
- auditoria exige trilhas imutaveis

Arquitetura:
- conta de log central
- CloudTrail org trail
- bucket de log com Object Lock
- bloqueios SCP para impedir alteracao de trilha