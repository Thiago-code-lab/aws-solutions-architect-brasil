# Flashcards — Módulo 12: Dados e Analytics

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** O que é o Amazon Athena e qual é o modelo de precificação?  
**R:** Serviço serverless de query interativo usando SQL padrão diretamente em arquivos S3. Modelo: **pay-per-query** — $5 por TB de dados varridos. Otimizações para reduzir custo: usar formato colunar (Parquet/ORC) e particionamento (varrem menos dados).

---

**P:** Por que Parquet reduz o custo das queries no Athena?  
**R:** Parquet é **colunar**: Athena lê apenas as colunas necessárias para a query, ignorando as outras. Também tem compressão nativa (Snappy, Gzip). Resultado: escaneia menos dados → menos custos ($5/TB varrido). CSV é row-based — escaneia linha inteira mesmo se a query precise de 2 colunas.

---

**P:** O que é o AWS Glue Data Catalog?  
**R:** Repositório central de metadados (schemas, tabelas, partições) para dados no S3, RDS, Redshift etc. Funciona como metastore para Athena e EMR. Glue Crawlers populam o Catalog automaticamente. Cada data source tem um "database" com "tables" no Catalog.

---

**P:** Qual é a diferença entre Glue ETL Jobs e AWS Glue DataBrew?  
**R:** **Glue ETL Jobs:** código Python/Scala (Spark) para transformações customizadas complexas. **AWS Glue DataBrew:** interface visual de preparação de dados sem código. Mais de 250 transformações pré-construídas (deduplicate, filter, format). Para analistas de dados sem habilidades de programação.

---

**P:** Quais são os 3 tipos de nó em um cluster Amazon EMR?  
**R:** **Master Node:** coordena o cluster (YARN ResourceManager, HDFS NameNode). **Core Node:** processa tasks + armazena dados HDFS (YARN NodeManager). **Task Node:** apenas processa tasks, sem armazenamento HDFS — ideal para Spot Instances (perder Task Node não afeta dados).

---

**P:** Quando usar Amazon MSK em vez de Amazon Kinesis?  
**R:** **MSK:** quando já usa Apache Kafka em on-premises e quer migrar sem mudar produtores/consumidores (fully managed Kafka); quando precisa de protocol Kafka nativo. **Kinesis:** serviço proprietário AWS mais integrado; sem operação de brokers; melhor integração com Lambda, Firehose etc. MSK = lift-and-shift Kafka; Kinesis = nativo AWS.

---

**P:** Qual é a diferença entre Redshift Multi-AZ e Redshift Cross-Region Snapshot?  
**R:** **Multi-AZ:** cluster Redshift com réplicas em múltiplas AZs — HA para failover automático (disponível no RA3). **Cross-Region Snapshot:** backup do cluster copiado para outra região para DR. Snapshots: automáticos (1-35 dias). Para migração/restore em outra região.

---

**P:** O que é Redshift Concurrency Scaling?  
**R:** Feature que adiciona automaticamente clusters transientes quando a demanda de concorrência excede a capacidade do cluster principal. Queries roteadas para os clusters de scaling. Créditos gratuitos: 1 hora por dia de cluster principal ativo. Custo adicional além dos créditos.

---

**P:** O que é o SPICE no Amazon QuickSight?  
**R:** Super-fast Parallel In-memory Calculation Engine: cache in-memory dos dados importados no QuickSight. Dashboards ficam ultra-rápidos pois não precisam fazer query ao banco a cada interação. Quota por usuário (GB). Refresh agendado ou sob demanda para atualizar os dados.

---

**P:** Quais são os casos de uso do Amazon Kinesis Data Firehose?  
**R:** Entrega de streaming data para **destinos**: S3, Redshift (via S3 staging), OpenSearch, Splunk, HTTP endpoints customizados. Características: near-real-time (buffering de 60s a 900s ou 1-128 MB), transformação opcional com Lambda, conversão de formato (JSON→Parquet), compressão. Não requer consumer — entrega automática.

---

**P:** O que é o AWS Lake Formation e qual problema resolve?  
**R:** Simplifica construção de **Data Lakes** seguros no S3. Problema: segurança granular em data lakes é complexa. Solução: Fine-Grained Access Control (FGAC) — controle por coluna, row-level filters, cell-level security. Centraliza permissões em vez de policy individual por bucket/pasta.

---

**P:** Qual é a diferença entre OpenSearch Service e Amazon Kendra?  
**R:** **OpenSearch:** search e analytics engine para logs, métricas, full-text search, visualizações (Kibana/OpenSearch Dashboards). **Kendra:** search engine com IA para documentos corporativos — entende linguagem natural ("quais são as férias?"), busca em PDFs, Word, SharePoint. OpenSearch = técnico/analytics; Kendra = search inteligente de documentos.

---

**P:** O que é Redshift Spectrum?  
**R:** Extensão do Redshift que permite fazer query de dados diretamente no **S3** sem carregar no Redshift. Usa nós de escalabilidade independente (Spectrum layer). Acessa o Glue Data Catalog para metadata. Ideal para: dados históricos/frios no S3 + dados quentes no Redshift em uma única query SQL.

---

**P:** Como funciona o particionamento no Athena e por que é importante?  
**R:** Particionamento: organizar dados no S3 em pastas hierárquicas por campo de alta cardinalidade (ex: `s3://bucket/year=2024/month=01/day=15/`). Na query: `WHERE year=2024 AND month=01` → Athena varrre apenas essas partições. Reduz drasticamente dados escaneados → menor custo e melhor performance.

---

**P:** Quais são as diferenças entre Kinesis Data Streams e Kinesis Data Firehose?  
**R:** **KDS:** streaming em tempo real, consumers customizados (Lambda, apps, Kinesis Analytics), retenção 1-365 dias, shards configuráveis. Requer consumer code. **KDF:** delivery gerenciado para destinos (S3, Redshift, OpenSearch), near-real-time (buffer), sem consumer code, compressão/transformação automática. KDS = flexibilidade; KDF = simplicidade de entrega.

---

**P:** O que é o AWS Glue Streaming ETL?  
**R:** Extensão do Glue para processar dados **em streaming** de Kinesis Data Streams, Kafka/MSK ou SQS. Processa micro-batches continuamente (streaming ETL). Usa Apache Spark Structured Streaming. Alternativa gerenciada ao Apache Flink para transformações de streaming.

---

**P:** Como funciona o Amazon EMR Serverless?  
**R:** Executa workloads Spark/Hive sem gerenciar clusters. Submit um job → AWS provisiona automaticamente os recursos necessários → executa → desaloca. Sem nós para gerenciar. Pay-per-vCPU-second e GB-second. Ideal para workloads intermitentes ou imprevisíveis. Sem cold start de provisionamento manual de cluster.

---

**P:** O que é Amazon OpenSearch UltraWarm e Cold Tiers?  
**R:** Camadas de armazenamento econômicas para dados históricos no OpenSearch: **UltraWarm:** dados menos acessados em S3 (warm tier), mais barato que instâncias hot, ainda queryable em segundos. **Cold:** ainda mais barato, dados arquivados, latência maior para acesso. Migration: índices movidos entre hot → warm → cold conforme ficam mais antigos.

---

**P:** O que é o AWS Glue Workflow?  
**R:** Orquestração de pipelines ETL no Glue. Permite criar grafos de execução com triggers, crawlers e jobs. Trigger types: on-demand, schedule (cron), event-based (completion de outro job). Visualização gráfica no console. Para pipelines ETL complexos multi-step sem Step Functions.

---

**P:** Quando usar Amazon EMR vs AWS Glue para ETL?  
**R:** **Glue:** ETL simples-médio, sem infraestrutura para gerenciar, integração nativa com Catalog/S3/Redshift, ideal para engenheiros não-Spark. **EMR:** workloads Spark/Hive/Presto complexas, controle total sobre cluster config, múltiplos frameworks (HDFS local), workloads HPC, maior flexibilidade e performance para big data.

---

**P:** O que é Amazon Kinesis Data Analytics (agora Amazon Managed Service for Apache Flink)?  
**R:** Serviço gerenciado para aplicações Apache Flink — processa streams em real-time com SQL ou Java/Python/Scala. Input: KDS ou MSK. Output: S3, KDS, KDF, Lambda etc. Use cases: detecção de anomalias em tempo real, aggregações janeladas (sliding/tumbling windows), joins de múltiplos streams.

---

**P:** Qual é o modelo de cobrança do Amazon Athena?  
**R:** $5.00 por TB de dados **escaneados** (não por tempo de query). Sem cobrança por queries que falham. Para reduzir custo: (1) Compressão — dados menores = menos TB; (2) Colunar (Parquet/ORC) — escaneia apenas colunas necessárias; (3) Particionamento — escaneia apenas partições relevantes.

---

**P:** O que é Amazon QuickSight Q?  
**R:** Engine de NLP (Natural Language Processing) do QuickSight: permite fazer perguntas em linguagem natural em português/inglês sobre os dashboards. Ex: "qual foi a receita em janeiro?" → QuickSight interpreta e gera visualização automaticamente. Requer treinamento de topicos com os dados.

---

**P:** O que é o DMS (Database Migration Service) e quando usar SCT?  
**R:** **DMS:** migração de banco de dados com mínima downtime (ongoing replication CDC). Suporta homo e heterogêneo. **SCT (Schema Conversion Tool):** converte schema + código (stored procedures, views) entre engines diferentes (Oracle → PostgreSQL, SQL Server → Aurora). DMS = migra dados; SCT = converte schema incompatível. Homogêneo (MySQL→MySQL): DMS sem SCT. Heterogêneo: DMS + SCT.

---

**P:** O que é Amazon Redshift Serverless?  
**R:** Redshift sem necessidade de provisionar ou gerenciar clusters. AWS automaticamente provisiona e escala capacidade baseado na workload. Pay-per-use (RPU-hours — Redshift Processing Units). Ideal para: workloads intermitentes, ambientes de dev/test, analytics esporádicos. Vs cluster: sem gerenciamento, mas custo pode ser maior para workloads constantes.
