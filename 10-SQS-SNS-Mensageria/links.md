# Links e Recursos — SQS, SNS e Mensageria (Módulo 10)

## Documentação Oficial AWS

- [Amazon SQS — Documentação Oficial](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [SQS — Filas Standard vs FIFO](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html)
- [SQS — Dead-Letter Queues (DLQ)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [SQS — Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
- [SQS — Long Polling](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-short-and-long-polling.html)
- [Amazon SNS — Documentação Oficial](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [SNS — Filter Policies](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)
- [SNS — Fan-out Pattern](https://docs.aws.amazon.com/sns/latest/dg/sns-common-scenarios.html)
- [Amazon EventBridge — Documentação](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [Amazon MQ — Documentação](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/welcome.html)

## FAQs

- [Amazon SQS FAQ](https://aws.amazon.com/sqs/faqs/)
- [Amazon SNS FAQ](https://aws.amazon.com/sns/faqs/)

## Whitepapers

- [Messaging Patterns on AWS](https://docs.aws.amazon.com/whitepapers/latest/develop-deploy-low-latency-applications/messaging.html)

## Artigos de Blog AWS

- [Standard vs FIFO: choosing the right queue](https://aws.amazon.com/blogs/compute/building-a-messaging-platform-using-sqs-and-sns/)
- [SQS DLQ best practices](https://aws.amazon.com/blogs/compute/handling-failures-with-sqs-dead-letter-queues/)
- [SNS to SQS fan-out pattern](https://aws.amazon.com/blogs/compute/exploring-the-amazon-sns-message-filtering-feature/)

## Ferramentas de Estudo

- [Tutorials Dojo — Amazon SQS Cheat Sheet](https://tutorialsdojo.com/amazon-sqs/)
- [Tutorials Dojo — Amazon SNS Cheat Sheet](https://tutorialsdojo.com/amazon-sns/)

## Comparativo Serviços de Mensageria

| Critério | SQS Standard | SQS FIFO | SNS | EventBridge |
|---------|-------------|----------|-----|-------------|
| Ordenação | Não garantida | Garantida | N/A | N/A |
| Throughput | Ilimitado | 300 msg/s (3000 c/ batching) | Ilimitado | 10k eventos/s |
| Deduplicação | Não | Sim (ID) | Não | Não |
| Entrega | At-least-once | Exactly-once | Push (fans-out) | Push (targets) |
| Consumers | Pull | Pull | Push (subscriptions) | Push (rules) |
| Uso típico | Desacoplamento | Pagamentos, pedidos | Notificações, fan-out | Eventos de serviços AWS |
| Retenção msg | 1 min–14 dias | 1 min–14 dias | N/A | N/A |
