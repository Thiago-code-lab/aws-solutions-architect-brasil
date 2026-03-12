# Módulo 15 — Migração e Transferência de Dados

## As 7 Estratégias de Migração (7 Rs)

| Estratégia | Nome | Descrição | Esforço |
|---|---|---|---|
| **Retire** | Aposentar | Descontinua aplicação: não tem valor de negócio | Nenhum |
| **Retain** | Reter | Mantém on-prem por agora (compliance, dívida técnica, revisitar depois) | Nenhum |
| **Rehost** | Lift & Shift | Move para EC2 sem mudanças (ex: MGN) | Baixo |
| **Relocate** | Relocar | Move para AWS sem mudanças (VMware Cloud on AWS, Kubernetes) | Baixo |
| **Replatform** | Lift & Reshape | Pequenas otimizações sem refatoração do core (ex: mover para RDS em vez de Oracle em EC2) | Médio |
| **Repurchase** | Substituir | Migra para SaaS (ex: CRM → Salesforce, email → Microsoft 365) | Médio |
| **Refactor / Re-architect** | Refatorar | Reescreve aproveitando cloud-native (ex: monolito → microserviços + Lambda + containers) | Alto |

---

## AWS Migration Hub

Painel central de rastreamento de migração:
- Visualiza progresso de todos os servidores sendo migrados
- Integra com **Application Discovery Service**, **MGN**, **DMS**
- **Migration Hub Orchestrator**: coordena e automatiza sequências de migração

---

## AWS Application Discovery Service

Coleta dados do ambiente on-prem para planejamento de migração:

| Modo | Como funciona |
|---|---|
| **Agentless Discovery** (Connector) | VM appliance no VMware vCenter; descobre VMs, CPU, memória, storage sem instalar agente |
| **Agent-based Discovery** | Instala ADS Agent em cada servidor; coleta dados mais detalhados (running processes, conexões de rede) |

Dados exportados para Migration Hub ou S3 para análise com Athena.

---

## AWS Application Migration Service (MGN/SMS)

**AWS MGN** (substituto do SMS - Server Migration Service):
- **Rehost (lift-and-shift)** automatizado para EC2
- Instala replication agent no servidor fonte → replica blocos de disco continuamente para AWS
- **Cutover**: testando em staging → janela de manutenção → cutover final

```
On-prem Server
  │
  ├── MGN Replication Agent instalado
  │     └── Replication contínua → S3/EBS na AWS
  │
  ├── Launch Template (tipo de instância, VPC, SG)
  │
  └── Test → Cutover → EC2 em produção
           └── RTO/RPO muito baixo (minutos)
```

---

## AWS Database Migration Service (DMS)

Migração de bancos de dados:

| Tipo | Origens/Destinos | SCT |
|---|---|---|
| **Homogênea** | Oracle→Oracle, MySQL→RDS MySQL, PostgreSQL→Aurora | Não necessário |
| **Heterogênea** | Oracle→Aurora, SQL Server→PostgreSQL | **SCT obrigatório** |

**Schema Conversion Tool (SCT)**: converte schema + stored procedures para o dialeto do banco destino

**CDC (Change Data Capture)**: replica mudanças contínuas após migração inicial (suporta replicação em curso com zero downtime)

**DMS Replication Instance**: EC2 que executa o trabalho de replicação; precisa ter conectividade com source e target

**Sources suportadas**: Oracle, SQL Server, MySQL, PostgreSQL, MongoDB, SAP, IBM DB2, S3
**Targets suportados**: todos acima + DynamoDB, Redshift, Kafka, OpenSearch, DocumentDB

---

## AWS Snow Family

Transferência física de dados (edge computing e migração massiva):

| Produto | Capacidade | Edge Computing | Uso Típico |
|---|---|---|---|
| **Snowcone** | 8-14 TB | Sim (2 vCPU, 4 GB) | Ambientes físicos pequenos, IoT |
| **Snowball Edge Storage Optimized** | 80 TB HDD + 1 TB SSD | Sim (40 vCPU, 80 GB) | Migração de dados em larga escala |
| **Snowball Edge Compute Optimized** | 42 TB + 28 TB SSD | Sim (52 vCPU, 208 GB, GPU opcional) | Análise local, ML na borda |
| **Snowmobile** | **100 PB** (exabyte-scale) | Não | Migração de datacenter inteiro |

**Regra prática**: se o upload levaria mais de 1 semana via internet → usar Snow Family

Processo:
1. AWS envia o dispositivo
2. Você carrega dados localmente
3. Envia de volta para AWS
4. AWS carrega dados para S3

Para **Snowball após S3**: lifecycle policy para Glacier se necessário.

---

## AWS DataSync

Transferência AUTOMATIZADA e agendada de dados:

| De → Para | Protocolo | Velocidade |
|---|---|---|
| On-prem → S3/EFS/FSx | NFS, SMB | Até 10 Gbps por agente |
| S3 → S3 (cross-region/account) | S3 API | Alta |
| EFS → EFS | NFS | Alta |

- **Instalação**: DataSync Agent (VM appliance) no on-prem; conecta via Direct Connect/VPN ou internet
- **Preserva metadados**: timestamps, permissões Unix
- **Verificação de integridade**: verifica checksums nos dois lados
- **Scheduled**: cria tarefas com agendamento (incremental após primeira full transfer)

**DataSync vs Storage Gateway vs Snow Family:**
```
DataSync: transferência agendada/automatizada (migration + sync)
Storage Gateway: acesso híbrido contínuo (não é uma ferramenta de migração)
Snow Family: quando a rede é insuficiente ou muito lenta
```

---

## AWS Transfer Family

Protocolos de transferência gerenciados para S3 e EFS:

| Protocolo | Porta |
|---|---|
| SFTP (SSH FTP) | 22 |
| FTPS (FTP over TLS) | 990 |
| FTP (não criptografado) | 21 |
| AS2 (Applicability Statement 2) | 8080/HTTPS |

- Mantém endpoints existentes (não precisa mudar clientes)
- Auth via IAM, Cognito, custom Lambda (para LDAP/AD existente)
- Ideal para: parceiros B2B que usam SFTP; migrações de servidores SFTP legados para S3

---

## Outros Serviços de Migração

| Serviço | Função |
|---|---|
| **AWS Mainframe Modernization** | Migra e moderniza mainframes IBM/Micro Focus para AWS |
| **CloudEndure Disaster Recovery** | Continuous replication para DR (substituído pelo MGN para migração) |
| **Migration Hub Refactor Spaces** | Migração incremental de monolito para microserviços |

---

## Dicas de Prova

- **7 Rs**: identificar a estratégia correta por cenário é pergunta frequente
  - "Sem mudança, mova para EC2" = Rehost (MGN)
  - "MySQL em EC2 → RDS" = Replatform
  - "Sem valor, desligar" = Retire
- **DMS**: para migração de BD; SCT = conversão de schema heterogênea
- **DataSync** vs **Snow Family**: DataSync = rede disponível + automação; Snow = sem rede boa
- **Snowmobile** = apenas para **exabyte** scale (>100 PB); Snowball Edge = 80 TB por dispositivo
- Transfer Family: clientes **não precisam mudar nada** → AWS recebe transferências e armazena em S3/EFS
- **MGN** inclui **continuous replication** → RTO muito baixo no momento do cutover
- Application Discovery Service: agentless (VMware VMs via connector) vs agent-based (qualquer OS, mais detalhado)
- DMS com CDC = **zero-downtime migration**: replica ongoing mudanças enquanto testando
- DataSync preserva metadados (timestamps, permissões); Snow Family não necessariamente
- **SCT** (Schema Conversion Tool) é separado do DMS — ferramenta desktop para converter DDL e procedimentos
