# Questões de Prova — Módulo 05: Amazon S3 e Armazenamento

<!-- Domínio SAA-C03: Design Resilient Architectures / Cost-Optimized Architectures -->

---

**1.** Uma empresa armazena logs de acesso com padrão imprevisível de acesso por período indeterminado. Precisam de menor custo possível sem gerenciar tiers manualmente. Qual classe S3 é mais adequada?

- A) S3 Standard-IA
- B) S3 Glacier Flexible Retrieval
- C) S3 Intelligent-Tiering
- D) S3 One Zone-IA

<details>
<summary>Resposta</summary>
**C — S3 Intelligent-Tiering**
Automaticamente move objetos entre tiers (Frequent, Infrequent, Archive Instant, Archive) conforme padrão de acesso. Ideal quando o padrão é imprevisível. One Zone-IA seria barco se o dado não puder tolerar perda; Standard-IA incorre custo de recuperação em acesso frequente.
</details>

---

**2.** Um bucket S3 precisa ser acessível publicamente para hospedar um site estático. O administrador ativou "Block Public Access" na conta mas quer liberar apenas este bucket. O que deve fazer?

- A) Não é possível — Block Public Access de conta é irrevogável
- B) Desabilitar Block Public Access na conta, depois aplicar bucket policy pública
- C) Habilitar "requester pays" no bucket para permitir acesso público
- D) Criar um endpoint S3 e acessar via CloudFront sem tornar o bucket público

<details>
<summary>Resposta</summary>
**D — Usar CloudFront com OAC (recomendado)**
A melhor prática é NÃO tornar o bucket público. Use CloudFront com Origin Access Control (OAC). Mas se a pergunta exige bucket público: a resposta correta é B — desabilitar Block Public Access na conta e aplicar bucket policy. A opção D é arquiteturalmente superior e é o que AWS recomenda.
</details>

---

**3.** Uma empresa quer replicar objetos S3 de us-east-1 para ap-southeast-1 para reduzir latência de usuários na Ásia. Qual recurso habilitar?

- A) S3 Transfer Acceleration
- B) Cross-Region Replication (CRR)
- C) Same-Region Replication (SRR)
- D) S3 Multi-Region Access Points

<details>
<summary>Resposta</summary>
**B — Cross-Region Replication (CRR)**
CRR replica objetos assincronamente entre buckets em regiões diferentes. Requer versionamento habilitado em ambos. SRR é para mesma região. Transfer Acceleration acelera uploads do cliente para S3, não replica objetos. Multi-Region Access Points roteia para o bucket mais próximo mas não replica sozinho.
</details>

---

**4.** Qual tipo de encriptação no S3 permite que o cliente gerencie as chaves mas a AWS faça a encriptação/decriptação?

- A) SSE-S3 (AES-256)
- B) SSE-KMS
- C) SSE-C
- D) Client-Side Encryption (CSE)

<details>
<summary>Resposta</summary>
**C — SSE-C**
SSE-C: o cliente fornece a chave em cada request HTTP; AWS encripta/decripta mas nunca armazena a chave. SSE-S3: AWS gerencia tudo. SSE-KMS: AWS gerencia via KMS (keys podem ser CMKs do cliente). CSE: o cliente encripta antes de enviar, AWS vê apenas bytes encriptados.
</details>

---

**5.** Um objeto no S3 precisa ser acessado por um usuário externo sem credenciais AWS por 1 hora. Qual mecanismo usar?

- A) ACL pública no objeto
- B) URL pre-assinada (presigned URL)
- C) Bucket policy com condition aws:SourceIp
- D) VPC Endpoint com policy aberta

<details>
<summary>Resposta</summary>
**B — Presigned URL**
Presigned URLs concedem acesso temporário (tempo configurável) a um objeto específico, usando as credenciais de quem gerou a URL. Expiram após o TTL. ACL pública exporia o objeto indefinidamente. SourceIp restricts por IP, não por tempo.
</details>

---

**6.** Uma empresa tem objetos S3 que não podem ser apagados por 7 anos por regulação financeira. Qual recurso garantirá que nem mesmo administradores possam apagar?

- A) MFA Delete
- B) Versionamento com lifecycle policy
- C) S3 Object Lock no modo Compliance
- D) S3 Object Lock no modo Governance

<details>
<summary>Resposta</summary>
**C — S3 Object Lock modo Compliance**
No modo Compliance, NENHUM usuário (incluindo root) pode apagar ou alterar o objeto antes do retention period expirar. No modo Governance, usuários com permissão `s3:BypassGovernanceRetention` podem burlar. MFA Delete protege apagamento do bucket, não garante retenção mínima.
</details>

---

**7.** Uma aplicação faz uploads de arquivos de 1 TB para S3. O que é necessário para garantir confiabilidade e eficiência no upload?

- A) Usar S3 Transfer Acceleration exclusivamente
- B) Usar Multipart Upload (obrigatório acima de 5 GB)
- C) Usar S3 Batch Operations
- D) Subir para um bucket regional e depois replica com CRR

<details>
<summary>Resposta</summary>
**B — Multipart Upload**
Multipart Upload é **obrigatório** para objetos acima de **5 GB** e recomendado para objetos acima de 100 MB. Divide o arquivo em partes paralelas, aumenta velocidade e resiliência (partes com falha são reenviadas individualmente). Transfer Acceleration é para reduzir latência geográfica, não para confiabilidade de upload.
</details>

---

**8.** Um cliente web precisa fazer upload de arquivos diretamente do browser para um bucket S3 em domínio diferente. O que habilitar no bucket?

- A) Transfer Acceleration
- B) CORS (Cross-Origin Resource Sharing)
- C) Bucket Policy com s3:GetObject público
- D) ACL com permissão AllUsers

<details>
<summary>Resposta</summary>
**B — CORS**
CORS controla quais origens (domínios externos) podem fazer requests para o bucket S3 diretamente do browser. Sem CORS configurado, o browser bloqueará requisições cross-origin por segurança. Isso é necessário para uploads diretos do browser (direct-to-S3).
</details>

---

**9.** Uma empresa quer minimizar custos armazenando logs de auditoria que raramente serão acessados mas que, quando necessário, precisam estar disponíveis em menos de 5 minutos. Qual classe S3?

- A) S3 Glacier Deep Archive
- B) S3 Glacier Flexible Retrieval
- C) S3 Glacier Instant Retrieval
- D) S3 Standard-IA

<details>
<summary>Resposta</summary>
**C — S3 Glacier Instant Retrieval**
Glacier Instant Retrieval oferece recuperação em **milissegundos**, com custo menor que Standard-IA. Ideal para dados raramente acessados mas que precisam de acesso imediato quando solicitados. Glacier Flexible Retrieval demora minutos a horas. Deep Archive demora horas.
</details>

---

**10.** Um sistema de arquivos precisa ser compartilhado simultaneamente entre 100 instâncias EC2 Linux em múltiplas AZs. Qual serviço AWS usar?

- A) Amazon S3
- B) Amazon EBS (gp3)
- C) Amazon EFS
- D) Amazon FSx for Windows File Server

<details>
<summary>Resposta</summary>
**C — Amazon EFS**
EFS é NFS gerenciado para Linux, multi-AZ, permite acesso concurrent de múltiplas instâncias/AZs. EBS é block storage acoplado a uma única instância (exceto io2 Multi-Attach, limitado a 16 instâncias na mesma AZ). FSx for Windows é para ambientes Windows. S3 tem semântica diferente (object storage).
</details>

---

**11.** Uma empresa data center precisa migrar 10 TB de dados semanais por backups para AWS S3. A conexão de internet é limitada. Qual serviço usar sem AWS SnowFamily?

- A) AWS DataSync via internet
- B) AWS Storage Gateway — Tape Gateway
- C) AWS DataSync via Direct Connect
- D) S3 Transfer Acceleration

<details>
<summary>Resposta</summary>
**C — AWS DataSync via Direct Connect**
DataSync é o serviço de migração/sincronização de dados. Com Direct Connect, os dados trafegam por link dedicado privado, evitando a internet. Storage Gateway Tape substitui fitas físicas mas não otimiza a transferência em massa. Transfer Acceleration usa internet (CloudFront edge).
</details>

---

**12.** Uma empresa precisa que um servidor on-premises acesse arquivos armazenados no S3 usando protocolo NFS/SMB com cache local para baixa latência. Qual serviço?

- A) AWS DataSync
- B) Amazon EFS com VPN
- C) AWS Storage Gateway — File Gateway
- D) AWS Storage Gateway — Volume Gateway (Stored mode)

<details>
<summary>Resposta</summary>
**C — File Gateway**
File Gateway expõe buckets S3 como file shares NFS/SMB para aplicações on-premises. Cache local mantém arquivos recentes para baixa latência. Dados são armazenados no S3 como objetos. DataSync é para migração em lote, não acesso contínuo. Volume Gateway trabalha com block storage (iSCSI).
</details>

---

**13.** Qual afirmação sobre S3 Lifecycle Policies é VERDADEIRA?

- A) Objetos podem transitar de S3 Standard diretamente para S3 Glacier Deep Archive sem passar por outros tiers
- B) Objetos só podem transitar para tiers mais baratos (não é possível voltar para Standard)
- C) Para transitar para Standard-IA ou One Zone-IA, os objetos devem ter pelo menos 180 dias
- D) S3 Intelligent-Tiering não pode ser destino de uma Lifecycle Policy

<details>
<summary>Resposta</summary>
**A — Correto**
Lifecycle Policies permitem transições diretas entre quaisquer classes (ex: Standard → Deep Archive diretamente). B é verdade (lifecycle só transita para tiers mais baratos; restaurar é manual). C está errado — o mínimo para IA tiers é **30 dias**. D está errado — Intelligent-Tiering pode ser destino.
</details>

---

**14.** Um bucket S3 recebe 100.000 PUTs por segundo distribuídos por 10 prefixos diferentes. Qual é o throughput esperado por prefixo?

- A) 100 PUT/s por prefixo
- B) 3.500 PUT/s por prefixo
- C) 5.500 GET/s por prefixo
- D) 10.000 PUT/s por prefixo (auto-scaling)

<details>
<summary>Resposta</summary>
**B — 3.500 PUT/s por prefixo**
O S3 suporta **3.500 PUT/COPY/POST/DELETE** e **5.500 GET/HEAD** por segundo **por prefixo**. Distribuindo a carga por 10 prefixos diferentes, você consegue 35.000 PUTs/s total. Isso é linear — mais prefixos = mais throughput.
</details>

---

**15.** Uma empresa quer receber uma notificação imediata quando um arquivo específico for uploaded para S3. Qual configuração usar?

- A) S3 Inventory com relatório diário
- B) CloudTrail com filtro de eventos S3
- C) S3 Event Notifications → SNS ou Lambda
- D) EventBridge Scheduler com polling do bucket

<details>
<summary>Resposta</summary>
**C — S3 Event Notifications**
S3 Event Notifications dispara eventos em tempo real para SNS, SQS ou Lambda quando objetos são criados, deletados, restaurados, etc. É a forma mais direta e de menor latência. CloudTrail registra chamadas API mas não é para "notificação imediata". EventBridge também pode ser usado (S3 integra com EventBridge) mas a forma mais direta é S3 Event Notifications → Lambda.
</details>
