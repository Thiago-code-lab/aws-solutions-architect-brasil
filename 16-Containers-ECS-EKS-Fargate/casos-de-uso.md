# Casos de Uso — Módulo 10: Containers (ECS, EKS, Fargate)

## Caso 1: Microserviços com ECS Fargate + ALB + Service Connect

**Cenário:** Startup de fintech com 5 microserviços. Equipe pequena quer containers sem gerenciar infra. Comunicação inter-serviços segura com observabilidade.

**Arquitetura:**
```
Internet → Route 53 → ALB (api.fintech.com)
                        ├── /payments/*  → ECS Service: PaymentService (Fargate)
                        ├── /accounts/*  → ECS Service: AccountService (Fargate)
                        └── /auth/*      → ECS Service: AuthService (Fargate)

Comunicação interna (ECS Service Connect):
  PaymentService → http://accountservice:8080/validate
  PaymentService → http://authservice:8080/verify
  [Service Connect = DNS interno, retries automáticos, métricas no CloudWatch]

IAM (Task Roles):
  PaymentService  → DynamoDB (payments table) + SQS (payment events)
  AccountService  → RDS Aurora (accounts DB)
  AuthService     → DynamoDB (sessions) + Secrets Manager (JWT keys)

ECR: repositório separado por serviço
Secrets: Secrets Manager → injetado via Task Definition (Execution Role)
Logs: awslogs driver → CloudWatch Logs Groups por serviço
```

---

## Caso 2: EKS com Managed Node Groups para Plataforma de Dados

**Cenário:** Empresa com equipe DevOps experiente em Kubernetes. Precisa de plataforma de dados com Spark, Kafka e APIs REST containerizadas.

**Arquitetura:**
```
EKS Cluster (us-east-1)
  ├── Node Group: general (m6i.xlarge, 4 nodes, On-Demand)
  │     └── Pods: API services, monitoring, ingress controllers
  │
  ├── Node Group: compute-intensive (c6i.4xlarge, 0-20 nodes, Spot)
  │     └── Pods: Spark executors, Kafka brokers
  │
  └── Fargate Profile: namespace=monitoring
        └── Pods: Prometheus, Grafana (serverless)

Add-ons AWS:
  ├── AWS Load Balancer Controller → ALB por Ingress Kubernetes
  ├── Amazon VPC CNI → pods com IPs da VPC
  ├── AWS EBS CSI Driver → PVCs para Kafka persistent storage
  └── ADOT (AWS Distro for OpenTelemetry) → traces para X-Ray

IRSA:
  ServiceAccount: spark-sa → IAM Role → S3 (data lake read/write)
  ServiceAccount: kafka-sa → IAM Role → MSK (acesso ao cluster Kafka)
```

---

## Caso 3: Deployment Blue/Green no ECS com CodeDeploy

**Cenário:** API crítica com requerimento de zero-downtime deployments e rollback em < 5 minutos em caso de problema.

**Proceso:**
```
Git Push → CodePipeline
  ├── CodeBuild: docker build → ECR push (v2.0)
  └── CodeDeploy (ECS Blue/Green):
        1. Cria novo task set "Green" com imagem v2.0
        2. ALB redireciona % de tráfego gradualmente para Green
           [Canary: 10% por 5 min → se ok → 90% → 100%]
        3. Health Check period: 5 minutos
        4. Sucesso: termina tasks Blue
        ---- OU ----
        4. Falha detectada → CodeDeploy reverte ALB para Blue em < 1 min
        5. Green tasks terminadas automaticamente

Configuração ECS Service:
  Deployment Type: CODE_DEPLOY
  ALB: 2 target groups (Blue + Green)
  CodeDeploy App: appspec.yml define hooks Lambda para smoke tests
```

---

## Caso 4: Workload de ML com EKS + Karpenter + GPU Nodes

**Cenário:** Equipe de ML precisa treinar modelos com GPUs (p3.8xlarge) que ficam ociosos entre experimentos. Custo é crítico.

**Arquitetura:**
```
EKS Cluster
  └── Karpenter (node provisioner) ← ao invés de Cluster Autoscaler
        ├── NodeClass: GPU (p3.8xlarge Spot, max 10 nodes)
        └── NodePool: ml-training (namespace=ml, tolerations=gpu)

Fluxo de Treinamento:
1. Data Scientist submete Job Kubernetes (spec: GPU request)
2. Karpenter detecta pod Pending (sem node disponível)
3. Karpenter provê GPU Spot node em ~30 segundos
4. Pod inicia treinamento ML
5. Job completa → pod terminado
6. Karpenter termina node após 2 min de ociosidade

IRSA para acesso ao S3:
  ServiceAccount: ml-training → IAM Role → S3 (datasets, model artifacts)
  
FSx for Lustre (CSI Driver):
  PVC → PersistentVolume → FSx for Lustre (integrado com S3)
  [Alta velocidade de leitura dos datasets durante treinamento]
```

**Economia:** Spot GPU instance = 70% de desconto. Karpenter termina nodes em minutos de inatividade vs Cluster Autoscaler que esperava 10+ minutos.

---

## Caso 5: Migração para ECS com App Runner e Esteira CI/CD

**Cenário:** Startup migra de VM on-premises para containers sem experiência em Kubernetes ou ECS. Precisa CI/CD simples.

**Arquitetura Simplificada (App Runner):**
```
GitHub Repo (main branch)
  └── CodePipeline:
        ├── CodeBuild: docker build → ECR
        └── App Runner Deploy: atualiza service com nova imagem
              └── App Runner Service:
                    ├── Auto-scaling: min=1, max=10 containers
                    ├── HTTPS automático (ACM gerenciado)
                    ├── Custom domain: api.startup.io
                    └── Source: ECR image

VPC Connector (App Runner + VPC):
  App Runner → VPC Connector → RDS (subnet privada)
               [App Runner originalmente sem acesso à VPC; VPC Connector resolve]

Limites App Runner (motivação futura para ECS):
  ✗ Sem controle de VPC completo
  ✗ Sem múltiplos containers por service
  ✗ Sem persistent volumes
  ✗ Apenas HTTP workloads
  → Quando atingir esses limites: migrar para ECS Fargate
```

**Progressão recomendada:**
```
App Runner (início) → ECS Fargate (quando precisar de controle) → EKS (quando precisar K8s)
```
