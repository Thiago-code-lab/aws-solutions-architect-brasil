# Cheatsheet — Módulo 11: Serverless, Lambda e API Gateway

## Limites Críticos do Lambda

| Parâmetro | Valor |
|---|---|
| Timeout máximo | 15 minutos |
| Memória | 128 MB a 10.240 MB |
| Payload síncrono (request/response) | 6 MB cada |
| Payload assíncrono | 256 KB |
| Tamanho do package (zip) | 50 MB (zip) / 250 MB (descomprimido) |
| Concurrent executions por conta/região | 1.000 (default, aumentável) |
| Layers máximos por função | 5 |
| Variáveis de ambiente | 4 KB total |
| Tmp storage (/tmp) | 512 MB a 10.240 MB |

---

## Tipos de Invocação Lambda

| Tipo | Quem Usa | Retry | DLQ |
|---|---|---|---|
| **Síncrono** | API GW, ALB, SDK, CLI | Nenhum (cliente retry) | N/A |
| **Assíncrono** | S3, SNS, EventBridge, SES | 2 retries automáticos | SQS/SNS via Destinations |
| **Event Source Mapping** | SQS, Kinesis, DynamoDB Streams, MSK | Retry até expirar (Kinesis) ou maxReceiveCount (SQS) | DLQ configurada na fila |

---

## Concurrency Lambda

| Tipo | Descrição | Custo |
|---|---|---|
| **On-demand** | Escala automático conforme demanda | Padrão (pay-per-use) |
| **Reserved** | Garante capacity exclusiva para a função; limita ao máximo definido | Padrão |
| **Provisioned** | Pré-aquece instâncias (elimina cold start) | $0.015/GB-hora provisionada |

**Throttling:** se `concurrent executions > reserved/account limit` → **HTTP 429** (sync) ou evento vai para fila de retry (async).

---

## API Gateway — Tipos de API

| Tipo | Features | Custo | Melhor Para |
|---|---|---|---|
| **REST API** | Stages, caching, usage plans, API keys, full transform | Médio | APIs produção completas |
| **HTTP API** | JWT/OAuth nativo, sem mapping templates | ~70% mais barato | Microsserviços simples |
| **WebSocket API** | Conexões bidirecionais persistentes | Por conexão + mensagem | Chat, real-time updates |

---

## API Gateway Throttling

| Limite | Valor Padrão |
|---|---|
| Request rate por conta | 10.000 req/s |
| Burst | 5.000 req concorrentes |
| HTTP 429 | Too Many Requests (quando excede) |

---

## Integrações do API Gateway

| Tipo | Descrição |
|---|---|
| **Lambda Proxy** | Event completo para Lambda; Lambda retorna statusCode + headers + body |
| **Lambda Custom** | Mapping templates (VTL) transformam request/response |
| **HTTP Proxy** | Redireciona request HTTP sem transformação |
| **AWS Service** | Chama AWS API diretamente (ex: SQS SendMessage sem Lambda) |
| **Mock** | Retorna resposta estática (para testes, CORS OPTIONS) |
| **VPC Link** | Acessa NLB em VPC privada |

---

## Autorizadores do API Gateway

| Tipo | Como Funciona |
|---|---|
| **Lambda Authorizer Token** | Token no header → Lambda avalia → retorna IAM Policy (ALLOW/DENY) |
| **Lambda Authorizer Request** | Headers + query params → Lambda → IAM Policy; flexível |
| **Cognito User Pools** | Token JWT do Cognito validado nativamente pelo API GW |

Cache de resultado: até **3600 segundos** (configurable, default 300s).

---

## Lambda Execution Environment Lifecycle

```
INIT phase     → download code, start runtime, execute init code (outside handler)
INVOKE phase   → execute handler function
SHUTDOWN phase → cleanup (response to SIGTERM)
```

**Cold Start** ocorre em: primeira invocação, após período idle, novo deployment, aumento de concurrency além das instâncias warm.

---

## Lambda + VPC — Diagrama de Conectividade

```
┌─────────────────────────────────────────────────────────────┐
│ AWS CLOUD                                                   │
│  ┌─────────────────┐     ┌─────────────────────────────┐   │
│  │   Lambda        │     │   VPC                       │   │
│  │   (hyperplane)  │─ENI─│  ┌──────────┐ ┌──────────┐ │   │
│  │                 │     │  │ Private  │ │ RDS / EC2│ │   │
│  └─────────────────┘     │  │ Subnet   │ │          │ │   │
│                          │  └──────────┘ └──────────┘ │   │
│                          │         ↓ NAT GW (se precisa internet) │   │
│                          └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Sem NAT GW**: Lambda em VPC não tem acesso à internet (SSM, DynamoDB precisam de VPC Endpoint ou NAT GW).

---

## Lambda vs Outros Computes — Quando Usar

| Cenário | Serviço |
|---|---|
| Event-driven, < 15 min, stateless | Lambda |
| Containerizado, microservices, > 15 min | ECS Fargate |
| Precisa de GPU, HPC, software específico | EC2 |
| Job batch longo (horas/dias) | AWS Batch |
| Workflow orquestrado multi-step | Step Functions + Lambda |
| ML Training/Inference | SageMaker |

---

## Dicas de Prova — Padrões Comuns

| Pista no Enunciado | Resposta Provável |
|---|---|
| "Cold start" + Java | Lambda SnapStart |
| "Latência consistente" servlerless | Provisioned Concurrency |
| "Autenticação personalizada" API GW | Lambda Authorizer |
| "JWT / OAuth" API GW | HTTP API + JWT Authorizer |
| "Lambda não acessa RDS" | Verificar SG + VPC config + NAT GW |
| "Lambda precisa de 30 min" | Lambda NÃO serve → ECS Fargate ou AWS Batch |
| "Deploy gradual 10% → 100%" | Lambda Alias com pesos (canary) |
| "Múltiplos ambientes (dev/staging/prod)" | API GW Stages |
| "Montar dependências entre funções" | Lambda Layers |
| "Lambda não deve ser invocada mais de N vezes/s" | Reserved Concurrency |
