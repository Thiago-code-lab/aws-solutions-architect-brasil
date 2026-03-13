# Módulo 17 — Casos de Uso Reais: Arquiteturas Multi-Serviço

## Sobre Este Módulo

Arquiteturas reais integram múltiplos serviços AWS. Este módulo apresenta padrões arquiteturais recorrentes no exame SAA-C03, onde a combinação certa de serviços determina a solução ótima.

---

## Caso 1: E-commerce de Alta Disponibilidade

**Requisitos**: Aplicação web global, escalável, com pagamentos seguros e catálogo de produtos.

```
Usuários Globais
    ↓
Route 53 (DNS + Health Checks + failover/latency routing)
    ↓
CloudFront (CDN — assets estáticos em edge locations)
    ↓
WAF + Shield Standard (proteção DDoS na borda)
    ↓
ALB (Application Load Balancer — distribui para múltiplas AZs)
    ↓
Auto Scaling Group → EC2 (app tier) ou ECS Fargate
    │
    ├── RDS Aurora MySQL (Multi-AZ para pedidos/usuários)
    │     └── Aurora Read Replicas (leitura pesada)
    ├── ElastiCache Redis (cache de sessão e catálogo)
    ├── DynamoDB (carrinho de compras — latência baixa)
    │
    └── S3 (imagens de produto) → CloudFront origin
         └── SQS (filas de pedidos) → Lambda (processamento async)
              └── SNS (notificações) → SES (email confirmação)
```

**Serviços-chave e justificativas:**
- Aurora Multi-AZ: disponibilidade para dados transacionais  
- ElastiCache: reduz latência e carga no banco de dados
- SQS desacopla processamento de pedidos (tolerância a falha na integração)
- CloudFront reduz latência mundial e custo de transferência

---

## Caso 2: Pipeline de Dados em Tempo Real

**Requisitos**: Ingestão de eventos de IoT, processamento em tempo real, analytics e alertas.

```
Dispositivos IoT
    ↓
IoT Core (MQTT) ou Kinesis Data Streams
    ↓
Lambda (transformação de eventos) ─── DynamoDB (estado atual dispositivo)
    │
    ├── Kinesis Firehose → S3 (raw data lake, Parquet)
    │         └── Glue Crawlers → Glue Catalog
    │               └── Athena (queries ad-hoc analistas)
    │
    └── Kinesis Data Analytics (Apache Flink)
          └── Anomaly detection → SNS → Lambda → PagerDuty
                                → CloudWatch Alarms
```

**Serviços-chave e justificativas:**
- Kinesis Data Streams: ingestão de alto volume em tempo real
- Kinesis Firehose: near real-time delivery para S3 (zero código de gerenciamento)
- Lambda ESM: processa eventos de Kinesis em batches  
- Athena: análise sem servidor dos dados no S3

---

## Caso 3: Aplicação Serverless com Autenticação

**Requisitos**: API REST serverless, autenticação de usuários, dados variáveis.

```
Mobile/Web App
    ↓
Amazon Cognito User Pool (signup/login/MFA/tokens JWT)
    ↓
API Gateway HTTP API (+ JWT Authorizer com Cognito)
    ↓
Lambda Functions:
    ├── GET /products   → DynamoDB (leitura, DAX para cache)
    ├── POST /orders    → DynamoDB + EventBridge
    ├── GET /files      → Pre-signed URL S3 retornada ao cliente
    └── POST /files     → Pre-signed URL S3 para upload direto

EventBridge:
    └── Rule: order.created → Lambda (enviar email) → SES
    └── Rule: order.created → SQS → Lambda (processar pagamento)

Monitoramento:
    X-Ray (tracing API GW → Lambda → DynamoDB)
    CloudWatch Logs (structured logging)
    CloudWatch Alarms (Lambda errors, throttles)
```

**Serviços-chave e justificativas:**
- Cognito User Pool: auth completo (OAuth2/OIDC) sem código próprio
- API GW HTTP API: custo 71% menor que REST API para endpoints simples
- Pre-signed URLs S3: clientes fazem upload/download direto no S3 (não passa pela Lambda)
- DAX: cache in-memory para DynamoDB (µs para leituras repetidas)

---

## Caso 4: Disaster Recovery Multi-Region

**Requisitos**: RTO < 15 min, RPO < 5 min para aplicação crítica.

```
Região Primária (us-east-1)
    ├── RDS Aurora (primary)     ──[global database]──→  Região DR (us-west-2)
    ├── DynamoDB Global Tables   ──[multi-master]──────→  DynamoDB Global Table
    ├── S3 (dados aplicação)     ──[CRR]──────────────→  S3 réplica
    └── EC2 ASG (app)

Route 53:
    ├── Health Check → ALB us-east-1 (primary, weight 100)
    └── Failover Record → ALB us-west-2 (DR, weight 0)
           └── Trigger: health check falha → DNS TTL flip para us-west-2

DR Region (us-west-2) — Warm Standby:
    ├── Aurora Secondary (read, promotes to primary on failover)
    ├── DynamoDB Global Table (já ativo — leitura local disponível)
    ├── EC2 ASG (capacidade reduzida — escala ao receber tráfego)
    └── ALB
```

**Estratégia: Warm Standby** → RTO ~10-15 min, RPO ~1-5 min (replicação assíncrona Aurora ~1s)

---

## Caso 5: Migração de Monolito para Microserviços

**Requisitos**: Migrar aplicação legada .NET para arquitetura moderna, sem downtime.

```
Fase 1 — Strangler Fig Pattern:
  Monolito original → Route 53 (via path-based routing no ALB)
      ├── /api/auth → Novo serviço auth (Fargate + Cognito)
      ├── /api/catalog → Novo serviço catálogo (Lambda + DynamoDB)
      └── /* → Monolito (EC2) ainda em produção

Fase 2 — Async Decoupling:
  Monolito emite eventos → SNS/EventBridge
    └── Microserviços consomem eventos (substituindo chamadas síncronas)

Fase 3 — Complete Migration:
  ECS/EKS (containers) ou Lambda (serverless) por domínio de negócio
  API Gateway (facade única para todos microserviços)
  Service Connect ou App Mesh (comunicação inter-serviços)
```

**Padrões usados:**
- **Strangler Fig**: substitui partes incrementalmente, nunca cutover total de uma vez
- **Event-driven**: desacopla serviços com EventBridge/SNS/SQS
- **API Gateway as facade**: clientes não precisam conhecer topologia interna

---

## Caso 6: Analytics de Big Data com Lake House

**Requisitos**: Data lake escalável para analytics em batch e em tempo real.

```
Fontes Diversas:
  ├── RDS/Aurora → DMS (CDC) → S3 (raw) 
  ├── SaaS APIs → Lambda → S3
  ├── Clickstream → Kinesis Firehose → S3
  └── Files → DataSync → S3

S3 Lake (zonas):
  raw/ → bronze/ (Glue ETL) → silver/ (Parquet + particionado) → gold/ (aggregated)

Catálogo:
  Glue Crawlers → Glue Data Catalog → Lake Formation (FGA permissions)

Consumo:
  Athena → reporting ad-hoc (paga por scan)
  Redshift Spectrum → DW queries misturando S3 + Redshift
  QuickSight (SPICE) → dashboards executivos
  SageMaker → modelos ML sobre dados gold/
```

---

## Padrões Recorrentes no Exame

| Padrão | Serviços | Contexto |
|---|---|---|
| Fan-out | SNS → múltiplos SQS | Notificar vários sistemas ao mesmo tempo |
| Desacoplamento | SQS entre producer/consumer | Buffer, tolerar picos, retry |
| CQRS | DynamoDB (write) + ElastiCache (read) | Separar leituras de escritas para otimizar |
| Event Sourcing | Kinesis/EventBridge → Lambda → DynamoDB | Audit trail + processamento assíncrono |
| Cache-aside | Lambda → ElastiCache miss → DynamoDB | Reduzir latência de leituras repetidas |
| Blue/Green | CodeDeploy ou R53 weighted | Zero-downtime deployment |

---

## Dicas de Prova

- Questões de arquitetura: procure o serviço **mais gerenciado** que satisfaz os requisitos
- "Alta disponibilidade" → Multi-AZ (RDS Multi-AZ, ASG em múltiplas AZs, ELB)
- "Baixa latência global" → CloudFront (assets), Global Accelerator (TCP/UDP), Aurora Global DB
- "Desacoplamento" → SQS (ponto-a-ponto) ou SNS (fan-out)
- "Processamento assíncrono" → SQS + Lambda ou EventBridge + Lambda
- "Econômico para cargas variáveis" → Lambda (serverless), Fargate On-Demand, Spot ASG
- "Segurança em camadas" → WAF + Shield + SG + NACL + KMS + Macie
