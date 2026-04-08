# Cheatsheet - AWS Organizations, Governanca e Custos

## Mapa rapido

| Necessidade | Servico/Padrao | Dica de prova |
|---|---|---|
| Guardrail de permissao em varias contas | SCP | Lembrar: nao concede, apenas restringe |
| Provisionar landing zone padrao | Control Tower | Menor overhead para multi-account |
| Acesso centralizado de usuarios | IAM Identity Center | Evita IAM users em cada conta |
| Alertar estouro de orcamento | AWS Budgets | Alerta antes da fatura fechar |
| Analise detalhada de gastos | CUR + Athena | Visao granular por servico/tag |
| Reduzir custo de compute previsivel | Savings Plans / RI | Compromisso de uso reduz preco |

## Otimizacao de custo

- rightsizing de EC2 e RDS
- desligamento de ambientes nao produtivos
- classes de storage adequadas (S3 lifecycle)
- usar Spot em workloads stateless
- revisar transferencias entre AZ/regioes

## Frases gatilho

- "across multiple AWS accounts" => Organizations/Control Tower
- "prevent specific actions" => SCP
- "cost visibility by team" => Cost Allocation Tags + CUR
- "predictable usage" => RI/Savings Plans
- "variable workload" => Savings Plans ou on-demand + autoscaling