# Cheatsheet — Módulo 09: Desacoplamento (SQS, SNS, EventBridge)

## SQS Standard vs FIFO

| | Standard Queue | FIFO Queue |
|---|---|---|
| Ordering | Best-effort (pode reordenar) | Exato (First-In-First-Out) |
| Entrega | At-least-once (pode duplicar) | Exactly-once (com deduplication ID) |
| Throughput | Quase ilimitado | 300 TPS (3.000 com batching) |
| Deduplication | Não | Sim (5 min janela) |
| Message Groups | Não | Sim (paralelismo dentro do FIFO) |
| Custo | Menor | Maior |

## SQS — Parâmetros Críticos

| Parâmetro | Padrão | Máximo | Descrição |
|---|---|---|---|
| VisibilityTimeout | 30s | 12h | Tempo invisível após leitura |
| Message Retention | 4 dias | 14 dias | Tempo na fila antes de expirar |
| Max Message Size | 256 KB | 256 KB | Payload máximo (S3 para maiores) |
| Long Polling waitTime | 0 (short) | 20s | Tempo de espera por mensagens |
| Delay Queue | 0 | 15 min | Delay antes de ficar visível |
| Dead Letter Queue maxReceiveCount | - | Configurável | Tentativas antes de ir para DLQ |

## SNS — Destinations por Tipo

| Destination | Standard Topic | FIFO Topic |
|---|---|---|
| SQS | Sim | Apenas SQS FIFO |
| Lambda | Sim | Não |
| HTTP/HTTPS | Sim | Não |
| Email | Sim | Não |
| SMS | Sim | Não |
| Firehose | Sim | Não |
| SQS FIFO | Não | Sim |

## Kinesis Data Streams vs Firehose

| | Kinesis Data Streams | Kinesis Firehose |
|---|---|---|
| Gerenciamento | Manual (shards, consumers) | Totalmente gerenciado |
| Latência | ~200ms (real-time) | Near real-time (60s ou 1 MB buffer mín.) |
| Consumidores | Múltiplos (KCL, Lambda, etc.) | Destinos fixos |
| Replay | Sim (1-365 dias retenção) | Não |
| Destinos | Consumidores customizados | S3, Redshift, OpenSearch, Splunk, HTTP |
| Shards | Manual ou Auto Scaling | Auto-scaling gerenciado |
| Transformação | Não (processado pelo consumer) | Lambda opcional antes do destino |

## SQS vs SNS vs EventBridge vs Kinesis

| | SQS | SNS | EventBridge | Kinesis |
|---|---|---|---|---|
| Modelo | Pull (consumer faz polling) | Push (entrega para subscribers) | Push (rules → targets) | Pull + Push (EFO) |
| Pattern | Task queue | Pub/Sub fan-out | Event routing | Data streaming |
| Retenção | 1 min - 14 dias | Sem retenção | Archive (opcional) | 1-365 dias |
| Replay | Não (DLQ para falhas) | Não | Sim (Archive + Replay) | Sim |
| Ordenação | FIFO opcional | FIFO opcional | Não garantida | Por shard |
| Melhor para | Workers assíncronos | Fan-out 1:N | Disparar ações em eventos | Analytics de stream |

## Padrão Fan-Out com SNS → SQS
```
Producer
  └── SNS Topic (order.events)
        ├── SQS Queue → Consumer A (estoque)     [FilterPolicy: {eventType: [order.created]}]
        ├── SQS Queue → Consumer B (faturamento) [FilterPolicy: {eventType: [order.created, order.completed]}]
        └── Lambda → Consumer C (auditoria)      [Sem filtro - recebe todos]
```

## EventBridge Event Pattern (exemplo)
```json
{
  "source": ["aws.rds"],
  "detail-type": ["RDS DB Instance Event"],
  "detail": {
    "EventID": ["RDS-EVENT-0006"],
    "SourceType": ["DB_INSTANCE"]
  }
}
```

## Dicas Rápidas de Prova
- SQS FIFO: NOME da fila deve terminar em `.fifo`
- SQS DLQ: deve ser mesmo tipo (Standard DLQ para Standard; FIFO DLQ para FIFO)
- **Fan-out clássico**: SNS → múltiplas SQS Queues (não múltiplas SNS → um SQS)
- Kinesis Shard = 1 MB/s entrada, 2 MB/s saída (shared) ou por consumer com Enhanced Fan-Out
- **Amazon MQ = lift-and-shift** de brokers com AMQP/MQTT/STOMP; não escala infinitamente como SQS
- EventBridge Partner Bus: SaaS (Salesforce, Datadog) → eventos direto no EventBridge sem código de polling
- SQS Max Receive Count + DLQ = poison pill isolation (standard anti-pattern para mensagens problemáticas)
- Kinesis Firehose: transforma com Lambda; carrega para S3/Redshift/OpenSearch; near-real-time (buffer)
- Para evento em tempo real com múltiplos consumers independentes: Kinesis Data Streams > SQS
- EventBridge Pipe = source → (optional filter + enrich) → target sem Lambda de roteamento
