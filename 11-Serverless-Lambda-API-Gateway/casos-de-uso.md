# Casos de Uso Reais — Serverless, Lambda e API Gateway (Módulo 11)

## Caso 1 — API REST Serverless para E-commerce

**Contexto:** Startup de e-commerce precisa de uma API para catálogo de produtos e criação de pedidos. Espera picos de tráfego em datas comemorativas (Black Friday, Natal). Orçamento limitado — não quer pagar por capacidade ociosa.

**Requisitos:**
- Escalonamento automático sem intervenção manual
- Custo proporcional ao uso (sem tráfego = sem custo)
- Latência < 500ms para leitura de produtos
- Autenticação JWT para rotas de pedidos

**Arquitetura:**
```
Cliente Mobile/Web
       │
       ▼
  CloudFront (cache GET /produtos)
       │
       ▼
 API Gateway (REST)
  ├── GET /produtos     → Lambda (sem auth) → DynamoDB (catálogo)
  ├── GET /produtos/{id}→ Lambda (sem auth) → DynamoDB
  ├── POST /pedidos     → Lambda (JWT auth) → DynamoDB + SNS
  └── GET /pedidos/{id} → Lambda (JWT auth) → DynamoDB
           │
     Cognito Authorizer (valida JWT)
           │
     DynamoDB (pedidos, TTL em itens de sessão)
           │
     SNS → SQS → Lambda (processar pagamento async)
```

**Decisões de Design:**
| Decisão | Alternativa Descartada | Motivo |
|---------|----------------------|--------|
| Lambda + DynamoDB | EC2 + RDS | Sem capacidade ociosa; DynamoDB escala horizontalmente |
| Cognito Authorizer | Lambda Authorizer | Menor operação; Cognito gerencia rotação de JWTs |
| CloudFront na frente | Só API GW | Cache de GETs reduz invocações Lambda |
| SNS → SQS async | Lambda sync para pagamento | Desacopla; pagamento pode demorar sem timeout |

**Estimativa de Custo (1M req/mês):**
- API Gateway: ~$3.50
- Lambda: ~$0.20 (Free Tier cobre)
- DynamoDB On-Demand: ~$1.25
- **Total: < $5/mês** vs EC2 t3.small ($13/mês ocioso)

---

## Caso 2 — Pipeline de Processamento de Imagens

**Contexto:** Plataforma de mídia social precisa redimensionar e aplicar filtros em imagens enviadas pelos usuários. Volume de 50.000 uploads/dia com picos de 500 uploads/minuto.

**Requisitos:**
- Processar imagens assincronamente (usuário não espera)
- Garantir que nenhuma imagem seja perdida (at-least-once)
- Múltiplas saídas: thumbnail 100px, médio 600px, original
- Armazenar resultados em CDN global

**Arquitetura:**
```
Usuário faz upload
       │
       ▼
  S3 (bucket: uploads-originais)
       │ S3 Event Notification
       ▼
  SQS (buffer — garante at-least-once)
       │
       ▼
  Lambda (processImage)
  ├── Lê imagem do S3
  ├── Resize com Pillow Layer
  ├── Grava thumbnail/médio/original em S3 (bucket: processadas)
  └── Atualiza DynamoDB (status = "processada")
       │
       ▼
  CloudFront → S3 (bucket: processadas)
  
  DLQ (SQS) → SNS → Email (alertas de falha)
  CloudWatch → Alarm → SNS (Lambda errors > 10)
```

**Pontos de Atenção:**
- Lambda Layer com biblioteca Pillow (máx 250 MB descomprimido)
- Concorrência reservada para evitar throttling do S3
- SQS visibility timeout > tempo máximo do Lambda (ex: Lambda 30s → timeout SQS 35s)
- DLQ configurado com maxReceiveCount=3

---

## Caso 3 — Webhook Fanout com SNS → Lambda

**Contexto:** Plataforma SaaS precisa notificar múltiplos sistemas quando um evento de negócio ocorre (ex: "cliente fechou contrato"): CRM, ERP, sistema de cobrança, e-mail marketing.

**Requisitos:**
- Evento publicado 1 vez → múltiplos sistemas recebem
- Falha em 1 sistema não afeta os outros
- Cada sistema tem seu próprio ritmo de processamento

**Arquitetura:**
```
App (publica evento)
       │
       ▼
  SNS Tópico: contrato-fechado
  ├── Assinante 1: SQS → Lambda (atualizar CRM)
  ├── Assinante 2: SQS → Lambda (atualizar ERP)
  ├── Assinante 3: SQS → Lambda (gerar cobrança)
  └── Assinante 4: SQS → Lambda (disparar e-mail marketing)

  Cada SQS tem sua própria DLQ
  Cada Lambda tem CloudWatch Log Group separado
```

**Por que SQS entre SNS e Lambda (e não SNS → Lambda diretamente)?**
- SQS garante persistência se Lambda falhar
- SQS permite controle de concorrência (Lambda não fica sobrecarregado)
- SQS permite DLQ por assinante (falha no CRM não contamina ERP)
- SNS → Lambda diretamente: perda de evento se Lambda falha (sem retry nativo)

---

## Caso 4 — API GraphQL com AppSync + Lambda

**Contexto:** App mobile de rede social precisa de subscriptions em tempo real (notificações no feed), queries otimizadas, e resolvers customizados para lógica de negócio.

**Requisitos:**
- Atualizações em tempo real (WebSocket nativo)
- Resolvers diretos para DynamoDB (sem Lambda para CRUD simples)
- Resolvers Lambda apenas para lógica complexa (recomendar amigos)
- Autenticação com Cognito User Pool

**Arquitetura:**
```
App Mobile
    │ GraphQL (HTTP/WebSocket)
    ▼
AWS AppSync
    ├── Query getUserProfile → DynamoDB Resolver (direto)
    ├── Query getFeed       → DynamoDB Resolver (direto)
    ├── Mutation createPost → DynamoDB Resolver + Pipeline Resolver
    ├── Query recommendations → Lambda Resolver (ML logic)
    └── Subscription onNewPost → WebSocket push automático
    
Cognito User Pool (autenticação AppSync)
DynamoDB (users, posts, follows)
Lambda (recommendation engine)
```

**AppSync vs API Gateway:**
| Critério | AppSync | API GW REST |
|---------|---------|------------|
| Subscriptions real-time | Nativo WebSocket | Requer API GW WebSocket separado |
| Query otimizada | GraphQL (1 request) | REST múltiplos endpoints |
| Resolvers diretos a DB | DynamoDB, Aurora, Dynamo | Sempre via Lambda |
| Complexidade | Maior setup inicial | Simples para REST puro |

---

## Caso 5 — Processamento em Lote com Lambda e Step Functions

**Contexto:** Empresa de análise financeira precisa processar 1 milhão de transações diárias até às 06h. Cada transação passa por 3 etapas sequenciais: validação, enriquecimento (consulta API externa), e cálculo de score.

**Requisitos:**
- Processar tudo em < 4 horas
- Retry automático em falha de etapa
- Relatório final de transações rejeitadas
- Visibilidade do progresso em tempo real

**Arquitetura:**
```
EventBridge Scheduler (02:00 UTC diario)
       │
       ▼
  Step Functions (Express Workflow)
  ┌─────────────────────────────────┐
  │ Map State (processa em paralelo) │
  │   ├── Lambda: Validar Transação  │
  │   │     (retry: 3x, backoff 2s)  │
  │   ├── Lambda: Enriquecer Dados   │
  │   │     (retry: 5x, backoff 10s) │
  │   └── Lambda: Calcular Score     │
  │         (retry: 2x, backoff 1s)  │
  └─────────────────────────────────┘
       │
       ▼
  DynamoDB (scores calculados)
  S3 (relatório JSON de rejeitados)
  SNS → Email (notificação conclusão)
  CloudWatch Logs (detalhes de cada execução)
```

**Por que Step Functions e não SQS em cadeia?**
- Step Functions: visibilidade de estado, retry configurável por etapa, Map State nativo para paralelismo
- SQS em cadeia: funciona, mas sem visibilidade do estado do pipeline, retry difícil de configurar por etapa
- Lambda timeout 15min: suficiente para 1 transação, Step Functions orquestra o lote

**Custo para 1M transações/dia:**
- Step Functions Express: $1.00 por 1M execuções de estado
- Lambda: ~$0.20 por 1M invocações
- DynamoDB: ~$1.25 write
- **Total: < $3/dia**
