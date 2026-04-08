# Módulo 05 — Amazon S3 e Armazenamento

## Objetivo

Neste módulo, você vai dominar decisões de arquitetura de armazenamento na AWS, escolhendo serviços e classes de armazenamento com base em custo, latência, durabilidade, segurança e requisitos de recuperação para cenários de prova e de produção.

## Serviços AWS principais

- Amazon S3
- Amazon EFS
- Amazon FSx
- AWS Storage Gateway
- AWS DataSync

## Arquitetura e trade-offs

## Classes de Armazenamento do S3

O Amazon S3 oferece um espectro de classes que equilibram custo, durabilidade e latência de acesso:

| Classe | Disponibilidade | AZs | Latência | Mínimo de armazenamento | Caso de uso |
|---|---|---|---|---|---|
| S3 Standard | 99,99% | ≥3 | ms | Nenhum | Dados acessados frequentemente |
| S3 Intelligent-Tiering | 99,9% | ≥3 | ms | 30 dias | Padrão de acesso imprevisível |
| S3 Standard-IA | 99,9% | ≥3 | ms | 30 dias | Backups, DR acessados mensalmente |
| S3 One Zone-IA | 99,5% | 1 | ms | 30 dias | Dados reproduzíveis, thumbnails |
| S3 Glacier Instant Retrieval | 99,9% | ≥3 | ms | 90 dias | Arquivos acessados 1x/trimestre |
| S3 Glacier Flexible Retrieval | 99,99% | ≥3 | minutos–horas | 90 dias | Arquivos que podem esperar horas |
| S3 Glacier Deep Archive | 99,99% | ≥3 | 12–48h | 180 dias | Conformidade, retenção de 7-10 anos |

> **Durabilidade:** Todas as classes têm 11 noves (99,999999999%) exceto One Zone-IA (99,5% disponibilidade, mesma durabilidade mas em 1 AZ).

### S3 Intelligent-Tiering
Move objetos automaticamente entre tiers com base no acesso:
- **Frequent Access** (padrão) → **Infrequent Access** (30 dias sem acesso) → **Archive Instant** (90 dias) → **Archive Access** (90–730 dias, opcional) → **Deep Archive Access** (180–730 dias, opcional)  
- Cobra taxa de monitoramento por objeto (não há taxa por recuperação no Frequent/Infrequent)

---

## Lifecycle Policies

Regras que automaticamente transitam objetos entre classes ou os deletam:

```
Objeto criado (Standard)
    ↓ 30 dias → Standard-IA
    ↓ 90 dias → Glacier Instant Retrieval
    ↓ 180 dias → Glacier Deep Archive
    ↓ 365 dias → DELETE (expiração)
```

Pontos importantes:
- Transição mínima entre Standard e Standard-IA/One Zone-IA: **30 dias**
- Lifecycle pode se aplicar a versões não-correntes (com versioning habilitado)
- **Lifecycle não aplica a objetos < 128 KB** (custo-efetivo não compensa para Glacier)
- Pode filtrar por prefixo, tags ou tamanho de objeto

---

## Versionamento e Replicação

### Versioning
- Habilitado no nível do bucket — não pode ser desabilitado após ativado (apenas suspenso)
- Cada versão do objeto tem um `VersionId` único
- `DELETE` em objeto versionado cria um **delete marker** — objeto não é deletado fisicamente
- Para deletar permanentemente: deve especificar o `VersionId`

### Replicação
| Tipo | Descrição |
|---|---|
| **CRR** (Cross-Region Replication) | Origem e destino em regiões diferentes. DR, conformidade, latência diminuída para leituras |
| **SRR** (Same-Region Replication) | Mesma região. Agregação de logs, produção → staging, conformidade |

Requisitos:
- Versioning **habilitado** em ambos os buckets (origem e destino)
- IAM Role com permissão de `s3:ReplicateObject` no destino
- Replicação é **prospectiva** — objetos anteriores à ativação não são replicados (use S3 Batch para objetos existentes)
- S3 Replication Time Control (RTC): garante 99,99% dos objetos replicados em 15 minutos (SLA pago)

---

## Segurança no S3

### Controle de Acesso
- **Bucket Policy** (resource-based): JSON, permite acesso cross-account, condições (IP, VPC endpoint, MFA, etc.)
- **ACLs** (legado): nível de objeto ou bucket. AWS recomenda desabilitar ACLs e usar somente bucket policies
- **Access Points**: simplifam políticas para múltiplos aplicativos em um bucket — cada access point tem sua própria política

### Pre-signed URLs
- URLs temporárias com credenciais embutidas do gerador
- Pode ser para GET ou PUT
- Expiração configurável (máx 7 dias para SDK, 12h para `aws s3 presign`)
- Usa as permissões de quem gerou a URL no momento do acesso

### CORS
Cross-Origin Resource Sharing — necessário quando uma aplicação web em `domain-a.com` faz requisição ao bucket `domain-b.s3.amazonaws.com`. Configurado via regras CORS no bucket (AllowedOrigins, AllowedMethods, AllowedHeaders).

### Block Public Access
- **4 configurações** que bloqueiam: novas ACLs, qualquer ACL, novas bucket policies com acesso público, qualquer política
- Habilitado por padrão em novos buckets
- Pode ser configurado no nível de account (AWS Organizations)

---

## Encriptação no S3

| Tipo | Gerenciamento da Chave | Onde a encriptação ocorre |
|---|---|---|
| **SSE-S3** | AWS (AES-256, chave por objeto) | Server-side (padrão, sem custo adicional) |
| **SSE-KMS** | Você (CMK ou aws/s3) | Server-side, auditado pelo CloudTrail, custo de KMS API |
| **SSE-C** | Você (chave enviada na requisição) | Server-side, AWS encripta mas não armazena a chave |
| **CSE** | Você (encripta antes de enviar) | Client-side, S3 armazena ciphertext |

A partir de 2023, SSE-S3 é o padrão para novos buckets (a menos que configurado de outra forma).

**Bucket Key (SSE-KMS):** Reduz chamadas à API do KMS em até 99% usando uma chave derivada por bucket, em vez de chamar o KMS para cada objeto. Habilitado por padrão em novos buckets com SSE-KMS.

---

## Performance do S3

- **Prefixos**: S3 escala para **3.500 PUTs/s e 5.500 GETs/s por prefixo**. Distribuir objetos em múltiplos prefixos (ex: hash do nome) maximiza throughput
- **Multipart Upload**: recomendado para objetos > 100 MB, **obrigatório** > 5 GB. Paraleliza uploads em partes
- **S3 Transfer Acceleration**: usa CloudFront Edge Locations para aceleração de upload global via rede backbone da AWS
- **S3 Byte-Range Fetches**: paraleliza downloads dividindo o objeto em ranges (semelhante ao multipart mas para leitura)

---

## S3 Object Lock (WORM)

Impede que objetos sejam deletados ou sobrescritos por um período definido ou indefinidamente:

| Modo | Comportamento |
|---|---|
| **Governance** | Apenas usuários com `s3:BypassGovernanceRetention` podem sobrescrever ou deletar |
| **Compliance** | **Ninguém** (incluindo root) pode deletar ou modificar enquanto ativa, nem alterar o modo |

- **Legal Hold**: WORM sem data de expiração, ativado/desativado por quem tem `s3:PutObjectLegalHold`
- Requer versioning habilitado
- Ideal para conformidade regulatória (SEC Rule 17a-4, FINRA, CFTC)

---

## S3 Event Notifications

Dispara notificações em operações de objeto (`s3:ObjectCreated:*`, `s3:ObjectRemoved:*`, etc.):
- **Destinos**: SNS topic, SQS queue, Lambda function, **EventBridge**
- Para eventos via EventBridge: todas as operações S3 ficam disponíveis com filtros avançados e múltiplos destinos

---

## Amazon S3 Select e Glacier Select

- **S3 Select**: executa queries SQL simples (`SELECT * FROM S3Object WHERE...`) diretamente no S3, filtrando dados antes de transferi-los — reduz custo e latência
- **Suporta**: CSV, JSON, Parquet (comprimidos com GZIP/BZIP2)

---

## Outros Serviços de Armazenamento

### Amazon EFS (Elastic File System)
- **Sistema de arquivos NFS** gerenciado, compatível com Linux (NFS v4.1)
- **Multi-AZ** por padrão (Standard) ou Single-AZ para EC2 na mesma AZ (custo menor)
- Classes: Standard e Standard-IA (igual ao S3, lifecycle move entre eles)
- **Performance modes**: General Purpose (padrão, baixa latência) e Max I/O (alta concorrência)
- **Throughput modes**: Bursting (escala com armazenamento), Provisioned (fixo), Elastic (recomendado — auto-adjusts)
- Cobrado por GB armazenado (mais caro que EBS, mas compartilhado entre instâncias)

### Amazon FSx
| Tipo | Para | Protocolo | Destaques |
|---|---|---|---|
| FSx for Windows File Server | Apps Windows / AD | SMB | AD nativo, DFS Namespaces, NTFS |
| FSx for Lustre | HPC, ML, Media | POSIX | Sub-ms latência, integra com S3, scratch vs persistent |
| FSx for NetApp ONTAP | Migração on-prem | NFS/SMB/iSCSI | Deduplicação, capacidade multi-protocolo |
| FSx for OpenZFS | Migração de ZFS | NFS | Snapshots, clone, compressão |

### AWS Storage Gateway
| Tipo | Protocolo | Caso de uso |
|---|---|---|
| File Gateway | NFS/SMB | Extensão on-prem para S3 (arquivos viram objetos) |
| Volume Gateway (Cached) | iSCSI | Dados primários no S3, cache local; backups como EBS snapshots |
| Volume Gateway (Stored) | iSCSI | Dados primários on-prem, backup async no S3 |
| Tape Gateway | iSCSI VTL | Substitui fita física, backup vai para S3/Glacier |

### AWS DataSync
- Transferência de dados **incremental** e agendada entre on-prem e AWS (S3, EFS, FSx)
- Agente instalado on-prem, compressão e encriptação em trânsito
- Até **10 Gbps** de throughput, verifica integridade de dados

---

## Armadilhas comuns na prova

- **S3 Standard por padrão** — sempre resposta segura quando sem requisitos de custo
- **One Zone-IA** apenas para dados **reproduzíveis** (thumbnails, transcodificados) — não usar para backup crítico
- **Glacier Instant** = acesso em ms; **Glacier Flexible** = minutos/horas; **Deep Archive** = 12–48h
- **Object Lock Compliance** = ninguém deleta, nem root — FINRA, SEC compliance
- **Pre-signed URL** usa permissões de quem gerou — se a role for revogada, a URL para de funcionar
- **SSE-KMS com Bucket Key** reduz chamadas KMS em ~99% — melhor custo
- **CRR requer versioning em ambos os buckets**
- FSx for Lustre: integra com S3 como data repository — HPC lê/escreve direto no Lustre, sincroniza com S3
- EFS: cobrado por uso real; EBS: cobrado por capacidade provisionada

## Lab hands-on

Para prática guiada, utilize o laboratório de S3 avançado em [07-S3-Avancado-Labs/lab.md](../07-S3-Avancado-Labs/lab.md).
Notas de custo: priorize arquivos pequenos para testes de lifecycle/replicação, use poucos objetos por etapa e remova buckets, versões e replication rules ao final.

## Questões práticas

- [questoes.md](./questoes.md)

## Revisão rápida / cheatsheet

- [cheatsheet.md](./cheatsheet.md)
- [flashcards.md](./flashcards.md)
