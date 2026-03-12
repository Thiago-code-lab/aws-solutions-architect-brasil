# Questões de Prova — Módulo 10: Containers (ECS, EKS, Fargate)

<!-- Domínio SAA-C03: Design High-Performing Architectures / Design Resilient Architectures -->

---

**1.** Uma equipe quer rodar containers na AWS sem gerenciar servidores EC2. Qual combinação permite isso com ECS?

- A) ECS com EC2 Launch Type
- B) ECS com Fargate Launch Type
- C) ECS com External Launch Type
- D) ECS com Auto Scaling Groups manualmente configurados

<details>
<summary>Resposta</summary>
**B — ECS com Fargate**
Fargate é serverless: você define a task (CPU, RAM, imagem), e a AWS gerencia todo o host subjacente. Sem EC2 para provisionar, patchear ou dimensionar. EC2 Launch Type exige que você registre instâncias EC2 no cluster (você gerencia o fleet). External é para servidores on-prem (ECS Anywhere).
</details>

---

**2.** Qual é a diferença entre Task Role e Task Execution Role no ECS?

- A) Task Role é para o ECS agent; Execution Role é para o container
- B) Task Role é para a aplicação dentro do container; Execution Role é para o ECS gerenciar a task (pull imagem ECR, push logs)
- C) Ambas são equivalentes — distinção apenas de nomenclatura
- D) Task Role se aplica ao EC2 Launch Type; Execution Role ao Fargate

<details>
<summary>Resposta</summary>
**B — Correto**
**Task Role**: políticas IAM para o que a **aplicação** pode fazer (ex: ler S3, escrever DynamoDB). Obtido via Task Metadata Endpoint (IMDS da task). **Task Execution Role**: para o ECS e Fargate agent: pull imagem do ECR, enviar logs para CloudWatch, ler Secrets Manager/SSM Parameter Store. Ambas são necessárias quando a aplicação acessa AWS e usa ECR.
</details>

---

**3.** Um container em ECS com launch type Fargate precisa que cada task tenha seu próprio Security Group para controle granular de rede. Qual modo de rede habilitar?

- A) bridge mode
- B) host mode
- C) awsvpc mode
- D) none mode

<details>
<summary>Resposta</summary>
**C — awsvpc mode**
Com `awsvpc`, cada task recebe **uma ENI (Elastic Network Interface) dedicada** com IP privado da VPC. Isso permite associar Security Groups individuais por task. É o **único modo suportado pelo Fargate**. `bridge` usa portas dinâmicas do host EC2. `host` usa a rede do host diretamente.
</details>

---

**4.** Uma empresa usa ECS EC2 launch type e quer distribuir tráfego entre múltiplos containers na mesma instância EC2, com portas diferentes atribuídas dinamicamente. Qual configuração no ALB?

- A) Application Load Balancer com target type "instance" e dynamic port mapping
- B) Network Load Balancer com IP targets fixos
- C) Application Load Balancer com target type "ip" no awsvpc mode
- D) Classic Load Balancer com sticky sessions

<details>
<summary>Resposta</summary>
**A — ALB com dynamic port mapping (bridge mode)**
No ECS EC2 com bridge mode, a nenhuma porta fixa é definida (ou usa hostPort=0) — o ECS atribui portas dinâmicas do host. O ALB com target type "instance" usa o ECS service para registrar automaticamente os pares (instância, porta dinâmica). Com awsvpc, usa-se target type "ip" (cada task tem seu IP único).
</details>

---

**5.** Qual componente do ECS define os containers (imagem Docker, CPU, RAM, variáveis de ambiente, volumes, IAM) de uma task?

- A) ECS Service
- B) ECS Cluster
- C) Task Definition
- D) Capacity Provider

<details>
<summary>Resposta</summary>
**C — Task Definition**
Task Definition é o blueprint JSON que define: imagem(ns) Docker, CPU/RAM alocados, mapeamento de portas, volumes montados, variáveis de ambiente, Task Role, Task Execution Role, log driver, etc. É versionado. ECS Service usa a Task Definition para lançar e manter N tasks em execução.
</details>

---

**6.** Uma empresa usa Amazon EKS e quer rodar pods sem gerenciar worker nodes (serverless). Qual configuração usar?

- A) EKS Managed Node Groups com Auto Scaling
- B) EKS Self-managed Nodes com Spot instances
- C) EKS com Fargate Profiles
- D) EKS com AWS Batch integration

<details>
<summary>Resposta</summary>
**C — EKS Fargate Profiles**
Fargate Profiles no EKS definem quais pods (por namespace/labels) devem rodar no Fargate. AWS provê e gerencia o compute; você não vê nem gerencia EC2 nodes. Managed Node Groups gerenciam EC2 nodes automaticamente mas você ainda escolhe tipos, AMIs, etc. Fargate = zero node management.
</details>

---

**7.** Qual é o custo aproximado do control plane de um cluster EKS por hora?

- A) Gratuito — apenas os worker nodes são cobrados
- B) $0,10/hora por cluster EKS (~$72/mês)
- C) $0,50/hora por cluster
- D) Cobrado por número de pods em execução

<details>
<summary>Resposta</summary>
**B — $0,10/hora por cluster (~$72/mês)**
EKS cobra $0,10 por hora pelo control plane gerenciado (API server, etcd, etc.) — isso é fixo independente do número de nodes/pods. ECS não tem custo de cluster; apenas compute (EC2 ou Fargate). Isso é um diferencial importante ao escolher ECS vs EKS para workloads pequenos.
</details>

---

**8.** Uma imagem Docker foi enviada ao ECR. Como garantir que apenas as 10 imagens mais recentes por repositório sejam retidas (apagar imagens antigas automaticamente)?

- A) S3 Lifecycle Policy integrada ao ECR
- B) ECR Lifecycle Policy com regra de contagem de imagens
- C) Lambda com CloudWatch Events limpando imagens semanalmente
- D) ECR Image Scanning com auto-delete de imagens antigas

<details>
<summary>Resposta</summary>
**B — ECR Lifecycle Policy**
ECR Lifecycle Policies permitem criar regras como: manter apenas as N imagens mais recentes por tag prefix, apagar imagens não-tagueadas após X dias, etc. Aplicadas automaticamente pelo ECR sem computação adicional. Image Scanning detecta vulnerabilidades (CVEs) — não gerencia retenção.
</details>

---

**9.** O que é IRSA (IAM Roles for Service Accounts) no EKS?

- A) Permite que pods EKS usem roles IAM sem precisar de credenciais hardcoded
- B) Serviço para federar identidades IAM em múltiplos clusters EKS
- C) Mecanismo de autenticação de nodes EKS com o control plane
- D) Permite acesso cross-account entre clusters EKS

<details>
<summary>Resposta</summary>
**A — IAM Roles para pods individuais**
IRSA permite associar um IAM Role a um Kubernetes Service Account. Pods que usam esse Service Account recebem credenciais temporárias da AWS automaticamente (via OIDC). Isso é o equivalente ao Task Role no ECS — sem colocar credenciais em variáveis de ambiente ou usar o role do node inteiro para o pod.
</details>

---

**10.** Uma empresa quer usar ECS ou EKS. A equipe tem pouca experiência mas precisa integrar com Kubernetes Helm Charts de terceiros. Qual escolher?

- A) ECS — mais simples e sem custo de cluster
- B) EKS — suporte nativo a Kubernetes e ecossistema (Helm, Operators, etc.)
- C) App Runner — elimina necessidade de orquestração
- D) EC2 com Docker Compose

<details>
<summary>Resposta</summary>
**B — EKS**
Se o requisito explícito é compatibilidade com Kubernetes (Helm Charts, Operators, kubectl, RBAC nativo), EKS é a escolha. ECS não é Kubernetes e não suporta Helm. Se a prioridade fosse simplicidade sem Kubernetes, ECS (especialmente + Fargate) seria melhor. A presença de Helm Charts como requisito determina EKS.
</details>

---

**11.** Como funciona a escalabilidade automática de tasks no ECS com base em métricas da fila SQS?

- A) ECS Service Auto Scaling com Target Tracking Policy baseada em ApproximateNumberOfMessagesVisible
- B) Lambda function que invoca ECS RunTask conforme a fila cresce
- C) SQS Auto Scaling Group trigger para ECS Cluster
- D) CloudWatch Alarm → SNS → ECS TaskSet update

<details>
<summary>Resposta</summary>
**A — ECS Service Auto Scaling com Target Tracking**
ECS Service Auto Scaling suporta Target Tracking, Step Scaling e Scheduled Scaling. Para SQS: crie uma CloudWatch Custom Metric com `ApproximateNumberOfMessagesVisible / número_de_tasks`, configure Target Tracking para manter essa proporção num alvo (ex: 10 mensagens por task). ECS escala tasks automaticamente com base na carga da fila.
</details>

---

**12.** Qual serviço AWS é mais adequado para deploy rápido de uma aplicação web containerizada com zero configuração de infraestrutura (sem ECS, ALB, etc.)?

- A) ECS + Fargate + ALB (manual)
- B) AWS Copilot CLI
- C) AWS App Runner
- D) Elastic Beanstalk com Docker

<details>
<summary>Resposta</summary>
**C — AWS App Runner**
App Runner é PaaS para containers web: você fornece a imagem ECR (ou código fonte) e App Runner cria e gerencia toda a infra (load balancer, auto-scaling, HTTPS, custom domain). Zero configuração de ECS/ALB. Elastic Beanstalk com Docker é mais flexível mas mais complexo. Copilot automatiza ECS/Fargate mas ainda expõe os componentes.
</details>

---

**13.** Qual o mecanismo seguro recomendado para que um container ECS acesse credenciais de banco de dados armazenadas no Secrets Manager?

- A) Injetar a credencial como variável de ambiente hardcoded na Task Definition
- B) Montar um volume EFS com o arquivo de credenciais
- C) Referenciar o Secrets Manager ARN na Task Definition — ECS injeta automaticamente como env var ou arquivo
- D) Lambda sidecar container que busca a credencial e passa via shared volume

<details>
<summary>Resposta</summary>
**C — Referenciamento de Secrets Manager na Task Definition**
ECS suporta injeção de segredos diretamente da Task Definition: no campo `secrets`, referencie o ARN do Secrets Manager (ou SSM Parameter Store). O ECS Execution Role puxa o valor no momento do launch da task e injeta como variável de ambiente. Nenhuma credencial fica hardcoded. C é o padrão AWS recomendado.
</details>

---

**14.** O que diferencía ECS Capacity Provider de simplesmente usar Auto Scaling Groups manualmente no ECS EC2 Launch Type?

- A) Capacity Provider gerencia automaticamente o scaling de instâncias EC2 baseado em tasks pendentes, sem intervenção manual
- B) Capacity Provider é exclusivo para Fargate, não funciona com EC2
- C) Não há diferença — Capacity Provider é apenas um alias para ASG
- D) Capacity Provider elimina a necessidade de Task Definitions

<details>
<summary>Resposta</summary>
**A — Gestão automática de scaling do cluster**
Capacity Provider com Managed Scaling: escala automaticamente o Auto Scaling Group do cluster EC2 com base em tasks pendentes (bin packing otimizado). Sem CP, você gerencia manualmente ou configura scaling policies simples. Com CP, o ECS "sabe" quantas instâncias são necessárias para acomodar todas as tasks e escala proativamente antes de tasks ficarem pendentes.
</details>

---

**15.** Uma empresa quer verificar vulnerabilidades CVE nas imagens Docker no ECR antes que sejam deployadas. Qual configuração?

- A) SonarQube integrado ao ECR
- B) ECR Image Scanning (Basic via Clair ou Enhanced via Amazon Inspector)
- C) GuardDuty com ECR integration
- D) CodeBuild step que executa Trivy manualmente

<details>
<summary>Resposta</summary>
**B — ECR Image Scanning**
ECR oferece dois tipos de scanning: **Basic** (usa Clair open-source, verifica CVEs do OS) habilitado on-push ou manual. **Enhanced Scanning** (via Amazon Inspector) verifica CVEs no OS e em linguagens de programação (Python packages, Node.js modules, etc.) com resultados no Console Inspector e EventBridge events para automação. Amazon Inspector → ECR = scanning contínuo das imagens.
</details>
