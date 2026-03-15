# Casos de Uso — Módulo 06: Banco de Dados

## Caso 1: E-commerce com RDS Multi-AZ e Read Replicas

**Cenário:** E-commerce com picos de tráfego sazonais. Catálogo é lido 20x mais do que escrito. Necessidade de failover automático.

**Arquitetura:**
```
                ┌─ RDS PostgreSQL Primary (us-east-1a)
                │    ├── OLTP: pedidos, pagamentos
ALB → App      ─┤    └── Multi-AZ Standby (us-east-1b) ← sync replica
Servers        │
                └─ RDS Read Replica x3 (us-east-1c)
                     └── OLTP leitura: catálogo, pesquisa

ElastiCache Redis ← Cache de sessão + catálogo mais buscado
```

**Configurações-chave:**
- Primary endpoint: escrita + leituras críticas (consistência forte)
- Read Replica endpoint (Reader): leituras do catálogo (consistência eventual ok)
- ElastiCache Redis: cache de 1 hora para páginas de produto
- RDS Multi-AZ: standby em AZ diferente, failover ~1-2 min

---

## Caso 2: Aplicação Serverless com DynamoDB e DAX

**Cenário:** API de leaderboard em tempo real para jogo mobile com 1 milhão de usuários ativos simultâneos.

**Arquitetura:**
```
Mobile App → API Gateway → Lambda
                              ├── DynamoDB Table (Players)
                              │     ├── PK: player_id
                              │     └── GSI: score-index (PK: game_id, SK: score)
                              └── DAX Cluster (cache layer)
                                    └── Latência < 1ms para top players
```

**DynamoDB Design:**
```
Table: GameLeaderboard
  PK: game_id#season
  SK: player_id
  GSI1: score-index (PK: game_id#season, SK: score) → top 100 por game

TTL: season_end_timestamp → apaga dados da temporada anterior automaticamente
```

**Por que DAX?** Lambda cold starts + 1M usuários → DynamoDB teria microsegundos mas DAX garante < 100µs para qualquer query. GSI permite `query` eficiente por ranking sem scan.

---

## Caso 3: Aurora Global Database para Aplicação Multi-Regional

**Cenário:** SaaS global com usuários na América (us-east-1) e Europa (eu-west-1). Regulação europeia exige dados na Europa.

**Arquitetura:**
```
América (us-east-1) — PRIMARY
  └── Aurora PostgreSQL Cluster
        ├── Writer Instance
        ├── Reader Instance x2
        └── US App Servers (escrita + leitura)

                    ↓ Replicação < 1 segundo (Global Database)

Europa (eu-west-1) — SECONDARY
  └── Aurora PostgreSQL Cluster (read-only)
        ├── Reader Instance x2 (leitura local GDPR)
        └── EU App Servers (leitura apenas)

Failover:
  Aurora Managed Planned Failover → promove EU a Primary em < 1 minuto
  RPO < 1s | RTO < 1 minuto
```

---

## Caso 4: Pipeline de Análise com Redshift + Spectrum

**Cenário:** Equipe de BI precisa cruzar dados de vendas históricos no DW com dados fresh de um data lake S3.

**Arquitetura:**
```
Dados Históricos (DW)          Dados Frescos (Data Lake)
  └── Redshift Cluster             └── S3 (Parquet, particionado por data)
        └── tabela: vendas_hist           └── Glue Catalog (schema dos arquivos)

Query unificada (Redshift Spectrum):
SELECT h.regiao, f.produto, SUM(h.receita) + SUM(f.receita) as total
FROM vendas_hist h
JOIN spectrum.vendas_fresh f ON h.produto_id = f.produto_id  ← S3 via Spectrum
WHERE f.data >= CURRENT_DATE - 7
GROUP BY 1, 2
```

**Benefício:** Sem ETL diário para carregar dados S3 no DW. Joins em tempo real entre DW e Data Lake. Redshift MPP distribui o processamento do Spectrum entre os nodes.

---

## Caso 5: Prevenção de Fraude com DynamoDB + ElastiCache

**Cenário:** Sistema bancário precisa verificar se um cart payment está sendo feito de um IP ou dispositivo suspeito em < 10ms.

**Arquitetura:**
```
Pagamento → API
  ├── ElastiCache Redis → Check: ip_blacklist, device_fingerprint (hits < 1ms)
  │         └── Cache Miss → DynamoDB → redis (write-through)
  │
  ├── DynamoDB: fraud_signals_table
  │     PK: signal_type (ip|device|account)
  │     SK: signal_value
  │     Atributos: risk_score, last_seen, blocked_until (TTL)
  │
  └── EventBridge: "fraud_detected" → SNS → Email/Lambda → Block Account
```

**Pattern usado:** Write-through cache (ao detectar novo sinal de fraude, escreve no Redis E DynamoDB simultaneamente). TTL no DynamoDB para expirar sinais antigos automaticamente. DAX poderia substituir Redis+DynamoDB se toda a lógica ficar no DynamoDB.
