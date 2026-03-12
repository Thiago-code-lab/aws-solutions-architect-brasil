# Questões — Módulo 12: Dados e Analytics

> **Domínio SAA-C03**: Design de Arquiteturas de Alta Performance  
> **Dificuldade**: Média

---

**1.** Uma empresa tem dados brutos em S3 em formato JSON (10 TB). Analistas fazem queries frequentes apenas em 3 das 50 colunas disponíveis. Qual mudança teria o maior impacto no custo e performance das queries Athena?

- A) Mover os dados para o Redshift
- B) Converter os arquivos JSON para formato Parquet e particionar por data
- C) Usar S3 Intelligent-Tiering para reduzir custo de storage
- D) Habilitar S3 Transfer Acceleration

<details><summary>Resposta</summary>

**B** — Parquet é colunar: Athena lê apenas as 3 colunas necessárias (vs todas as 50 em JSON). Particionamento evita full table scan. Combinados: redução de custo e tempo de query de 80-95%. Menor custo por TB escaneado + queries muito mais rápidas.

</details>

---

**2.** Um time de Data Science precisa fazer ETL sobre 100 GB de dados diários no S3, transformando JSONs em Parquet e aplicando limpeza de dados. Qual serviço AWS é mais adequado para isso sem gerenciar servidores?

- A) AWS Glue (ETL jobs serverless com Spark gerenciado)
- B) Amazon EMR com cluster permanente
- C) Lambda para processamento arquivo a arquivo
- D) AWS Batch com containers

<details><summary>Resposta</summary>

**A** — Glue ETL jobs: Spark serverless, escala automaticamente, paga por DPU-hora apenas enquanto o job roda. EMR requer gerenciamento de cluster. Lambda tem limite de 15 minutos e 10 GB de memória (insuficiente para 100 GB de dados). Glue é o padrão para ETL serverless na AWS.

</details>

---

**3.** Uma equipe de analytics quer criar um catálogo centralizado dos datasets armazenados em S3, tornando-os pesquisáveis via Athena sem configurar manualmente cada tabela. Qual serviço resolver isso automaticamente?

- A) Amazon Macie
- B) AWS Glue Crawlers + Glue Data Catalog
- C) AWS Lake Formation
- D) Amazon OpenSearch

<details><summary>Resposta</summary>

**B** — Glue Crawlers: varrem o S3, inferem schemas automaticamente e criam/atualizam tabelas no Glue Data Catalog. Athena usa o Glue Data Catalog como metastore. In minutos, todos os datasets S3 ficam consultáveis via Athena.

</details>

---

**4.** Uma empresa opera um cluster Kafka on-premises e quer migrar para AWS sem reescrever produtores e consumidores. Qual serviço AWS usar?

- A) Amazon Kinesis Data Streams
- B) Amazon SQS FIFO
- C) Amazon MSK (Managed Streaming for Apache Kafka)
- D) Amazon EventBridge

<details><summary>Resposta</summary>

**C** — MSK é Kafka gerenciado: compatível com APIs Kafka nativas. Produtores e consumidores existentes continuam funcionando sem modificações. Kinesis tem APIs próprias (incompatível com Kafka).

</details>

---

**5.** Uma empresa tem um cluster Redshift e as queries de analistas estão degradando a performance de outros usuários. Qual feature do Redshift resolve isso com o mínimo de configuração?

- A) Aumentar o tamanho do cluster para ra3.16xlarge
- B) Habilitar Concurrency Scaling para adicionar automaticamente capacidade durante picos de queries
- C) Migrar para Redshift Serverless
- D) Criar read replicas do Redshift

<details><summary>Resposta</summary>

**B** — Concurrency Scaling: quando múltiplos usuários submetem queries simultâneas e a fila aumenta, o Redshift automaticamente provisiona clusters adicionais para processar queries extras. Os primeiros 60 minutos por dia de scaling são gratuitos. Não há read replicas no Redshift (diferente do RDS).

</details>

---

**6.** Um arquiteto precisa criar um pipeline que entregue eventos de clickstream de uma web app para S3 em formato Parquet para análise posterior com Athena. Qual é a arquitetura mais simples?

- A) Kinesis Data Streams → Lambda → S3 (convertendo para Parquet na Lambda)
- B) Kinesis Data Firehose com transformação de format para Parquet (via Glue Data Catalog) → S3
- C) API Gateway → Lambda → Glue Job → S3
- D) MSK → Glue Streaming → S3

<details><summary>Resposta</summary>

**B** — Kinesis Firehose suporta conversão de formato nativa para Parquet/ORC usando o Glue Data Catalog (define o schema). Configuração sem código, auto-escala, compressão automática e entrega para S3 com bufferização. Solução mais simples e gerenciada.

</details>

---

**7.** Uma empresa precisa que analistas de negócio construam dashboards interativos sobre dados em S3 sem mover dados para outro sistema. Qual é a melhor solução?

- A) Redshift + QuickSight
- B) Athena + QuickSight (com SPICE para dashboards interativos)
- C) OpenSearch + Kibana
- D) EMR + Jupyter Notebooks

<details><summary>Resposta</summary>

**B** — Athena consulta S3 diretamente (sem mover dados); QuickSight conecta ao Athena e pode importar os resultados para o SPICE (cache in-memory) para dashboards ultra-rápidos. Sem infraestrutura para gerenciar. Redshift (A) precisaria mover os dados.

</details>

---

**8.** Uma empresa tem um Elasticsearch cluster self-managed rodando em EC2 para log analytics. Querem migrar para um serviço gerenciado mantendo as APIs e dashboards Kibana existentes. Qual serviço usar?

- A) Amazon Kinesis Data Analytics
- B) Amazon OpenSearch Service (com suporte a OpenSearch Dashboards/Kibana)
- C) Amazon Athena
- D) AWS Glue

<details><summary>Resposta</summary>

**B** — Amazon OpenSearch Service é o sucessor gerenciado do Elasticsearch Service. Compatível com APIs Elasticsearch e Kibana (via OpenSearch Dashboards). Migração de clusters Elasticsearch auto-gerenciados para OpenSearch Service é suportada.

</details>

---

**9.** Uma equipe de dados precisa controlar quais usuários podem acessar quais tabelas e colunas no data lake (S3 + Glue Catalog). Qual serviço implementa permissões a nível de coluna?

- A) S3 Bucket Policies
- B) Glue Data Catalog com IAM policies
- C) AWS Lake Formation com column-level security
- D) Amazon Macie

<details><summary>Resposta</summary>

**C** — Lake Formation: gerencia permissões de data lake com granularidade de banco/tabela/coluna/row (Cell-Level Security). Integra com Athena, Redshift Spectrum e EMR. S3 Bucket Policies e IAM policies não operam no nível de colunas de uma tabela.

</details>

---

**10.** Uma empresa armazena dados históricos em S3 mas precisa executar queries SQL ad-hoc sobre esses dados combinados com dados "quentes" em um cluster Redshift. Qual feature do Redshift usar?

- A) Redshift Data Sharing
- B) Redshift Spectrum para query dados S3 como tabelas externas
- C) Migrar todos os dados históricos para Redshift
- D) Usar Athena para todos os dados e abandonar o Redshift

<details><summary>Resposta</summary>

**B** — Redshift Spectrum: cria external tables no Redshift apontando para dados S3 (via Glue Catalog). Queries que referenciam external tables são distribuídas para thousands de nodes Spectrum no S3 — sem mover dados. Combina dados quentes (Redshift) e frios (S3) em um único SQL.

</details>

---

**11.** Um time de ML precisa armazenar features calculadas para uso tanto em treinamento (batch) quanto em inferência em tempo real (online) sem inconsistências. Qual serviço AWS oferece isso?

- A) S3 para armazenar CSVs de features
- B) SageMaker Feature Store com offline store (S3) e online store (baixa latência)
- C) DynamoDB para features online e S3 para features offline separados
- D) ElastiCache Redis para todas as features

<details><summary>Resposta</summary>

**B** — SageMaker Feature Store: mantém offline store (S3, para treino) e online store (baixa latência, para inferência em tempo real) sincronizados. Garante consistência: a mesma feature definition é usada em treino e produção, evitando training/serving skew.

</details>

---

**12.** Uma empresa processa grandes volumes de dados com Apache Spark no EMR. Para reduzir custos, querem usar Spot Instances para os nós de processamento. Qual tipo de node deve usar Spot?

- A) Master node
- B) Core nodes
- C) Task nodes
- D) B e C

<details><summary>Resposta</summary>

**C** — Task nodes são ideais para Spot: processam dados mas não armazenam dados HDFS. Se forem interrompidos, o job pode redistribuir o trabalho. Core nodes armazenam dados HDFS — se forem perdidos (Spot interruption), dados são perdidos. Master node = nunca Spot (coordena o cluster inteiro).

</details>

---

**13.** Uma empresa quer consultar dados de um banco de dados DynamoDB e de um bucket S3 em uma única query SQL usando Athena. Qual feature usar?

- A) DynamoDB Streams exportados para S3 antes das queries
- B) Athena Federated Query com Lambda Data Source Connector para DynamoDB
- C) Glue ETL para mover dados DynamoDB para S3 antes das queries
- D) AWS Glue DataBrew para unificar as fontes

<details><summary>Resposta</summary>

**B** — Athena Federated Query: usa Lambda connectors para consultar fontes não-S3 (DynamoDB, RDS, Redshift, CloudWatch, etc.) diretamente no SQL. O conector DynamoDB traduz consultas Athena para DynamoDB API. Queries entre S3 e DynamoDB em um único SQL.

</details>

---

**14.** O time de analytics precisa detectar anomalias em métricas de vendas em tempo real (ex: queda súbita de conversão). Qual combinação de serviços implementa isso?

- A) Kinesis Data Streams → Kinesis Data Analytics (Apache Flink) → SNS para alertas
- B) S3 + Athena com queries agendadas por Lambda → SNS
- C) CloudWatch Metrics com Anomaly Detection → CloudWatch Alarm → SNS
- D) A e C são válidas dependendo da fonte de dados

<details><summary>Resposta</summary>

**D** — Kinesis Analytics (A): para dados já em streaming (clickstream, apps). CloudWatch Anomaly Detection (C): para métricas de aplicações e infraestrutura. A escolha depende da fonte: se os dados de vendas vêm de uma stream, use Kinesis Analytics. Se são métricas do sistema, use CloudWatch.

</details>

---

**15.** Uma empresa precisa migrar seu data warehouse Oracle on-prem para Amazon Redshift. O schema Oracle usa stored procedures e tem estrutura de tabelas diferente do Redshift. Qual é o processo correto?

- A) DMS direto do Oracle para Redshift (suportado nativamente)
- B) SCT (Schema Conversion Tool) para converter schema + procedures, depois DMS para migrar dados
- C) Export CSV do Oracle → S3 → Redshift COPY command
- D) EMR para transformar os dados antes de carregar no Redshift

<details><summary>Resposta</summary>

**B** — Migração heterogênea Oracle → Redshift: SCT primeiro para converter DDL, stored procedures e código proprietário Oracle para SQL compatível com Redshift (identifica também o que não pode ser convertido automaticamente). Depois DMS para mover os dados (com suporte a full load e CDC para zero-downtime).

</details>
