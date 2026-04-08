# 23 AWS Organizations, Governanca e Custos

## Objetivos do modulo

- desenhar ambientes multi-account com seguranca e governanca central
- aplicar SCPs, tagging e controle de acesso por Organizational Unit
- dominar servicos de custo no contexto de arquitetura: Budgets, Cost Explorer, CUR, Savings Plans e RI
- responder cenarios do SAA-C03 que pedem reducao de custo com menor overhead operacional

## Conceitos fundamentais

No SAA-C03, governanca e custo aparecem junto com seguranca e operacao. A ideia central e estruturar contas para reduzir risco, melhorar visibilidade financeira e aplicar guardrails sem travar o time.

## Arquitetura multi-account recomendada

- Management account apenas para governanca e billing
- OUs separadas por ambiente e criticidade (Sandbox, Dev, Prod, Security)
- Contas dedicadas para logs e seguranca
- Acesso humano central por IAM Identity Center

## Servicos e praticas chave

- AWS Organizations: hierarquia de contas e consolidated billing
- SCP: define teto de permissoes por OU/conta
- AWS Control Tower: baseline multi-account com guardrails
- AWS Budgets: alertas proativos por limite de gasto ou uso
- Cost Explorer: analise de tendencia e recomendacoes
- Cost and Usage Report (CUR): base detalhada para chargeback/showback
- Savings Plans e Reserved Instances: compromisso para reduzir custo de compute
- Cost Allocation Tags: rastrear custo por time/produto/ambiente

## Dicas de exame

- SCP nao concede permissoes; apenas restringe.
- "Least operational overhead" em multi-account aponta para Control Tower e Identity Center.
- Para visibilidade granular de custo em escala, CUR + Athena e alternativa recorrente.
- Para workloads estaveis, RI/Savings Plans costuma ser resposta de otimizacao.
- Spot e melhor para cargas tolerantes a interrupcao.

## Anti-padroes

- usar conta unica para tudo
- usar root user no dia a dia
- sem padrao de tags obrigatorias
- sem limites/alertas de custo por ambiente
- sem segregacao de contas de log e seguranca

## Links relacionados

- [Cheatsheet](./cheatsheet.md)
- [Casos de uso](./casos-de-uso.md)
- [Questoes](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Links oficiais](./links.md)