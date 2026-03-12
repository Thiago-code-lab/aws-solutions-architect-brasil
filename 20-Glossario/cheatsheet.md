# Cheatsheet — Glossário AWS (Módulo 20)

## Siglas e Acrônimos — Referência Rápida

| Sigla | Significado | Contexto |
|-------|-------------|---------|
| **ACL** | Access Control List | S3 object/bucket access; NACL em VPC |
| **ACM** | AWS Certificate Manager | Certificados TLS/SSL gratuitos |
| **AMI** | Amazon Machine Image | Template para instâncias EC2 |
| **ARN** | Amazon Resource Name | Identificador único de recurso AWS |
| **ASG** | Auto Scaling Group | Grupo de EC2 com escalonamento automático |
| **AZ** | Availability Zone | Data center(s) isolado dentro de uma Region |
| **BAA** | Business Associate Agreement | Contrato para dados HIPAA/saúde |
| **BYOL** | Bring Your Own License | Usar licença própria em EC2 Dedicated |
| **CDN** | Content Delivery Network | CloudFront distribui conteúdo globalmente |
| **CIDR** | Classless Inter-Domain Routing | Notação de blocos IP (ex: 10.0.0.0/16) |
| **CMK** | Customer Managed Key | Chave KMS gerenciada pelo cliente |
| **CORS** | Cross-Origin Resource Sharing | Política S3/API GW para acesso cross-domain |
| **CRR** | Cross-Region Replication | Replicação S3 entre regiões |
| **DAX** | DynamoDB Accelerator | Cache in-memory para DynamoDB |
| **DNSSEC** | DNS Security Extensions | Verificação autenticidade registros DNS |
| **EBS** | Elastic Block Store | Volume de disco para EC2 |
| **EC2** | Elastic Compute Cloud | Serviço de máquinas virtuais |
| **ECS** | Elastic Container Service | Orquestrador de containers |
| **EFS** | Elastic File System | Sistema de arquivos NFS gerenciado |
| **EIP** | Elastic IP | IP público estático para EC2/NAT GW |
| **EKS** | Elastic Kubernetes Service | Kubernetes gerenciado |
| **EMR** | Elastic MapReduce | Processamento big data (Spark/Hadoop) |
| **ETL** | Extract, Transform, Load | Pipeline de dados (AWS Glue) |
| **FSx** | Amazon FSx | Sistemas de arquivos gerenciados (Windows, Lustre) |
| **GS** | General Purpose SSD | Tipo EBS — gp2/gp3 |
| **HA** | High Availability | Alta disponibilidade |
| **IAM** | Identity and Access Management | Controle de acesso AWS |
| **IGW** | Internet Gateway | Permite acesso à internet em VPC |
| **IOPS** | Input/Output Operations Per Second | Métrica de performance de disco |
| **KDS** | Kinesis Data Streams | Stream de dados em tempo real |
| **KDF** | Kinesis Data Firehose | Entrega streaming para S3/Redshift/ES |
| **KMS** | Key Management Service | Gerenciamento de chaves de criptografia |
| **MFA** | Multi-Factor Authentication | Autenticação em múltiplos fatores |
| **MGN** | Application Migration Service | Lift-and-shift de servidores para AWS |
| **MTBF** | Mean Time Between Failures | Tempo médio entre falhas |
| **MTTR** | Mean Time To Recovery | Tempo médio de recuperação |
| **NACL** | Network Access Control List | Firewall stateless para subnets VPC |
| **NLB** | Network Load Balancer | LB camada 4 (TCP/UDP), alta performance |
| **ALB** | Application Load Balancer | LB camada 7 (HTTP/HTTPS) |
| **CLB** | Classic Load Balancer | LB legado (não recomendado) |
| **NAT** | Network Address Translation | NAT GW/Instance para subnets privadas |
| **OIDC** | OpenID Connect | Protocolo de identidade federada |
| **OAC** | Origin Access Control | CloudFront acessa S3 privado (novo) |
| **OAI** | Origin Access Identity | CloudFront acessa S3 privado (legado) |
| **PCI DSS** | Payment Card Industry Data Security Standard | Compliance para dados de cartão |
| **RPO** | Recovery Point Objective | Quantidade máxima de dados que pode perder |
| **RTO** | Recovery Time Objective | Tempo máximo para recuperar após falha |
| **SAML** | Security Assertion Markup Language | Federação de identidade SSO |
| **SCP** | Service Control Policy | Política de controle em AWS Organizations |
| **SDK** | Software Development Kit | Bibliotecas para desenvolvimento AWS |
| **SG** | Security Group | Firewall stateful para instâncias/recursos |
| **SLA** | Service Level Agreement | Acordo de nível de serviço (uptime) |
| **SNS** | Simple Notification Service | Pub/sub, notificações |
| **SOC** | Service Organization Control | Relatório de auditoria (SOC1/SOC2/SOC3) |
| **SQS** | Simple Queue Service | Fila de mensagens gerenciada |
| **SRR** | Same-Region Replication | Replicação S3 dentro da mesma região |
| **SSE** | Server-Side Encryption | Criptografia no servidor (S3/DynamoDB) |
| **SSM** | Systems Manager | Gerenciamento de instâncias, parameters |
| **STS** | Security Token Service | Tokens temporários (AssumeRole) |
| **TGW** | Transit Gateway | Hub de conectividade VPC multi-VPC |
| **TTL** | Time To Live | Tempo de expiração (DNS, cache, DynamoDB) |
| **VPC** | Virtual Private Cloud | Rede virtual privada na AWS |
| **VPN** | Virtual Private Network | Conexão criptografada on-premises ↔ AWS |
| **WAF** | Web Application Firewall | Proteção layer 7 (SQLi, XSS) |
| **WORM** | Write Once Read Many | S3 Object Lock — imutabilidade |

---

## Confusões Frequentes no Exame

| Conceito A | vs | Conceito B | Como Diferenciar |
|------------|----|-----------|--------------|}
| **RTO** | vs | **RPO** | RTO = tempo de recuperação; RPO = dados perdidos (ponto no passado) |
| **Disponibilidade** | vs | **Durabilidade** | Disponibilidade = posso acessar agora? Durabilidade = dados foram perdidos? |
| **Throughput** | vs | **Latência** | Throughput = quanto por segundo; Latência = quanto tempo demora 1 operação |
| **Multi-AZ** | vs | **Read Replica** | Multi-AZ = HA/failover (standby syncrono); Read Replica = escalar leituras |
| **SG** | vs | **NACL** | SG = stateful (retorno automático); NACL = stateless (regras entrada E saída) |
| **Secrets Manager** | vs | **SSM Param Store** | Secrets = rotação automática, maior custo; SSM = configuraçõesgerais, gratuito para Standard |
| **KMS CMK** | vs | **KMS AWS-managed** | CMK = você controla rotação/acesso; AWS-managed = AWS controla |
| **SNS** | vs | **SQS** | SNS = push para N assinantes (fan-out); SQS = pull pela aplicação (1 consumidor por msg) |
| **Elasticity** | vs | **Scalability** | Elasticity = escala e desescala automaticamente; Scalability = capacidade de crescer |
| **IaaS** | vs | **PaaS** | IaaS = EC2 (você gerencia SO); PaaS = Elastic Beanstalk/RDS (AWS gerencia SO/infra) |

---

## Tipos de Armazenamento

| Tipo | Serviço | Protocolo | Quando Usar |
|------|---------|-----------|-------------|
| **Block** | EBS | iSCSI / NVMe | Boot volume, DB, app que precisa de disco local |
| **File** | EFS | NFS v4.1 | Compartilhamento entre EC2, apps Linux |
| **File** | FSx for Windows | SMB / CIFS | Apps Windows, Active Directory |
| **Object** | S3 | HTTP REST | Backup, mídia, logs, data lake |
| **Archive** | S3 Glacier | HTTP | Compliance, backup long-term |
| **In-Memory** | ElastiCache | Redis/Memcached | Cache de sessão, dados quentes |

---

## Modelos de Responsabilidade Compartilhada

| Responsabilidade | AWS | Cliente |
|-----------------|-----|---------|
| Hardware físico | ✅ | |
| Hipervisor | ✅ | |
| Rede física | ✅ | |
| Sistema Operacional (EC2) | | ✅ |
| Patches do SO | | ✅ |
| Configuração Security Groups | | ✅ |
| Dados do cliente | | ✅ |
| Criptografia de dados | Ferramentas | Configuração |
| IAM (usuários/roles) | | ✅ |
| RDS SO patches | ✅ | |
| RDS dados | | ✅ |

---

## Consistência no DynamoDB

| Tipo | O que significa | Quando Usar |
|------|----------------|-------------|
| **Eventual Consistency** | Leitura pode retornar dado levemente desatualizado | Padrão; menor custo; apps tolerantes |
| **Strong Consistency** | Leitura sempre retorna dado mais recente | 2x custo de leitura; quando precisar garantia |

---

## SLAs de Disponibilidade — Referência

| Porcentagem | Downtime/ano | Downtime/mês | Serviço Exemplo |
|------------|-------------|-------------|----------------|
| 99% | ~87,6h | ~7,3h | Serviços básicos |
| 99,9% | ~8,7h | ~43,8min | EC2, S3 Standard-IA |
| 99,95% | ~4,4h | ~21,9min | RDS Multi-AZ |
| 99,99% | ~52,6min | ~4,4min | S3 Standard, DynamoDB |
| 99,999% | ~5,3min | ~26s | Route 53 |

---

## Dicas de Prova

| Pista na Questão | Resposta Esperada |
|-----------------|------------------|
| "identificar recurso AWS pelo ID único" | ARN (Amazon Resource Name) |
| "acessar API AWS com código" | SDK + IAM Role (nunca access key hardcoded) |
| "bloco de IPs para VPC" | CIDR (ex: 10.0.0.0/16) |
| "certificado SSL gratuito para ALB/CloudFront" | ACM (AWS Certificate Manager) |
| "manter conformidade com leis de saúde" | HIPAA + BAA com AWS |
| "dados que não podem ser alterados após gravação" | S3 Object Lock (WORM) |
| "failover automático banco de dados" | RDS Multi-AZ (não Read Replica) |
| "escalar leitura banco de dados" | RDS Read Replica (não Multi-AZ) |
| "SG retorna resposta automaticamente" | Stateful (SG) vs Stateless (NACL) |
| "conectar múltiplas VPCs como hub-and-spoke" | Transit Gateway (TGW) |
