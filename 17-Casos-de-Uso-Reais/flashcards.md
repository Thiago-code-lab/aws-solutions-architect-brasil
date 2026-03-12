# Flashcards — Módulo 17: Casos de Uso Reais

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** O que é o padrão "Strangler Fig" e como implementar na AWS?  
**R:** Padrão de migração gradual de monólito para microsserviços. Novas funcionalidades são construídas como microsserviços; funcionalidades antigas são migradas incrementalmente. Na AWS: usar API Gateway ou Application Load Balancer para rotear tráfego — `/nova-feature` → Lambda/ECS, `/legado/*` → monólito EC2. Gradual sem big-bang rewrite.

---

**P:** Como arquitetar uma aplicação de e-commerce altamente disponível na AWS?  
**R:** Frontend: CloudFront + S3 (SPA). Backend: ALB + ECS/EC2 em 2+ AZs com Auto Scaling. Banco: RDS Aurora Multi-AZ (escrita) + Read Replicas (leitura). Cache: ElastiCache (sessões, produtos populares). Carrinho/estoque: DynamoDB (alta escala). Pagamentos: SQS (exactly-once via deduplicationId). imagens: S3 + CloudFront.

---

**P:** Qual é o padrão fan-out e quando usar na AWS?  
**R:** Fan-out: um evento dispara múltiplos consumidores em paralelo. Implementação: SNS Topic → múltiplas SQS Queues (ou Lambda). Cada subscriber processa independentemente. Use quando: um evento precisa acionar múltiplas ações (order_placed → SQS_email + SQS_fulfillment + SQS_analytics + SQS_audit). Desacopla publisher de múltiplos consumers.

---

**P:** Como implementar exactly-once processing em pagamentos na AWS?  
**R:** SQS FIFO Queue com **MessageDeduplicationId** (hash do transactionId). Se a mesma transação é enviada duas vezes com o mesmo DeduplicationId dentro da janela de 5 minutos, apenas uma é entregue. No processamento: verificação idempotente no banco (check se transactionId já foi processado antes de debitar).

---

**P:** Qual arquitetura para um pipeline de IoT com 100.000 dispositivos?  
**R:** Dispositivos → IoT Core (MQTT) → Rules Engine → múltiplos destinos: Kinesis Data Streams (streaming analytics), DynamoDB (últimas leituras), S3 (via Kinesis Firehose, dados históricos). Para alertas em tempo real: Lambda Consumer no KDS. Para análises históricas: Athena no S3. Visualização: QuickSight ou Grafana.

---

**P:** Como usar o padrão CQRS (Command Query Responsibility Segregation) na AWS?  
**R:** Separar escrita e leitura em modelos/stores diferentes. Write path (Command): API Gateway → Lambda → DynamoDB (source of truth). Read path (Query): dados desnormalizados em ElastiCache ou OpenSearch para leitura rápida. Sincronização: DynamoDB Streams → Lambda → atualiza read model. Write otimizado para consistência; read otimizado para performance.

---

**P:** Como arquitetar um Data Lake serverless na AWS?  
**R:** Ingestão: Kinesis Firehose (streaming) + DataSync (batch). Storage: S3 com particionamento por data. Catalog: Glue Crawler → Glue Data Catalog. Transform: Glue ETL Jobs (raw → curated → analytics zones). Query: Athena (SQL). Segurança: Lake Formation (FGA por coluna/row). Visualização: QuickSight. Orquestração: Step Functions ou MWAA.

---

**P:** O que é Event Sourcing e como implementar na AWS?  
**R:** Em vez de salvar state atual, salvar todos os eventos que levaram ao estado: `order_created`, `payment_processed`, `shipped`. State atual é rebuilt replaying events. AWS: DynamoDB Streams ou Kinesis como event log. EventBridge para publicar eventos. Vantagem: auditoria completa, temporal queries (estado em qualquer ponto no tempo), debugging.

---

**P:** Como implementar autenticação com Cognito em uma SPA + S3?  
**R:** (1) Cognito User Pool para autenticação (username/password, OAuth social). (2) Cognito Identity Pool para trocar token do User Pool por credenciais AWS temporárias (IAM Role assumida). (3) Frontend usa credenciais temporárias para chamar API Gateway (autenticado via Cognito Authorizer) ou acessar S3 diretamente. Sem backend de autenticação para gerenciar.

---

**P:** Qual arquitetura para uma plataforma de streaming de vídeo (Netflix-like)?  
**R:** Upload: S3 + pre-signed URL. Transcodagem: S3 Event → SQS → EC2/ECS com MediaConvert ou Elemental. CDN: CloudFront com origens S3 (vídeo processado). DRM: CloudFront Signed URLs ou Signed Cookies. Metadados: DynamoDB ou Aurora. Recomendações: SageMaker Personalize. Analytics: Kinesis → Redshift/Athena.

---

**P:** Como implementar multi-tenant com isolamento por tenant na AWS?  
**R:** Estratégias: **(1) Silo model:** conta AWS por tenant (máximo isolamento, custo administrativo alto). **(2) VPC por tenant:** compartilha conta, VPCs separadas. **(3) S3 prefixes:** `s3://bucket/tenant-id/data/` com IAM conditions `Condition: {"StringLike": {"s3:prefix": "${aws:PrincipalTag/TenantId}/*"}}`. **(4) DynamoDB partition key** = tenantId. Escolha depende do requisito de isolamento.

---

**P:** Qual é o padrão de arquitetura serverless típico do exame SAA-C03?  
**R:** API GW → Lambda → DynamoDB (CRUD simples). Variações: + S3 pré-signed URLs para upload direto. + Cognito para auth. + SQS para desacoplar Lambda de processamento pesado. + EventBridge para agendamento. + Step Functions para workflows multi-step. + Lambda@Edge para personalização no CDN. + RDS Proxy para Lambda que conecta a RDS (connection pooling).

---

**P:** Como detectar fraudes em tempo real com ML na AWS?  
**R:** Transactions → API GW → Lambda → Amazon Fraud Detector (ou SageMaker endpoint) → resultado em <100ms. Para transações históricas: Kinesis Data Streams → Lambda (chama FM endpoint). Armazenar decisões: DynamoDB. Re-treinar modelos periodicamente: Kinesis → S3 → SageMaker Training Job → atualizar endpoint. Audit trail: CloudTrail + DynamoDB.

---

**P:** O que é o padrão "Cache-Aside" e como implementar com ElastiCache?  
**R:** Aplicação gerencia o cache manualmente. Read: check ElastiCache → hit: retornar dado. Miss: buscar no RDS → salvar no ElastiCache → retornar. Write: atualizar RDS → **invalidar** cache (delete da key). Contrário de Write-Through (cache sempre atualizado na escrita). Cache-Aside: lazy loading, menos memória usada, eventualmente consistente.

---

**P:** Como implementar Blue/Green Deployment com zero downtime na AWS?  
**R:** Opção 1: **ELB + ASG** — criar novo Target Group (green) com nova versão, shift tráfego no ALB Listener Rule gradualmente. Opção 2: **CodeDeploy Blue/Green** — automatiza processo + rollback automático. Opção 3: **Lambda aliases + weighted routing** (10% v2 → 100% v2). Opção 4: **Elastic Beanstalk** — swap de environment URLs. Rollback: redirecionar tráfego de volta.

---

**P:** Como arquitetar search de produtos em e-commerce com alta performance?  
**R:** DynamoDB como source of truth → DynamoDB Streams → Lambda → OpenSearch Service (indexação). Front-end búsca no OpenSearch (full-text, filtros, autocomplete, facets). Benefício: OpenSearch retorna em <10ms para queries complexas. DynamoDB não suporta full-text search nativamente. Dados mapeados com campos analisados (tokenizados) no OpenSearch.

---

**P:** Qual são as principais patterns de integração entre serviços AWS?  
**R:** **(1) Sync:** API Gateway → Lambda (cliente espera). **(2) Async:** API GW → Lambda → SQS → outro Lambda (fire-and-forget). **(3) Fan-out:** SNS → múltiplas SQS. **(4) Streaming:** KDS → Lambda/KDA. **(5) Schedule:** EventBridge Scheduler → Lambda. **(6) Workflow:** Step Functions → múltiplos serviços orquestrados. Cada pattern pra um caso de uso.

---

**P:** Como garantir conformidade HIPAA em arquitetura AWS?  
**R:** (1) Usar serviços HIPAA-eligible (EC2, RDS, S3, Lambda, etc. — ver BAA). (2) Assinar Business Associate Agreement (BAA) com AWS. (3) Criptografia: KMS CMKs, encryption at rest e in transit. (4) Audit logging: CloudTrail + CloudWatch Logs. (5) Access control: IAM com princípio de least-privilege. (6) Network: VPC privada + VPN/DX para acesso. (7) Data backup com criptografia.

---

**P:** Como implementar DLQ (Dead Letter Queue) em uma arquitetura serverless?  
**R:** Invocação **assíncrona** (SNS, S3 Events): DLQ configurada diretamente na função Lambda (SNS retries 2x, Lambda retries 2x). Mensagens falhadas vão para SQS DLQ. **Event Source Mapping** (SQS): DLQ é configurada no SQS (não no Lambda). SQS tentará por maxReceiveCount, então move para SQS DLQ. Preferred: usar Lambda Destinations para invocações assíncronas (mais flexível).

---

**P:** Qual arquitetura para aplicação backend com baixo RTO (<5 min) e RPO (<1 min) multi-região?  
**R:** Active-Active: Route 53 com health checks + Latency/Failover Routing. API stateless em ambas regiões (ECS/Lambda). Banco: Aurora Global Database (replicação <1s, failover automático em ~1 min). Cache: ElastiCache Global Datastore (réplica cross-region). S3 CRR para assets. Sessões: DynamoDB Global Tables. CloudFront com origens em múltiplas regiões.

---

**P:** O que é o padrão Saga e quando usar na AWS?  
**R:** Gerenciar transações distribuídas em microsserviços. Cada serviço faz uma transação local e publica um evento. Se fase seguinte falhar: compensating transactions desfazem passos anteriores. AWS: Step Functions (orquestração — sabe o estado global) ou Coreografia via SNS/EventBridge (cada serviço reage a eventos). Use quando: precisa de consistência eventual em transações multi-serviço.

---

**P:** Como usar SSM Parameter Store para configuração de aplicações?  
**R:** Hierarquia de parâmetros: `/app/prod/database/password` (SecureString com KMS). Lambda/ECS busca parâmetros na inicialização via SDK. Vantagens sobre env vars: (1) versionamento; (2) secretos criptografados; (3) auditável (CloudTrail); (4) mudança sem redeploy (Lambda pode ler no cold start). Para rotação automática: Secrets Manager.

---

**P:** Como implementar processamento de imagem serverless na AWS?  
**R:** Upload: pre-signed S3 URL (direto do browser para S3, sem passar pelo backend). Upload trigger: S3 Event → Lambda. Lambda processa: Pillow/Sharp → resize/thumbnail → salva resultado em S3. CDN: CloudFront distribui imagens processadas. Para processamento pesado: S3 Event → SQS → Lambda (throttled) ou EC2 Spot via ASG. Metadados: DynamoDB.

---

**P:** Qual é a arquitetura típica para sistema de notificações em tempo real?  
**R:** Backend → EventBridge (ou SNS) → múltiplos canais: (1) Email: SES. (2) SMS: SNS → SMS. (3) Push notifications: SNS → GCM/APNs. (4) In-app real-time: API GW WebSocket API + Lambda + DynamoDB (connection tracking). (5) Slack: Lambda → Slack API. Fan-out pattern para múltiplos canais simultâneos sem acoplar o backend.

---

**P:** Como o exame testa conhecimento de arquiteturas multi-camada (3-tier)?  
**R:** **Presentation:** CloudFront + S3 (SPA) ou ALB. **Application:** EC2 ASG ou ECS em subnets privadas (sem acesso direto à internet). **Data:** RDS em subnet de dados privada (aceita conexão apenas do SG da camada application). O exame testa: fluxo de SGs correto, que bancos ficam em subnets privadas, que ALB fica em subnet pública, que EC2 de aplicação não precisa de IP público.
