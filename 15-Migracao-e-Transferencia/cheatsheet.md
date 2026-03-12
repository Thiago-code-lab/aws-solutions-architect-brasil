# Cheatsheet — Módulo 15: Migração e Transferência

## Os 7 Rs — Estratégias de Migração

| Estratégia | Nome | Mudança | Exemplo |
|---|---|---|---|
| **Retire** | Descontinuar | Nenhuma | App não mais usado |
| **Retain** | Manter on-premises | Nenhuma | Compliance, latência |
| **Rehost** | Lift-and-shift | Mínima | EC2 = VM on-premises |
| **Relocate** | Mover VMware | Mínima | VMware Cloud on AWS |
| **Replatform** | Lift-and-optimize | Pontual | RDS em vez de MySQL self-managed |
| **Repurchase** | Trocar por SaaS | Mudança de produto | Salesforce em vez de CRM próprio |
| **Refactor** | Rearchitect | Grande | Microservices + Lambda + DynamoDB |

---

## Snow Family — Capacidade e Uso

| Dispositivo | Storage | Compute | Quando Usar |
|---|---|---|---|
| **Snowcone Small** | 8 TB SSD | 2 vCPUs, 4 GB RAM | Locais remotos, peso: 2,1 kg |
| **Snowcone Large** | 14 TB SSD | 4 vCPUs, 4 GB RAM | Capacidade maior em campo |
| **Snowball Edge Storage** | 80 TB HDD | 40 vCPUs | Migração de dados grande escala |
| **Snowball Edge Compute** | 42 TB HDD | 52 vCPUs, GPU opcional | Edge computing pesado |
| **Snowmobile** | 100 PB | N/A | Datacenter inteiro |

**Regra de decisão:** `tamanho_dados / bandwidth_disponível > 1 semana` → usar Snow Family.

---

## Comparativo: AWS MGN vs DMS vs DataSync

| | MGN | DMS | DataSync |
|---|---|---|---|
| **Tipo** | Rehost de servidores | Migração de banco | Transferência de arquivos |
| **Source** | Qualquer servidor (Win/Linux) | Banco de dados | NFS, SMB, S3, HDFS |
| **Target** | EC2 | RDS, Aurora, Redshift | S3, EFS, FSx |
| **Replicação** | Contínua (block-level) | CDC (transações) | Batch ou agendada |
| **Downtime** | Minutos (cutover manual) | Mínimo (CDC) | N/A |
| **Prerequisito** | Agente no servidor | Task DMS configurada | DataSync Agent on-premises |

---

## DMS + SCT — Quando Usar Cada Um

| Migração | DMS | SCT |
|---|---|---|
| MySQL → MySQL RDS | ✅ | ❌ Não necessário |
| Oracle → Oracle RDS | ✅ | ❌ Não necessário |
| Oracle → Aurora PostgreSQL | ✅ (dados) | ✅ (schema conversion) |
| SQL Server → RDS PostgreSQL | ✅ (dados) | ✅ (schema conversion) |
| Oracle → Redshift | ✅ (dados) | ✅ (schema conversion) |

**Regra:** mesmo engine (homogêneo) = apenas DMS. Engines diferentes (heterogêneo) = SCT primeiro + DMS.

---

## AWS DataSync — Casos de Uso

| Caso de Uso | Configuração |
|---|---|
| On-premises NFS → S3 | DataSync Agent on-prem, Location NFS + S3 |
| S3 → EFS | Sem agent (AWS-to-AWS), Location S3 + EFS |
| HDFS on-prem → S3 | Agent on-prem, Location HDFS + S3 |
| SMB/Windows → S3 | Agent on-prem, Location SMB + S3 |
| S3 → FSx for Windows | Sem agent, Location S3 + FSx WFS |

**Preserva:** POSIX permissions, timestamps, ACLs. **Verifica:** integridade via checksums end-to-end.

---

## AWS Transfer Family — Protocolos

| Protocolo | Porta | Criptografia | Caso de Uso |
|---|---|---|---|
| **SFTP** | 22 | TLS | Padrão para transferência segura de arquivos |
| **FTPS** | 990/21 | TLS explicit/implicit | Sistemas legados que precisam de FTP + TLS |
| **FTP** | 21 | Nenhuma | Redes internas (não recomendado para internet) |
| **AS2** | 443 | TLS + S/MIME | EDI B2B (Electronic Data Interchange) |

Storage backend: **S3** ou **EFS**.

---

## Migration Hub — Fases de Migração

```
Fase 1: DISCOVER
   Application Discovery Service
   ↓ (inventário + dependências)

Fase 2: ASSESS  
   Migration Evaluator (TCO)
   Migration Hub (priorização)
   ↓ (business case)

Fase 3: MIGRATE
   MGN (servidores) / DMS (bancos) / DataSync (arquivos)
   ↓ (migração + replicação)

Fase 4: CUTOVER
   Parar source → Validar target → Redirecionar tráfego
   ↓

Fase 5: MODERNIZE
   Refactoring pós-migração (opcional)
```

---

## Dicas de Prova — Padrões Comuns com Migração

| Pista no Enunciado | Resposta Provável |
|---|---|
| "Mover servidor Windows para EC2 com downtime mínimo" | AWS MGN |
| "Migrar Oracle para Aurora PostgreSQL" | SCT (schema) + DMS (dados + CDC) |
| "Terabytes de dados, internet seria lenta" | AWS Snowball Edge |
| "100 PB de dados para AWS" | AWS Snowmobile |
| "Mover arquivos NFS para S3 preservando permissões" | AWS DataSync |
| "Parceiro externo precisa de SFTP" | AWS Transfer Family |
| "Migração de VM VMware com mínima mudança" | AWS MGN ou Relocate (VMware Cloud) |
| "Descobrir dependências de aplicações on-prem" | Application Discovery Service |
| "Calcular TCO antes de migrar" | AWS Migration Evaluator |
| "Acompanhar progresso de migração centralizado" | AWS Migration Hub |
| "Acesso contínuo de clientes via FTP para S3" | AWS Transfer Family |
| "Replicar mudanças de banco em tempo real durante migração" | DMS com CDC |
| "Dados de sensor edge sem conectividade confiável" | Snowball Edge (Compute Optimized) |
