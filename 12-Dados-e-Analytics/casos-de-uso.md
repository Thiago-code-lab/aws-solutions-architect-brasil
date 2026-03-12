# Casos de Uso Reais — Dados e Analytics (Módulo 12)

## Caso 1 — Data Lake Centralizado para Empresa de Varejo

**Contexto:** Rede de varejo com 500 lojas precisa centralizar dados de vendas (POS), estoque (ERP), comportamento web (clickstream) e dados externos (clima, feriados) para análise integrada.

**Requisitos:**
- Ingestão de múltiplas fontes com diferentes formatos e frequências
- Isolamento de acesso por departamento (financeiro não vê dados de RH)
- Query ad hoc pelos analistas sem provisionamento de clusters
- Custo proporcional à quantidade de dados analisados

**Arquitetura:**
```
FONTES DE DADOS:
POS (batch diário CSV) ──────────────────┐
ERP (batch noturno JSON) ────────────────┤ → S3 Raw Zone
Clickstream (real-time) → KDF ──────────┤    (dados brutos)
APIs externas (Lambda agendada) ─────────┘
                                         │
                                    AWS Glue ETL
                                    (job diário)
                                         │
                                    S3 Processed Zone
                                    (Parquet + particionado)
                                         │
                              ┌──────────┴──────────┐
                              │                     │
                           Athena                Redshift Spectrum
                        (query ad hoc)          (dashboards BI)
                              │
                     Lake Formation
                     (controle acesso por coluna/tabela)
                     
Glue Data Catalog (metadados centralizados)
CloudWatch → Glue Job alerts
```

**Estratégia de Particionamento S3:**
```
s3://data-lake/processed/vendas/
  year=2024/
    month=01/
      day=15/
        loja=SP001/
          dados.parquet
```
→ Athena filtra apenas as partições necessárias (reduz custo de scan)

**Controle de Acesso Lake Formation:**
| Perfil | Acesso |
|--------|--------|
| Analista Financeiro | vendas.*, faturamento.* (sem PII) |
| Analista Marketing | clickstream.*, campanhas.* |
| Data Scientist | Todas as tabelas (read-only) |
| ETL Service Role | Todas (leitura raw + escrita processed) |

---

## Caso 2 — Pipeline de Análise de Fraudes em Tempo Real

**Contexto:** Fintech precisa detectar transações fraudulentas em tempo real (<1 segundo) durante o processamento de pagamentos.

**Requisitos:**
- Análise de cada transação em < 500ms
- Enriquecer transação com histórico do usuário
- Score de risco via modelo ML
- Armazenar todas as transações para treinamento futuro

**Arquitetura:**
```
App Pagamento
     │ PUT record
     ▼
Kinesis Data Streams (shard por região)
     │ Lambda trigger (batch 100, window 1s)
     ▼
Lambda (fraud-scorer)
├── Lê histórico em DynamoDB (< 5ms com DAX)
├── Calcula features (freq, valor, geolocalização)
├── Invoca SageMaker Endpoint (scoring ML)
└── Se score > 0.8: publica em SNS (alerta fraude)
│
├── Kinesis Data Firehose (todas transações)
│        └── S3 (treino futuro + compliance)
│
└── DynamoDB (atualiza perfil usuário)

SNS → Lambda → bloquear transação + notificar usuário
CloudWatch → Alarm (latência > 300ms ou error rate > 1%)
```

**Kinesis vs SQS para este caso:**
- **Kinesis:** múltiplos consumidores do mesmo stream, ordenação por shard, replay de eventos
- **SQS:** 1 consumidor por mensagem, sem ordenação garantida (Standard), sem replay

---

## Caso 3 — Warehouse Analítico para Relatórios de Negócio

**Contexto:** Empresa de logística precisa de dashboards de performance (entregas no prazo, custo por rota, produtividade motoristas) para diretoria, com dados atualizados a cada hora.

**Requisitos:**
- Relatórios complexos com JOINs entre múltiplas tabelas
- Dados consolidados de 5 anos históricos
- Atualização a cada 1 hora
- QuickSight para visualização dos executivos

**Arquitetura:**
```
RDS Aurora (sistema operacional)
     │ DMS (Change Data Capture - CDC)
     ▼
S3 (staging zone — Parquet)
     │ Redshift COPY (a cada 1h)
     ▼
Redshift Cluster (Warehousing)
├── Tabelas dimensão: dim_motorista, dim_rota, dim_veiculo
├── Tabelas fato: fato_entrega, fato_custo
└── Materialized Views (KPIs pré-calculados)
     │
     ▼
Amazon QuickSight (dashboards diretoria)
     │
Redshift Spectrum → S3 (consultar dados históricos >2 anos sem mover para Redshift)
```

**Distribuição Redshift:**
| Tabela | Distribution Key | Sort Key |
|--------|-----------------|---------|
| fato_entrega | motorista_id (JOIN freq) | data_entrega |
| fato_custo | rota_id | data |
| dim_motorista | ALL (pequena tabela) | — |

---

## Caso 4 — Análise de Logs de Aplicação com Athena

**Contexto:** Startup de SaaS precisa analisar logs de API (erros, latência, usuários ativos) sem manter um cluster Elasticsearch. Volume: 10 GB de logs/dia.

**Requisitos:**
- Análise de logs dos últimos 90 dias
- Queries por data, endpoint, código de erro
- Custo controlado (Elasticsearch cluster ~$500/mês)
- Alertas de anomalias de erro

**Arquitetura:**
```
ALB Access Logs → S3 (logs-brutos/alb/year=../month=../day=..)
CloudWatch Logs → Kinesis Data Firehose → Conversão Parquet → S3 (logs-processados/)
Lambda → Custom Logs → CloudWatch → KDF → S3

S3 (logs-processados, particionado por data)
     │
Glue Crawler (atualiza partições diariamente)
     │
Glue Data Catalog
     │
Athena (queries ad hoc)
     │
QuickSight (dashboards de erro rate, p99 latência)

CloudWatch Logs Insights → Queries rápidas (últimas hrs)
EventBridge → Lambda (alerta se error rate > 5% em 5 min)
```

**Custo Athena vs Elasticsearch:**
| Item | Athena | Elasticsearch |
|------|--------|--------------|
| 10GB scan/query | $0.05 | — |
| 100 queries/dia | ~$5/dia | cluster fixo |
| Cluster base | $0 | ~$500/mês |
| Break-even | favorável se < 100 queries/dia complexas | — |

---

## Caso 5 — ETL em Tempo Real com Kinesis + Glue Streaming

**Contexto:** Empresa de marketplaces recebe 50.000 eventos/minuto de cliques, impressões e conversões. Precisa ter dados disponíveis para análise de campanhas com atraso máximo de 5 minutos.

**Requisitos:**
- Latência de ingestão a análise: < 5 minutos
- Enriquecimento de evento (adicionar dados de produto do catálogo)
- Deduplicação (mesmo evento pode chegar duplicado)
- Custo controlado (sem cluster sempre ligado)

**Arquitetura:**
```
App (tracker JS/mobile)
     │ POST /track
     ▼
API Gateway → Kinesis Data Streams (10 shards)
                     │
              Glue Streaming ETL
              ├── Lê do KDS a cada 60s
              ├── Enriquece com catálogo (DynamoDB lookup)
              ├── Deduplica por event_id (janela 5min)
              ├── Converte para Parquet
              └── Escreve em S3 (particionado por hora)
                     │
              S3 (processed/events/)
                     │
              Athena (query por campanha/produto — até 5min atrás)
              QuickSight (dashboard tempo quase-real)
              
EventBridge → Glue Job (trigger a cada 5min como alternativa simpler)
```

**Glue Streaming vs Kinesis Data Analytics (Flink):**
| Critério | Glue Streaming | KDA / Flink |
|---------|---------------|-------------|
| Complexidade setup | Baixa | Alta |
| Latência mínima | ~60s | Ms–segundos |
| Janelas de tempo | Básico | Avançado (sliding, tumbling) |
| Custo | DPU/hora | KPU/hora |
| Quando usar | ETL near-real-time simples | CEP, anomaly detection real-time |
