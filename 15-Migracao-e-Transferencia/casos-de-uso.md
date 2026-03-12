# Casos de Uso Reais — Migração e Transferência (Módulo 15)

## Caso 1 — Lift-and-Shift de Data Center Corporativo

**Contexto:** Empresa manufatureira com 200 servidores on-premises (Windows Server + SQL Server, Linux + Oracle) precisa migrar para AWS em 12 meses. Deseja mínimo de retrabalho inicial (lift-and-shift) com modernização posterior.

**Requisitos:**
- Janela de cutover < 2 horas por servidor
- Zero perda de dados durante migração
- Replicação contínua antes do cutover
- Plano de rollback em caso de falha

**Arquitetura de Migração com MGN:**
```
FASE 1 — DESCOBERTA (semanas 1-4):
AWS Application Discovery Service (discovery agent nos servidores)
├── Mapeia dependências entre apps (quais servidores se comunicam)
├── Coleta: CPU, memória, disco, tráfego de rede
└── Migration Hub → agrupa em waves de migração

FASE 2 — REPLICAÇÃO (semanas 5-20):
On-premises → AWS MGN (Application Migration Service)
        AWS Replication Agent instalado em cada servidor
                │ Replicação contínua por TCP 1500
                ▼
        AWS Staging Area (instâncias replication servers)
                │
                ▼
        EBS Volumes (cópia dos discos on-premises)

FASE 3 — TEST CUTOVER (semanas 21-23):
MGN → Launch Test Instance (launch template configurado)
├── Validar app funciona na AWS
├── Testar conectividade com outros serviços
└── Verificar performance (mesma ou melhor)

FASE 4 — CUTOVER PRODUCTION:
├── Congelar tráfego on-premises (manutenção 30min)
├── MGN finaliza replicação (delta sync)
├── Launch production instances
├── Atualizar DNS (Route 53 → novos IPs)
└── Monitorar 48h antes de descomissionar on-premises
```

**Plano de Rollback:**
- MGN mantém instâncias de staging por 90 dias após cutover
- DNS pode ser revertido em < 5 minutos (Route 53 TTL baixo)
- On-premises não descomissiona até validação completa

---

## Caso 2 — Migração de Banco PostgreSQL para Aurora com DMS

**Contexto:** FinTech com banco PostgreSQL on-premises (500 GB, 100 transações/segundo) precisa migrar para Amazon Aurora PostgreSQL sem interromper operações. Sistema processa pagamentos 24/7.

**Requisitos:**
- Zero downtime durante migração
- Janela de cutover < 10 minutos
- Validar integridade dos dados após migração
- Capacidade de rollback em 15 minutos

**Arquitetura com DMS (Change Data Capture):**
```
FASE 1 — CARGA COMPLETA:
PostgreSQL On-Prem ──── DMS Replication Instance ──── Aurora PostgreSQL
   (source endpoint)          (r5.xlarge)              (target endpoint)
                         Full Load Task
                         ├── Copia todas as tabelas
                         ├── Desabilita FK constraints (temporário)
                         └── Cria índices após carga (mais rápido)
                         
Duração estimada: 4-6 horas (500GB)

FASE 2 — REPLICAÇÃO CDC (enquanto Full Load ainda roda):
DMS → Change Data Capture (captura INSERT/UPDATE/DELETE)
     └── Aplica mudanças em Aurora em tempo real
     └── Latência replicação: < 1 segundo
     
CUTOVER (quando lag replication ≈ 0):
1. Aplicação: modo read-only (ou maintenance page 5min)
2. DMS: verificar lag = 0 (sem dados pendentes)
3. Alterar connection string da app → Aurora endpoint
4. AWS SCT validou: mesma estrutura + checksums iguais
5. Retomar operação normal
6. Monitorar 24h = rollback disponível (DMS ainda ativo no sentido inverso)

ROLLBACK:
DMS task reversa (Aurora → PostgreSQL on-prem) pré-configurada
Alteração connection string de volta em < 5 minutos
```

**Diferença Full Load vs CDC:**
| Modo | O que faz | Quando usar |
|------|-----------|-------------|
| Full Load | Cópia completa única | Tráfego pode ser interrompido |
| CDC | Replica mudanças incrementais | Migração zero-downtime |
| Full Load + CDC | Cópia + acompanha mudanças | **Padrão recomendado** |

---

## Caso 3 — Transferência de 500 TB com AWS Snowball Edge

**Contexto:** Empresa de petróleo com base remota (sem fibra, link satelital de 100 Mbps) precisa transferir 500 TB de dados sísmicos históricos para S3 e depois processá-los com EMR.

**Requisitos:**
- Transferência em < 30 dias
- Dados criptografados durante transporte
- Sem dependência de link de internet (link lento)
- Processamento de subset dos dados na borda antes do envio

**Cálculo de Viabilidade:**
```
500 TB via internet (100 Mbps):
  500 TB = 500 × 1024 × 8 Gbits = 4.096.000 Gbits
  @ 100 Mbps = 0.1 Gbps
  Tempo = 4.096.000 / 0.1 / 3600 / 24 = ~474 dias
  
Snowball Edge Storage Optimized (80 TB utilizável):
  500 TB / 80 TB = 7 dispositivos
  Empacotamento + envio + ingestão: ~3 semanas
  ✓ MUITO MAIS RÁPIDO
```

**Processo:**
```
AWS Console → Solicitar 7 Snowball Edge Storage Optimized
                    │ Entrega em 2-5 dias úteis
                    ▼
Base Remota:
  ├── Conectar Snowball à rede local (10 GbE)
  ├── Snowball Client → transferir dados para dispositivo
  ├── (opcional) Snowball Edge compute: Lambda local para pré-processamento
  └── Criptografia automática (AES-256, chave no KMS)
  
Envio para AWS (transportadora)
                    │
                    ▼
AWS Data Center → Ingestão automática em S3
                    │
                    ▼
S3 (dados sísmicos) → EMR (processamento Spark)
                    → Athena (análise ad hoc)
                    
Notificação: SNS email quando ingestão completa
Verificação: MD5 checksum automático pelo serviço
```

---

## Caso 4 — Migração Heterogênea Oracle → Aurora PostgreSQL

**Contexto:** Empresa de RH deseja eliminar licença Oracle ($500K/ano) migrando para Aurora PostgreSQL. Banco tem 2 TB com stored procedures, packages PL/SQL, e funções Oracle-specific.

**Requisitos:**
- Converter código PL/SQL para PL/pgSQL (incompatível nativamente)
- Identificar incompatibilidades antes de começar
- Migrar dados com mínimo downtime
- Documentar todas as conversões para auditoria

**Processo com SCT + DMS:**
```
FASE 1 — ASSESSMENT:
AWS SCT (Schema Conversion Tool)
├── Conecta ao Oracle on-premises
├── Gera relatório de compatibilidade:
│   ├── 70% código convertido automaticamente ✓
│   ├── 20% requer ajuste manual (lógica Oracle-specific)
│   └── 10% precisa ser reescrito (funcionalidades sem equivalente)
└── Prioridade: corrigir os 30% manuais antes de continuar

FASE 2 — CONVERSÃO:
SCT converte automaticamente:
├── Tabelas, índices, views → CREATE TABLE/INDEX/VIEW
├── PL/SQL básico → PL/pgSQL
└── Sequences → Sequences PostgreSQL

Time de DBA corrige manualmente:
├── CONNECT BY (hierarquia) → Recursive CTE
├── ROWNUM → ROW_NUMBER() OVER()
├── Oracle packages → PostgreSQL schemas + functions
└── Implicit cursors → Explicit cursors PgSQL

FASE 3 — MIGRAÇÃO DADOS:
DMS Full Load + CDC (igual caso 2, mas fonte Oracle → Aurora PG)
Diferença: SCT deve ser aplicado antes do DMS

FASE 4 — VALIDAÇÃO:
AmazonDMS data validation (hashes de tabelas comparados automaticamente)
Testes de regressão (suite de queries críticas devem retornar mesmo resultado)
```

---

## Caso 5 — Transfer Family para Integração B2B com Parceiros

**Contexto:** Empresa de logística precisa receber arquivos de 50 parceiros via SFTP (padrão legado do mercado) e processá-los automaticamente. Atualmente mantém 2 servidores SFTP on-premises.

**Requisitos:**
- Manter protocolo SFTP (parceiros não mudam)
- Processar arquivos automaticamente ao chegar
- Cada parceiro vê apenas sua pasta
- Alta disponibilidade (sem manutenção de servidores)

**Arquitetura:**
```
Parceiros (50 empresas)
    │ SFTP (porta 22)
    ▼
AWS Transfer Family (SFTP endpoint gerenciado)
├── DNS personalizado: sftp.empresa.com.br (Route 53)
├── Autenticação: AWS Secrets Manager (chave SSH por parceiro)
└── Storage: S3 (bucket por parceiro ou prefixo por parceiro)

S3 Structure:
  s3://logistica-parceiros/
    parceiro-A/incoming/   (parceiro A só acessa este prefixo)
    parceiro-B/incoming/   (parceiro B só acessa este prefixo)

Controle de Acesso:
IAM Policy por parceiro: 
  s3:PutObject apenas em /{parceiro}/{userid}/*
  s3:GetObject apenas em /{parceiro}/{userid}/*
  
PROCESSAMENTO AUTOMÁTICO:
S3 Event (s3:ObjectCreated) → Lambda (process-file)
├── Valida formato do arquivo (CSV, EDI)
├── Move para /processed/ após validação
├── Insere registros em DynamoDB
└── SNS → sistema interno atualiza status entrega

MONITORAMENTO:
CloudWatch Logs (cada conexão SFTP logada)
CloudTrail (quem fez upload de qual arquivo, quando)
CloudWatch Alarm (arquivo não recebido de parceiro crítico em 24h)
```

**Benefícios vs SFTP on-premises:**
| Item | On-premises | Transfer Family |
|------|------------|----------------|
| HA | Manual (2 VMs) | Nativo Multi-AZ |
| Escalabilidade | Limitada | Automática |
| Manutenção | Patches manuais | Zero |
| Custo | Servidores + licenças | Por hora + por GB |
| Auditoria | Logs manuais | CloudTrail nativo |
