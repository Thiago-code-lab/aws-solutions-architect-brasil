# Flashcards — Módulo 05: Amazon S3 e Armazenamento

---

**P:** Quais são as 7 classes de armazenamento S3 em ordem decrescente de custo de armazenamento?
**R:** Standard → Intelligent-Tiering → Standard-IA → One Zone-IA → Glacier Instant Retrieval → Glacier Flexible Retrieval → Glacier Deep Archive

---

**P:** Qual é o tempo mínimo de armazenamento cobrado para S3 Standard-IA?
**R:** 30 dias (abaixo disso, cobra como se fosse 30 dias)

---

**P:** Qual é o tempo mínimo de armazenamento para S3 Glacier Deep Archive?
**R:** 180 dias

---

**P:** Quantas cópias S3 Standard mantém e em quantas AZs?
**R:** Mínimo 3 cópias em pelo menos 3 AZs da mesma região (durabilidade 11 9s = 99,999999999%)

---

**P:** Qual classe S3 armazena em apenas 1 AZ? Qual o risco?
**R:** S3 One Zone-IA. Risco: se a AZ for destruída, os dados são perdidos. Disponibilidade 99,5% vs 99,9% das outras classes

---

**P:** O que é S3 Intelligent-Tiering e quando usar?
**R:** Move objetos automaticamente entre tiers (Frequent Access, Infrequent Access, Archive tiers) baseado em acesso. Ideal para padrões imprevisíveis. Cobra pequena taxa de monitoramento mas sem retrieval fee

---

**P:** Quais são os requisitos para habilitar CRR ou SRR?
**R:** Versionamento deve estar habilitado em AMBOS os buckets (origem e destino). Objetos existentes não são replicados automaticamente (usar S3 Batch Replication para isso)

---

**P:** Qual é a diferença entre SSE-S3, SSE-KMS e SSE-C?
**R:** SSE-S3: AWS gerencia chaves (AES-256). SSE-KMS: chaves gerenciadas via AWS KMS (audit trail, permissões finas). SSE-C: cliente fornece a chave em cada request; AWS encripta/decripta mas não armazena a chave

---

**P:** O que é S3 Bucket Key e qual o benefício?
**R:** Reduz o número de chamadas ao KMS ao criar uma chave envelope por bucket (não por objeto). Reduz custo de SSE-KMS em até 99%

---

**P:** Qual é o throughput por prefixo S3 para PUT e GET?
**R:** 3.500 PUTs/COPY/POST/DELETE e 5.500 GET/HEAD por segundo **por prefixo**. Distribuir em múltiplos prefixos aumenta linearmente o throughput total

---

**P:** Quando é obrigatório usar Multipart Upload no S3?
**R:** Obrigatório para objetos acima de **5 GB**. Recomendado para objetos acima de **100 MB**

---

**P:** Qual é a diferença entre Object Lock Governance e Compliance?
**R:** Governance: usuários com permissão `s3:BypassGovernanceRetention` podem apagar/modificar. Compliance: NINGUÉM pode apagar (nem root/AWS) durante o retention period. Compliance é irrevogável

---

**P:** O que é Legal Hold no S3 Object Lock?
**R:** Protege o objeto de apagamento independente do retention period. Pode ser aplicado/removido por qualquer usuário com permissão `s3:PutObjectLegalHold`. Não expira automaticamente

---

**P:** Para que serve S3 Transfer Acceleration?
**R:** Usa CloudFront edge locations para acelerar uploads do cliente usuario para S3 (especialmente clientes geograficamente distantes da região S3). Tráfego vai ao edge CloudFront mais próximo, depois pela rede backbone AWS até o S3

---

**P:** Qual a diferença entre Gateway Endpoint e Interface Endpoint para S3?
**R:** Gateway Endpoint: gratuito, adicionado como rota na route table, apenas dentro da VPC. Interface Endpoint (PrivateLink): tem custo ($/hora + $/GB), cria ENI com IP privado, acessível de on-prem via DX/VPN

---

**P:** O que é o Amazon FSx for Lustre e qual o caso de uso principal?
**R:** File system de alta performance baseado no Lustre (HPC open source). Integra nativamente com S3 (lê/escreve diretamente). Casos: ML training, HPC, renderização de vídeo, processamento paralelo de dados

---

**P:** Quais são os 4 tipos de AWS Storage Gateway?
**R:** File Gateway (NFS/SMB → S3), Volume Gateway Cached (iSCSI, dados primários no S3, cache local), Volume Gateway Stored (iSCSI, dados primários local, backup S3), Tape Gateway (VTL → S3/Glacier)

---

**P:** O que é S3 Select e qual o benefício?
**R:** Permite executar queries SQL simples (SELECT) sobre objetos S3 (CSV, JSON, Parquet) sem baixar o arquivo completo. Reduz dados transferidos e custo. Alternativa simples ao Athena para queries pontuais

---

**P:** Como funciona o S3 Event Notifications?
**R:** Dispara eventos para SNS, SQS ou Lambda quando ações ocorrem no bucket (s3:ObjectCreated, s3:ObjectRemoved, s3:ObjectRestore, etc.). Pode filtrar por prefixo e sufixo de chave. Alternativa: usar EventBridge (mais flexível)

---

**P:** Qual é a diferença entre EFS e EBS?
**R:** EFS: NFS4 managed, multi-AZ, acesso concurrent de múltiplas instâncias/AZs, escala automaticamente, Linux only. EBS: block storage, tipicamente acoplado a 1 instância (exceto io2 Multi-Attach), single-AZ, Windows/Linux
