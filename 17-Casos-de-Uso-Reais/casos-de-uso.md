# Casos de Uso Reais — Arquiteturas Multi-Serviço (Módulo 17)

## Caso 1 — Plataforma SaaS Multi-Tenant com Isolamento por Tier

**Contexto:** Empresa de software B2B oferece plataforma de gestão para clientes de 3 tamanhos: Free (milhares de clientes), Business (500 clientes), Enterprise (50 grandes contratos). Cada tier exige nível diferente de isolamento e garantia de performance.

**Arquitetura Híbrida (Pool + Silo):**
```
CAMADA DE ROTEAMENTO:
Route 53 → ALB → API Gateway

Lógica de Roteamento por Tenant:
├── ENTERPRISE (silo): stack dedicada
│     ALB → ECS dedicado → Aurora dedicado (sem compartilhamento)
│     DDoS: Shield Advanced individual
│     SLA: 99.99%
│
├── BUSINESS (bridge): infra compartilhada, DB separado
│     ALB → ECS compartilhado → Aurora Schema por tenant
│     Compute: shared, Storage: isolado
│     SLA: 99.9%
│
└── FREE (pool): tudo compartilhado
      ALB → ECS compartilhado → DynamoDB (tenantId = partition key)
      SLA: 99%

IDENTIFICAÇÃO DE TENANT:
JWT token → API GW Authorizer (Lambda)
Lambda extrai tenantId → injeta em header X-Tenant-ID
ECS app usa X-Tenant-ID para rotear fila correta / DB correto

NOISY NEIGHBOR PROTECTION (camada Free):
SQS por tenant-tier: Business tem fila separada de Free
Lambda reserved concurrency: Business = 100, Free = 20
DynamoDB: on-demand (escala por tenant sem afetar outros)
```

---

## Caso 2 — Migração Strangler Fig de Monolito Java para Microsserviços

**Contexto:** Sistema ERP legado (Java monolítico, 15 anos) precisa ser modernizado gradualmente. Não é possível reescrever tudo de uma vez — risco alto e custo proibitivo. Estratégia: extrair módulos um a um sem downtime.

**Processo Strangler Fig:**
```
ESTADO INICIAL:
Tudo no monolito Java:
CLI → Servidor Java → PostgreSQL Oracle

FASE 1 — INSERIR PROXY (Strangler):
CLI → API GW / ALB ← NOVO PONTO DE ENTRADA
        ├── /api/estoque  → Monolito Java (legado)
        ├── /api/clientes → Monolito Java (legado)
        ├── /api/pedidos  → Monolito Java (legado)
        └── /api/relatorio→ Monolito Java (legado)

FASE 2 — EXTRAIR MÓDULO DE ESTOQUE:
← Desenvolver microsserviço estoque em paralelo
← Testes de integração com dados reais
ALB Route: /api/estoque → NOVO Lambda + DynamoDB
           (outros /api/* ainda vão para monolito)

FASE 3 — EXTRAIR MÓDULO DE PEDIDOS:
/api/pedidos → NOVO ECS Service + Aurora PostgreSQL
/api/estoque → Lambda + DynamoDB (já migrado)
/api/clientes, /api/relatorio → Monolito ainda

FASE FINAL — "MATAR" O MONOLITO:
Quando último módulo extraído:
ALB não tem mais rotas para monolito
Monolito Java descomissionado
100% microsserviços na AWS

ANTI-PADRÃO EVITADO:
❌ "Big Bang" — reescrever tudo antes de ir para produção
✓ Incremental — cada extração validada em produção antes da próxima
```

---

## Caso 3 — Pipeline de Processamento de Pedidos com Saga Pattern

**Contexto:** Marketplace online com 3 microsserviços independentes: Pedidos, Estoque e Pagamento. Quando um pedido é criado, os 3 devem ser consistentes — se o pagamento falha, o estoque deve ser liberado.

**Saga Coreografada (Event-Driven):**
```
HAPPY PATH:
Pedido criado → SQS (PedidoCriado)
                     │
              Lambda InventoryService
              ├── Reserva estoque
              └── SQS (EstoqueReservado)
                            │
                     Lambda PaymentService  
                     ├── Cobra pagamento
                     └── SQS (PagamentoAprovado)
                                    │
                             Lambda OrderService
                             └── Confirma pedido (status = CONFIRMADO)
                             └── SNS (PedidoConfirmado) → notifica usuário

COMPENSAÇÃO (Rollback):
PaymentService falha → SQS (PagamentoRejeitado)
                              │
                       Lambda InventoryService
                       └── Libera estoque (compensação)
                       └── SQS (EstoqueLiberado)
                                   │
                            Lambda OrderService
                            └── Cancela pedido (status = CANCELADO)
                            └── SNS → notifica usuário
```

**Saga Orquestrada (Step Functions):**
```
Step Functions Express Workflow:
State 1: Task (ReservarEstoque) → retry 3x, catch → CancelarPedido
State 2: Task (ProcessarPagamento) → retry 2x, catch → LiberarEstoque → CancelarPedido
State 3: Task (ConfirmarPedido)
State 4: Task (NotificarUsuario)

Compensação (Catch):
LiberarEstoque → CancelarPedido → NotificarFalha

Vantagem vs Coreografia:
- Visibilidade completa do estado em cada step
- Retry configurável por step
- Fácil debug (X-Ray integrado)
```

---

## Caso 4 — Event Sourcing para Sistema de Auditoria Financeira

**Contexto:** Banco de investimentos precisa de histórico completo e imutável de todas as mudanças em contas e portfólios. Regulação exige rastreabilidade de 10 anos + capacidade de "replay" para auditoria.

**Arquitetura Event Sourcing:**
```
WRITE MODEL:
App → API GW → Lambda (command handler)
                    │ NÃO atualiza estado diretamente
                    │ GRAVA EVENTO imutável
                    ▼
              DynamoDB (event store — tabela de eventos)
              ├── PK: aggregateId (ex: conta-123)
              ├── SK: version (1, 2, 3...)
              ├── eventType: "SaldoDebitado", "SaldoCreditado"
              ├── payload: {valor: 5000, moeda: "BRL"}
              └── timestamp: ISO 8601
              
DynamoDB Streams → Lambda (projections updater)
                        └── ElastiCache (saldo atual — leitura rápida)
                        └── Athena/Redshift (relatórios analíticos)

READ MODEL (CQRS):
App → API GW → Lambda (query handler)
                    └── ElastiCache (leitura rápida do estado atual)

REPLAY PARA AUDITORIA:
Query: SELECT * FROM event_store WHERE accountId = 'conta-123' ORDER BY version
      → Replay todos eventos → recalcular estado em qualquer ponto no tempo
      
S3 (event store backup, compactado por mês, Object Lock 10 anos)
```

---

## Caso 5 — Plataforma de Transmissão Ao Vivo com IVS + Lambda

**Contexto:** Plataforma de ensino online quer oferecer aulas ao vivo com baixíssima latência (< 5 segundos), chat em tempo real, enquetes durante a transmissão, e gravação automática para replay posterior.

**Arquitetura:**
```
TRANSMISSÃO:
Professor → OBS/Encoder (RTMP) → Amazon IVS (Interactive Video Service)
                                        │
                                  CloudFront CDN (entrega global)
                                        │
                                  Players dos alunos (< 5s latência)

CHAT EM TEMPO REAL:
Aluno → API GW WebSocket
             ├── Lambda (onConnect) → DynamoDB (connectionIds)
             ├── Lambda (sendMessage) → DynamoDB + broadcast
             └── Lambda (onDisconnect) → remove connectionId
             
Broadcast para todos na sala:
Lambda → API GW Management API → POST /connections/{connectionId}

ENQUETES:
Professor → API REST → Lambda → DynamoDB (enquete + respostas)
Evento IVS Timed Metadata → Player exibe enquete sincronizado com vídeo

GRAVAÇÃO E REPLAY:
IVS → S3 automático (gravação completa)
     └── MediaConvert (transcodificação para múltiplas resoluções)
            └── S3 (HLS adaptativo)
                └── CloudFront (CDN para replay)

MÉTRICAS:
IVS → CloudWatch (viewers, bitrate, buffering ratio)
Custom: Lambda → CloudWatch Metrics (mensagens/min, respostas enquete)
```

**IVS vs MediaLive:**
| Critério | IVS | MediaLive |
|---------|-----|-----------|
| Latência | < 5s (ultra-low latency) | 15-30s (padrão) |
| Configuração | Baixa (gerenciado) | Alta (muitos parâmetros) |
| Interatividade | Nativa (timed metadata, chat) | Limitada |
| Custo | Por hora + GB | Por hora (mais caro para escala) |
| Quando usar | Lives interativas simples | Broadcasting broadcast TV |
