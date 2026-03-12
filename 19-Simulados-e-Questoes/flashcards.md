# Flashcards — Módulo 19: Simulados e Questões

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards — estratégia de prova e revisão de padrões

---

**P:** Quais são os 4 domínios do SAA-C03 e seus pesos?  
**R:** **(1) Design Resilient Architectures: 26%** — HA, DR, decoupling. **(2) Design High-Performing Architectures: 24%** — scaling, caching, databases. **(3) Design Secure Applications and Architectures: 30%** — IAM, encryption, network security. **(4) Design Cost-Optimized Architectures: 20%** — EC2 pricing, storage tiers, serverless. Segurança é o maior domínio.

---

**P:** Como identificar no exame quando usar Lambda vs ECS vs EC2?  
**R:** **Lambda:** stateless, <15 min, responde a eventos, escala automática. **ECS/Fargate:** containerizado, >15 min possível, microservices, precisa de Docker. **EC2:** controle total de OS, software específico (licença, GPU), carga contínua, SSH access, compliance OS-level. Pistas: "sem gerenciar servidores" → Lambda/Fargate; "lift-and-shift" → EC2.

---

**P:** Como identificar quando usar RDS vs DynamoDB no exame?  
**R:** **RDS:** dados relacionais, ACID transactions, SQL complexo com JOINs, schema fixo, aplicação existente que usa SQL. **DynamoDB:** alta escala (milhões de req/s), schema flexível, latência de milissegundos, serverless, key-value/document model, evitar JOINs. Pista no exame: "relational" → RDS; "milisseconds at any scale" / "key-value" → DynamoDB.

---

**P:** Quando usar SQS vs SNS vs EventBridge no exame?  
**R:** **SQS:** fila ponto-a-ponto, decoupling, processamento assíncrono durável, controle de throughput. **SNS:** publicar para múltiplos subscribers simultaneamente (fan-out), notificações push (email, SMS, Lambda). **EventBridge:** roteamento baseado em event patterns, integração com SaaS, scheduler, audit trail de events. SQS+SNS = fan-out com buffering.

---

**P:** Qual é a pista no texto da questão que indica usar ElastiCache?  
**R:** Palavras-chave: "slow database queries", "repetidas queries iguais", "session data", "leaderboard", "real-time", "sub-millisecond latency", "hot data". Redis: quando precisa de data structures (sets, sorted sets para leaderboards), pub/sub, persistência opcional. Memcached: cache simples multi-threaded, sem persistência necessária.

---

**P:** Quando usar Route 53 Failover vs Latency vs Weighted Routing?  
**R:** **Failover:** active/passive DR — tráfego vai para primary, failover automático para secondary se health check falha. **Latency:** rota usuários para a região de menor latência (performance global). **Weighted:** A/B testing, distribuição gradual de tráfego (canary). **Geolocation:** compliance — usuários da EU vai apenas para região EU. **Multi-Value:** múltiplos registros saudáveis retornados (não é load balancer real).

---

**P:** Como interpretar "custo mais baixo" com armazenamento S3 no exame?  
**R:** Pensar no padrão de acesso: **Standard** = frequente (qualquer frequência). **Standard-IA** = acesso infrequente mas retrieval rápido quando necessário. **Glacier Instant** = acesso ocasional (milissegundos). **Glacier Flexible** = acesso raro (<1/mês), 1-12h retrieval. **Deep Archive** = < acesso anual, 12-48h retrieval. Lower cost to store, higher cost to retrieve.

---

**P:** O que fazer quando a questão pede "mais seguro" para acesso a S3?  
**R:** Hierarquia de segurança S3: (1) **Block Public Access** (nível de conta/bucket). (2) **Bucket Policy** com Deny ou Allow granular. (3) **IAM Policy** para usuários/roles. (4) **Pre-signed URLs** para acesso temporário. (5) **OAC** (Origin Access Control) para CloudFront. (6) **Object Lock** para WORM. Para "mais seguro": Block Public Access + OAC para CloudFront, sem ACLs.

---

**P:** Por que VPC Peering não é sempre a melhor solução para conectar VPCs?  
**R:** **Limitações do VPC Peering:** (1) Não transitive — A↔B e B↔C não implica A↔C (precisa peering A↔C também). (2) CIDR ranges não podem sobrepor. (3) Para N VPCs: N*(N-1)/2 peerings (escala mal). **Alternativa para hub-and-spoke:** AWS Transit Gateway — conecta dezenas/centenas de VPCs e VPNs via hub central. Exame testa: muitas VPCs = Transit Gateway; poucas VPCs = peering.

---

**P:** Como identificar quando usar CloudFront vs ALB no exame?  
**R:** **CloudFront:** conteúdo global (assets, API global), cache de conteúdo estático/dinâmico, proteção DDoS (Shield Standard grátis), S3 origin, Lambda@Edge. **ALB:** load balancing regional dentro de uma VPC, routing por path/host, target groups (EC2, Lambda, IP), WebSockets, integração com WAF regional. Pista: "global" + "cache" → CloudFront; "regional" + "routing" → ALB.

---

**P:** Quando um SG (Security Group) é mais adequado que NACL?  
**R:** **SG é melhor quando:** controle no nível de instância, stateful (retorno automático), aceita referência de outros SGs (ex: "permitir do SG do ALB"). **NACL é melhor quando:** precisa de regras DENY explícitas (ex: bloquear IP específico), segurança no nível de subnet, compliance que exige ACL. Exame: "bloquear IP malicioso" → NACL Deny rule (SG não tem Deny).

---

**P:** O que significa "decoupled" em arquitetura no contexto do exame?  
**R:** Componentes que operam independentemente sem chamadas síncronas diretas. Se um componente falha ou fica lento, não cascadeia para outros. Implementação: **SQS** (buffering assíncrono), **SNS** (pub/sub), **EventBridge** (event-driven). Pista no exame: "tightly coupled" → solução com SQS/SNS para decoupling. "Escalável independentemente" → decoupled.

---

**P:** Qual é a diferença entre KMS CMK e KMS AWS Managed Key?  
**R:** **AWS Managed Key** (ex: `aws/s3`): rotação anual automática, cada serviço tem a sua, sem custo adicional, sem controle de policy. **Customer Managed Key (CMK):** você controla a key policy, auditável no CloudTrail, pode desabilitar, custo (~$1/mês), rotação configurável. Exame: "audit key usage" ou "controlar quem usa a chave" → CMK. Simples encryption sem controle → AWS Managed Key.

---

**P:** Quando usar SSM Secrets Manager vs SSM Parameter Store?  
**R:** **Secrets Manager:** rotação automática nativa (Lambda, RDS, Redshift, DocumentDB), versionamento, integração direta com RDS. Mais caro (~$0.40/secret/mês). **Parameter Store Standard:** gratuito, sem rotação automática nativa, simples configuração (não-secret). **Parameter Store Advanced:** suporta Parameter Policies (TTL, alertas). Se a questão menciona "automatic rotation" → Secrets Manager.

---

**P:** O que identificar primeiro ao ler uma questão do exame?  
**R:** Sequência de leitura: **(1)** Última frase — geralmente especifica o requisito real ("qual é a solução mais custo-efetiva?"). **(2)** Restrições explícitas — "sem gerenciar servidores", "menor mudança de código", "RPO < 1 hora". **(3)** Contexto — tipo de dado, volume, frequência de acesso. **(4)** Eliminação — qual opção viola DIRETAMENTE um requisito?

---

**P:** Como o exame testa conhecimento de Multi-AZ vs Multi-Region?  
**R:** **Multi-AZ:** proteção contra falha de AZ (disponibilidade), mesma região, latência low, failover automático (RDS Multi-AZ, ALB, Aurora). **Multi-Region:** proteção contra falha de região (DR), latência mais alta, geralmente latency routing ou failover com Route 53. Exame: "datacenter failure" → Multi-AZ. "region failure" / "disaster recovery" → Multi-Region.

---

**P:** Qual é a diferença entre Active-Active e Active-Passive multi-region?  
**R:** **Active-Active:** ambas regiões recebem tráfego simultaneamente (Route 53 Latency/Weighted). Sem downtime em failover. Aurora Global Database (escrita em primária, leitura em qualquer). **Active-Passive:** só a região primária recebe tráfego. Failover manual ou automático para secundária se primária falha (Route 53 Failover). Active-Active = menor RTO, maior custo.

---

**P:** Qual pista leva a escolher Aurora em vez de RDS MySQL?  
**R:** Pistas para Aurora: "leitura de alta performance" (Read Replicas até 15, vs 5 do RDS), "escala automática" (Aurora Serverless), "failover <30s" (Aurora retém dados em cluster volume distribuído, vs Multi-AZ ~1-2 min), "multi-region" (Aurora Global Database), "zero-downtime patching". Custo: Aurora ~20% mais caro que RDS MySQL, mas equivalente com mais features.

---

**P:** Como reconhecer um problema de cold start Lambda no exame?  
**R:** Pistas: "latência inconsistente", "primeira request lenta", "picos esporádicos de latência", "Java + Lambda". Solução no exame: **Provisioned Concurrency** (elimina cold start, instâncias pré-aquecidas) ou **SnapStart** (Java 11+). NOT a solução: aumentar memória (reduz duration mas não elimina cold start), Arm64/Graviton (melhor custo/perf mas não resolve cold start).

---

**P:** Quais são as condições IAM mais testadas no exame?  
**R:** **aws:RequestedRegion** — restringe a região. **aws:PrincipalTag** — baseado em tag do usuário/role. **s3:prefix** — prefix do S3. **aws:SourceVpc/VPCe** — restringe acesso a recursos dentro de VPC/VPC Endpoint. **aws:MultiFactorAuthPresent** — exige MFA. **StringLike/StringEquals** — comparação com wildcards. Exame testa: "apenas da VPC" → `aws:SourceVpc` condition.

---

**P:** O que é uma SCP (Service Control Policy) e quando usar?  
**R:** Política no AWS Organizations que define **limite máximo de permissões** para contas membro — mesmo que uma conta tenha IAM Admin, SCP pode bloquear ações (ex: proibir ações fora de us-east-1). SCP não concede permissões — apenas restringe. Casos: conformidade multi-conta (proibir regiões não aprovadas, proibir deletar CloudTrail). SCP + account IAM = effective permissions.

---

**P:** Como eliminar opções wrongas em questões de segurança do exame?  
**R:** Elimine automaticamente: (1) **Access Keys hardcoded** em código. (2) **Credenciais em variáveis de ambiente** quando IAM Roles são possíveis. (3) Portas **0.0.0.0/0 abertas** desnecessariamente. (4) S3 **Public Read ACL** quando há alternativa. (5) Dados sensíveis em **texto plano** (plaintext). AWS security principle: assume role > access keys; least privilege; encrypt by default.

---

**P:** O que o exame testa sobre EBS vs EFS vs S3?  
**R:** **EBS:** block storage montado em uma EC2, single-AZ (exceto io2 Multi-Attach), alta IOPS, banco de dados, OS. **EFS:** file system NFS compartilhado entre múltiplas EC2, multi-AZ, escalável automaticamente. **S3:** object storage, acesso via HTTP API, não montável como filesystem (EFS/EBS são), praticamente ilimitado, ideal para dados não-estruturados, backup, data lake.

---

**P:** Quando usar AWS Batch vs Lambda para processamento em lote?  
**R:** **Lambda:** jobs curtos (<15 min), event-driven, pequenos. **AWS Batch:** jobs longos (horas/dias), computação pesada (HPC, ML training, rendering), precisa de GPU, grande volume de jobs paralelos. Gerencia: provisiona EC2/Fargate dynamicamente, fila de jobs, prioridades. Ideal para cargas que excedem limites do Lambda.

---

**P:** Como o exame testa o princípio de least privilege?  
**R:** Cenários: **(1)** EC2 precisa ler apenas um bucket S3 → IAM Role com `s3:GetObject` naquele bucket específico (não `s3:*` ou AdministratorAccess). **(2)** Lambda precisa escrever no DynamoDB → `dynamodb:PutItem` na tabela específica (não full access). **(3)** Cross-account: trust policy + permission policy combinados. Sempre: **mínimo necessário para o mínimo de tempo necessário**.
