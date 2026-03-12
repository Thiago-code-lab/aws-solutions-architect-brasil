# Cheatsheet — Módulo 05: Amazon S3 e Armazenamento

## Classes de Armazenamento S3

| Classe | Disponibilidade | AZs | Retrieval Fee | Min. Duração | Uso Principal |
|---|---|---|---|---|---|
| S3 Standard | 99,99% | ≥ 3 | Nenhuma | Nenhuma | Dados frequentemente acessados |
| S3 Intelligent-Tiering | 99,9% | ≥ 3 | Nenhuma | Nenhuma | Padrão de acesso imprevisível |
| S3 Standard-IA | 99,9% | ≥ 3 | Por GB | 30 dias | Acesso infrequente, recuperação rápida |
| S3 One Zone-IA | 99,5% | 1 | Por GB | 30 dias | Dados reproduzíveis, baixo custo |
| S3 Glacier Instant | 99,9% | ≥ 3 | Por GB | 90 dias | Arquivos raramente acessados, recuperação ms |
| S3 Glacier Flexible | 99,99% | ≥ 3 | Por GB | 90 dias | Arquivos, recuperação minutos/horas |
| S3 Glacier Deep Archive | 99,99% | ≥ 3 | Por GB | 180 dias | Long-term backup, recuperação horas |

## Tipos de Encriptação S3

| Tipo | Gerencia Chave | Gerencia Encriptação | Casos |
|---|---|---|---|
| SSE-S3 | AWS | AWS | Menor overhead operacional |
| SSE-KMS | AWS KMS (CMK opcional) | AWS | Audit trail, controle de acesso à chave |
| SSE-C | Cliente | AWS | Cliente quer total controle da chave |
| CSE | Cliente | Cliente | Compliance que exige encriptação cliente-side |

## Replicação S3

| | CRR | SRR |
|---|---|---|
| Regiões | Regiões diferentes | Mesma região |
| Requisito | Versionamento em AMBOS | Versionamento em AMBOS |
| Objetos existentes | Não replicados (usar S3 Batch) | Não replicados |
| Delete replicated | Opcional com delete marker sync | Opcional |

## Performance por Prefixo
- **3.500 PUT/COPY/POST/DELETE** por segundo por prefixo
- **5.500 GET/HEAD** por segundo por prefixo
- Multipart Upload: obrigatório > **5 GB**, recomendado > **100 MB**
- S3 Transfer Acceleration: usa edge CloudFront → backbone AWS → S3

## Object Lock
| Modo | Quem pode remover | Período |
|---|---|---|
| Governance | Usuários com `s3:BypassGovernanceRetention` | Configurável |
| Compliance | NINGUÉM (nem root) | Configurável, irrevogável |
| Legal Hold | Qualquer um com `s3:PutObjectLegalHold` | Sem data de expiração |

## Outros Serviços de Armazenamento

| Serviço | Protocolo | Multi-AZ | OS | Caso de Uso |
|---|---|---|---|---|
| EBS | Block (iSCSI) | Não (zonal) | Win/Linux | Volume único para EC2 |
| EFS | NFS v4 | Sim | Linux | Compartilhado entre N EC2 |
| FSx for Windows | SMB/NTFS | Sim | Windows | AD, DFS |
| FSx for Lustre | Lustre | Sim | Linux | HPC, ML, big data |
| FSx for ONTAP | NFS/SMB/iSCSI | Sim | Win/Linux | Migração NetApp |
| FSx for OpenZFS | NFS | Sim | Linux | Migração ZFS/POSIX |

## Storage Gateway

| Tipo | Protocolo | Onde Persiste |
|---|---|---|
| File Gateway | NFS/SMB | S3 (com cache local) |
| Volume Cached | iSCSI | S3 (com cache local) |
| Volume Stored | iSCSI | On-premises (backup async para S3) |
| Tape Gateway | iSCSI VTL | S3 / S3 Glacier |

## Dicas Rápidas de Prova
- Versionamento = **pré-requisito** para CRR, SRR e Object Lock
- Bucket Policy vs ACL: prefira Bucket Policy (ACL é legado)
- Block Public Access: 4 configurações independentes; aplica na conta E no bucket
- CORS: necessário para browser JavaScript fazer requests cross-origin para S3
- Pre-signed URL: gerada com credenciais do criador, expira em tempo configurável
- S3 Inventory: liste objetos do bucket de forma assíncrona (diárias/semanais) — alternativa ao LIST API
- S3 Select: SQL simples sobre objetos S3 (até 50% menos dados transferidos)
- DataSync: migração de dados on-prem → AWS ou entre serviços AWS (suporta NFS, SMB, HDFS, S3, EFS, FSx)
