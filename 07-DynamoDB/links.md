# Links e Recursos — DynamoDB (Módulo 07)

## Documentação Oficial AWS

- [Amazon DynamoDB — Documentação Oficial](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [DynamoDB — Índices Secundários Globais (GSI)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)
- [DynamoDB — Índices Secundários Locais (LSI)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html)
- [DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
- [DynamoDB TTL — Time to Live](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
- [DynamoDB Auto Scaling](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/AutoScaling.html)
- [DynamoDB On-Demand vs Provisioned](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html)
- [DAX — DynamoDB Accelerator](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)
- [DynamoDB Transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
- [DynamoDB Condition Expressions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html)

## FAQs

- [Amazon DynamoDB FAQ](https://aws.amazon.com/dynamodb/faqs/)

## Whitepapers

- [Best Practices for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/BestPractices.html)
- [DynamoDB — NoSQL Design Patterns](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-general-nosql-design.html)

## Artigos de Blog AWS

- [NoSQL Design Patterns for DynamoDB](https://aws.amazon.com/blogs/database/nosql-design-patterns-for-dynamodb/)
- [GSI Overloading - advanced indexing](https://aws.amazon.com/blogs/database/using-global-secondary-indexes-for-dynamic-queries/)
- [Single-table design with DynamoDB](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/)

## Ferramentas de Estudo

- [Tutorials Dojo — Amazon DynamoDB Cheat Sheet](https://tutorialsdojo.com/amazon-dynamodb/)
- [NoSQL Workbench for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)

## Tabela Comparativa DynamoDB

| Critério | On-Demand | Provisioned |
|---------|-----------|-------------|
| Planejamento de capacidade | Não | Sim (RCU/WCU) |
| Escalonamento | Automático | Manual ou Auto Scaling |
| Custo por req | Maior | Menor (uso previsível) |
| Ideal para | Tráfego imprevisível | Tráfego estável |
| Free tier | Não | Sim (25 WCU/RCU) |

| Modo de Leitura | Definição |
|----------------|-----------|
| Eventually Consistent | Padrão — 1/2 do custo em RCU |
| Strongly Consistent | Dados mais recentes — 1 RCU por 4KB |
| Transactional Read | 2x RCU — garante atomicidade |
