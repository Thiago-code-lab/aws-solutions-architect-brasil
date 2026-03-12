# Módulo 11 — Serverless: Lambda e API Gateway

## Computação Serverless na AWS

No modelo serverless, você não provisiona nem gerencia servidores — define apenas o código e a AWS cuida do resto (provisionamento, escalabilidade, patches, HA). Você paga apenas pelo que executar.

---

## AWS Lambda

### Modelo de Execução

```
Event Source → Lambda Service → Execution Environment
                                  └── Runtime (Python/Node/Java/Go/Ruby/.NET)
                                        └── seu-handler(event, context)
```

- **Handler**: ponto de entrada (`modulo.funcao`, ex: `app.lambda_handler`)
- **Event**: payload JSON com dados do trigger (varia por event source)
- **Context**: objeto com metadata (request_id, remaining_time, function_name, memory_limit)

### Limites Importantes

| Parâmetro | Limite | Notas |
|---|---|---|
| Timeout | 3s padrão, **15 minutos** máximo | Configure conforme necessidade |
| Memória | 128 MB a **10.240 MB** (10 GB) | CPU escala proporcionalmente |
| Armazenamento /tmp | 512 MB padrão, até **10.240 MB** | Efêmero por execution environment |
| Payload de entrada (sync) | 6 MB | - |
| Payload de entrada (async) | 256 KB | - |
| Package size (zip) | 50 MB (direto), 250 MB (S3) | 250 MB descomprimido |
| Layers | Até 5 por função | Total 250 MB descomprimido |
| Concorrência padrão | 1.000 por região | Aumentar via support ticket |

### Modelos de Invocação

| Modelo | Quem invoca | Comportamento em Erro | Exemplos |
|---|---|---|---|
| **Síncrono** | Caller aguarda resposta | Caller lida com o erro | API GW, ALB, Lambda invoke direto |
| **Assíncrono** | Lambda retorna 202 e processa depois | Retry automático 2x + DLQ | S3, SNS, EventBridge, CloudWatch |
| **Event Source Mapping** | Lambda policia a fonte | Retry até expirar + DLQ/bisect | SQS, Kinesis, DynamoDB Streams, Kafka |

### Lambda Concurrency (Concorrência)

```
Account Concurrency = 1.000 (default, aumentável)

                ┌─ Reserved Concurrency (garantida para função X)
                │   [bloqueia concorrência de outros]
Function Pool  ─┤
                └─ Unreserved Concurrency (compartilhado entre demais)

Provisioned Concurrency:
  Pré-aquece N execução environments → elimina cold start
  Cobrado por hora (mesmo sem invocações)
  Requerido para: aplicações sensíveis à latência (APIs interativas)
```

**Burst Limit (por região):**
- Escalonamento inicial: 3.000 concurrent executions (us-east-1, us-west-2, eu-west-1)
- Regiões menores: 500-1.000 inicial
- Expansão: +500/minuto após o burst

### Cold Start

Ocorre quando Lambda precisa criar um novo execution environment:
1. Download do pacote (código + layers)
2. Inicializar o runtime
3. Executar código de inicialização (fora do handler)

**Mitigações:**
- **Provisioned Concurrency**: ambientes pré-aquecidos (custo adicional)
- **Lambda SnapStart** (Java): snapshot do ambiente após inicialização; restaura em lugar de reinicializar (**Java 11+ apenas**)
- Manter pacote pequeno; evitar imports pesados no handler

### Lambda Layers

- Pacotes compartilhados entre múltiplas funções (bibliotecas, dependências, ferramentas)
- Separação entre código de aplicação e dependências
- Versioned; pode ser compartilhado entre contas AWS
- Limite: 5 layers por função, 250 MB total (descomprimido)

### Lambda em VPC

```
Lambda (dentro VPC):
  ├── Acessa recursos privados (RDS, ElastiCache, EC2)
  ├── Usa sua própria ENI (Hyperplane ENI — compartilhada entre funções na mesma VPC/subnet/SG)
  └── Para acessar internet: precisa de NAT Gateway (subnet pública + IGW)
      [Lambda em VPC NÃO tem acesso à internet por padrão]
```

**Impacto no cold start:** Lambda em VPC usa Hyperplane ENI (não ENI dedicada desde 2019) — impacto mínimo.

### Lambda Event Source Mapping (ESM)
Para SQS, Kinesis, DynamoDB Streams, MSK:
- Lambda **policia** a fonte (não a fonte empurra)
- Processa em **batches** (batch size configurável)
- **Filtering**: processa apenas eventos com certos atributos (ex: apenas `eventType = ORDER`)
- **Bisect on Error**: divide o batch em 2 ao falhar para isolar a mensagem problemática
- **DLQ ou Destination**: mensagens que falham todas as tentativas vão para SQS/SNS/S3

---

## Amazon API Gateway

### Tipos de API

| Tipo | Uso | Custo | Features |
|---|---|---|---|
| **REST API** | APIs REST completas | Maior | Cache, usage plans, request validation, X-Ray, WAF |
| **HTTP API** | APIs simples + alto throughput | ~71% menor | Lambda proxy, JWT, OIDC/OAuth2, menos features |
| **WebSocket API** | Conexões bidirecionais persistentes | Por conexão + mensagem | Chat, alertas real-time, $connect/$disconnect/$default |

### API Gateway REST — Componentes

```
API → Resource (/users) → Method (GET, POST, PUT, DELETE)
                              ├── Method Request (validação de entrada, autorização)
                              ├── Integration Request (mapeamento → backend)
                              ├── Backend (Lambda, HTTP, AWS Service, Mock)
                              ├── Integration Response (mapeamento de volta)
                              └── Method Response (status codes, headers)
```

### Tipos de Integração

| Tipo | Como funciona |
|---|---|
| **Lambda Proxy** | API GW passa o evento completo para Lambda; Lambda retorna response no formato esperado |
| **Lambda Custom** | Você define mapping templates (Velocity) para transformar request/response |
| **HTTP Proxy** | Proxia para endpoint HTTP externo |
| **HTTP Custom** | Proxia para HTTP com transformação |
| **AWS Service** | Integra diretamente com serviço AWS (ex: SQS SendMessage, DynamoDB PutItem) |
| **Mock** | Retorna resposta sem backend real (para testes) |

### Authorizers

| Tipo | Como funciona | Cache |
|---|---|---|
| **Lambda Authorizer** | Lambda recebe token/header, retorna IAM policy (Allow/Deny) | TTL configurável (0-3600s) |
| **Cognito User Pool** | Valida JWT token do Cognito User Pool | Automático via token |
| **IAM Authorization** | Assina requests com AWS SigV4 (para clientes AWS internos) | N/A |

### Throttling

- **Account limit**: 10.000 RPS + 5.000 burst (REST API)
- **Usage Plans**: rate (RPS) e quota (requests/dia ou semana) por cliente via API Key
- HTTP 429 quando throttled
- Throttling pode ser configurado por stage e por método

### API Gateway Cache
- Por stage; TTL padrão 300s (5 min), até 3.600s
- Cache key: query strings e headers configurados
- Invalidar por request: `Cache-Control: max-age=0` + permissão necessária
- Tamanhos: 0,5 GB a 237 GB

### Stages e Deploy
- API não fica disponível até fazer um **deployment para um stage**
- Stage variables: como variáveis de ambiente para stages (ex: endpoint do backend muda por stage)
- **Canary Deployments**: enviar % do tráfego do stage para nova versão

---

## AWS SAM (Serverless Application Model)

Framework para IaC de aplicações serverless:
```yaml
Transform: AWS::Serverless-2016-10-31

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.handler
      Runtime: python3.12
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /hello
            Method: GET
```
`sam build && sam deploy --guided` → cria/atualiza CloudFormation stack

---

## Arquitetura Serverless Típica

```
Client
  └── API Gateway (REST/HTTP API)
        ├── Authorizer: Lambda / Cognito
        └── Integration: Lambda Proxy
              ├── Lambda A (GET /users)    → DynamoDB
              ├── Lambda B (POST /orders)  → DynamoDB + SNS (async)
              └── Lambda C (GET /reports)  → S3 Select / Athena

Async:
  SNS → SQS → Lambda (processador) → SES (email)
  EventBridge → Lambda (scheduler) → Relatórios em S3

Monitoramento:
  CloudWatch Logs (automático)
  X-Ray (tracing distribuído)
```

---

## Dicas de Prova

- Lambda **não** tem estado entre invocações — use DynamoDB/ElastiCache para estado
- `/tmp` persiste apenas na vida do execution environment (não entre invocações diferentes)
- Lambda **grátis** até 1 milhão de invocações e 400.000 GB-segundos/mês
- **Reserved Concurrency = 0** → função efetivamente desabilitada
- API Gateway HTTP API NÃO suporta: response caching, request validation, WAF nativo, UsagePlan/ApiKey
- Lambda SnapStart: Java apenas; snapshot após init phase; restaurado com `CRaC` (Coordinated Restore at Checkpoint)
- SQS com Lambda: VisibilityTimeout da fila deve ser ≥ 6× timeout da função Lambda
- Lambda layers vs container image: container image (até 10 GB) permite empacotar tudo; não usa layers
- Event Source Mapping com Kinesis/DynamoDB Streams: Lambda processa em shards paralelamente (1 concurrent invoc per shard por default)
- API Gateway não pode ser acessado diretamente de dentro de VPC sem VPC Link ou Resource Policy
