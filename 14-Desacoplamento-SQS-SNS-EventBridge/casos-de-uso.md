# Casos de Uso — Módulo 09: Desacoplamento (SQS, SNS, EventBridge)

## Caso 1: Fan-Out com SNS + SQS para Processamento de Pedidos

**Cenário:** E-commerce precisa que cada novo pedido acione: atualização de estoque, geração de nota fiscal e envio de email de confirmação — independentemente.

**Arquitetura:**
```
Order Service (Producer)
  └── SNS Topic: order-events
        ├── SQS Queue: inventory-queue → Lambda: InventoryService
        │     └── [FilterPolicy: {eventType: ["order.created"]}]
        │           └── DLQ: inventory-dlq (maxReceiveCount=3)
        │
        ├── SQS Queue: invoice-queue → Lambda: InvoiceService
        │     └── [FilterPolicy: {eventType: ["order.created", "order.completed"]}]
        │           └── DLQ: invoice-dlq
        │
        └── Lambda: EmailService (direto) → SES
              └── [FilterPolicy: {eventType: ["order.created"]}]

Benefits:
✓ Order Service não conhece os consumers
✓ Falha no InvoiceService não afeta InventoryService
✓ Cada queue tem seu próprio DLQ e retry
✓ Adicionar novo consumer = nova SQS subscription (zero impacto nos outros)
```

---

## Caso 2: Worker Assíncrono com SQS + Auto Scaling

**Cenário:** Sistema de processamento de vídeo. Usuários fazem upload e os vídeos precisam ser transcodificados. Carga varia muito ao longo do dia.

**Arquitetura:**
```
S3 (upload) → S3 Event Notification → SQS Queue: video-transcode
                                              │
                                        SNS Alarm (CW metric)
                                              │
                              ApproximateNumberOfMessagesVisible ÷ FleetSize
                                              │
                                    ASG Target Tracking Policy
                                              │
                                Auto Scale EC2 Workers (min=1, max=20)
                                              │
                        EC2 Worker: polling SQS → transcode → S3 (output)
```

**Por que não Lambda?** Transcodificação pode demorar 30+ minutos (Lambda máximo = 15min). EC2 com ASG baseado em SQS metric é o padrão para workloads de longa duração.

**Otimização de custo:** Workers podem ser Spot Instances (até 90% desconto). Se spot for interrompida, a mensagem SQS reaparece após VisibilityTimeout → outro worker pega.

---

## Caso 3: Pipeline de IoT com Kinesis + Lambda

**Cenário:** 10.000 sensores IoT enviam dados de temperatura a cada segundo. Dados precisam ser armazenados no S3 e alertas gerados em tempo real para temperaturas anormais.

**Arquitetura:**
```
IoT Sensors (10.000 × 1 msg/s = 10K msgs/s)
  └── AWS IoT Core (MQTT)
        └── IoT Rule → Kinesis Data Stream (10 shards × 1K req/s = 10K/s)
                            ├── Lambda Consumer (Enhanced Fan-Out)
                            │     └── Analisa temperatura em real-time
                            │           └── > 80°C → SNS → PagerDuty
                            │
                            └── Kinesis Data Firehose
                                  └── (buffer 60s) → S3 (Parquet)
                                        └── Glue Catalog → Athena queries
```

**KDS vs Firehose:** KDS para alertas real-time (Lambda, < 200ms). Firehose para ingestão batch no S3 (near real-time, sem código de consumer).

---

## Caso 4: Event-Driven com EventBridge para Automação de Operações

**Cenário:** Equipe de ops quer automações: ao escalar instâncias EC2, notificar equipe; ao criar usuário IAM, verificar políticas; ao atualizar código no CodePipeline, atualizar wiki.

**Regras EventBridge:**
```
Default Event Bus (eventos AWS)
  ├── Rule 1: EC2 Instance State Change → RUNNING
  │     └── Target: Lambda → Slack webhook ("Nova instância i-xxx em us-east-1")
  │
  ├── Rule 2: IAM CreateUser API (via CloudTrail)
  │     └── Target: Lambda → IAM Compliance Check → SNS se violação
  │
  └── Rule 3: CodePipeline → Execution Succeeded
        └── Target: Step Functions → Confluence API Update

Custom Event Bus: app-events
  └── PutEvents → application custom events
        ├── Rule: source = "app.orders" → Step Functions (complex workflows)
        └── Rule: source = "app.billing" → Firehose → S3 (audit log)
```

---

## Caso 5: Desacoplamento com Amazon MQ para Migração de Sistema Legado

**Cenário:** Sistema bancário legado Java usa ActiveMQ para comunicação entre microserviços. Empresa quer mover para AWS sem reescrever o código.

**Estratégia Lift-and-Shift:**
```
BEFORE (on-premises):
  ServiceA (AMQP) → ActiveMQ Broker → ServiceB (AMQP)
                                     → ServiceC (STOMP)

AFTER (AWS - Amazon MQ):
  ServiceA (AMQP) → Amazon MQ (Active/Standby) → ServiceB (AMQP)
                    ├── Broker 1 (us-east-1a) Active       → ServiceC (STOMP)
                    └── Broker 2 (us-east-1b) Standby
```

**Estratégia de modernização incremental:**
```
Phase 2 (Future):
  ServiceA → Amazon MQ → Bridge Lambda → SNS/SQS
  [Lambda consome do MQ e republica no SNS]
  [Novos serviços usam SQS/SNS; legados continuam com MQ]
  [Migração gradual sem big-bang]
```

**Por que não ir direto para SQS?** Código de produção usa AMQP/STOMP client libraries. Mudar para SQS SDK exigiria refatoração significativa e testes. Amazon MQ = zero mudanças no código.
