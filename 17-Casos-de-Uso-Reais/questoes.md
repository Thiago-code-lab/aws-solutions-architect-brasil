# Questões — Módulo 17: Casos de Uso Reais

> **Domínio SAA-C03**: Todos os domínios (multi-serviço)  
> **Dificuldade**: Alta

---

**1.** Uma empresa de streaming de vídeo precisa de uma arquitetura que suporte 1 milhão de usuários simultâneos, sirvam vídeos com baixa latência globalmente, e escale automaticamente. Quais serviços AWS compõem a arquitetura ideal?

- A) EC2 + ELB + RDS
- B) CloudFront + S3 (conteúdo estático) + ALB + Auto Scaling Group + Aurora (metadados)
- C) API Gateway + Lambda + DynamoDB
- D) Global Accelerator + EC2 + RDS Multi-AZ

<details><summary>Resposta</summary>

**B** — CloudFront faz cache dos vídeos em edge locations globalmente (baixa latência, reduz origem). S3 armazena os arquivos de vídeo (escala ilimitada, custo baixo). ALB + ASG escala a camada de aplicação (metadados, autenticação, playlists). Aurora para dados transacionais com Read Replicas para alto throughput de leitura.

</details>

---

**2.** Uma plataforma de e-commerce precisa processar pagamentos garantindo que cada transação seja processada exatamente uma vez, mesmo em caso de retry. Qual arquitetura garante isso?

- A) API Gateway → Lambda → SQS Standard → Lambda processador
- B) API Gateway → Lambda → SQS FIFO → Lambda processador com idempotency key
- C) API Gateway → Lambda → DynamoDB → SNS
- D) API Gateway → Lambda direta (retry explícito no cliente)

<details><summary>Resposta</summary>

**B** — SQS FIFO com MessageDeduplicationId = exactly-once delivery. No processador, use a transaction ID como Idempotency Key no DynamoDB (conditional put: só grava se não existir) para garantir exactly-once processing mesmo se a mensagem for re-entregue. Dupla proteção: SQS FIFO (dedup) + DynamoDB conditional write.

</details>

---

**3.** Uma startup de fintech precisa de uma arquitetura que detecte fraudes em transações em tempo real (latência < 100ms), processe 10.000 transações por segundo e mantenha auditoria completa. Qual arquitetura implementa isso?

- A) S3 + Athena para análise batch de fraudes (revisar após o dia)
- B) API GW → Lambda → SageMaker endpoint (modelos ML em tempo real) → DynamoDB (resultado) + Kinesis → S3 + Athena (auditoria)
- C) RDS Aurora com stored procedures de detecção de fraude
- D) EC2 com aplicação de fraud detection rodando 24/7

<details><summary>Resposta</summary>

**B** — Para tempo real com ML: SageMaker real-time endpoint retorna prediction em <100ms. A transação também vai para Kinesis Firehose → S3 para auditoria em Athena. DynamoDB armazena os resultados de prevenção. Lambda orquestra tudo, escalando automaticamente para 10.000 TPS.

</details>

---

**4.** Uma empresa de SaaS multi-tenant precisa isolar os dados de cada cliente (tenant) em S3, garantindo que um tenant nunca acesse dados de outro. Qual mecanismo implementa isso?

- A) Um único bucket S3 com prefixos por tenant e bucket policy genérica
- B) Um bucket S3 por tenant com bucket policy + Cognito Identity Pool com assume role por tenant
- C) S3 compartilhado com ACLs por objeto para cada tenant
- D) S3 Object Ownership com ACLs habilitadas por objeto

<details><summary>Resposta</summary>

**B** — Isolamento forte: bucket por tenant + Cognito Identity Pool (federa usuários para roles IAM específicas de cada tenant). A bucket policy do tenant-bucket permite acesso apenas da role do tenant. Nenhum cross-tenant access é possível. Alternativa (A) com prefixos é mais fácil de implementar mas menos seguro (uma misconfiguration pode expor todos os tenants).

</details>

---

**5.** Uma empresa quer migrar de uma arquitetura monolítica para microserviços sem downtime, usando o padrão Strangler Fig. Como implementar isso na AWS?

- A) Criar todos os microserviços primeiro e fazer cutover total num fim de semana
- B) ALB com path-based routing: novas rotas vão para microserviços, rotas legadas vão para o monolito; migrar gradualmente
- C) Route 53 com weighted routing dividindo tráfego entre monolito e microserviços
- D) B é o padrão Strangler Fig correto; o tráfego é migrado incrementalmente por domínio

<details><summary>Resposta</summary>

**D** — Strangler Fig: ALB como facade. Novas funcionalidades e rotas migradas vão para microserviços (ECS/Lambda); funcionalidades legadas ainda vão para o monolito. Gradualmente, todas as rotas são migradas e o monolito é "estrangulado" (desativado). Route 53 weighted seria mais para blue/green do monolito inteiro, não migração incremental.

</details>

---

**6.** Uma empresa de IoT tem 100.000 dispositivos enviando telemetria a cada 10 segundos. Os dados precisam ser processados em tempo real para alertas e armazenados para analytics histórico. Qual arquitetura usar?

- A) IoT Core → Lambda (por mensagem) → DynamoDB + S3
- B) IoT Core → Kinesis Data Streams → Lambda (alertas em tempo real) + Kinesis Firehose → S3 (armazenamento) + Athena (analytics)
- C) MQTT broker EC2 → SQS → Lambda
- D) API Gateway → Lambda → DynamoDB Streams → Analytics

<details><summary>Resposta</summary>

**B** — IoT Core: MQTT/HTTPS gerenciado para 100k dispositivos. Kinesis Data Streams absorve 100k × 10s = 10k mensagens/segundo. Lambda processa stream para alertas em tempo real. Kinesis Firehose paralelamente entrega para S3 em Parquet. Athena faz analytics histórico. Solução serverless, gerenciada e escalável.

</details>

---

**7.** Uma empresa de marketplace precisa notificar 5 times diferentes (email, mobile push, auditoria, analytics, CRM) quando um pedido é criado. Qual é o padrão mais adequado?

- A) REST API que chama os 5 sistemas em sequência
- B) SNS topic (pedido criado) → 5 SQS queues (uma por sistema) → cada sistema consome sua fila (fan-out pattern)
- C) EventBridge com 5 rules apontando para cada sistema
- D) B e C são válidos; SNS fan-out é mais simples; EventBridge é melhor para regras condicionais complexas

<details><summary>Resposta</summary>

**D** — SNS Fan-out: SNS → múltiplas SQS (ou Lambda diretamente) em paralelo. Simples e direto. EventBridge: melhor quando diferentes eventos precisam de diferentes filtros/regras (ex: só notificar analytics pra pedidos > R$500). Para o caso simples descrito, SNS fan-out é suficiente.

</details>

---

**8.** Uma empresa precisa de um sistema de busca de produtos que suporte full-text search, filtros por categoria/preço, e autocompletion. A base tem 10 milhões de produtos. Qual serviço AWS usar?

- A) DynamoDB com GSI para filtros por categoria
- B) Amazon OpenSearch Service com índices de produtos
- C) Amazon Aurora com Full-Text Search
- D) Amazon Kendra

<details><summary>Resposta</summary>

**B** — OpenSearch: motor de busca full-text, filtros compostos, autocompletion (completion suggester), fuzzy matching, relevance scoring. 10 milhões de documentos é viável num cluster OpenSearch de tamanho adequado. DynamoDB não tem full-text search. Kendra é para enterprise search de documentos, não catálogo de produtos.

</details>

---

**9.** Uma empresa quer construir um data lake serverless onde engenheiros de dados possam fazer ETL em Python, analistas possam fazer queries SQL e cientistas de dados possam treinar modelos ML, todos sobre os mesmos dados em S3. Qual arquitetura?

- A) EMR permanente para todas as workloads
- B) S3 + Glue Data Catalog + Glue Studio (ETL) + Athena (SQL) + SageMaker (ML)
- C) Redshift como central de todos os dados
- D) Aurora + QuickSight para todos os casos

<details><summary>Resposta</summary>

**B** — S3 como storage universal (dados brutos e processados). Glue Data Catalog: metastore compartilhado. Glue Studio: ETL Python/Spark serverless. Athena: SQL sobre S3 (serverless). SageMaker: ML sobre dados S3. Cada persona usa a ferramenta ideal, todos acessam os mesmos dados no S3 via Glue Catalog.

</details>

---

**10.** Uma aplicação web precisa autenticar usuários com Google, Facebook e email/senha, e depois acessar DynamoDB e S3 diretamente do browser (client-side) sem passar pelo backend. Qual serviço implementa isso?

- A) IAM Users para cada usuário da aplicação
- B) Amazon Cognito: User Pools (auth) + Identity Pools (credenciais AWS temporárias para acesso client-side)
- C) API Gateway com Lambda Authorizer para validar tokens OAuth
- D) AWS SSO para usuários de aplicações web

<details><summary>Resposta</summary>

**B** — Cognito User Pool: signup/login com email/senha + federação com Google, Facebook (social). Identity Pool: troca o token Cognito (ou social) por credenciais AWS temporárias (STS assume role). O browser pode então acessar S3 e DynamoDB diretamente com as credenciais temporárias, sem expor credenciais permanentes.

</details>

---

**11.** Uma empresa de mídia precisa processar vídeos enviados por usuários (transcodificação para múltiplos formatos e resoluções). O processamento é pesado (30-60 minutos por vídeo) e o volume é imprevisível. Qual arquitetura?

- A) Lambda para transcodificação (timeout máximo 15 min — insuficiente)
- B) S3 evento → SQS → ECS Fargate tasks (escala baseada em queue depth) para transcodificação
- C) EC2 On-Demand com Auto Scaling
- D) AWS Elemental MediaConvert (serviço gerenciado específico para transcodificação)

<details><summary>Resposta</summary>

**D** — AWS Elemental MediaConvert: serviço gerenciado específico para transcodificação de vídeo. Suporta múltiplos formatos de input/output, presets de resolução, DRM, legendas. S3 → MediaConvert → S3. Serverless, paga por minuto de output transcodificado. Alternativa: S3 → SQS → Fargate tasks com FFmpeg (B) — mais flexível mas mais complexo.

</details>

---

**12.** Uma empresa precisa de um ambiente de desenvolvimento cloud onde os desenvolvedores escrevam, executem e depurem código diretamente do browser, sem configurar ambientes locais. Qual serviço AWS?

- A) AWS Cloud9 (IDE baseado em browser com EC2 em background)
- B) AWS CloudShell
- C) Amazon WorkSpaces
- D) AWS CodeStar

<details><summary>Resposta</summary>

**A** — AWS Cloud9: IDE completo no browser com editor de código, terminal integrado, debugger e suporte a Lambda development. Roda em EC2 (ou SSM para instâncias existentes). Colaboração em tempo real. CloudShell (B) é só terminal sem IDE. WorkSpaces (C) é desktop virtual completo.

</details>

---

**13.** Uma plataforma de blog tem picos de tráfego previsíveis (lançamentos de artigos) que duram 2 horas. A maioria do conteúdo é estático. Qual é a arquitetura mais custo-efetiva?

- A) Auto Scaling Group com escala baseada em CPU
- B) CloudFront cacheando conteúdo estático (artigos, imagens) → origin ALB + pequeno ASG (apenas para conteúdo dinâmico)
- C) Lambda para todo conteúdo (incluindo geração dinâmica de HTML)
- D) EC2 com memoria alta para cache em memória local

<details><summary>Resposta</summary>

**B** — CloudFront absorve a maior parte do tráfego de conteúdo estático sem chegar no origin. Apenas requests não-cacheáveis (search, comments, user specific) chegam no backend. O ASG pode ser mínimo. Durante picos, CloudFront escala automaticamente (global infrastructure). Custo-efetivo pois elimina a maioria da carga do backend.

</details>

---

**14.** Uma empresa precisa garantir que é possível restaurar sua aplicação em uma nova região em menos de 1 hora após uma falha regional. Qual combinação de estratégias implementar?

- A) Snapshots diários de EBS + AMIs copiadas para segunda região
- B) Multi-region AMIs + Aurora Global DB + S3 CRR + Route 53 health check failover + IaC (CloudFormation)
- C) Apenas S3 CRR para backup dos dados
- D) AWS Backup com cross-region backup para todos os recursos

<details><summary>Resposta</summary>

**B** — Para RTO < 1 hora: AMIs pré-copiadas (EC2 pronto para lançar), Aurora Global DB (BD replicado com RPO < 1s), S3 CRR (dados replicados), Route 53 failover automático, e CloudFormation para provisionar rapidamente a infraestrutura. AWS Backup (D) é bom para backups mas não coordena o failover orquestrado.

</details>

---

**15.** Uma empresa de saúde precisa armazenar prontuários médicos com controles rigorosos: apenas médicos autorizados acessam registros de seus pacientes, auditoria de todos os acessos, criptografia com chaves do cliente, e conformidade HIPAA. Qual arquitetura?

- A) RDS MySQL com usuários por médico
- B) S3 com AES256 + IAM policies por médico
- C) DynamoDB (dados) + KMS CMK (criptografia) + IAM com resource-based conditions + CloudTrail Data Events (auditoria) + AWS Artifact (certificação HIPAA)
- D) Aurora com SSL + Cognito User Pools

<details><summary>Resposta</summary>

**C** — Conformidade HIPAA completa: DynamoDB com KMS CMK (chave gerenciada pelo cliente = controle total de criptografia); IAM com condições (`dynamodb:LeadingKeys` para restringir por patient_id do médico-paciente); CloudTrail Data Events: registra todas as leituras no DynamoDB; AWS Artifact: obtém documentos BAA (Business Associate Agreement) necessários para HIPAA.

</details>
