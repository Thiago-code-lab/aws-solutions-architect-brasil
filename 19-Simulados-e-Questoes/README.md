# Módulo 19 — Simulados e Questões

## Estratégia para o Exame SAA-C03

### Formato do Exame
- **65 questões** (múltipla escolha/múltipla seleção)
- **130 minutos** (2h10min)
- **Score mínimo**: 720/1000 (escala de pontuação de 100-1000)
- **Domínios cobertos:**
  - Design de Arquiteturas Resilientes: 26%
  - Design de Arquiteturas de Alta Performance: 24%
  - Design de Aplicações Seguras: 30%
  - Design de Arquiteturas Otimizadas em Custo: 20%

### Estratégia de Resposta
1. Leia o **requisito principal** destacado na questão (geralmente em negrito ou citado explicitamente)
2. Elimine **distractors** óbvios (serviços errados para o contexto)
3. Escolha a resposta **mais gerenciada** que satisfaz **todos** os requisitos
4. Para questões de múltipla seleção: identifique quantas respostas são pedidas (2 ou 3)
5. Tempo por questão: **~2 minutos**; marque questões difíceis para revisão

---

## Simulado Completo 1 — 65 Questões

### Seção A — Arquiteturas Resilientes (Questões 1-17)

**1.** Uma empresa precisa executar um banco de dados MySQL com failover automático entre Zonas de Disponibilidade e que suporte leituras de alta carga. Qual combinação de serviços resolve isso?
- A) RDS MySQL Single-AZ com Read Replicas
- B) Amazon Aurora MySQL Multi-AZ com Aurora Read Replicas
- C) Amazon RDS MySQL Multi-AZ com Read Replicas
- D) Amazon DynamoDB com DAX

<details><summary>Resposta</summary>

**B** — Aurora MySQL Multi-AZ tem failover automático em ~30s (muito mais rápido que RDS Multi-AZ) E suporta até 15 Read Replicas de baixa latência. Aurora Read Replicas fazem parte do cluster e promovem automaticamente em caso de failover.

</details>

---

**2.** Uma aplicação crítica em EC2 precisa de RTO < 15 minutos e RPO < 1 hora. Qual estratégia de DR na AWS é mais econômica para esses requisitos?
- A) Multi-Site Active-Active em duas regiões
- B) Backup & Restore com snapshots automáticos no S3
- C) Warm Standby com capacidade reduzida em uma segunda região
- D) Pilot Light com componentes mínimos em segunda região

<details><summary>Resposta</summary>

**D** — Pilot Light: mantém apenas os componentes essenciais (BD replicado, AMIs prontas) rodando na segunda região em estado mínimo. RTO de 10-15 min (provisionar o restante) com custo bem menor que Warm Standby. RPO depende da frequência de replicação; com Aurora Global DB ou DMS CDC, RPO < 1h é factível.

</details>

---

**3.** Uma empresa tem um endpoint crítico. Quando a região primária fica indisponível, o tráfego deve ser roteado automaticamente para uma segunda região. Qual serviço implementa isso?
- A) AWS Global Accelerator com endpoint groups
- B) Amazon Route 53 com Failover routing
- C) Amazon CloudFront com origin failover
- D) A e B são ambas válidas

<details><summary>Resposta</summary>

**D** — Route 53 Failover routing monitora health checks e muda automaticamente o DNS para o endpoint secundário. Global Accelerator também faz failover automático entre regiões via anycast. Ambos são válidos; Route 53 atua no nível DNS, Global Accelerator atua no nível de rede (menor latência de failover).

</details>

---

**4.** Uma fila SQS está recebendo mensagens de alta prioridade misturadas com mensagens de baixa prioridade. As mensagens de alta prioridade precisam ser processadas imediatamente. O que fazer?
- A) Configurar message attributes e filtrar no consumer
- B) Criar duas filas SQS separadas e direcionar mensagens por prioridade
- C) Usar SQS FIFO para garantir ordem de processamento
- D) Usar SNS Message Filtering para separar as mensagens

<details><summary>Resposta</summary>

**B** — SQS não tem suporte nativo a prioridade de mensagens. A solução é criar filas separadas (alta e baixa prioridade) e ter workers que verificam primeiro a fila de alta prioridade. SQS FIFO garante ordem mas não prioridade entre mensagens.

</details>

---

**5.** Uma empresa precisa de um sistema que garanta que cada mensagem seja processada exatamente uma vez, mesmo com reativações de consumers. Qual fila AWS usar?
- A) Amazon SQS Standard
- B) Amazon SQS FIFO com deduplication ID
- C) Amazon SNS FIFO
- D) Amazon Kinesis Data Streams

<details><summary>Resposta</summary>

**B** — SQS FIFO com Message Deduplication ID garante exatamente uma entrega (exactly-once processing). SQS Standard garante ao menos uma entrega (duplicatas possíveis). Kinesis pode ter ao menos uma entrega dependendo da configuração do consumer.

</details>

---

**6.** Uma aplicação web usa ALB + Auto Scaling Group + RDS Aurora. Durante picos, o banco de dados fica sobrecarregado com leituras. A solução mais custo-efetiva para escalar leituras é:
- A) Aumentar a classe da instância RDS principal
- B) Adicionar Aurora Read Replicas e configurar a string de conexão da aplicação para o endpoint Reader
- C) Implementar ElastiCache Redis como cache de sessão
- D) Migrar para DynamoDB com DAX

<details><summary>Resposta</summary>

**B** — Aurora Read Replicas escalam leituras horizontalmente com latência mínima; o Reader Endpoint distribui automaticamente entre réplicas. É mais econômico e direto que as outras opções para cargas de leitura SQL existentes.

</details>

---

**7.** Uma empresa quer garantir que suas funções Lambda críticas sempre tenham capacidade disponível sem cold start. Qual configuração implementa isso?
- A) Reserved Concurrency com valor alto
- B) Provisioned Concurrency em uma version ou alias da Lambda
- C) Aumentar a memória alocada da Lambda para 10 GB
- D) Configurar Lambda em VPC para reduzir cold starts

<details><summary>Resposta</summary>

**B** — Provisioned Concurrency pré-aquece N execution environments, eliminando cold start completamente para esses N concurrents. Reserved Concurrency garante capacidade mas não elimina cold start. VPC não ajuda com cold start (Hyperplane ENI desde 2019 minimizou esse efeito).

</details>

---

**8.** Uma instância EC2 falha e precisa ser substituída automaticamente pelo mesmo endereço IP privado e armazenamento dos dados. Qual solução implementa isso?
- A) Auto Scaling Group com min/max/desired = 1
- B) EC2 Instance Recovery via CloudWatch Alarm
- C) AMI automática + CloudFormation
- D) Instância EC2 com Placement Group

<details><summary>Resposta</summary>

**B** — EC2 Instance Recovery (alarm action "Recover Instance") move a instância para novo hardware mantendo o mesmo IP privado, Elastic IP, instance ID e dados EP (EBS). ASG cria nova instância com IPs novos.

</details>

---

**9.** Uma empresa deve garantir que objetos S3 críticos sejam replicados automaticamente para outra região, e que os metadados e versões também sejam replicados. O que configurar?
- A) S3 Transfer Acceleration
- B) S3 Cross-Region Replication (CRR) com versioning habilitado nos dois buckets
- C) S3 Same-Region Replication (SRR) com Lambda
- D) Evento S3 → Lambda → CopyObject para outro bucket

<details><summary>Resposta</summary>

**B** — CRR requer versioning habilitado em ambos os buckets (source e destination). Replica automaticamente novos objetos e versões para a região de destino. SRR é dentro da mesma região.

</details>

---

**10.** Uma aplicação usa DynamoDB e começa a receber erros `ProvisionedThroughputExceededException`. A empresa quer escalar automaticamente sem gerenciar manualmente RCU/WCU. O que implementar?
- A) Aumentar manualmente as RCU/WCU provisionadas
- B) Migrar para DynamoDB On-Demand mode
- C) Adicionar DAX na frente do DynamoDB
- D) Migrar para RDS para suporte a mais capacidade

<details><summary>Resposta</summary>

**B** — On-Demand mode escala automaticamente com base no tráfego; não há throttling por capacidade provisionada. DynamoDB Auto Scaling (com provisioned mode) também funciona mas tem delay de resposta. On-Demand tem custo por request (mais caro para carga previsível alta, ideal para carga variável/imprevista).

</details>

---

**11.** Uma empresa tem um S3 bucket com dados sensíveis e precisa garantir que nenhum objeto seja excluído acidentalmente por 7 anos. Qual configuração implementa isso?
- A) S3 Versioning + MFA Delete
- B) S3 Object Lock em modo Compliance com retention period de 7 anos
- C) S3 Glacier Vault Lock com policy de retenção
- D) S3 Bucket Policy negando DeleteObject

<details><summary>Resposta</summary>

**B** — S3 Object Lock em modo Compliance: **nem o root account** pode excluir ou modificar objetos durante o retention period. Perfeito para WORM compliance. Modo Governance permite que admins com permissão especial sobrescrevam. Bucket Policy pode ser alterada por admin; MFA Delete não impede todos os tipos de exclusão.

</details>

---

**12.** Uma aplicação de microserviços em ECS precisa que cada container acesse um secret de banco de dados armazenado no AWS Secrets Manager. Qual é a forma mais segura de fornecer o segredo ao container?
- A) Passar o segredo como variável de ambiente no task definition
- B) Usar a integração nativa ECS + Secrets Manager no task definition com `secrets` field
- C) Armazenar o segredo no S3 criptografado e a Lambda busca antes do container iniciar
- D) Embutir o segredo no container image

<details><summary>Resposta</summary>

**B** — ECS suporta injeção de secrets do Secrets Manager e Parameter Store diretamente em variáveis de ambiente ou como volumes — o ECS agent busca o valor na execução. O secret nunca fica exposto na imagem ou task definition em texto claro. A task execution role precisa de permissão para `secretsmanager:GetSecretValue`.

</details>

---

**13.** Uma solução de analytics processa grandes arquivos CSV de 10 GB no S3. As queries via Athena são lentas e caras. Qual melhoria de uma única ação teria mais impacto?
- A) Converter arquivos para Parquet e particionar por data
- B) Mover para Redshift para queries mais rápidas
- C) Aumentar o tamanho do bloco no S3
- D) Usar S3 Select para filtrar antes de enviar ao Athena

<details><summary>Resposta</summary>

**A** — Parquet é formato colunar comprimido: Athena lê apenas as colunas necessárias e escaneia muito menos dados. Particionamento por data evita full table scans. Combinados, podem reduzir o custo e tempo de query em 90%+. Esta é a otimização mais impactante para Athena.

</details>

---

**14.** Uma empresa precisa que os logs dos Lambda e ECS containers sejam centralizados, com retenção de 90 dias e possibilidade de queries analíticas. Qual solução implementar?
- A) Kinesis Firehose → S3 → Athena
- B) CloudWatch Logs com grupo de logs, retenção de 90 dias e CloudWatch Logs Insights
- C) Elasticsearch (OpenSearch) com agente de log
- D) CloudWatch Logs Subscription → Kinesis Firehose → S3 + Athena

<details><summary>Resposta</summary>

**B** — Lambda e ECS enviam logs automaticamente para CloudWatch Logs. Configurar retenção de 90 dias por grupo. CloudWatch Logs Insights permite queries analíticas nos logs diretamente. Solução mais simples e custo-efetiva para o caso descrito.

</details>

---

**15.** Uma instância EC2 em subnet privada precisa fazer chamadas a uma API externa pela internet sem expor IP privado. O que configurar?
- A) Adicionar um Elastic IP à instância
- B) Criar um NAT Gateway em subnet pública e adicionar rota 0.0.0.0/0 à route table da subnet privada
- C) Configurar VPC Peering com uma VPC pública
- D) Habilitar DNS resolution na VPC

<details><summary>Resposta</summary>

**B** — NAT Gateway em subnet pública (com IGW) permite que instâncias em subnets privadas iniciem conexões para a internet. O NAT traduz o IP privado para o IP do NAT Gateway. Elastic IP na instância privada não funciona sem IGW na route table.

</details>

---

**16.** Uma empresa quer garantir que apenas requests assinadas com SigV4 possam acessar seu S3 bucket. Qual policy aplicar no bucket?
- A) Bucket Policy com `"Effect": "Deny"` para `"Principal": "*"` sem condição `aws:SecureTransport`
- B) Bucket Policy com Condition `"StringEquals": {"s3:authType": "REST-QUERY-STRING"}`
- C) Bucket ACL configurando acesso privado
- D) Amazon Macie para monitorar acessos não autorizados

<details><summary>Resposta</summary>

**A** — A condição correta é `"Condition": {"Bool": {"aws:SecureTransport": "false"}}` com Deny para forçar HTTPS. Para forçar SigV4 (autenticação AWS), remover acesso público + ter IAM policies apenas para identidades AWS. Amazon Macie é detecção, não controle de acesso.

</details>

---

**17.** Uma empresa usa AWS Organizations com múltiplas contas. Precisa impedir que desenvolvedores em contas-membro criem recursos em regiões não aprovadas. Qual serviço implementar?
- A) IAM Permission Boundaries em cada conta membro
- B) Service Control Policy (SCP) no Organizations OU aplicando Deny para regiões não aprovadas
- C) AWS Config Rules em cada conta membro
- D) IAM Role com condição de região em cada conta membro

<details><summary>Resposta</summary>

**B** — SCPs são aplicadas no nível de OU ou conta no Organizations e limitam o que qualquer identity (mesmo root) pode fazer nas contas membros. Uma SCP `Deny + Condition: aws:RequestedRegion NotEquals [us-east-1, us-west-2]` bloqueia todas as ações em regiões não listadas.

</details>

---

### Seção B — Performance (Questões 18-33) — Referência Rápida

| # | Área | Resposta Esperada |
|---|---|---|
| 18 | EC2 high compute workload | C5/C6i, placement group cluster |
| 19 | Baixa latência inter-serviços mesma AZ | Placement Group Cluster |
| 20 | Alta IOPS para banco de dados | EBS io2 Block Express |
| 21 | Conteúdo estático global baixa latência | CloudFront |
| 22 | TCP/UDP aceleração global | Global Accelerator |
| 23 | Cache em memória para leituras repetidas | ElastiCache Redis |
| 24 | DynamoDB leitura µs | DAX (DynamoDB Accelerator) |
| 25 | ECS tasks comunicação interna | Service Connect ou Service Discovery (Cloud Map) |
| 26 | Lambda com grande dependência Python | Lambda Container Image (até 10 GB) |
| 27 | Queries S3 sem mover dados | Amazon Athena |
| 28 | Upload S3 arquivos grandes otimizado | Multipart Upload + Transfer Acceleration |
| 29 | EFS alta performance (HPC) | EFS Max I/O ou FSx for Lustre |
| 30 | Redshift queries lentas em concurrent users | Concurrency Scaling |
| 31 | API GW throttling para proteger backend | Usage Plans + Rate Limits |
| 32 | Graviton instances | Melhor preço/performance para cargas Cloud-native |
| 33 | Lambda cold start Java critico | SnapStart (Java 11+) ou Provisioned Concurrency |

---

### Seção C — Segurança (Questões 34-54) — Referência Rápida

| # | Área | Resposta Esperada |
|---|---|---|
| 34 | Detectar instâncias comprometidas | GuardDuty |
| 35 | Detectar dados sensíveis no S3 | Macie |
| 36 | Scan de vulnerabilidades EC2/Lambda | Inspector |
| 37 | Cross-account role | sts:AssumeRole com trust policy |
| 38 | Rotação automática de secrets RDS | Secrets Manager com rotation Lambda |
| 39 | Criptografia dados DynamoDB em repouso | AWS KMS CMK (Customer Managed Key) |
| 40 | Bloquear SQL injection na API | WAF com AWS Managed Rules |
| 41 | Acesso EC2 sem key pair | SSM Session Manager |
| 42 | S3 bucket policy vs IAM policy | S3 Bucket Policy para cross-account; IAM para same-account |
| 43 | CloudFront + S3 — prevent direct S3 access | Origin Access Control (OAC) |
| 44 | Compliance PCI DSS na AWS | Shared Responsibility + AWS Artifact (reports) |
| 45 | Encrypt EBS volume existente | Snapshot → Encrypted Copy → Restore |
| 46 | Auditar quais APIs foram chamadas | CloudTrail (Management Events) |
| 47 | Multitenancy — isolar dados por tenant | DynamoDB com partition key por tenant + Resource Policy |
| 48 | EKS pod permissions | IRSA (IAM Roles for Service Accounts) |
| 49 | Key rotation automática | KMS Key Rotation anual (AWS Managed Keys auto; CMK configurável) |
| 50 | VPC endpoint S3 sem internet | Gateway VPC Endpoint para S3 (gratuito) |
| 51 | Logs de acesso ao S3 | S3 Server Access Logging ou CloudTrail Data Events |
| 52 | Prevent accidental deletion CloudFormation | Termination Protection + DeletionPolicy: Retain |
| 53 | ACM certificado renovação | Automático para ACM-provisioned certificates |
| 54 | Network Firewall vs WAF | Network Firewall = Layer 4 (VPC nível); WAF = Layer 7 (HTTP) |

---

### Seção D — Custo (Questões 55-65) — Referência Rápida

| # | Área | Resposta Esperada |
|---|---|---|
| 55 | Instâncias não prod fora do horário comercial | Evento EventBridge → Lambda → Stop/Start EC2 |
| 56 | Carga de trabalho batch tolerante a interrupção | Spot Instances |
| 57 | Comprometimento de longo prazo conhecido | Reserved Instances ou Savings Plans |
| 58 | Rightsizing EC2 | Compute Optimizer |
| 59 | S3 acesso imprevisível | S3 Intelligent-Tiering |
| 60 | Transferência de dados — custo zero | Dados para EC2 na mesma região são gratuitos; saída tem custo |
| 61 | Redshift vs Athena — quando usar | Athena para ad-hoc; Redshift para DW analítico recorrente |
| 62 | EFS vs EBS custo | EBS mais barato mas single-instance; EFS multi-instance mas mais caro/GB |
| 63 | Lambda vs EC2 para workload esporádico | Lambda mais econômico (paga por invocação, sem custo idle) |
| 64 | NAT GW custo | ~$0.045/hora + $0.045/GB processado; use VPC Endpoint para S3/DynamoDB (gratuito) |
| 65 | Multi-AZ RDS custo | ~2x custo de Single-AZ; necessário para HA; não economize nisso em produção |

---

## Gabarito Resumido — Simulado 1

| Q | A | Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|---|---|
| 1 | B | 14 | B | 27 | Athena | 40 | WAF | 53 | Automático |
| 2 | D | 15 | B | 28 | Multipart | 41 | SSM | 54 | Network FW |
| 3 | D | 16 | A | 29 | FSx Lustre | 42 | Bucket Policy | 55 | EventBridge |
| 4 | B | 17 | B | 30 | Concurrency | 43 | OAC | 56 | Spot |
| 5 | B | 18 | C5/C6i | 31 | Usage Plans | 44 | Artifact | 57 | RI/SP |
| 6 | B | 19 | Cluster PG | 32 | Graviton | 45 | Snapshot | 58 | Optimizer |
| 7 | B | 20 | io2 | 33 | SnapStart | 46 | CloudTrail | 59 | Int-Tiering |
| 8 | B | 21 | CloudFront | 34 | GuardDuty | 47 | Partition key | 60 | Gratuito |
| 9 | B | 22 | Global Acc | 35 | Macie | 48 | IRSA | 61 | Redshift DW |
| 10 | B | 23 | ElastiCache | 36 | Inspector | 49 | KMS Rotation | 62 | EBS mais barato |
| 11 | B | 24 | DAX | 37 | AssumeRole | 50 | GW Endpoint | 63 | Lambda |
| 12 | B | 25 | Service Conn | 38 | Secrets Mgr | 51 | S3 Logging | 64 | VPC Endpoint |
| 13 | A | 26 | Container | 39 | CMK | 52 | Termination P | 65 | Multi-AZ prod |
