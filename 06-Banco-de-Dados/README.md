# Módulo 06 — Banco de Dados

## Amazon RDS

Serviço gerenciado para bancos de dados relacionais. Suporta MySQL, PostgreSQL, MariaDB, Oracle, SQL Server e Amazon Aurora.

### Multi-AZ
- **Standby** passivo em outra AZ — recebe replicação síncrona via storage-level replication
- Failover automático em 1–2 minutos (DNS atualiza para o standby)
- O standby **não pode ser lido** — não é um read replica
- Backups são feitos no standby (sem impacto na primária)
- Multi-AZ é para **disponibilidade** (HA), não para escalabilidade de leitura

### Read Replicas
- Replicação **assíncrona** da primária — pode haver lag
- Até 15 read replicas (Aurora), 5 (RDS padrão)
- Podem estar em outra região (**Cross-Region Read Replica**)
- Podem ser promovidas a banco independente (quebra replicação)
- **Usadas para:** escalar leitura, analytics sem impacto na primária, migração

### Backups e Snapshots
| Tipo | Retenção | Quando ocorre | Restauração |
|---|---|---|---|
| Automated Backup | 1–35 dias | Janela de manutenção | Ponto no tempo (PITR) |
| Manual Snapshot | Indefinido | Você dispara | Para snapshot específico |

- PITR (Point-in-Time Recovery): restaura para qualquer segundo dentro do período de retenção
- Automated Backup habilitado por padrão; retenção padrão = 7 dias
- Snapshots copiáveis entre regiões para DR

### RDS Proxy
- Pool de conexões entre aplicação e banco — reduz overhead de abertura/fechamento
- Ideal para Lambda (funções criam/destroem conexões rapidamente)
- Suporta failover Multi-AZ mais rápido (menos conexões para renegociar)
- Requer estar na mesma VPC

---

## Amazon Aurora

Banco de dados relacional compatível com MySQL/PostgreSQL, desempenho até **5x MySQL e 3x PostgreSQL** do RDS equivalente.

### Arquitetura de Cluster
```
Writer Endpoint        Reader Endpoint
      │                      │
      ▼                      ▼
  Primary Instance    ← Replica 1
                      ← Replica 2 (todas as réplicas compartilham o mesmo storage)
                      ← Replica N (até 15)

Storage Layer (Shared):
  6 cópias dos dados → 3 AZs (2 cópias por AZ)
  Quorum: 4/6 para escrita, 3/6 para leitura
  Auto-healing: detecta e repara bits corrompidos automaticamente
  Auto-scaling de storage: incrementos de 10 GB, até 128 TB
```

### Aurora vs RDS Multi-AZ
| Aurora | RDS Multi-AZ |
|---|---|
| Réplicas de leitura (até 15) no mesmo cluster | Standby passivo (não legível) |
| Failover em <30 segundos | Failover em 1–2 minutos |
| Storage compartilhado entre instâncias | Replicação de storage separada |
| Aurora Serverless v2 disponível | Não há serverless |

### Aurora Global Database
- **Replicação cross-region** em < 1 segundo (mediana)
- Até 5 regiões secundárias, cada uma com até 16 read replicas
- Promoção de região secundária a primária em < 1 minuto (DR)
- Útil para latência de leitura global e recuperação cross-region

### Aurora Serverless v2
- Escala de 0,5 a 128 ACUs (Aurora Capacity Units) em incrementos finos de 0,5 ACU
- Ideal para workloads variáveis, intermitentes, test/dev
- Cobra por ACU-hora a cada segundo

---

## Amazon DynamoDB

NoSQL key-value e document store totalmente gerenciado. Escala horizontalmente para qualquer throughput.

### Modelo de Dados
- **Partition Key** (obrigatório): determina a partição física — deve ter alta cardinalidade
- **Sort Key** (opcional): permite múltiplos itens por partition key, habilita queries de range
- **Items**: até 400 KB cada
- **Atributos**: tipados (String, Number, Binary, Boolean, Null, List, Map, Set)

### Índices Secundários
| Tipo | Partition Key | Sort Key | Tamanho | Consistência |
|---|---|---|---|---|
| **LSI (Local Secondary Index)** | Mesma da tabela | Diferente | Até 10 GB por partition key | Eventual ou Strong |
| **GSI (Global Secondary Index)** | Diferente | Qualquer | Ilimitado | Eventual apenas |

- LSI: criado apenas durante a criação da tabela, não pode ser adicionado depois
- GSI: pode ser adicionado a qualquer momento, tem RCU/WCU próprias

### Capacity Modes
| Modo | Como paga | Quando usar |
|---|---|---|
| **Provisioned** | RCU + WCU pré-definidos | Tráfego previsível, custo menor |
| **On-Demand** | Por operação lida/escrita | Tráfego imprevisível, paga pelo uso |

- 1 RCU = 1 strongly consistent read de 4 KB / 2 eventually consistent reads de 4 KB
- 1 WCU = 1 write de até 1 KB

### DAX (DynamoDB Accelerator)
- Cache in-memory totalmente gerenciado para DynamoDB
- Latência de **microsegundos** (vs milissegundos do DynamoDB direto)
- Drop-in substituição da API DynamoDB (mesmo endpoint, sem mudança de código)
- Ideal para aplicações com leitura intensiva e dados quentes (hot keys)

### DynamoDB Streams
- Captura mudanças (insert, update, delete) em ordem cronológica por item
- Retenção: 24 horas
- Usa cases: acionar Lambda em mudanças, replicação cross-region customizada, auditoria, computação derivada

### DynamoDB TTL
- Atributo de timestamp (Unix epoch) em cada item
- DynamoDB deleta itens expirados automaticamente (eventualmente, sem custo de WCU)
- Útil para: sessões, carrinho de compra, tokens temporários, dados de telemetria

---

## Amazon ElastiCache

Cache in-memory gerenciado. Dois engines:

### Redis vs Memcached
| Característica | Redis | Memcached |
|---|---|---|
| Estruturas de dados | Strings, Lists, Sets, Sorted Sets, Hashes, HLL | Apenas strings |
| Multi-threading | Single-thread (por padrão, Redis 6+ tem I/O threads) | Multi-thread |
| Persistence | RDB snapshots + AOF | Nenhuma |
| Replication | Réplicas de leitura (cluster mode off/on) | Não |
| Failover automático | Sim (Redis Sentinel/Cluster) | Não |
| Pub/Sub | Sim | Não |
| Lua scripting | Sim | Não |
| **Multi-AZ** | Sim | Não |
| Usar para | Sessões, leaderboards, pub/sub, filas leves, ML features | Cache simples multi-threaded |

### Redis Cluster Mode
- **Cluster Mode Off**: 1 shard, até 5 read replicas, Multi-AZ possível, simples
- **Cluster Mode On**: até 500 shards, distribui dados automaticamente (sharding), escala para PiB de dados

### Caching Patterns
| Padrão | Descrição |
|---|---|
| **Lazy Loading** | Cache miss → busca DB → armazena cache. Dados podem ficar stale |
| **Write-Through** | Escreve no cache e no DB juntos. Cache sempre atual, mas write latency maior |
| **Cache Aside** | App gerencia o cache manualmente |

---

## Amazon Redshift

Data warehouse col-oriented (MPP — Massively Parallel Processing) para analytics de grande escala.

- **Cluster**: Leader node (query planning) + Compute nodes (armazenam e processam)
- **Redshift Spectrum**: executa queries SQL em dados no S3 sem carrega-los no cluster
- **Columnar storage**: compressão excelente, otimizado para agregações
- **Enhanced VPC Routing**: força tráfego COPY/UNLOAD a passar pela VPC (não pela internet)
- **Snapshots**: automáticos (retenção 1–35 dias) ou manuais; cross-region possível
- **Redshift Serverless**: auto-scaling sem gerenciar cluster, paga por RPU-segundo

---

## Bancos Especializados

| Serviço | Tipo | Caso de Uso |
|---|---|---|
| **Amazon DocumentDB** | Document (compatível MongoDB) | Catálogos, CMS, perfis de usuário |
| **Amazon Neptune** | Graph | Redes sociais, detecção de fraude, grafos de conhecimento |
| **Amazon Keyspaces** | Wide column (compatível Cassandra) | IoT, telemetria, time-series de grande escala |
| **Amazon QLDB** | Ledger (imutável, auditável) | Registros financeiros, supply chain, histórico de transações |
| **Amazon Timestream** | Time-series | IoT, DevOps metrics, telemetria de séries temporais |

---

## Como Escolher o Banco Certo (Exame)

```
Dado relacional + esquema fixo + ACID?
  ├── Alta disponibilidade + compatibilidade MySQL/PostgreSQL → Aurora
  └── Oracle/SQL Server (licença específica) → RDS

Dados semi-estruturados (JSON) flexível, escala massiva?
  └── DynamoDB (key-value/document)

Analytics, BI, data warehouse?
  └── Redshift

Cache de alta velocidade?
  ├── Sessão, leaderboard, pub/sub → ElastiCache Redis
  └── Cache simples multi-threaded → ElastiCache Memcached

Grafos (friends of friends, detecção de fraude)?
  └── Neptune

Auditoria imutável (não pode alterar histórico)?
  └── QLDB

IoT / time-series de alta ingestão?
  └── Timestream

MongoDB workload migrado?
  └── DocumentDB
```

---

## Dicas de Prova

- **RDS Multi-AZ ≠ Read Replica**: Multi-AZ é HA síncrona (standby não lê); Read Replica é escala assíncrona (legível)
- **Aurora armazena 6 cópias em 3 AZs**, quorum 4/6 para escrita
- **Aurora Global Database** < 1s replicação cross-region — perfeito para RPO global baixo
- **DynamoDB Partition Key** deve ter alta cardinalidade para distribuição uniforme (hot partition = throttling)
- **GSI** = pode criar a qualquer momento; **LSI** = somente na criação da tabela
- **DAX** = µs latência; apenas para DynamoDB; não ajuda para writes (write-through)
- **ElastiCache Redis** tem Multi-AZ, persistence, replication — use quando esses recursos importam
- **Redshift Spectrum** = query em S3 sem carregar no cluster (serverless query on data lake)
- **QLDB** = ledger imutável — perguntas sobre auditoria de histórico financeiro
- **RDS Proxy** = conexão pool — ideal para Lambda + RDS
