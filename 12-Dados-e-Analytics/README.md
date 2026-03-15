# Módulo 12 — Dados e Analytics

## Visão Geral do Ecossistema de Analytics AWS

```
Ingestão → Armazenamento → Processamento → Análise → Visualização
  Kinesis    S3 (Data Lake)   EMR / Glue     Athena    QuickSight
  MSK        Redshift DW      Spark          OpenSearch BI tools
  DataSync   OpenSearch       Lambda         Redshift
```

---

## Amazon Athena

SQL serverless sobre dados no S3:

- **Pay-per-query**: $5 por TB de dados escaneados
- Sem infraestrutura para gerenciar; lê formatos: Parquet, ORC, Avro, JSON, CSV
- Usa **Glue Data Catalog** como metastore
- Suporta particionamento → **crucial para reduzir custo** (evita full table scan)
- **Federated Query**: consulta outras fontes de dados via Lambda connectors (DynamoDB, RDS, Redshift, CloudWatch)
- **Athena for Apache Spark**: notebooks interativos com Spark (sem cluster EMR)

### Otimizações para Reduzir Custo e Latência

```sql
-- Particionamento (particiona por data → scan apenas partições relevantes)
MSCK REPAIR TABLE vendas; -- atualiza partições no Glue Catalog

-- Colunar (Parquet/ORC escaneiam apenas colunas necessárias)
SELECT amount FROM vendas WHERE year=2024 AND month=01
-- Só escaneia coluna 'amount' nas partições de jan/2024 → 99% menos dados
```

---

## AWS Glue

ETL serverless e catálogo de dados:

| Componente | Função |
|---|---|
| **Glue Data Catalog** | Metastore central: tabelas, schemas, localização S3 |
| **Crawlers** | Descobre e registra schemas automaticamente (S3, RDS, DynamoDB, etc.) |
| **ETL Jobs** | Python/Scala Spark jobs gerenciados (escala automática de DPUs) |
| **Glue Studio** | Interface visual para ETL jobs |
| **DataBrew** | Preparação de dados visual (limpeza, normalização) sem código |
| **Glue Streaming** | ETL de streaming em tempo real (Kinesis, Kafka, S3) |
| **Workflow** | Orquestração de jobs ETL com triggers e dependências |

**DPU (Data Processing Unit)**: 4 vCPUs + 16 GB RAM. Jobs cobrados por DPU-hora.

---

## Amazon EMR (Elastic MapReduce)

Cluster gerenciado para big data com Apache Hadoop, Spark, Hive, Presto, HBase:

| Modo | Descrição |
|---|---|
| **EMR on EC2** | Cluster de instâncias EC2 (Master + Core + Task nodes) |
| **EMR on EKS** | Spark/Presto rodando como pods Kubernetes no EKS |
| **EMR Serverless** | Submete jobs sem gerenciar cluster (auto-scale de workers) |

**Storage options:**
- **HDFS** (in-cluster, efêmero): máximo throughput para processamento
- **EMRFS → S3**: storage persistente; dados sobrevivem ao término do cluster

**Nodes:**
- **Master** (Primary): coordena o cluster (NameNode, YARN ResourceManager)
- **Core**: armazena dados HDFS + processa
- **Task**: apenas processa (sem HDFS); ideal para Spot instances

---

## Amazon OpenSearch Service

Search + analytics engine baseado em Elasticsearch/OpenSearch:
- **Search**: full-text search, autocompletion, faceted search para aplicações
- **Log Analytics**: CloudWatch Logs/Kinesis Firehose → OpenSearch → Kibana/Dashboards
- **Real-time Analytics**: indexação quase em tempo real para dashboards BI

| Deployment | Nodes | HA |
|---|---|---|
| Multi-AZ | Réplicas shards em diferentes AZs | SIM (2-3 AZs) |
| Dedicated Master | Nós master separados dos data nodes | Recomendado para prod |
| UltraWarm | Tier de storage mais barato para dados "mornos" | Read-only; S3-backed |
| Cold Storage | Armazena índices em S3, carrega quando necessário | Ainda mais barato |

---

## Amazon MSK (Managed Streaming for Apache Kafka)

Kafka gerenciado na AWS:
- Cria e gerencia clusters Kafka (broker nodes, zookeeper)
- Compatível com APIs Kafka nativas (producers/consumers existentes têm zero mudanças)
- **MSK Connect** (Kafka Connect gerenciado): conectores source/sink para S3, RDS, OpenSearch, etc.
- **MSK Serverless**: capacidade automática, sem gerenciar brokers
- Retenção de dados configurável; integra com IAM e VPC
- **Quando usar MSK vs Kinesis?** MSK: migração de Kafka existente; ecossistema Kafka (connectors, streams); compliance. Kinesis: AWS-native mais simples, sem Kafka expertise necessária.

---

## Amazon Redshift

Data Warehouse OLAP com MPP (Massively Parallel Processing):
- Clusters: 1 Leader Node + 1-128 Compute Nodes
- **Columnar storage**: cada coluna armazenada separadamente → compressão + scan eficiente
- **Distribution styles**: EVEN, KEY, ALL (para tabelas dimensão pequenas)

| Feature | Descrição |
|---|---|
| Redshift Spectrum | Query S3 como tabelas externas sem COPY |
| Materialized Views | Pré-computação incremental de queries complexas |
| Concurrency Scaling | Auto-adiciona cluster capacity para picos de queries simultâneas |
| AQUA | Hardware-accelerated cache (apenas ra3 nodes) |
| Redshift Serverless | Capacidade automática por query (paga por RPU-hour) |
| Data Sharing | Leitura de dados entre clusters sem copiar (mesmo ou cross-account) |

---

## Amazon QuickSight

BI serverless e dashboards:
- Conecta a: S3 (Athena), Redshift, RDS, OpenSearch, third-party (Salesforce, etc.)
- **SPICE** (Super-fast Parallel In-memory Calculation Engine): cache in-memory para dashboards ultra-rápidos
- **Embedding**: dashboards embarcados em aplicações web com row-level security por usuário
- **ML Insights**: anomaly detection automática, forecasting, narrativas em linguagem natural
- Pagamento por usuário/sessão (sem cluster para provisionar)

---

## AWS Lake Formation

Governança e segurança do Data Lake sobre S3 + Glue Catalog:
- **Fine-grained access control**: permissões por banco/tabela/coluna/row (cell-level security)
- **Blueprint**: automatiza criação de data lake a partir de uma fonte de dados
- **DataFilters**: compartilhar apenas subsets de tabelas entre contas
- **Tagged-based access control (LF-TBAC)**: tags em recursos e identidades para controle de acesso

---

## Pipeline Analytics Típico

```
Dados Brutos (on-prem, RDs, APIs)
  ├── Kinesis Data Firehose → S3 (raw layer)
  ├── DMS → S3 (migração RDS/Oracle)
  └── DataSync → S3 (arquivos on-prem)

S3 (raw/bronze)
  └── Glue Crawlers → Glue Catalog
        └── Glue ETL Jobs (Spark)
              └── S3 (processed/silver/gold - Parquet particionado)
                    ├── Athena (queries ad-hoc, pay-per-scan)
                    ├── Redshift Spectrum (DW queries misturadas)
                    └── QuickSight (dashboards BI via SPICE)

Streaming:
  Kinesis/Kafka → Glue Streaming / Flink (Kinesis Analytics) → OpenSearch (real-time dash)
```

---

## Dicas de Prova

- Athena: **Parquet + particionamento** = combinação padrão para minimizar custo
- Glue Catalog é compartilhado entre Athena, Redshift Spectrum, EMR — metastore único
- **EMR node types**: Master (coordena), Core (HDFS + compute), Task (compute only = Spot ideal)
- MSK não tem retenção máxima; Kinesis tem 365 dias de retenção máxima
- Redshift cluster vs Redshift Serverless: cluster para carga previsível; serverless para carga variável ou intermitente
- OpenSearch: NOT serverless (em cloud) — você gerencia nodes e replication; usa compute instances
- Lake Formation FGA overrides IAM policies for Glue Catalog permissions
- AQUA (Redshift): hardware acceleration para aggregate queries; disponível apenas em ra3 nodes
- QuickSight SPICE: dados importados, cache in-memory; refresh configurável (schedule ou manual)
- Athena Federated Query usa Lambda Data Source Connectors → executa queries em fontes não-S3
