# Flashcards — Módulo 10: Containers (ECS, EKS, Fargate)

---

**P:** Qual é a hierarquia de objetos no ECS?
**R:** Cluster → Service → Task → Container. Task Definition define o blueprint (imagem, CPU, RAM, vars, volumes, IAM). Service mantém N tasks rodando. Cluster é o agrupamento lógico de recursos

---

**P:** O que é Task Role vs Task Execution Role no ECS?
**R:** Task Role: IAM role para a **aplicação** dentro do container (permissão para acessar S3, DynamoDB, etc.). Task Execution Role: IAM role para o **ECS Agent** (pull imagem do ECR, push logs para CloudWatch, ler Secrets Manager/SSM)

---

**P:** Qual é o único modo de rede suportado pelo Fargate?
**R:** **awsvpc mode**. Cada task Fargate recebe uma ENI dedicada com IP privado da VPC. Permite Security Groups individuais por task. Bridge e host modes só funcionam no EC2 Launch Type

---

**P:** Qual a vantagem do awsvpc mode no ECS EC2?
**R:** Cada task recebe ENI própria com IP da VPC → pode ter SG individual por task (não por instância). Facilita controle de acesso granular e compatibilidade com ALB target type "ip". Desvantagem: limite de ENIs por instância EC2

---

**P:** Qual o limite de CPU e RAM que pode ser configurado em uma task Fargate?
**R:** CPU: 0,25 vCPU a 16 vCPU. RAM: proporcional ao CPU (ex: 0,25 vCPU → 0,5-2 GB; 16 vCPU → 32-120 GB). Cobrado por vCPU-segundo e GB-segundo enquanto a task está ativa

---

**P:** Quais são os 3 Launch Types disponíveis no ECS?
**R:** EC2 (você gerencia as instâncias EC2 do cluster), Fargate (serverless, AWS gerencia o host), External/ECS Anywhere (servidores on-premises ou edge registrados como cluster ECS)

---

**P:** O que é ECS Capacity Provider?
**R:** Gerencia automaticamente o Auto Scaling Group de instâncias EC2 do cluster baseado em tasks pendentes, otimizando bin packing. Com "Managed Scaling" habilitado, o ECS aumenta/diminui instâncias automaticamente antes de tasks ficarem queued

---

**P:** Qual é o custo do control plane de um cluster EKS?
**R:** $0,10/hora por cluster EKS (~$72/mês). ECS não cobra pelo cluster (apenas pelo compute: EC2 ou Fargate). Considere este custo fixo para workloads pequenos em EKS

---

**P:** O que são EKS Managed Node Groups?
**R:** AWS gerencia o lifecycle dos EC2 worker nodes (provisioning, patching, updates com zero downtime, draining). Você escolhe o tipo de instância e AMI. Diferente do Fargate (sem nodes) e Self-managed nodes (você gerencia tudo manualmente)

---

**P:** O que é IRSA (IAM Roles for Service Accounts) no EKS?
**R:** Permite associar IAM Roles a Kubernetes Service Accounts. Pods que usam o Service Account recebem credenciais temporárias AWS via OIDC (sem hardcode). Equivalente ao ECS Task Role mas para pods Kubernetes — granularidade por pod/deployment

---

**P:** O que é Amazon VPC CNI no EKS?
**R:** Plugin de rede padrão do EKS. Aloca IPs diretamente da VPC para cada pod (não usa overlay network). Resultado: pods têm IPs nativos da VPC, comunicam diretamente com outros recursos AWS, podem ter SGs aplicados diretamente (EKS Security Groups for Pods)

---

**P:** O que é o ECR Lifecycle Policy e como funciona?
**R:** Regras que apagam automaticamente imagens baseado em critérios: age (dias), count (manter N mais recentes), tag status (tagged/untagged). Exemplos: "apagar imagens untagged com mais de 7 dias" ou "manter apenas as 5 imagens mais recentes por tag prefix"

---

**P:** Qual a diferença entre ECR Image Scanning Basic e Enhanced?
**R:** Basic (Clair): verifica CVEs do SO (OS packages). Enhanced (Amazon Inspector): verifica CVEs no SO E em linguagens de programação (Python, Node.js, Java packages). Enhanced = mais detalhado. Resultados disponíveis no Inspector Console + EventBridge events

---

**P:** Como containers ECS acessam segredos (credenciais de DB) de forma segura?
**R:** Na Task Definition, campo `secrets`: referencie ARN do Secrets Manager ou SSM Parameter Store. O ECS Agent (via Task Execution Role) busca o segredo no momento do launch e injeta como variável de ambiente ou arquivo. Nenhuma credencial hardcoded

---

**P:** O que é AWS App Runner?
**R:** PaaS serverless para containers web. Você fornece imagem ECR ou código fonte → App Runner cria automaticamente load balancer, HTTPS, auto-scaling, custom domain. Zero configuração de ECS/Fargate/ALB. Mais simples que ECS, menos controle

---

**P:** O que é ECS Service Connect?
**R:** Service Mesh gerenciado pela AWS para comunicação service-to-service dentro de um ECS cluster. Oferece service discovery, roteamento, retries, timeouts e observabilidade (métricas, tracing) sem configurar App Mesh ou Envoy manualmente

---

**P:** Qual é o modo de scaling no EKS para pods e para nodes?
**R:** Pods: **HPA** (Horizontal Pod Autoscaler) — escala número de pods baseado em CPU/RAM/métricas custom. Nodes: **Cluster Autoscaler** (escala node groups) ou **Karpenter** (provisionamento de nodes mais rápido e eficiente, AWS-native). Fargate nodes: escalam automaticamente por pod

---

**P:** O que é EKS Fargate Profile?
**R:** Seletor que define quais pods rodam no Fargate (por namespace e labels). Pod que corresponde ao profile é automaticamente agendado no Fargate (sem instâncias EC2). Pods que não correspondem vão para node groups. Cada pod Fargate = micro-VM isolada

---

**P:** Como funciona a autenticação no Amazon ECR para Docker pull?
**R:** Usa o AWS CLI: `aws ecr get-login-password --region ... | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com`. Token válido por 12 horas. Kubernetes/EKS: usa ECR credential helper ou IRSA para renovação automática

---

**P:** O que é AWS Copilot CLI?
**R:** CLI que automatiza deploy de aplicações containerizadas no ECS/Fargate seguindo best practices. Cria automaticamente: ECR, Task Definition, ECS Service, ALB, VPC, CodePipeline, SSM/Secrets. Abstrai a complexidade sem remover controle como o App Runner faz
