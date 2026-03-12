# Casos de Uso — Módulo 05: Amazon S3 e Armazenamento

## Caso 1: Hospedagem de Site Estático com CloudFront e Segurança

**Cenário:** Empresa quer hospedar um site estático (React SPA) com baixa latência global, bucket S3 privado e HTTPS.

**Arquitetura:**
```
Usuário (Global)
  └── Route 53 (Alias → CloudFront)
        └── CloudFront Distribution
              └── S3 Bucket (PRIVADO - Block Public Access ON)
                    └── Origin Access Control (OAC) → Bucket Policy
```

**Configurações-chave:**
- S3: Block Public Access = ON; Versioning = ON para rollback
- CloudFront: OAC habilitado; Viewer Protocol Policy = HTTPS Only
- Bucket Policy: `Allow cloudfront.amazonaws.com com condition aws:SourceArn = distribution ARN`
- ACM Certificate: us-east-1 (obrigatório para CloudFront)
- Route 53: Alias record `www.empresa.com` → CloudFront distribution

**Por que este design?** S3 privado + OAC impede acesso direto ao bucket; CloudFront é o único caminho de acesso. CloudFront faz caching global → menor latência. HTTPS enforçado em todas as camadas.

---

## Caso 2: Backup Automatizado com Lifecycle e Glacier

**Cenário:** Empresa financeira armazena relatórios mensais por 7 anos. Acesso nos primeiros 90 dias é frequente; depois, raramente acessado.

**Lifecycle Policy:**
```
S3 Standard → (90 dias) → S3 Standard-IA → (180 dias) → S3 Glacier Instant Retrieval
           → (365 dias) → S3 Glacier Deep Archive → (7 anos) → Expire
```

**Object Lock configurado:**
- Modo Compliance com retention de 2.555 dias (7 anos)
- Legal Hold adicional para documentos em auditoria

**Por que este design?** Lifecycle reduz custo progressivamente conforme o dado "envelhece". Object Lock Compliance garante conformidade regulatória sem possibilidade de apagamento prematuro.

---

## Caso 3: Data Lake Multi-Região com Replicação

**Cenário:** Empresa global precisa de dados disponíveis em us-east-1 (processamento) e ap-southeast-1 (conformidade de dados na Ásia).

**Arquitetura:**
```
Produção (us-east-1)             Leitura/Conformidade (ap-southeast-1)
  └── S3 Bucket [Primary]   →CRR→  S3 Bucket [Replica]
        ├── S3 Event Notification       └── S3 Inventory (compliance)
        │     └── SQS → Lambda (ETL)
        └── S3 Inventory (CSV diário)
```

**Configurações:**
- Versioning: ON em ambos os buckets
- CRR: replicação de todos os objetos (ou por prefix `data/raw/`)
- Encryption: SSE-KMS com different CMKs por região
- Access Points: um por equipe de dados com prefix isolation

---

## Caso 4: Upload Direto do Browser com Pre-signed URL

**Cenário:** Plataforma de fotos permite usuários fazerem upload de imagens direto para S3 sem passar pelo servidor da aplicação.

**Fluxo:**
```
1. Browser → API Gateway → Lambda
              └── Lambda gera pre-signed URL (PutObject, TTL 15min)
2. Browser → PUT diretamente para S3 usando pre-signed URL
3. S3 Event Notification → SQS → Lambda (processamento da imagem)
4. Lambda → escreve thumbnail em outro bucket
```

**Configurações:**
- CORS no bucket S3: AllowedOrigins = `https://app.empresa.com`, AllowedMethods = PUT
- Bucket Policy: permite apenas requests com a presigned URL signature
- CloudFront na frente para servir imagens processadas (GET)

---

## Caso 5: Migração para AWS com Storage Gateway e DataSync

**Cenário:** Data center corporativo com servidores Windows acessando shares de arquivo. Empresa quer migrar para S3 gradualmente.

**Migração em Fases:**
```
FASE 1 (Convivência):
  Windows Servers → File Gateway (NFS/SMB) → S3 Bucket
  [File Gateway fica on-prem; dados primários no S3; cache local]

FASE 2 (Migração em Massa):
  DataSync Agent (on-prem) → AWS DataSync → S3 Bucket
  [Migração de dados históricos com verificação de integridade]

FASE 3 (Pós-migração):
  Windows Servers → Amazon FSx for Windows (AWS) via Direct Connect
  [File system gerenciado na AWS com AD integration]
```

**Por que Storage Gateway primeiro?** Permite convivência gradual enquanto equipes se adaptam, sem big-bang cutover. DataSync para migração em massa dos dados existentes.
