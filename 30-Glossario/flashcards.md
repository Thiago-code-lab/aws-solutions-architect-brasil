# Flashcards — Módulo 20: Glossário

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 30 flashcards — definições de termos e acrônimos AWS

---

**P:** O que significa SLA no contexto AWS?  
**R:** **Service Level Agreement** — acordo de nível de serviço que define o uptime garantido de um serviço AWS. Ex: EC2 SLA = 99.99% por região. Se a AWS não cumprir, clientes recebem créditos de serviço. SLA diferente de SLO (Service Level Objective — meta interna) e SLI (Service Level Indicator — métrica real).

---

**P:** O que é um CDN e como o CloudFront o implementa?  
**R:** **Content Delivery Network** — rede de servidores distribuídos globalmente que cacheia conteúdo próximo dos usuários. CloudFront: 500+ Points of Presence (PoPs) em 90+ cidades. Reduz latência (usuário acessa PoP local vs servidor distante), reduz carga na origem, cached responses para conteúdo estático.

---

**P:** O que é Idempotência e por que é importante em APIs e filas?  
**R:** **Idempotência:** executar a mesma operação múltiplas vezes produz o mesmo resultado. Ex: `DELETE /order/123` executado 3 vezes = resultado idêntico a executar 1 vez. Importante em: sistemas de fila (SQS entrega ao-less-once, pode re-entregar), retries HTTP, pagamentos (evitar cobranças duplicadas). Design: usar IDs únicos de transação, check-before-write.

---

**P:** O que é throughput vs latência?  
**R:** **Throughput:** quantidade de dados processados por unidade de tempo (GB/s, requests/s). **Latência:** tempo para processar uma única operação (ms, µs). Podem ser inversamente relacionados: processar mais itens em batch aumenta throughput mas pode aumentar latência individual. No exame: "muitos dados" → throughput; "resposta rápida para usuário" → latência.

---

**P:** O que é um SQS Message Visibility Timeout?  
**R:** Após um consumer pegar uma mensagem do SQS, ela fica **invisível** na fila por `VisibilityTimeout` (default: 30s, max: 12h). Se o consumer processar com sucesso e deletar a mensagem: removida permanentemente. Se o consumer falhar/timeout: a mensagem se torna visível novamente para outros consumers. Evita duplo processamento.

---

**P:** O que é NAT Gateway e por que instâncias privadas precisam dele?  
**R:** **Network Address Translation Gateway** — permite instâncias em subnets privadas (sem IP público) acessarem a internet para outbound connections (atualizações, APIs externas), mas a internet não pode iniciar conexões com elas. NAT GW fica na subnet pública com Elastic IP. Route da subnet privada: `0.0.0.0/0 → NAT GW`. Gerenciado, escalável, altamente disponível por AZ.

---

**P:** O que é CIDR Notation?  
**R:** **Classless Inter-Domain Routing** — notação para endereçamento IP. Ex: `10.0.0.0/16` = 65.536 IPs (10.0.0.0 a 10.0.255.255). O `/16` = 16 bits fixos (prefixo), 16 bits variáveis. `/24` = 256 IPs. Menor o número após `/`, maior o bloco. VPC AWS: `/16` max, `/28` min (11 IPs usáveis — AWS reserva 5).

---

**P:** O que é um Bastion Host e qual é a alternativa moderna?  
**R:** **Bastion Host (Jump Box):** EC2 em subnet pública com porta SSH aberta. Acesso: SSH para bastion → SSH para instâncias privadas. Problema: superfície de ataque (SSH exposto). **Alternativa moderna:** AWS Systems Manager Session Manager — sem porta 22 aberta, sem bastion, auditável, funciona em subnets privadas sem internet.

---

**P:** O que é o modelo de pagamento "pay-as-you-go" na AWS?  
**R:** **Pagar pelo uso real**, sem compromisso mínimo (para serviços On-Demand). Sem taxa de setup. Sem contratos de longo prazo obrigatórios. Oposto do on-premises: não precisa comprar hardware para pico de capacidade. Na AWS: EC2 On-Demand faturado por segundo; Lambda por milissegundo; S3 por GB/mês; RDS por hora.

---

**P:** O que é um Managed Service na AWS e qual seu benefício?  
**R:** Serviço onde a AWS gerencia a infraestrutura subjacente: patches, hardware, HA, backups. Ex: RDS (AWS gerencia OS, patches do MySQL, backups), Lambda (AWS gerencia run time, escala), DynamoDB (AWS gerencia storage, replicação). Você é responsável por: configuração, dados, IAM. Reduz operational overhead.

---

**P:** O que é um Elastic IP Address?  
**R:** **IP público estático** persistente na sua conta AWS. Ao contrário do IP público de EC2 que muda a cada reinicialização, o Elastic IP persiste. Associa-se a uma instância EC2 ou NAT Gateway. Gratuito enquanto associado a uma instância running; cobrado ($0.005/hr) quando não associado (para evitar desperdício). Máximo 5 EIPs por conta por padrão.

---

**P:** O que é DNS (Domain Name System) no contexto AWS?  
**R:** Serviço que traduz nomes de domínio (`www.exemplo.com`) para endereços IP. AWS: **Route 53** é o serviço DNS gerenciado. Tipos de record: **A** (nome → IPv4), **AAAA** (nome → IPv6), **CNAME** (nome → outro nome), **Alias** (Route 53 específico, aponta para AWS resources — ELB, CloudFront, S3 Website).

---

**P:** O que é edge computing no contexto AWS?  
**R:** Processamento de dados **próximo da fonte** (dispositivos, usuários) em vez de no data center central. AWS edge: CloudFront PoPs (cache + Lambda@Edge), AWS Outposts (hardware AWS no data center do cliente), Snow Family (Snowball Edge — compute em campo), IoT Greengrass (Lambda em dispositivos IoT), Wavelength (dentro de rede 5G de telecom).

---

**P:** O que é TLS/SSL e como AWS o implementa?  
**R:** **Transport Layer Security** — protocolo de criptografia para comunicação segura pela internet (HTTPS). AWS: **ACM (AWS Certificate Manager)** provisiona, gerencia e renova automaticamente certificados TLS para: CloudFront, ALB, API Gateway. Terminação TLS: CloudFront ou ALB descriptografa e reencripta (ou passa para backend). Gratuito via ACM.

---

**P:** O que é eventual consistency vs strong consistency?  
**R:** **Strong Consistency:** leitura sempre retorna o dado mais recente após write. **Eventual Consistency:** leitura pode retornar dado antigo temporariamente, mas eventualmente converge para o estado atual. AWS: S3 default = strong consistency (desde 2020). DynamoDB: **Eventually Consistent Reads** (padrão, 2x capacity units mais eficiente) e **Strongly Consistent Reads** (disponível, usa mais RCUs).

---

**P:** O que é um Availability Zone (AZ)?  
**R:** Data center(s) fisicamente separado(s) dentro de uma região AWS, com energia, rede e cooling independentes. Conexão entre AZs: rede privada de baixa latência (<1ms). Cada AZ tem ID único real (ex: `use1-az1`), mas nomes (`us-east-1a`) variam por conta (mapeamento aleatório). Falha em uma AZ não deve afetar outras. Multi-AZ = HA contra falha de AZ.

---

**P:** O que é o princípio de Least Privilege?  
**R:** Dar a cada identidade (usuário, role, serviço) **apenas as permissões necessárias** para o trabalho e nada mais. Reduz blast radius: se credencial comprometida, atacante tem acesso mínimo. Na AWS: IAM policies específicas (não `*`), Resource-based policies, SCPs para limitar contas. Best practice: começar com zero permissões e adicionar conforme necessário.

---

**P:** O que é Infrastructure as Code (IaC)?  
**R:** Gerenciar e provisionar infraestrutura via código (declarativo ou imperativo) em vez de manualmente no console. Benefícios: reprodutível, versionável (Git), auditável, testável, documentação automática. AWS nativo: **CloudFormation** (declarativo YAML/JSON). Multi-cloud: **Terraform** (HCL). CDK: código em Python/TypeScript que gera CloudFormation. GitOps: IaC + Git como fonte de verdade.

---

**P:** O que é um Service Mesh e como AWS o implementa?  
**R:** Infraestrutura dedicada para comunicação entre microsserviços (service discovery, load balancing, circuit breaking, observability, mTLS). AWS: **AWS App Mesh** (baseado em Envoy proxy, integrado com ECS/EKS), **Amazon VPC Lattice** (service-to-service networking gerenciado mais simples). Resolve: como serviços se encontram e comunicam de forma segura e observável.

---

**P:** O que é Object Storage vs Block Storage vs File Storage?  
**R:** **Object Storage (S3):** arquivos com metadata e ID único, acesso via HTTP API, escala praticamente ilimitada, não montável como filesystem. **Block Storage (EBS):** volumes de baixo nível (como HD), montados em EC2, alta performance, IOPS configuráveis, ideal para bancos. **File Storage (EFS, FSx):** filesystem compartilhado NFS/SMB, múltiplas instâncias acessam simultaneamente.

---

**P:** O que é o modelo de precificação do AWS Savings Plans?  
**R:** Compromisso de gasto mínimo por hora (ex: $10/hr por 1 ou 3 anos), em troca de desconto. **Compute Savings Plans:** mais flexível — aplica para EC2 (qualquer tipo/região/OS), Fargate e Lambda. **EC2 Instance Savings Plans:** menos flexível (família específica + região), maior desconto (até 72%). Diferente de Reserved Instances: não reserva capacidade específica.

---

**P:** O que é CloudFormation Stack vs StackSet?  
**R:** **Stack:** conjunto de recursos AWS criados/gerenciados como unidade pelo CloudFormation em uma única conta/região. **StackSet:** implanta Stacks em **múltiplas contas e regiões** simultaneamente. Necessário permissões cross-account (IAM Roles). Use StackSets para: baseline de segurança em toda a organização, GuardDuty em todas as contas, regras AWS Config globais.

---

**P:** O que é o AWS Free Tier e quais são seus 3 tipos?  
**R:** **(1) Always Free:** nunca expira. Ex: Lambda 1M req/mês, DynamoDB 25 GB, CloudFront 1 TB, SNS 1M req. **(2) 12-Month Free:** após criar conta, 12 meses de uso limitado. Ex: EC2 t2.micro/t3.micro 750h/mês, S3 5 GB, RDS db.t2.micro 750h/mês. **(3) Trials:** períodos de teste de serviços específicos. Free Tier não cobre todos os recursos — verificar limites.

---

**P:** O que é Tag em recursos AWS e como contribui para Cost Allocation?  
**R:** **Tags:** key-value pairs aplicadas a recursos AWS. Exemplos: `Environment:prod`, `Project:phoenix`, `CostCenter:123`. **Cost Allocation Tags:** ativadas no Billing Console, aparecem no Cost Explorer permitindo filtrar e agrupar custos por tag. Melhores práticas: taggar todos os recursos, automação via CloudFormation/Terraform, SCP que exige tags obrigatórias.

---

**P:** O que é Multi-Factor Authentication (MFA) na AWS?  
**R:** Segundo fator de autenticação além da senha. AWS suporta: **Virtual MFA** (apps como Google Authenticator, Authy — TOTP), **Hardware MFA** (dispositivo físico TOTP ou FIDO2 — YubiKey), **SMS MFA** (descontinuado para IAM). Obrigatório para: root account protection, ações críticas (S3 MFA Delete, IAM delete com MFA condition). IAM best practice: MFA em todos os usuários com console access.

---

**P:** O que é o conceito de "Shared Responsibility Model" e com exemplo de S3?  
**R:** No S3: **AWS:** infraestrutura física, hardware dos servidores S3, rede, durability (11 noves), replicação interna dos objetos. **Cliente:** configuração de bucket policy, Block Public Access settings, criptografia de objetos (SSE-S3/KMS), controle de acesso (IAM policies), versionamento, replicação cross-region. O que acontece COM os dados é responsabilidade do cliente.

---

**P:** O que é o Amazon Resource Name (ARN) de uma Lambda Function?  
**R:** Formato completo: `arn:aws:lambda:us-east-1:123456789012:function:MinhaFuncao`. Componentes: `arn` (prefixo) : `aws` (partition) : `lambda` (serviço) : `us-east-1` (região) : `123456789012` (account ID) : `function` (resource type) : `MinhaFuncao` (resource name). Para alias: `:MinhaFuncao:PROD`. Para version: `:MinhaFuncao:1`.

---

**P:** O que é um "endpoint" no AWS?  
**R:** Pode significar: **(1) VPC Endpoint:** conexão privada entre VPC e serviço AWS. **(2) SageMaker Endpoint:** instância de inference servindo um modelo ML. **(3) API Endpoint:** URL de uma API (ex: `https://abc123.execute-api.us-east-1.amazonaws.com/prod/users`). **(4) DynamoDB Endpoint:** endpoint regional para acesso ao DynamoDB sem internet (VPC endpoint). Contexto determina o significado.

---

**P:** O que é durabilidade vs disponibilidade no S3?  
**R:** **Durabilidade:** probabilidade de não perder os dados. S3 Standard = **11 noves** (99.999999999%) — para 10 milhões de objetos, perder um a cada 10.000 anos. Armazena dados em 3 AZs. **Disponibilidade:** fração do tempo que o serviço está acessível. S3 Standard = 99.99%/ano. S3 One Zone-IA: durávelidade 11 noves (mas em 1 AZ), disponibilidade 99.5%. Durability ≠ availability.

---

**P:** O que é um security finding e como AWS os consolida?  
**R:** **Finding:** registro de um potencial problema de segurança detectado. Gerado por: GuardDuty (ameaças), Inspector (vulnerabilidades EC2/ECR/Lambda), Macie (dados sensíveis S3), Config (non-compliance), IAM Access Analyzer (permissões excessivas). **AWS Security Hub** consolida todos os findings em um dashboard centralizado, normalizado para AWS Security Finding Format (ASFF).
