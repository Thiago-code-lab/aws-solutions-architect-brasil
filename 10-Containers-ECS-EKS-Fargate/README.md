# Módulo 10 — Containers: ECS, EKS, Fargate

## Por Que Containers na AWS?

Containers oferecem consistência entre ambientes (dev/staging/prod), isolamento de dependências e densidade de workloads. A AWS oferece dois orquestradores principais: **ECS** (proprietary, simplesidade) e **EKS** (Kubernetes, portabilidade).

---

## Amazon ECS (Elastic Container Service)

### Conceitos Fundamentais
```
ECS Cluster
  └── Service (define quantas Tasks rodar + onde)
        └── Task (1-N containers rodando juntos)
              └── Task Definition (blueprint: imagem, CPU, RAM, ports, volumes, IAM role)
```

- **Task Definition**: arquivo JSON versionado que define containers (como Dockerfile mas para o ECS)
- **Task**: instância rodando de uma Task Definition (1 task = 1 ou mais containers co-localizados)
- **Service**: garante que N tasks estejam sempre rodando; integra com ALB/NLB; rolling updates
- **Cluster**: agrupamento lógico de resources (pode ser EC2 instances ou Fargate)

### Launch Types

| Launch Type | Infraestrutura | Gerenciamento | Controle |
|---|---|---|---|
| **EC2** | EC2 instances que você registra no cluster | Você gerencia as instâncias (AMI, patches, scaling) | Total sobre o host |
| **Fargate** | Serverless — AWS gerencia o host | Sem instâncias para gerenciar | Apenas task-level |
| **External (ECS Anywhere)** | Seus servidores on-premises | Você gerencia | Híbrido |

### Modos de Rede do ECS

| Modo | Descrição | Quando usar |
|---|---|---|
| **bridge** | Container ports mapeados para IPs dinâmicos do host | EC2, múltiplos containers no mesmo host |
| **host** | Container usa diretamente a rede do host | EC2, alta performance, sem NAT |
| **awsvpc** | Cada task recebe própria ENI com IP privado da VPC | **Recomendado** — Fargate obrigatório; isolamento de SG por task |
| **none** | Sem rede | Tasks offline |

Com `awsvpc`, cada task tem seu próprio Security Group — granularidade máxima de isolamento de rede.

### IAM no ECS
| Papel IAM | Para | Quem usa |
|---|---|---|
| **Task Role** | O que o container pode fazer (S3, DynamoDB, etc.) | A aplicação dentro do container |
| **Task Execution Role** | O que o ECS pode fazer para lançar a task (pull ECR, push CloudWatch Logs, Secrets Manager) | O ECS agent |

> **Nunca embuta credenciais no container** — use Task Role com `awsvpc` mode. As credenciais são obtidas via IMDS do task metadata endpoint.

### ECS + ALB: Service Discovery e Load Balancing
- **ALB com dynamic port mapping** (launch type EC2): containers registram portas dinâmicas no TG
- **NLB** para tráfego alto throughput ou non-HTTP
- **Service Connect**: Service Mesh gerenciado para comunicação service-to-service dentro do ECS (mTLS, observabilidade)
- **Cloud Map**: service discovery baseado em DNS para comunicação entre containers

### ECS Auto Scaling
- Escala o número de **tasks** (não instâncias — para EC2 Launch Type, escalar instâncias é separado)
- Políticas: Target Tracking, Step Scaling, Scheduled
- **ECS Capacity Provider**: gerencia automaticamente o scaling do cluster EC2 subjacente com base nas tasks pendentes

---

## AWS Fargate

Compute engine serverless para ECS e EKS. Você não gerencia servidores:

- Define: CPU (0,25 vCPU a 16 vCPU) + RAM (proporcional ao CPU)
- Cobrado por vCPU-hora e GB-hora enquanto task está rodando
- Cada task em Fargate obrigatoriamente usa `awsvpc` (própria ENI)
- **Spot Fargate**: até 70% de desconto para tasks interrompíveis
- **Fargate Savings Plans**: compromisso de uso para desconto

---

## Amazon EKS (Elastic Kubernetes Service)

Kubernetes gerenciado na AWS. AWS gerencia o control plane (API server, etcd); você gerencia os worker nodes.

### Modos de Worker Nodes

| Modo | Descrição | Flexibilidade |
|---|---|---|
| **Managed Node Groups** | AWS gerencia o lifecycle dos EC2 nodes (launch, update, terminate) | Alta — pode customizar AMI, tipo, etc. |
| **Self-managed Nodes** | Você gerencia completamente (launch template, userdata) | Total |
| **Fargate Profiles** | Pods rodam em Fargate (serverless) | Sem node management |

### Componentes EKS
- **Control Plane**: gerenciado pela AWS (multi-AZ por padrão, altamente disponível)
- **Worker Nodes**: EC2 ou Fargate onde os Pods rodam
- **Add-ons**: AWS gerencia componentes como CoreDNS, kube-proxy, Amazon VPC CNI, AWS Load Balancer Controller
- **Amazon VPC CNI**: pods recebem IPs diretamente da VPC (igual ao ECS awsvpc — cada pod = IP da VPC)

### EKS Storage
- **EBS**: volumes persistentes para pods (node-specific, não multi-attach)
- **EFS**: volumes compartilhados entre múltiplos pods/nodes
- **Amazon FSx for Lustre**: high-performance para ML/HPC workloads

### EKS vs ECS

| Aspecto | ECS | EKS |
|---|---|---|
| Orquestrador | Proprietário AWS | Kubernetes (open source) |
| Curva de aprendizado | Menor | Maior |
| Portabilidade | Apenas AWS | Qualquer Kubernetes (on-prem, GKE, AKS) |
| Ecossistema | Integrado com AWS natively | Helm charts, operadores Kubernetes |
| Multi-cloud/on-prem | ECS Anywhere (limitado) | EKS Anywhere, Kubernetes em qualquer lugar |
| Custo control plane | Sem custo de cluster | $0,10/hora por cluster EKS |

---

## Amazon ECR (Elastic Container Registry)

Registry privado (e público) de imagens Docker/OCI:

- **Repositórios privados**: imagens acessíveis apenas dentro da conta (ou cross-account via resource policy)
- **Repositórios públicos** (ECR Public): imagens públicas (hub.docker similar)
- **Image Scanning**: enhanced scanning via Amazon Inspector (CVEs, malware) ou básica
- **Lifecycle Policies**: descarta imagens antigas automaticamente (ex: manter apenas as 10 mais recentes)
- **Replication**: cross-region ou cross-account para DR e distribuição global

---

## Outros Serviços de Container

### AWS App Runner
- PaaS para containers web: você fornece o código ou imagem, App Runner gerencia tudo
- Auto-scaling, HTTPS, load balancing — sem configuração de ECS/Fargate/ALB
- Mais simples, menos controle

### AWS Copilot
- CLI para implantar aplicações containerizadas no ECS/Fargate com melhores práticas incorporadas
- Cria automaticamente: ECR, ECS Service, ALB, SSM Parameters, etc.

### AWS App Mesh
- Service Mesh baseado em **Envoy proxy** para ECS e EKS
- Observabilidade (tracing, metrics) e mTLS para comunicação service-to-service
- Alternativa AWS-native ao Istio

---

## Arquitetura de Referência: Microserviços com ECS Fargate

```
Route 53 → ALB (public)
              ├── /api/users/* → ECS Service: UserService (Fargate)
              │                       └── Task Role → DynamoDB (Users table)
              ├── /api/orders/* → ECS Service: OrderService (Fargate)
              │                       └── Task Role → RDS Aurora (Orders DB)
              └── /api/notify/* → ECS Service: NotifyService (Fargate)
                                      └── Task Role → SNS/SES

Comunicação interna:
  OrderService → SNS "order-placed" → SQS → NotifyService
  (fan-out via EventBridge para auditoria no S3)

ECR → Task Definition (imagem versionada)
CloudWatch Logs → Container logs (awslogs driver)
AWS X-Ray → Distributed tracing
```

---

## Quando Usar o Quê

| Necessidade | Serviço |
|---|---|
| Simplicidade máxima, app web containerizada | App Runner |
| Containers AWS-native sem gerenciar infra | ECS + Fargate |
| Containers AWS-native com controle do host | ECS + EC2 |
| Kubernetes para portabilidade | EKS |
| Kubernetes serverless | EKS + Fargate |
| Kubernetes on-premises | EKS Anywhere |

---

## Dicas de Prova

- **Task Role** = permissões da aplicação; **Task Execution Role** = permissões do ECS para gerenciar a task
- **awsvpc mode** = cada task tem ENI e SG próprios — isolamento de rede por task (Fargate obrigatório)
- **ECS EC2** ainda precisa de EC2 instances no cluster; **Fargate** = zero infra para gerenciar
- **Managed Node Groups** no EKS ≠ Fargate — ainda são EC2 nodes mas gerenciados pela AWS
- **ECR Lifecycle Policy** evita acúmulo de imagens antigas; usar para controle de custos
- EKS tem custo fixo de cluster ($0,10/hora ≈ $72/mês); ECS não cobra pelo cluster
- Para portabilidade multi-cloud: EKS (Kubernetes) é mais adequado que ECS
- **VPC CNI no EKS** = pods com IPs da VPC (como awsvpc no ECS) — importante para segurança e roteamento
- ECS Auto Scaling escala **tasks**; Capacity Provider escala o **cluster EC2** subjacente automaticamente
- **Service Connect** = nova alternativa ao App Mesh para comunicação service-to-service no ECS
