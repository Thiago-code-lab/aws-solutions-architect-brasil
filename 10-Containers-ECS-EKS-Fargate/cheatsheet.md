# Cheatsheet — Módulo 10: Containers (ECS, EKS, Fargate)

## ECS Launch Types

| | EC2 Launch Type | Fargate Launch Type | External (ECS Anywhere) |
|---|---|---|---|
| Gerencia instâncias? | SIM (você) | NÃO (AWS) | SIM (on-prem/edge) |
| Custo | EC2 + ECS (cluster grátis) | vCPU/hora + GB/hora | EC2-like por node |
| Controle | Total sobre o host | Apenas task level | Total sobre o hardware |
| Rede disponível | bridge, host, awsvpc | Apenas **awsvpc** | bridge, host, awsvpc |

## ECS Task Definition — Parâmetros Críticos

```json
{
  "family": "minha-app",
  "taskRoleArn": "arn:aws:iam::...:role/TaskRole",
  "executionRoleArn": "arn:aws:iam::...:role/TaskExecutionRole",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [{
    "name": "app",
    "image": "account.dkr.ecr.region.amazonaws.com/app:v1.0",
    "portMappings": [{"containerPort": 8080}],
    "secrets": [{"name": "DB_PASSWORD", "valueFrom": "arn:aws:secretsmanager:..."}],
    "logConfiguration": {"logDriver": "awslogs", "options": {"awslogs-group": "/ecs/app"}}
  }]
}
```

## ECS IAM Roles

| Role | Quem usa | Para que |
|---|---|---|
| Task Role | Aplicação no container | Acessar AWS services (S3, DynamoDB, SQS) |
| Task Execution Role | ECS Agent / Fargate | Pull imagem ECR, push CloudWatch Logs, ler Secrets Manager |
| Instance Profile | EC2 node (EC2 Launch Type) | ECS Agent se registrar no cluster |

## Modos de Rede ECS

| Modo | Funciona com | IP da Task | SG por Task | Notas |
|---|---|---|---|---|
| bridge | EC2 | IP do host + porta dinâmica | Não (SG do host) | Dynamic port mapping com ALB |
| host | EC2 | IP do host | Não | Sem NAT, alta performance |
| **awsvpc** | EC2 e **Fargate** | **IP próprio da VPC** | **SIM** | Recomendado; ENI por task |
| none | EC2 | Nenhum | Não | Tasks offline |

## ECS vs EKS — Quando Usar

| Critério | ECS | EKS |
|---|---|---|
| Simples + AWS-native | ✅ Melhor | Pode usar |
| Kubernetes compatível | ❌ | ✅ Obrigatório |
| Helm Charts / Operators | ❌ | ✅ |
| Multi-cloud / portabilidade | ❌ | ✅ |
| Custo de control plane | Gratuito | $0,10/hora ≈ $72/mês |
| Curva de aprendizado | Baixa | Alta |

## EKS — Tipos de Worker Nodes

| Tipo | Gerenciamento | Flexibilidade |
|---|---|---|
| Managed Node Groups | AWS (lifecycle EC2 gerenciado) | AMI, tipos, tamanho escolhidos por você |
| Self-managed Nodes | Você (launch template, userdata) | Total |
| Fargate Profiles | AWS (serverless) | Zero node management (por pod) |

## ECR — Configurações Importantes

| Feature | Descrição |
|---|---|
| Lifecycle Policy | Automatiza deleção de imagens por contagem ou idade |
| Image Scanning | Basic (Clair, OS CVEs) ou Enhanced (Inspector, OS + linguagens) |
| Replication | Cross-region e/ou cross-account (para DR, distribuição global) |
| Immutable Tags | Previne sobrescrita de tags existentes (ex: `latest` imutável) |
| Pull Through Cache | Proxy cache de registries públicos (DockerHub, ECR Public) |

## Fargate CPU/RAM Combinations

| vCPU | RAM (GB) |
|---|---|
| 0.25 | 0.5, 1, 2 |
| 0.5 | 1 a 4 |
| 1 | 2 a 8 |
| 2 | 4 a 16 |
| 4 | 8 a 30 |
| 8 | 16 a 60 |
| 16 | 32 a 120 |

## Container Services Comparison

| Serviço | Gerenciamento | Kubernetes? | Melhor para |
|---|---|---|---|
| ECS + EC2 | Médio (EC2 nodes) | Não | Controle + AWS native |
| ECS + Fargate | Baixo (serverless) | Não | Simplicidade sem infra |
| EKS + Node Groups | Médio | **Sim** | K8s gerenciado |
| EKS + Fargate | Baixo | **Sim** | K8s + serverless |
| App Runner | Zero | Não | Deploy ultra-rápido |
| Elastic Beanstalk | Baixo-médio | Não | Apps web tradicionais |

## Dicas Rápidas de Prova
- **Task Role** = o que a app faz; **Execution Role** = o que o ECS faz para lançar a task
- **awsvpc** = única opção no Fargate; dá ENI + IP dedicado + SG próprio por task
- **ECS custos**: cluster EC2 gratuito, Fargate cobra por vCPU-hora e GB-hora
- **EKS custos**: $0,10/hora pelo control plane (independente de nodes/pods)
- **IRSA (IAM Roles for Service Accounts)**: EKS equivalent do ECS Task Role — role por pod via OIDC
- **Managed Node Groups**: AWS gerencia EC2 (rolling update, drain) mas você escolhe tipo/AMI
- **Fargate Profile no EKS**: define quais pods (namespace/label) vão para Fargate
- Dynamic port mapping (ECS EC2 bridge mode): ALB target type = "instance"; com awsvpc = target type "ip"
- Para segredos em tasks: referenciar Secrets Manager/SSM na Task Definition (Execution Role com permissão)
- **ECR + VPC Endpoint**: instâncias em subnet privada fazem pull de imagens ECR sem NAT/internet
