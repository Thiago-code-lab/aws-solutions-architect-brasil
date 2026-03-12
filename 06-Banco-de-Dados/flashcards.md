# Flashcards — Módulo 06: Banco de Dados

---

**P:** Qual é a diferença de replicação entre RDS Multi-AZ e Read Replica?
**R:** Multi-AZ: replicação **síncrona** → zero RPO, failover automático (~1-2 min). Read Replica: replicação **assíncrona** → pequeno lag, para offload de leituras, failover manual (promover a standalone DB)

---

**P:** Quantas cópias os dados Aurora mantém e como?
**R:** 6 cópias em 3 AZs (2 por AZ). Write quorum: 4/6. Read quorum: 3/6. Suporta perda de 1 AZ sem impacto em reads; perda de 2 AZs sem impacto em writes

---

**P:** Qual o RTO/RPO do Aurora Global Database em caso de falha regional?
**R:** RPO < 1 segundo (replicação com lag de ~1s). RTO < 1 minuto (promover região secundária manualmente ou automaticamente via managed failover)

---

**P:** O que é RDS Proxy e quando usar?
**R:** Proxy gerenciado que faz connection pooling entre aplicações e RDS/Aurora. Reutiliza conexões, evita connection exhaustion. Crítico para Lambda (sem estado, abre nova conexão a cada invoke) e ECS (muitas tasks)

---

**P:** Quanto tempo o RDS PITR (Point-In-Time Recovery) permite restaurar?
**R:** Até os últimos 35 dias (configurável de 1 a 35 dias). Restaura para qualquer segundo dentro dessa janela usando backups + transaction logs

---

**P:** Quais são os 2 modos de capacidade do DynamoDB e quando usar cada um?
**R:** Provisioned: você define RCUs e WCUs + Auto Scaling. Use para carga previsível/consistente (mais barato). On-Demand: cobra por request. Use para carga imprevisível, novos workloads, spikes súbitos

---

**P:** O que é 1 RCU e 1 WCU no DynamoDB?
**R:** 1 RCU = 1 read forte consistência de até 4 KB (ou 2 reads eventually consistent). 1 WCU = 1 write de até 1 KB por segundo

---

**P:** Qual é a latência do DAX vs DynamoDB direto?
**R:** DynamoDB: milissegundos de latência. DAX (DynamoDB Accelerator): **microssegundos** (cache in-memory gerenciado, compatível com API DynamoDB)

---

**P:** Qual é a diferença entre LSI e GSI no DynamoDB?
**R:** LSI: mesma partition key, sort key diferente, criado APENAS na criação da tabela, usa throughput da tabela. GSI: partition key diferente, criado a qualquer momento, throughput independente

---

**P:** Por quanto tempo DynamoDB Streams retém os dados?
**R:** 24 horas. Streams captura item-level changes (INSERT, MODIFY, REMOVE) com imagem antes e/ou depois. Integra com Lambda (trigger) e DynamoDB Global Tables (replicação multi-região)

---

**P:** Qual é o padrão de leitura "Lazy Loading" no ElastiCache?
**R:** Cache Miss: busca do banco de dados → escreve no cache. Cache Hit: retorna do cache. Prós: só armazena dados solicitados. Contras: cache miss = 3 viagens (cache, DB, cache) + risco de dado stale

---

**P:** Qual é a diferença principal entre Redis e Memcached no ElastiCache?
**R:** Redis: persistência, replicação, Multi-AZ, Pub/Sub, Sorted Sets, Geospatial, Cluster Mode (sharding). Memcached: multi-threading, sem persistência, sem replicação, simples, pure cache. Para HA e estruturas complexas: Redis

---

**P:** O que é Redshift Enhanced VPC Routing?
**R:** Força o tráfego COPY e UNLOAD do Redshift a usar a VPC (ao invés da internet pública/endpoints públicos S3). Permite controle via VPC Flow Logs, SG, NACLs. Recomendado para segurança e conformidade

---

**P:** Quando usar Amazon Neptune?
**R:** Para dados e queries de grafos: redes sociais (amizades), fraud detection (conexões suspeitas), knowledge graphs, motores de recomendação. Suporta Property Graph (Gremlin, openCypher) e RDF (SPARQL)

---

**P:** Quando usar Amazon QLDB?
**R:** Ledger database com histórico imutável e verificável criptograficamente. Para: registros financeiros, supply chain tracking, audit logs onde nenhum dado pode ser apagado ou modificado retroativamente

---

**P:** Para qual workload Aurora Serverless v2 é ideal?
**R:** Cargas de trabalho com tráfego muito variável (ex: dev/test, aplicações com picos noturnos ou sazonais). Escala de 0,5 a 128 ACUs em incrementos de 0,5 ACU em ~1 segundo. Zero downtime durante escalas

---

**P:** O que é o endpoint de leitura e endpoint de escrita do Aurora Cluster?
**R:** Cluster Endpoint (escrita): aponta para a instância primary writer. Reader Endpoint: load balancer entre todas as Read Replicas. Aplicações devem usar o endpoint correto para separar reads de writes

---

**P:** O que é ElastiCache Cluster Mode (sharding no Redis)?
**R:** Distribui os key slots (partições do keyspace) entre múltiplos node groups (shards). Permite maior throughput total (escrita em paralelo em shards) e maior capacidade de dados do que um único nó. Cada shard tem réplicas para HA

---

**P:** O que é Amazon Keyspaces?
**R:** Apache Cassandra compatível e totalmente gerenciado na AWS. Para migração lift-and-shift de workloads Cassandra sem gerenciar clusters. CQL (Cassandra Query Language) compatível

---

**P:** Qual o uso do Amazon Timestream?
**R:** Time series database serverless. Para IoT data, DevOps metrics, telemetria de application performance. Armazena e analisa trilhões de eventos por dia. Queries com funções de análise temporal built-in
