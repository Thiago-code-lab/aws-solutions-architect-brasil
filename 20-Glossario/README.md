# Módulo 20 — Glossário AWS SAA-C03

## Sobre Este Módulo

Dicionário de termos e conceitos essenciais para o exame AWS Solutions Architect Associate (SAA-C03). Organizado por domínio para facilitar a revisão rápida.

---

## A

| Termo | Definição |
|---|---|
| **ACL (Access Control List)** | Lista de regras de acesso. Em S3: configuração legada de permissão por bucket/objeto. Em VPC: NACL — firewall stateless no nível de subnet |
| **ACM (AWS Certificate Manager)** | Serviço para provisionar, gerenciar e fazer deploy de certificados SSL/TLS gratuitos para uso com CloudFront, ALB, API Gateway |
| **ALB (Application Load Balancer)** | Load balancer de Layer 7 (HTTP/HTTPS). Suporta path-based e host-based routing, WebSocket, gRPC |
| **AMI (Amazon Machine Image)** | Template de instância EC2 contendo OS, aplicações e configurações. Específico por região (pode ser copiado entre regiões) |
| **ARN (Amazon Resource Name)** | Identificador único de recursos AWS. Formato: `arn:aws:service:region:account-id:resource` |
| **ASG (Auto Scaling Group)** | Grupo de instâncias EC2 que escala automaticamente baseado em políticas de scaling ou demanda |
| **Athena** | Serviço de query SQL serverless sobre dados no S3. Paga por TB escaneado |
| **Aurora** | Banco de dados relacional compatível MySQL/PostgreSQL criado pela AWS. 5x performance que MySQL, 3x PostgreSQL |
| **Availability Zone (AZ)** | Datacenter isolado dentro de uma região AWS. Múltiplas AZs por região garantem alta disponibilidade |
| **AWS Organizations** | Gerenciamento centralizado de múltiplas contas AWS com policies (SCPs) e faturamento consolidado |

---

## B

| Termo | Definição |
|---|---|
| **Bastion Host** | Instância EC2 em subnet pública usada como ponto de acesso SSH para instâncias em subnets privadas. Substituto mais seguro: SSM Session Manager |
| **Bedrock** | Serviço de IA Generativa da AWS. Acessa Foundation Models (Anthropic Claude, Amazon Titan, etc.) via API sem gerenciar infraestrutura |
| **Blue/Green Deployment** | Estratégia de deploy com dois ambientes idênticos; tráfego é migrado gradualmente ou totalmente do "blue" (atual) para o "green" (novo) |
| **Bucket Policy** | Política JSON baseada em recurso aplicada diretamente no S3 bucket. Controla acesso de qualquer entidade (cross-account) |
| **Burst** | Capacidade temporária além do limite provisionado. EC2 (instâncias T): créditos CPU burst. Lambda: 3.000 concurrent executions iniciais por região |

---

## C

| Termo | Definição |
|---|---|
| **CIDR (Classless Inter-Domain Routing)** | Notação para ranges de IP. Ex: `10.0.0.0/16` = 65.536 IPs. VPC CIDR define o address space da rede virtual |
| **CloudFormation** | IaC (Infrastructure as Code) da AWS. Templates YAML/JSON que definem e provisionam recursos AWS como código |
| **CloudFront** | CDN (Content Delivery Network) global da AWS com 400+ edge locations. Melhora latência e reduz carga na origem |
| **CloudTrail** | Serviço de auditoria que registra todas as API calls feitas na conta AWS. Essencial para compliance e investigação de incidentes |
| **CloudWatch** | Serviço de monitoramento e observabilidade. Coleta métricas, logs, cria alarmes e dashboards |
| **CMK (Customer Managed Key)** | Chave KMS criada e gerenciada pelo cliente. Permite controle total sobre políticas de uso e rotação |
| **Cognito** | Serviço de autenticação e autorização para aplicações web/mobile. User Pools (auth) + Identity Pools (federação AWS credentials) |
| **Cold Start** | Delay na primeira invocação de uma função Lambda quando um novo execution environment precisa ser inicializado |
| **Compliance** | Conformidade com regulações e padrões (PCI-DSS, HIPAA, SOC2, ISO 27001). AWS é responsável pela infraestrutura; cliente pela aplicação |
| **CRR (Cross-Region Replication)** | Replicação automática de objetos S3 para um bucket em outra região. Requer versioning habilitado |

---

## D

| Termo | Definição |
|---|---|
| **DAX (DynamoDB Accelerator)** | Cache in-memory para DynamoDB com latência de microssegundos. Totalmente gerenciado e compatível com DynamoDB API |
| **DLQ (Dead Letter Queue)** | Fila (SQS) ou tópico (SNS) que recebe mensagens que não puderam ser processadas após o número máximo de tentativas |
| **DMS (Database Migration Service)** | Serviço de migração de bancos de dados. Suporta migrações homogêneas e heterogêneas com opção de CDC (Change Data Capture) |
| **DynamoDB** | Banco NoSQL chave-valor totalmente gerenciado. Escalabilidade infinita, latência de milissegundos, multi-region com Global Tables |
| **DX (Direct Connect)** | Conexão de rede dedicada e privada entre on-prem e AWS. Largura de banda de 1, 10 ou 100 Gbps com latência consistente |

---

## E

| Termo | Definição |
|---|---|
| **EBS (Elastic Block Store)** | Volumes de disco de alta performance para EC2. Tipos: gp2/gp3 (SSD geral), io1/io2 (SSD alta performance), st1 (HDD throughput), sc1 (HDD cold) |
| **EC2 (Elastic Compute Cloud)** | Serviço de computação virtual (VMs) da AWS. Centenas de tipos de instâncias para diferentes cargas de trabalho |
| **ECR (Elastic Container Registry)** | Registry privado de imagens Docker gerenciado. Integrado com ECS, EKS e Lambda Container Image |
| **ECS (Elastic Container Service)** | Orquestrador de containers gerenciado pela AWS. Executa tasks em EC2 ou Fargate |
| **EFS (Elastic File System)** | Sistema de arquivos NFS gerenciado e elástico para Linux. Multi-AZ e multi-instance simultaneamente |
| **EKS (Elastic Kubernetes Service)** | Kubernetes gerenciado pela AWS. Reduz complexidade operacional do control plane Kubernetes |
| **ElastiCache** | Serviço de cache in-memory gerenciado. Dois engines: Redis (persistência, Pub/Sub, sorted sets) e Memcached (simples, multi-thread) |
| **Endpoint (VPC)** | Conexão privada entre VPC e serviços AWS sem sair para internet. Gateway (S3, DynamoDB — gratuito) ou Interface (PrivateLink — pago) |
| **ESM (Event Source Mapping)** | Configuração que permite Lambda receber eventos de Kinesis, DynamoDB Streams, SQS, MSK automaticamente |
| **EventBridge** | Barramento de eventos serverless. Roteia eventos entre serviços AWS, SaaS e aplicações customizadas via regras |

---

## F–G

| Termo | Definição |
|---|---|
| **Fargate** | Computação serverless para containers (ECS/EKS). Sem servidores para gerenciar; paga por vCPU/memória usados pela task |
| **FIFO Queue** | SQS FIFO: garante ordem de entrega e exatamente uma entrega (exactly-once). Limitado a 300 TPS (3.000 com batching) |
| **FSx** | Sistemas de arquivos gerenciados de alta performance: Lustre (HPC), Windows (SMB), NetApp ONTAP, OpenZFS |
| **Global Accelerator** | Serviço de rede que usa a rede privada AWS (anycast) para rotear tráfego TCP/UDP com menor latência global. 2 IPs anycast estáticos |
| **Glue** | Serviço ETL serverless e catálogo de dados. Crawlers descobrem schemas; Jobs processam dados com Spark gerenciado |
| **GuardDuty** | Serviço de detecção de ameaças usando ML. Analisa CloudTrail, VPC Flow Logs, DNS Logs para detectar comportamentos maliciosos |

---

## H–I

| Termo | Definição |
|---|---|
| **HA (High Availability)** | Capacidade do sistema de continuar operando mesmo com falhas de componentes. Geralmente exige resources em múltiplas AZs |
| **IAM (Identity and Access Management)** | Controle de acesso a recursos AWS. Usuários, grupos, roles e políticas definem quem pode fazer o quê |
| **IOPS (I/O Operations Per Second)** | Medida de performance de storage. EBS gp3 baseline 3.000 IOPS; io2 Block Express até 256.000 IOPS |
| **IRSA (IAM Roles for Service Accounts)** | Mecanismo EKS que associa IAM roles a Kubernetes service accounts. Pods obtêm credenciais AWS sem segredos hardcoded |
| **Inspector** | Avaliação automatizada de vulnerabilidades para EC2, Lambda Functions e imagens ECR. Usa CVE database |

---

## J–K

| Termo | Definição |
|---|---|
| **Jump Box** | Ver Bastion Host |
| **Kinesis Data Streams** | Serviço de streaming em tempo real. Producers → Shards → Consumers. Retenção até 365 dias; replay possível |
| **Kinesis Firehose** | Entrega near-real-time de dados para S3, Redshift, OpenSearch, Splunk. Totalmente gerenciado sem código de consumer |
| **KMS (Key Management Service)** | Serviço gerenciado de chaves de criptografia. Integrado com todos os serviços AWS que suportam criptografia |

---

## L–M

| Termo | Definição |
|---|---|
| **Lambda** | Computação serverless. Executa código em resposta a eventos sem provisionar servidores. Paga por invocação + duração (GB-segundos) |
| **Lake Formation** | Serviço de governança de data lake sobre S3. Gerencia permissões refinadas (coluna/linha) para Glue + Athena |
| **Macie** | Serviço de segurança de dados que usa ML para descobrir e proteger dados sensíveis no S3 (PII, chaves, credenciais) |
| **MGN (Application Migration Service)** | Serviço de lift-and-shift automatizado. Replica servidores continuamente para AWS e executa cutover com mínimo downtime |
| **MSK (Managed Streaming for Kafka)** | Apache Kafka gerenciado na AWS. Compatível com clientes Kafka existentes sem alterar código |
| **Multi-AZ** | Alta disponibilidade com recursos em múltiplas Availability Zones. Para RDS: failover automático síncrono para standby |

---

## N–O

| Termo | Definição |
|---|---|
| **NAT Gateway** | Componente de rede que permite instâncias em subnets privadas iniciarem conexões à internet sem serem acessíveis da internet |
| **NACL (Network Access Control List)** | Firewall stateless no nível de subnet. Avalia regras numeradas em ordem; separadas regras inbound e outbound |
| **NLB (Network Load Balancer)** | Load balancer Layer 4 (TCP/UDP/TLS). Extremamente alta performance (milhões de RPS), IP estático via EIP, latência ultra-baixa |
| **OAC (Origin Access Control)** | Método atual (substitui OAI) para que CloudFront acesse S3 de forma privada. O S3 bucket nega acesso direto pela internet |
| **OpenSearch** | Fork gerenciado do Elasticsearch. Search, log analytics e visualização com Kibana/OpenSearch Dashboards |

---

## P–Q

| Termo | Definição |
|---|---|
| **Parameter Store** | Armazenamento hierárquico de configuração e segredos no Systems Manager. Gratuito para standard (até 4 KB/parâmetro) |
| **Placement Group** | Agrupamento de instâncias EC2 por proximidade ou distribuição. Cluster (baixa latência), Spread (máxima disponibilidade), Partition (HBase/HDFS) |
| **PrivateLink** | AWS PrivateLink: expõe serviços privados via Interface Endpoint (ENI) sem tráfego passando pela internet |
| **QuickSight** | BI serverless da AWS. SPICE engine para cache in-memory; dashboards embarcáveis; ML Insights automático |

---

## R

| Termo | Definição |
|---|---|
| **RDS (Relational Database Service)** | Bancos relacionais gerenciados: MySQL, PostgreSQL, Oracle, SQL Server, MariaDB, e Aurora |
| **Read Replica** | Cópia assíncrona de banco de dados para escalar leituras. RDS: até 5 réplicas; Aurora: até 15 réplicas |
| **Rekognition** | Visão computacional como serviço. Análise de imagens e vídeos: objetos, rostos, texto, moderação de conteúdo |
| **Reserved Instance (RI)** | Compromisso de 1 ou 3 anos com desconto de até 72% vs On-Demand. Standard RI: fixo instância; Convertible RI: pode mudar tipo |
| **Route 53** | DNS gerenciado da AWS com 100% SLA. Suporta 8 políticas de roteamento + health checks |
| **RPO (Recovery Point Objective)** | Quantidade máxima de dados que pode ser perdida em evento de desastre (determina frequência de backup) |
| **RTO (Recovery Time Objective)** | Tempo máximo aceitável para restaurar sistema após falha (determina estratégia de DR) |

---

## S

| Termo | Definição |
|---|---|
| **S3 (Simple Storage Service)** | Armazenamento de objetos durável (11 9s), na prática ilimitado. Buckets por região; objetos até 5 TB |
| **SAM (Serverless Application Model)** | Framework de IaC simplificado para aplicações serverless Lambda/API GW/DynamoDB. Extension do CloudFormation |
| **Savings Plans** | Desconto de até 66% (Compute) ou 72% (EC2) em troca de compromisso de $$/hora por 1 ou 3 anos. Mais flexível que Reserved Instances |
| **SCT (Schema Conversion Tool)** | Ferramenta desktop que converte schema de banco de dados entre engines diferentes na migração heterogênea (junto com DMS) |
| **Security Group (SG)** | Firewall stateful no nível de instância/ENI. Regras apenas de Allow (sem Deny explícito). Referência a outros SGs por ID |
| **Shield** | Proteção contra DDoS. Standard (gratuito, automático para todos); Advanced (pago, proteção extra + DRT + WAF gratuito) |
| **Spot Instance** | EC2 com desconto de até 90% usando capacidade ociosa. Pode ser interrompida com aviso de 2 minutos |
| **SQS (Simple Queue Service)** | Fila de mensagens gerenciada. Standard (alta vazão, at-least-once) ou FIFO (ordem garantida, exactly-once) |
| **SSM (Systems Manager)** | Suite de ferramentas operacionais: Session Manager, Parameter Store, Patch Manager, Run Command, Automation |
| **SRR (Same-Region Replication)** | Replicação de objetos S3 entre buckets na mesma região (ex: consolidar logs, compliance) |
| **Step Functions** | Orquestrador de workflows serverless. Coordena lambdas, ECS tasks, e outros serviços em máquinas de estado visuais |

---

## T–Z

| Termo | Definição |
|---|---|
| **TGW (Transit Gateway)** | Hub de rede centralizado que conecta VPCs e on-prem em topologia hub-and-spoke. Roteamento transitivo (diferente de VPC Peering) |
| **Transcribe** | Speech-to-Text gerenciado. Suporta streaming, identificação de locutores, vocabulário customizado |
| **Transfer Family** | Servidor SFTP/FTPS/FTP/AS2 gerenciado com backend em S3 ou EFS. Clientes não precisam alterar ferramentas existentes |
| **Trusted Advisor** | Ferramenta que recomenda melhorias em custo, segurança, performance, fault tolerance e service limits |
| **VPC (Virtual Private Cloud)** | Rede virtual isolada na AWS. Você controla CIDR, subnets, routing tables, security groups e NACLs |
| **VPC Peering** | Conexão entre duas VPCs (mesma ou diferente conta/região) via rede AWS. Não-transitivo (A↔B, B↔C ≠ A↔C) |
| **WAF (Web Application Firewall)** | Firewall de aplicação web Layer 7. Protege contra SQL injection, XSS, e outros ataques via web ACLs e regras gerenciadas |
| **WORM (Write Once Read Many)** | Modelo de armazenamento onde dados não podem ser modificados após escritos. S3 Object Lock Compliance mode |
| **X-Ray** | Serviço de tracing distribuído. Visualiza latência, erros e performance entre serviços de uma aplicação |

---

## Tabela de Acrônimos

| Sigla | Significado |
|---|---|
| ACL | Access Control List |
| ACM | AWS Certificate Manager |
| ALB | Application Load Balancer |
| AMI | Amazon Machine Image |
| ARN | Amazon Resource Name |
| ASG | Auto Scaling Group |
| CDC | Change Data Capture |
| CDN | Content Delivery Network |
| CMK | Customer Managed Key |
| CRR | Cross-Region Replication |
| DAX | DynamoDB Accelerator |
| DLQ | Dead Letter Queue |
| DMS | Database Migration Service |
| DX | Direct Connect |
| EBS | Elastic Block Store |
| ECR | Elastic Container Registry |
| ECS | Elastic Container Service |
| EFS | Elastic File System |
| EKS | Elastic Kubernetes Service |
| ELB | Elastic Load Balancing |
| ENI | Elastic Network Interface |
| ESM | Event Source Mapping |
| FGA | Fine-Grained Access Control |
| FM | Foundation Model (Bedrock) |
| FSx | Amazon FSx (file systems) |
| IAM | Identity and Access Management |
| IGW | Internet Gateway |
| IOPS | Input/Output Operations Per Second |
| IRSA | IAM Roles for Service Accounts |
| IaC | Infrastructure as Code |
| KMS | Key Management Service |
| MGN | AWS Application Migration Service |
| MPP | Massively Parallel Processing |
| MSK | Managed Streaming for Kafka |
| NACL | Network Access Control List |
| NLB | Network Load Balancer |
| NLP | Natural Language Processing |
| OAC | Origin Access Control |
| OAI | Origin Access Identity (legado) |
| PII | Personally Identifiable Information |
| RI | Reserved Instance |
| RPO | Recovery Point Objective |
| RTO | Recovery Time Objective |
| SCP | Service Control Policy |
| SCT | Schema Conversion Tool |
| SG | Security Group |
| SNS | Simple Notification Service |
| SQS | Simple Queue Service |
| SRR | Same-Region Replication |
| SSE | Server-Side Encryption |
| SSM | AWS Systems Manager |
| TGW | Transit Gateway |
| TTS | Text-to-Speech |
| VGW | Virtual Private Gateway |
| VPC | Virtual Private Cloud |
| WAF | Web Application Firewall |
| WAT | Well-Architected Tool |
| WORM | Write Once Read Many |
