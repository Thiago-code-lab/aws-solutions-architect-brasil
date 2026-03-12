# Cheatsheet — Casos de Uso Reais (Módulo 17)

## Padrões de Arquitetura Fundamentais

| Padrão | Problema Resolvido | Serviços AWS Chave | Pista de Prova |
|--------|-------------------|-------------------|----------------|
| **Fan-Out** | 1 evento → múltiplos processadores | SNS → SQS (N filas) | "processar em paralelo por múltiplos sistemas" |
| **CQRS** | Leitura e escrita com modelos distintos | DynamoDB + ElastiCache / Aurora + Athena | "otimizar leitura sem impactar escrita" |
| **Event Sourcing** | Histórico imutável de estado | Kinesis / DynamoDB Streams | "audit trail completo / replay de eventos" |
| **Strangler Fig** | Migração incremental de monolito | ALB + Target Groups / API GW | "migrar sem downtime, rota por rota" |
| **Cache-Aside** | Reduzir latência de leitura | ElastiCache (Redis/Memcached) | "cache miss → DB, cache hit → rápido" |
| **Saga** | Transação distribuída (rollback) | Step Functions + SQS + Lambda | "coordenar transações entre microsserviços" |
| **Bulkhead** | Isolar falhas entre componentes | SQS separadas por prioridade/tenant | "falha em um não afeta outro" |
| **Circuit Breaker** | Evitar cascata de falhas | Lambda + DLQ + CloudWatch Alarm | "parar chamadas quando downstream falha" |

---

## Quando Usar Cada Tipo de Integração

| Integração | Quando Usar | Serviço | Latência |
|------------|-------------|---------|----------|
| **Síncrona** | Resposta imediata necessária | API GW → Lambda, ALB | < 30s |
| **Assíncrona simples** | Desacoplamento, tolerância a falhas | SQS → Lambda | segundos-minutos |
| **Fan-out** | 1 evento → N consumidores | SNS → múltiplas SQS | segundos |
| **Streaming** | Volume alto, tempo real | Kinesis Data Streams | ms-segundos |
| **Agendada** | Tarefas periódicas | EventBridge Scheduler → Lambda | horário |
| **Workflow** | Orquestração multi-step com retry | Step Functions | ms-horas |
| **Pub/Sub global** | Múltiplas regiões, múltiplos protocolos | SNS cross-region | segundos |

---

## Multi-Região: Active-Active vs Active-Passive

| Critério | Active-Active | Active-Passive |
|----------|--------------|----------------|
| **Objetivo** | Sem perda de disponibilidade | Menor custo de standby |
| **RPO** | ~0 | Minutos (depende replicação) |
| **RTO** | ~0 (failover automático) | 1–10 min |
| **Custo** | 2x (infraestrutura dupla) | ~1.2x (standby menor) |
| **Complexidade** | Alta (conflito de dados) | Baixa |
| **Route 53 policy** | Latency / Geolocation | Failover |
| **Banco** | Aurora Global (writer em ambas) | Aurora Global (1 writer) |
| **Caso de uso** | Global SaaS, finanças 24/7 | DR corporativo, backup regional |

---

## Arquiteturas por Vertical

### E-commerce de Alta Disponibilidade
```
Cliente → CloudFront → ALB (Multi-AZ)
                          ├── EC2 Auto Scaling (app)
                          └── ElastiCache (carrinho)
                               └── Aurora Multi-AZ (pedidos)
                                    └── DynamoDB (catálogo/inventário)
SNS → SQS → Lambda (notificações)
EventBridge → Lambda (processamento async pós-pedido)
```

### Pipeline de IoT
```
Dispositivos → IoT Core (MQTT)
                   ├── IoT Rules → Kinesis Data Streams → Lambda (real-time)
                   ├── IoT Rules → S3 (raw data) → Glue → Athena (batch)
                   └── IoT Rules → DynamoDB (estado atual dispositivo)
CloudWatch → SNS (alertas threshold)
```

### Plataforma de Streaming de Vídeo
```
Upload → S3 → Lambda → MediaConvert (transcoding)
                             └── S3 (múltiplas resoluções)
                                  └── CloudFront (CDN global)
DynamoDB (metadados vídeo)
Cognito (autenticação usuários)
```

### Data Lake Moderno
```
Dados → S3 (raw zone) → Glue ETL → S3 (processed zone)
                                         └── Athena (query ad hoc)
                                         └── Redshift Spectrum (BI)
Lake Formation (controle acesso por tabela/coluna)
Glue Data Catalog (metadados centralizados)
```

---

## Multi-Tenant SaaS no AWS

| Estratégia | Isolamento | Custo | Quando Usar |
|------------|-----------|-------|-------------|
| **Silo** (por tenant) | Máximo (stack dedicada) | Alto | Compliance, grandes contratos |
| **Pool** (compartilhado) | Lógico (tag/ID) | Baixo | Muitos tenants pequenos |
| **Híbrido** | Médio (tier-based) | Médio | Mix de clientes premium/free |
| | | | |
| **DynamoDB** | Partition key = tenantId | Baixo | Dados com padrão uniform |
| **RDS** | Schema por tenant | Médio | SQL, tenants médios |
| **Aurora** | Database por tenant | Alto | Grandes clientes enterprise |

---

## Padrões de Segurança por Cenário

| Cenário | Padrão Correto | Serviços |
|---------|---------------|---------|
| API pública com auth | JWT/OAuth2 | Cognito User Pool + API GW |
| API B2B (empresa-empresa) | mTLS / API Key | API GW + ACM |
| Acesso cross-account | IAM Role + AssumeRole | STS |
| Segredo em Lambda | Secrets Manager (não env var) | Secrets Manager + IAM |
| Criptografia dados S3 | SSE-KMS (CMK) | KMS + S3 |
| Proteção DDoS layer 7 | WAF + Shield Advanced | WAF + CloudFront |
| Auditoria quem fez o quê | CloudTrail + Athena query | CloudTrail + S3 + Athena |

---

## HIPAA / Compliance — Padrões de Exame

| Requisito | Solução AWS |
|-----------|------------|
| Dados PHI em repouso | KMS CMK (customer-managed) |
| Dados PHI em trânsito | TLS 1.2+ (ACM) |
| Logs imutáveis | CloudTrail + S3 Object Lock (WORM) |
| Acesso mínimo necessário | IAM granular + SCPs |
| BAA (Business Associate Agreement) | Assinar com AWS para serviços elegíveis |
| Isolamento rede | VPC + Security Groups + NACLs |

---

## Dead Letter Queue (DLQ) — Padrão de Tolerância a Falhas

```
SQS → Lambda (processa)
   ↓ maxReceiveCount excedido
  DLQ (SQS separada) → CloudWatch Alarm → SNS → Email/PagerDuty
                               ↓
                         Análise manual / Lambda de análise
```

- **maxReceiveCount:** 3–5 tentativas antes de mover para DLQ
- **DLQ de SQS:** outra fila SQS
- **DLQ de Lambda async:** SQS ou SNS
- **Exame:** se a questão fala em "reprocessar mensagens falhas" → DLQ

---

## Dicas de Prova

| Pista na Questão | Resposta Esperada |
|-----------------|------------------|
| "processar em paralelo por múltiplos consumidores" | SNS → múltiplas SQS (fan-out) |
| "desacoplar produtor de consumidor" | SQS (FIFO se ordem importa) |
| "orquestrar sequência de passos com retry/compensação" | Step Functions |
| "migração incremental sem derrubar sistema legado" | Strangler Fig (ALB rules) |
| "histórico completo de mudanças / replay" | Event Sourcing (Kinesis/DynamoDB Streams) |
| "reduzir latência leitura de DB" | ElastiCache (Cache-Aside) |
| "isolar falha de um serviço upstream" | SQS + DLQ + Circuit Breaker |
| "RTO ≈ 0, RPO ≈ 0, global" | Active-Active + Aurora Global + Route 53 Latency |
| "dados de telemetria IoT em tempo real" | IoT Core → Kinesis → Lambda |
| "múltiplos tenants, custo baixo" | Pool strategy (shared infra, tenantId partition key) |
| "notificar vários sistemas após evento de pedido" | SNS tópico com múltiplas assinaturas |
| "transação entre microsserviços com rollback" | Saga Pattern (Step Functions) |
