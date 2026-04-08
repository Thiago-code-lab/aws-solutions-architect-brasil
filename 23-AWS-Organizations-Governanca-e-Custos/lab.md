# Lab - Governanca Basica e Controle de Custos

## Objetivo
Configurar uma estrutura inicial de governanca e monitoramento financeiro.

## Passos

1. Crie uma Organizacao (se ainda nao existir).
2. Crie OUs: Sandbox, Dev e Prod.
3. Mova contas para OUs corretas.
4. Crie SCP de restricao regional (exemplo: apenas us-east-1 e sa-east-1).
5. Ative Cost Allocation Tags (Environment, CostCenter, Owner).
6. Crie AWS Budget mensal por conta com alerta por email/SNS.
7. Abra Cost Explorer e valide dados por servico e tag.

## Validacao

- Contas nas OUs corretas
- SCP aplicada e validada
- Alertas de budget funcionando
- Custos visiveis por tag no explorer

## Limpeza

- remover budgets de teste
- desanexar SCP de laboratorio