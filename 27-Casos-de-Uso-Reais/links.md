# Links e Recursos — Casos de Uso Reais (Módulo 17)

## Documentação Oficial AWS

- [AWS Step Functions — Documentação](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
- [Step Functions — Express vs Standard Workflows](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-standard-vs-express.html)
- [Amazon EventBridge — Documentação](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [Amazon SNS — Documentação](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [SNS — Message Filtering](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)
- [Amazon SQS — Documentação](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [Amazon AppFlow — Documentação](https://docs.aws.amazon.com/appflow/latest/userguide/what-is-appflow.html)
- [AWS Microservices Architecture](https://aws.amazon.com/microservices/)
- [Amazon IVS (Interactive Video Service) — Documentação](https://docs.aws.amazon.com/ivs/latest/userguide/what-is.html)

## FAQs

- [AWS Step Functions FAQ](https://aws.amazon.com/step-functions/faqs/)
- [Amazon SNS FAQ](https://aws.amazon.com/sns/faqs/)
- [Amazon EventBridge FAQ](https://aws.amazon.com/eventbridge/faqs/)

## Whitepapers

- [Microservices on AWS Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/microservices-on-aws/microservices-on-aws.html)
- [Building Event-Driven Architectures on AWS](https://docs.aws.amazon.com/whitepapers/latest/building-event-driven-architectures/building-event-driven-architectures.html)
- [SaaS on AWS Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/saas-architecture-fundamentals/saas-architecture-fundamentals.html)

## Artigos de Blog AWS

- [Implementing event-driven architectures with EventBridge](https://aws.amazon.com/blogs/compute/building-an-event-driven-application-with-amazon-eventbridge/)
- [Saga pattern with Step Functions](https://aws.amazon.com/blogs/compute/implementing-the-saga-pattern-with-aws-step-functions/)
- [Strangler Fig pattern migration on AWS](https://aws.amazon.com/blogs/architecture/modernizing-banking-applications-using-microservices-on-aws/)
- [Building SaaS multi-tenant architectures on AWS](https://aws.amazon.com/blogs/saas/aws-saas-factory-multi-tenant-isolation-patterns/)

## Vídeos re:Invent

- [Event-Driven Architecture (API203)](https://www.youtube.com/results?search_query=aws+reinvent+event+driven+architecture+API203)
- [Microservices on AWS (ARC307)](https://www.youtube.com/results?search_query=aws+reinvent+microservices+architecture+ARC307)
- [Step Functions for orchestration (SVS303)](https://www.youtube.com/results?search_query=aws+reinvent+step+functions+orchestration)
- [SaaS on AWS Multi-Tenancy (SDD404)](https://www.youtube.com/results?search_query=aws+reinvent+saas+multi+tenancy+SDD404)

## Ferramentas de Estudo

- [Tutorials Dojo — AWS Step Functions Cheat Sheet](https://tutorialsdojo.com/aws-step-functions/)
- [Tutorials Dojo — Amazon EventBridge Cheat Sheet](https://tutorialsdojo.com/amazon-eventbridge/)
- [AWS Skill Builder — Event-Driven Architectures Workshop](https://explore.skillbuilder.aws/learn/course/external/view/elearning/12246/building-event-driven-architectures-on-aws)

## Padrões de Arquitetura — Resumo

| Padrão | Descrição | Serviços AWS |
|--------|-----------|-------------|
| Fan-out | 1 produtor → N consumidores | SNS → SQS |
| Saga (coreografado) | Eventos locais por serviço | EventBridge, SNS |
| Saga (orquestrado) | Coordenador central | Step Functions |
| Event Sourcing | Imutabilidade de eventos | DynamoDB Streams, Kinesis |
| Strangler Fig | Migração incremental | API Gateway, ALB |
| CQRS | Separação leitura/escrita | DynamoDB + ElastiCache |
