# Cheatsheet — Módulo 12: Dados e Analytics

## Comparativo Rápido dos Serviços de Analytics

| Serviço | Tipo | Modelo | Melhor Para |
|---|---|---|---|
| **Athena** | Query interativa | Serverless, pay-per-TB | SQL ad-hoc em S3 |
| **Glue** | ETL | Serverless | Transformações ETL, Catalog |
| **EMR** | Big Data Cluster | EC2 gerenciado | Spark/Hive/Presto customizado |
| **Redshift** | Data Warehouse | Cluster ou Serverless | OLAP, BI, SQL analítico |
| **OpenSearch** | Search/Analytics | Cluster | Full-text search, logs, dashboards |
| **MSK** | Kafka gerenciado | Cluster | Streaming Kafka nativo |
| **Kinesis Data Streams** | Streaming | Shards | Streaming real-time customizável |
| **Kinesis Firehose** | Delivery streaming | Serverless | Entrega automática para S3/Redshift |
| **QuickSight** | BI/Visualização | SaaS | Dashboards, SPICE |

---

## Athena — Otimizações de Custo e Performance

| Técnica | Impacto | Implementação |
|---|---|---|
| **Formato colunar** (Parquet/ORC) | Reduz até 90% dados escaneados | Converter CSV → Parquet com Glue |
| **Compressão** (Snappy, GZIP) | Reduz tamanho dos arquivos | Configurar no job ETL |
| **Particionamento** | Escaneia apenas partições relevantes | `year=2024/month=01/day=15/` |
| **Bucketing** | Reduz data shuffle no join | `CLUSTERED BY (user_id) INTO 64 BUCKETS` |
| **CTAS** (Create Table As Select) | Transforma in-place para Parquet | `CREATE TABLE ... WITH (format='PARQUET')` |

**Preço:** $5.00 por TB escaneado. Free: DDL queries, queries falhas, queries com partições (exceto data escaneada).

---

## Kinesis Data Streams vs Kinesis Firehose

| Característica | KDS | KDF |
|---|---|---|
| Latência | Milissegundos (real-time) | Near real-time (60s-900s buffer) |
| Consumers | Customizados (Lambda, App, KDA) | Automático (S3, Redshift, OpenSearch) |
| Retação dos dados | 1-365 dias | Não há (entrega e descarta) |
| Reprocessamento | Sim (cursor de múltiplos consumers) | Não |
| Throughput | 1 MB/s por shard (write), 2 MB/s (read) | Automático |
| Code necessário | Sim (consumer code) | Não (configuração) |
| Transformação | Não nativo | Lambda opcional |

---

## Glue — Componentes

| Componente | Função |
|---|---|
| **Data Catalog** | Repositório central de schemas/metadados |
| **Crawler** | Descobre schemas e popula o Catalog automaticamente |
| **ETL Job** | Código Spark (Python/Scala) para transformação |
| **DataBrew** | Interface visual de transformação sem código |
| **Studio** | IDE visual para jobs Glue |
| **Workflow** | Orquestração de crawlers + jobs + events |
| **Streaming ETL** | ETL contínuo de Kinesis/Kafka |

---

## Redshift — Distribuição e Sort Keys

| Tipo | Quando Usar |
|---|---|
| DISTKEY: campo de join | Colocaliza linhas relacionadas nos mesmos nós |
| DISTKEY: ALL | Replica toda a tabela em cada nó (tabelas pequenas de dimensão) |
| DISTKEY: EVEN | Distribuição round-robin (sem join frequente) |
| SORTKEY: campo de filtro/range | Organiza fisicamente linhas — range scan mais eficiente |

---

## Redshift Serverless vs Provisionado

| | Serverless | Provisionado |
|---|---|---|
| **Gerenciamento** | Zero | Gerenciar cluster, resize |
| **Escalabilidade** | Automática | Manual ou Elastic Resize |
| **Custo** | RPU-hours (pay-per-use) | Hora do nó (independente de uso) |
| **Melhor para** | Intermitente, dev/test | Workloads previsíveis e contínuas |
| **Concurrency Scaling** | N/A | Clusters transientes automáticos |

---

## EMR — Tipos de Nó e Instâncias Recomendadas

| Nó | Função | Instância Recomendada | Spot OK? |
|---|---|---|---|
| **Master** | Coordena cluster (YARN RM, HDFS NN) | On-Demand ou Reserved | Não (SPOF) |
| **Core** | Processa + armazena HDFS | On-Demand ou Reserved | Não (perder dados HDFS) |
| **Task** | Apenas processa (sem HDFS) | **Spot** (ideal) | **Sim** — pode ser interrompido sem perder dados |

---

## Lake Formation — Controles de Acesso

| Nível | Exemplo |
|---|---|
| **Database** | Lista/describe tabelas |
| **Table** | Select de todas colunas |
| **Column** | Select apenas das colunas autorizadas |
| **Row** | Filtros de linhas (Row-level filters) |
| **Cell** | Combinação de coluna + linha |

Fine-Grained Access Control (FGAC) substitui gerenciamento de S3 bucket policies para data lakes.

---

## Dicas de Prova — Padrões Comuns

| Pista no Enunciado | Resposta Provável |
|---|---|
| "SQL ad-hoc em S3" | Athena |
| "Reduzir custo Athena" | Parquet + Particionamento |
| "ETL visual sem código" | AWS Glue DataBrew |
| "Schema discovery automático" | Glue Crawler → Glue Data Catalog |
| "Migrar Kafka para AWS" | Amazon MSK |
| "Entrega automática de stream para S3" | Kinesis Data Firehose |
| "Real-time com consumer customizado" | Kinesis Data Streams |
| "BI self-service para não técnicos" | QuickSight (SPICE) |
| "Full-text search + dashboards Kibana" | OpenSearch Service |
| "Dados históricos no S3 + query SQL" | Redshift Spectrum ou Athena |
| "Segurança granular colunar em data lake" | Lake Formation FGAC |
| "Stream processing com Apache Flink" | MSF (Managed Apache Flink) |
| "Oracle → Aurora de forma heterogênea" | SCT (schema) + DMS (dados) |
