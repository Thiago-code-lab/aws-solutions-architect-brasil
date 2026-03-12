# Módulo 09 — Desacoplamento: SQS, SNS, EventBridge

## Por Que Desacoplar?

Arquiteturas tightly-coupled criam cascatas de falhas: se o consumidor está lento ou falho, o produtor para. Serviços de mensageria e eventos introduzem um buffer ou roteador entre produtor e consumidor, aumentando resiliência, escalabilidade independente e tolerância a falhas.

---

## Amazon SQS (Simple Queue Service)

Serviço de fila de mensagens totalmente gerenciado.

### Standard vs FIFO

| Característica | Standard | FIFO |
|---|---|---|
| Throughput | **Ilimitado** | 300 TPS (sem batching), 3.000 TPS (com batching de 10) |
| Ordem | Best-effort (pode sair fora de ordem) | **Strict FIFO** (First In, First Out) |
| Deduplicação | Possível entregar duplicatas | **Exatamente uma vez** (deduplication window 5 min) |
| Naming | Qualquer nome | Deve terminar em `.fifo` |
| Uso típico | Alto throughput, tolerante a duplicatas | Pedidos financeiros, comandos em sequência |

### Visibility Timeout
- Quando uma mensagem é recebida, ela fica **invisível** por um período (padrão: 30s, máx: 12h)
- Durante esse tempo, o processamento deve concluir e deletar a mensagem
- Se o processamento falhar e o timeout expirar, a mensagem reaparece para outro consumidor
- **ChangeMessageVisibility**: estende o timeout se o processamento demorar mais

### DLQ (Dead-Letter Queue)
- Mensagens que falharam `maxReceiveCount` vezes são movidas para a DLQ
- Permite inspecionar e reprocessar mensagens problemáticas
- DLQ precisa ser do mesmo tipo da fila origem (FIFO → FIFO DLQ, Standard → Standard DLQ)
- **DLQ Redrive**: reprocessar mensagens da DLQ de volta à fila origem

### Long Polling
- `waitTimeSeconds` > 0 (máx 20s): a requisição de recebimento espera até ter mensagem ou timeout
- Reduz chamadas de API vazias (e custo) vs. Short Polling (retorna imediatamente, mesmo sem mensagem)
- **Habilitar Long Polling**: configurar na fila ou na chamada `ReceiveMessage`

### Outras Configurações Chave
| Parâmetro | Default | Máximo | Descrição |
|---|---|---|---|
| Message Retention | 4 dias | 14 dias | Tempo que mensagem fica na fila |
| Message Size | — | 256 KB | Por mensagem (use S3 + SQS Extended Client para maior) |
| Delay Queue | 0 | 15 min | Atraso antes de a mensagem ser visível |
| Receive Request Wait Time | 0 | 20s | Long polling window |

### SQS + Auto Scaling
- ASG escala baseado em `ApproximateNumberOfMessagesVisible` (via CloudWatch Custom Metric)
- Padrão: instâncias "worker" consomem da fila, ASG escala com o backlog

---

## Amazon SNS (Simple Notification Service)

Serviço de pub/sub totalmente gerenciado. Producers publicam em **Topics**, Subscribers recebem.

### Modelos de Distribuição
- **Fan-out**: 1 publicação → múltiplos subscribers (SQS, Lambda, HTTP, Email, SMS, Mobile Push)
- Tipicamente: aplicação publica no SNS → SNS distribui para múltiplas SQS filas (fan-out sem acoplamento)

```
Evento de Compra
    │
    ▼
SNS Topic: order-created
    ├── SQS: process-payment
    ├── SQS: update-inventory
    ├── SQS: send-email
    └── Lambda: update-analytics
```

### FIFO Topics
- Like FIFO SQS mas para pub/sub
- Subscribers devem ser SQS FIFO Queues
- 300 publishes/s, ordenado, sem duplicatas

### Message Filtering
- Cada subscriber pode ter uma **filter policy** (JSON) que define quais atributos de mensagem receber
- Reduz processamento desnecessário no subscriber
```json
{"event_type": ["ORDER_PLACED", "ORDER_FAILED"]}
```

### SNS vs SQS
| SNS | SQS |
|---|---|
| Pub/Sub (1 → N) | Fila (1:1 pull) |
| Push para subscribers | Consumers fazem poll/pull |
| Sem retenção de mensagem | Retenção até 14 dias |
| Fan-out | Ponto a ponto |

---

## Amazon EventBridge

Barramento de eventos serverless. Evolução do CloudWatch Events.

### Event Buses
| Tipo | Descrição |
|---|---|
| **Default** | Eventos de serviços AWS (EC2 state-change, S3 object created, etc.) |
| **Custom** | Você cria, para seus próprios eventos de aplicação |
| **Partner** | Eventos de SaaS parceiros (Zendesk, Datadog, PagerDuty, Auth0) |

### Rules
- Definem **event pattern** (what triggers) + **target** (what executes)
- Targets: Lambda, SQS, SNS, Step Functions, Kinesis, CodePipeline, EventBridge de outra conta, e 20+ outros
- **Schedule**: regras baseadas em cron ou rate (não precisa de evento)
- **Event Archive**: arquiva todos os eventos (ou filtrados) para replay
- **Event Replay**: reprocessa eventos arquivados (útil para debugging e novos consumers retroativos)

### Schema Registry
- Descobre e registra esquemas de eventos automaticamente
- Gera código bindings (Java, Python, TypeScript) para trabalhar com eventos tipados
- Valida eventos contra schemas

### EventBridge vs SNS vs SQS
| Aspecto | EventBridge | SNS | SQS |
|---|---|---|---|
| Modelo | Event bus (rules + targets) | Pub/Sub (topics) | Fila |
| Roteamento | Filtragem por conteúdo (JSON path) | Filter policies básicas | N/A |
| Fontes | AWS services, custom, SaaS partners | Suas aplicações | — |
| Replay | Sim (archive + replay) | Não | Não |
| Targets | 20+ tipos | SQS, Lambda, HTTP, etc. | Pull por consumers |
| Uso | Orquestração de eventos complexos | Fan-out simples | Buffer/decoupling |

---

## Amazon Kinesis

Plataforma para processamento de dados de streaming em tempo real.

### Kinesis Data Streams

- **Shards**: unidade de capacidade (1 MB/s entrada, 2 MB/s saída por shard)
- Retenção: 24h padrão, até 365 dias
- **Producers**: SDK, Kinesis Agent, Kinesis Producer Library (KPL — batching + compression)
- **Consumers**: Lambda, Kinesis Data Analytics, KCL (Kinesis Client Library), Firehose
- **Enhanced Fan-Out**: 2 MB/s por consumer por shard (push via HTTP/2, vs poll padrão)
- **Partition Key**: determina qual shard recebe o record (igual ao DynamoDB partition key — alta cardinalidade evita hot shards)

### Kinesis Data Firehose

- **Fully managed**, sem administração de shards (serverless)
- **Destinos**: S3, Redshift (via S3), OpenSearch, Splunk, DataDog, HTTP custom
- Não é armazenado (streaming, sem replay)
- **Buffer**: agrupa registros antes de entregar (buffer size ou buffer interval)
- Pode aplicar transformações com Lambda antes de entregar
- **Firehose ≠ Streams**: Firehose é near-real-time (latência de segundos/minutos); Streams é real-time

### Kinesis Data Streams vs Firehose
| Característica | Data Streams | Data Firehose |
|---|---|---|
| Latência | Real-time (<200ms) | Near-real-time (60s–900s buffer) |
| Gerenciamento | Manual (shards) | Serverless (automático) |
| Replay | Sim (até 365 dias) | Não |
| Destinos | Consumers customizados | S3, Redshift, OpenSearch, etc. |
| Custo | Por shard-hora | Por GB ingerido |

---

## Amazon MQ

Broker de mensagens gerenciado compatível com protocolos de mensageria abertos: **AMQP, MQTT, OpenWire, STOMP**.

- Suporta **Apache ActiveMQ** e **RabbitMQ**
- Diferente de SQS/SNS (protocolos proprietários da AWS)
- Usado para **lift-and-shift de aplicações existentes** que já usam ActiveMQ/RabbitMQ
- Deploy em 1 ou 2 AZs (Active/Standby para HA)
- Para novos projetos: prefira SQS/SNS (serverless, mais escalável)

---

## Dicas de Prova

- **SQS + SNS fan-out** = padrão clássico de desacoplamento — SNS distribui para múltiplas SQS
- **SQS FIFO**: exatamente-uma-vez + ordenado, mas limitado a 300/3.000 TPS
- **Visibility Timeout** menor que o tempo de processamento = duplicatas; maior = delay na reentrega
- **DLQ** é para mensagens que falharam `maxReceiveCount` vezes — não para mensagens lentas
- **Long Polling** (waitTimeSeconds=20) reduz custo e número de chamadas vazias
- **EventBridge** é preferível ao CloudWatch Events (EventBridge é a mesma coisa, mas com mais features)
- EventBridge **archive + replay** = debug retroativo e onboarding de novos consumers
- **Kinesis Data Streams** para real-time analytics + replay; **Firehose** para near-real-time → S3/Redshift
- **Hot shard no Kinesis** = partition key com baixa cardinalidade (ex: apenas "user"/"event") — distribuir melhor as chaves
- **Amazon MQ** = migração lift-and-shift de ActiveMQ/RabbitMQ; para novos projetos, use SQS/SNS
