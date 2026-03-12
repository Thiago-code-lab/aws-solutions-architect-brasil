# Cheatsheet — Módulo 06: Banco de Dados

## Comparativo RDS Multi-AZ vs Read Replica

| | Multi-AZ | Read Replica |
|---|---|---|
| Propósito | Alta disponibilidade / DR | Escalabilidade de leitura |
| Replicação | Síncrona | Assíncrona |
| RPO | ~0 (sem perda de dados) | Pequeno lag possível |
| RTO | ~1-2 minutos (DNS flip) | Manual (promoção a standalone) |
| Pode receber leituras? | NÃO (standby apenas) | SIM (apenas leitura) |
| Cross-region? | Não (mesma região) | Sim (até 5 por DB) |
| Failover | Automático | Manual |

## Aurora vs RDS

| | Aurora | RDS |
|---|---|---|
| Cópias | 6 (em 3 AZs) | 2 (Multi-AZ: primário + standby) |
| Write quorum | 4/6 | 1 (primary síncrona para standby) |
| Read Replicas | Até 15 | Até 5 |
| Failover RR | ~30 segundos | Promote manual |
| Global Database | Sim (< 1s lag) | Não (apenas DX/VPN manual) |
| Serverless | v2 (0,5-128 ACU) | Não (exceto Aurora) |
| Performance | 3x MySQL / 5x PostgreSQL | Padrão |

## DynamoDB — Valores-Chave

| Parâmetro | Valor |
|---|---|
| 1 RCU (Eventually Consistent) | 2 reads de 4 KB/s |
| 1 RCU (Strongly Consistent) | 1 read de 4 KB/s |
| 1 WCU | 1 write de 1 KB/s |
| DAX latência | Microssegundos |
| TTL | Gratuito; deleção eventual dentro de 48h |
| Streams retenção | 24 horas |
| Global Tables | Replicação multi-região ativa |

## DynamoDB Indexes

| | LSI | GSI |
|---|---|---|
| Quando criar | Apenas na criação da tabela | Qualquer momento |
| Partition Key | Mesma da tabela | Diferente |
| Sort Key | Diferente da tabela | Qualquer atributo |
| Throughput | Compartilha com a tabela | Independente |
| Consistência | Strong ou Eventually | Apenas Eventually |

## ElastiCache Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| Persistência | Sim (RDB, AOF) | Não |
| Replicação | Sim (Replica) | Não |
| Multi-AZ | Sim | Não |
| Cluster Mode | Sim (sharding) | Sim (multi-thread) |
| Estruturas | String, Hash, List, Set, Sorted Set, Geo, Streams | String apenas |
| Pub/Sub | Sim | Não |
| Caso de uso | Sessions, leaderboards, cache rico, queues | Cache simples, alta velocidade multi-thread |

## Caching Patterns

| Pattern | Cache Miss | Cache Hit | Prós | Contras |
|---|---|---|---|---|
| Lazy Loading | Busca DB → escreve cache | Retorna cache | Só armazena o necessário | 3 round-trips no miss; dados podem ficar stale |
| Write-Through | Escreve DB + cache | Retorna cache | Dados sempre frescos | Write penalty; caches de dados nunca lidos |
| Write-Behind | Escreve cache → async DB | Retorna cache | Menor latência de write | Risco de perda se cache falhar antes de persistir |

## Escolha de Banco de Dados

| Necessidade | Banco |
|---|---|
| Relacional OLTP (latência baixa) | RDS MySQL/PostgreSQL/Aurora |
| Relacional OLTP (alta performance) | Aurora |
| Relacional OLAP / Data Warehouse | Redshift |
| NoSQL key-value / documentos | DynamoDB |
| Cache in-memory | ElastiCache Redis |
| Grafos | Amazon Neptune |
| Documentos (MongoDB) | Amazon DocumentDB |
| Time series (IoT, metrics) | Amazon Timestream |
| Ledger imutável | Amazon QLDB |
| Cassandra gerenciado | Amazon Keyspaces |

## Dicas Rápidas de Prova
- RDS Multi-AZ endpoint não muda após failover (mesmo DNS CNAME)
- Aurora: 2 endpoints principais — Cluster Endpoint (write) + Reader Endpoint (read LB)
- DynamoDB Global Tables requer On-Demand ou ativa replicação via replica write capacity
- Redshift Spectrum: query dados S3 diretamente do Redshift sem COPY
- RDS Proxy: obrigatório quando Lambda acessa RDS (connection pooling, IAM auth, Secrets Manager)
- ElastiCache Redis Cluster Mode: distribui keyspace em múltiplos shards → maior capacity e write throughput
- Aurora Serverless v2: escala em ~1 segundo (não requer reinicialização como v1)
- PITR RDS retention: 1-35 dias; Aurora: até 35 dias
