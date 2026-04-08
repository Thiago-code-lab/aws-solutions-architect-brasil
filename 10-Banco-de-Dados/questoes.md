# Questões de Prova — Módulo 06: Banco de Dados

<!-- Domínio SAA-C03: Design Resilient Architectures / High-Performing Architectures -->

---

**1.** Uma aplicação web de e-commerce precisa de um banco de dados relacional com failover automático dentro da mesma região sem perda de dados. Qual configuração RDS usar?

- A) RDS com Read Replica em outra AZ
- B) RDS Multi-AZ Deployment
- C) RDS com backup automatizado habilitado
- D) Aurora com Multi-Master

<details>
<summary>Resposta</summary>
**B — RDS Multi-AZ Deployment**
Multi-AZ mantém réplica **síncrona** standby em outra AZ com failover automático (1-2 minutos). Read Replicas são **assíncronas** e servem para leitura, não failover automático sem intervenção. Backup automatizado não oferece failover.
</details>

---

**2.** Qual a diferença crítica entre Multi-AZ e Read Replicas no RDS?

- A) Multi-AZ usa replicação síncrona; Read Replicas usam replicação assíncrona
- B) Multi-AZ é para leitura e escrita; Read Replicas apenas para escrita
- C) Read Replicas garantem zero RPO; Multi-AZ tem RPO de minutos
- D) Multi-AZ pode ser cross-region; Read Replicas são apenas na mesma AZ

<details>
<summary>Resposta</summary>
**A — Correto**
Multi-AZ: síncrono → zero RPO, failover automático, standby não recebe tráfego de leitura. Read Replicas: assíncrono → pode ter lag (pequeno RPO), para offload de leituras, pode ser promovida a standalone DB. Read Replicas podem ser cross-region! Multi-AZ é apenas na mesma região.
</details>

---

**3.** Uma startup usa RDS MySQL com Aurora e percebe spike de latência quando funções Lambda escalam de 0 para 1.000 instâncias simultâneas, esgotando conexões RDS. Qual solução?

- A) Aumentar max_connections no parameter group
- B) Usar RDS Proxy entre as funções Lambda e o banco
- C) Escalar verticalmente para db.r6g.16xlarge
- D) Habilitar Enhanced Monitoring no RDS

<details>
<summary>Resposta</summary>
**B — RDS Proxy**
RDS Proxy gerencia um pool de conexões entre a aplicação (Lambda, ECS, etc.) e o RDS/Aurora, reutilizando conexões e evitando que cada Lambda abra sua própria conexão. É a solução arquitetural para o problema de "connection exhaustion" com Lambda. Aumentar max_connections é paliativo e tem limite.
</details>

---

**4.** Sobre Amazon Aurora, qual afirmação está CORRETA?

- A) Aurora armazena 2 cópias dos dados em 2 AZs (total 2 cópias)
- B) Write quorum requer 4 de 6 cópias; read quorum requer 3 de 6
- C) Aurora Failover para Read Replica demora ~30 minutos
- D) Aurora Serverless v2 não suporta escalonamento automático de capacidade

<details>
<summary>Resposta</summary>
**B — Correto**
Aurora mantém **6 cópias em 3 AZs** (2 por AZ). Write quorum: 4/6 cópias confirmadas. Read quorum: 3/6. Failover para Aurora Read Replica é automático e demora ~30 segundos (não 30 minutos). Aurora Serverless v2 escala de 0,5 a 128 ACUs automaticamente.
</details>

---

**5.** Uma empresa precisa de um banco NoSQL que responda em microssegundos para um catálogo de produtos com milhões de itens. Qual solução?

- A) DynamoDB com DAX
- B) ElastiCache Redis com RDS PostgreSQL
- C) Aurora com Read Replicas
- D) DynamoDB com Strong Consistency habilitado

<details>
<summary>Resposta</summary>
**A — DynamoDB com DAX**
DynamoDB tem latência de milissegundos. DAX (DynamoDB Accelerator) é cache in-memory gerenciado que reduz latência para **microssegundos**. ElastiCache com RDS adiciona complexidade operacional. Strong Consistency no DynamoDB aumenta latência (não reduz).
</details>

---

**6.** Qual modelo de capacidade do DynamoDB cobrar apenas pelo que for efetivamente consumido sem planejamento de capacidade?

- A) Provisioned Capacity Mode com Auto Scaling
- B) On-Demand Capacity Mode
- C) Reserved Capacity
- D) Provisioned com burst capacity

<details>
<summary>Resposta</summary>
**B — On-Demand Capacity Mode**
On-Demand cobra por cada read/write request efetivamente feito. Ideal para cargas imprevisíveis, novos workloads ou uso esporádico. Custa mais por request que Provisioned, mas zero planejamento. Provisioned + Auto Scaling ainda exige configuração de min/max; pode ter throttling se o scaling não acompanhar spikes súbitos.
</details>

---

**7.** Uma aplicação web precisa de sessões de usuário armazenadas em cache com TTL de 30 minutos, compartilhadas entre múltiplas instâncias EC2. Qual serviço?

- A) DynamoDB com TTL
- B) ElastiCache Redis
- C) ElastiCache Memcached
- D) RDS com índice em session_id

<details>
<summary>Resposta</summary>
**B — ElastiCache Redis**
Redis suporta estruturas de dados ricas, persistência, replicação, Multi-AZ e Pub/Sub. Para sessões de usuário com TTL, Redis é ideal. Memcached é mais simples, sem persistência, sem replicação. DynamoDB funciona mas tem latência de milissegundos vs microssegundos do Redis e não está "in-memory" em sentido estrito.
</details>

---

**8.** Uma empresa precisa executar queries SQL analíticas sobre 100 TB de dados de vendas históricas com performance de minutos, não horas. Qual banco de dados AWS usar?

- A) RDS PostgreSQL com índices
- B) Aurora Serverless v2
- C) Amazon Redshift
- D) DynamoDB com GSI

<details>
<summary>Resposta</summary>
**C — Amazon Redshift**
Redshift é OLAP (Online Analytical Processing) com arquitetura MPP (Massively Parallel Processing) para queries analíticas sobre petabytes de dados. RDS/Aurora são OLTP (transações). DynamoDB + GSI é NoSQL, não adequado para SQL analítico ad-hoc em 100 TB.
</details>

---

**9.** Qual a diferença entre LSI e GSI no DynamoDB?

- A) LSI pode ter chave de partição diferente; GSI mantém a mesma chave de partição
- B) LSI deve ser criado na criação da tabela; GSI pode ser criado a qualquer momento
- C) GSI usa RCU/WCU separados; LSI compartilha RCU/WCU da tabela base
- D) B e C estão corretas

<details>
<summary>Resposta</summary>
**D — B e C estão corretas**
LSI: criado **apenas na criação da tabela** (depois não é possível); mesma partition key, sort key diferente; compartilha throughput da tabela. GSI: criado a qualquer momento após a criação; partition key e sort key distintas; tem próprio throughput (WCU/RCU independentes). Ambos permitem queries em atributos não-chave.
</details>

---

**10.** Uma empresa quer usar Aurora e garantir RPO próximo de zero em caso de falha total de uma AWS Region. Qual recurso?

- A) Aurora Multi-AZ com 6 cópias
- B) Aurora Global Database
- C) Aurora com CRR (Cross-Region Read Replica)
- D) RDS Multi-Region

<details>
<summary>Resposta</summary>
**B — Aurora Global Database**
Aurora Global Database replica com lag < 1 segundo para até 5 regiões secundárias. Em caso de falha da região primária, pode promover uma região secundária em < 1 minuto (RTO < 1min, RPO < 1 segundo). CRR RDS/Aurora regular tem lag maior e failover manual mais demorado.
</details>

---

**11.** Uma aplicação legada Java usa JDBC para conectar a um banco Oracle on-premises. A empresa quer migrar para AWS sem reescrever a aplicação. Qual banco escolher?

- A) DynamoDB (NoSQL como substituto)
- B) RDS for Oracle
- C) Aurora PostgreSQL com schema conversion
- D) Amazon Keyspaces

<details>
<summary>Resposta</summary>
**B — RDS for Oracle**
Para migração lift-and-shift de Oracle sem reescrever, RDS for Oracle mantém compatibilidade total com Oracle. Aurora PostgreSQL + AWS SCT (Schema Conversion Tool) é uma opção para heterogeneous migration mas requer esforço de conversão. DynamoDB e Keyspaces são NoSQL — não compatíveis com JDBC Oracle sem reescrita.
</details>

---

**12.** Uma análise de Redshift precisa cruzar dados do data warehouse com dados fresh no S3, sem carregá-los ao DW. Qual recurso usar?

- A) Redshift COPY command
- B) Redshift Spectrum
- C) AWS Glue ETL
- D) Amazon Athena

<details>
<summary>Resposta</summary>
**B — Redshift Spectrum**
Redshift Spectrum permite executar queries SQL diretamente em dados no S3 (Parquet, CSV, JSON, etc.) como se fossem tabelas externas do Redshift, sem carregar os dados. Athena também consulta S3 mas é independente do Redshift — Spectrum une os dois mundos numa única query.
</details>

---

**13.** Qual banco de dados AWS é mais adequado para dados de grafos como redes sociais (amizades, conexões)?

- A) DynamoDB com adjacency list pattern
- B) Amazon Neptune
- C) Amazon DocumentDB
- D) Amazon QLDB

<details>
<summary>Resposta</summary>
**B — Amazon Neptune**
Neptune é graph database gerenciado, com suporte a Property Graph (Gremlin, openCypher) e RDF (SPARQL). Ideal para redes sociais, knowledge graphs, fraud detection, recomendações. DocumentDB é document store (MongoDB-compatible). QLDB é ledger imutável. DynamoDB adjacency list funciona mas é subótimo para traversals complexos.
</details>

---

**14.** Uma aplicação armazena sessões de usuário no DynamoDB. Sessões expiram após 24 horas e precisam ser automaticamente removidas. Qual recurso usar?

- A) Lambda com CloudWatch Events removendo itens diariamente
- B) DynamoDB Streams com Lambda de cleanup
- C) TTL (Time To Live) no DynamoDB
- D) S3 Lifecycle Policy integrada ao DynamoDB

<details>
<summary>Resposta</summary>
**C — TTL (Time To Live)**
DynamoDB TTL permite associar um atributo de timestamp a um item; quando o timestamp expira, DynamoDB automaticamente remove o item sem custo de WCU. Remoção ocorre em janela de 48h após expiração (não imediata, mas eficiente e sem custo). Lambda + CloudWatch é overengineering.
</details>

---

**15.** Qual é o RTO típico de um RDS Multi-AZ failover?

- A) Instantâneo (0 segundos) — automaticamente redireciona DNS
- B) 1 a 2 minutos — troca DNS para standby
- C) 5 a 10 minutos — standby precisa re-sincronizar logs
- D) 15 a 30 minutos — nova instância de DB é provisionada

<details>
<summary>Resposta</summary>
**B — 1 a 2 minutos**
RDS Multi-AZ failover troca o DNS CNAME para apontar para o standby em ~60-120 segundos. A aplicação reconecta ao mesmo endpoint DNS. Standby já está sincronizado (síncrono), então não há re-sincronização. Zero perda de dados com RPO ≈ 0. Aplicações devem usar JDBC connection retry para absorver o breve downtime.
</details>
