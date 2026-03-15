# Casos de Uso Reais — Machine Learning e IA (Módulo 13)

## Caso 1 — Sistema de Recomendação de Produtos

**Contexto:** E-commerce com 10 milhões de usuários precisa de recomendações personalizadas em tempo real na página de produto ("quem comprou X também comprou Y") e em campanhas de e-mail personalizadas por segmento.

**Requisitos:**
- Recomendações em tempo real (< 200ms por request)
- Modelo retreinado diariamente com novos dados de comportamento
- Não exige time de ML dedicado — usar serviço gerenciado
- Segmentação para campanhas de marketing batch

**Arquitetura:**
```
DADOS DE ENTRADA:
Clickstream → Kinesis → S3 (interações usuário-produto)
Pedidos     → S3 (histórico de compras)

TREINAMENTO (diário — batch):
S3 (interações) → Amazon Personalize
                  ├── Dataset Group
                  ├── Solution (algoritmo USER_PERSONALIZATION)
                  └── Campaign (endpoint de inferência)

INFERÊNCIA (tempo real):
App → Amazon Personalize Campaign → Recomendações JSON
App → ElastiCache (cache resultado por user_id, TTL 1h)

BATCH (campanhas email):
EventBridge (diário) → Lambda → Personalize Batch Inference
                                    └── S3 (recomendações por segmento)
                                              └── SES (e-mails personalizados)
```

**Personalize vs SageMaker para este caso:**
| Critério | Personalize | SageMaker |
|---------|-------------|-----------|
| Time ML necessário | Não | Sim |
| Algoritmos disponíveis | Recomendação/ranking | Qualquer |
| Tempo setup | Horas | Dias/semanas |
| Flexibilidade | Baixa | Alta |
| Custo | Por TPS + horas treinamento | Por instância |

---

## Caso 2 — Análise de Documentos Financeiros com Textract + Comprehend

**Contexto:** Seguradora precisa processar 5.000 documentos/dia (formulários de sinistro, laudos médicos, faturas) para acelerar análise de apólices. Processo manual leva 3 dias por sinistro.

**Requisitos:**
- Extrair dados estruturados de formulários (campos, tabelas)
- Detectar informações de PII (CPF, nome, data de nascimento) para proteção
- Classificar tipo de documento (sinistro/laudo/fatura)
- Verificar consistência dos dados extraídos

**Arquitetura:**
```
Documentos (PDF/imagem) → S3 (sinistros-entrada)
                                │ S3 Event Notification
                                ▼
                           Lambda (orquestrador)
                           ├── Textract StartDocumentAnalysis (async)
                           │     (extrai formulários + tabelas)
                           │     └── SNS callback → Lambda (resultado)
                           ├── Comprehend DetectPiiEntities
                           │     (detecta CPF, nome, data)
                           ├── Comprehend ClassifyDocument
                           │     (tipo: sinistro/laudo/fatura)
                           └── Comprehend DetectSentiment
                                 (urgência do cliente no texto livre)
                                │
                           DynamoDB (metadados + campos extraídos)
                           S3 (resultado estruturado JSON)
                           Step Functions (fluxo aprovação humana se confiança < 80%)
                           
A2I (Augmented AI) → Revisão humana quando Textract tem baixa confiança
```

**Redução de Tempo:**
- Antes: 3 dias (manual) → Depois: 2 horas (automatizado)
- Taxa de automação: 85% dos documentos sem revisão humana
- A2I cobre os 15% com baixa confiança

---

## Caso 3 — Chatbot de Atendimento com Lex v2 + Lambda

**Contexto:** Empresa de telecomunicações quer reduzir volume de chamadas (200.000/mês) para SAC oferecendo chatbot que resolve os 5 problemas mais comuns: verificar fatura, 2ª via boleto, cancelar serviços, verificar saldo de dados, alterar plano.

**Requisitos:**
- Compreender linguagem natural em português
- Autenticar usuário antes de operações sensíveis
- Escalar chamadas complexas para atendente humano
- Integrar com sistema de CRM existente (REST API)

**Arquitetura:**
```
Usuário (WhatsApp / App / Web)
       │
       ▼
Amazon Lex v2 (Bot em português)
  Intents:
  ├── ConsultarFatura → Lambda (consulta CRM API)
  ├── EmitirBoleto    → Lambda (gera boleto PDF → S3 → URL)
  ├── CancelarServico → Lambda (verifica contrato + chama CRM)
  ├── VerificarDados  → Lambda (consulta sistema de telemetria)
  └── AlterarPlano    → Lambda (opções plano + Step Functions workflow)
  
  Autenticação:
  ├── Slot: CPF + data nascimento
  └── Lambda validator → consulta DynamoDB (sessão autenticada)
  
  Fallback:
  └── Intent: TransferirAtendente → Amazon Connect (transferência)
  
Amazon Connect (call center integrado)
CloudWatch (métricas: intent recognition rate, transfer rate)
```

**Métricas de Sucesso:**
| KPI | Antes | Depois |
|-----|-------|--------|
| Resolução sem atendente | 0% | 65% |
| Tempo médio resolução | 8 min | 90 seg |
| Custo por interação | R$15 | R$0.12 |
| Satisfação (CSAT) | 3.2/5 | 4.1/5 |

---

## Caso 4 — Moderação de Conteúdo com Rekognition

**Contexto:** Plataforma de UGC (User Generated Content) com 500.000 uploads de imagem/vídeo por dia precisa remover automaticamente conteúdo impróprio (violência, nudez, drogas) antes de publicar.

**Requisitos:**
- Analisar 100% dos conteúdos antes de publicar
- Falsos positivos < 5% (conteúdo válido bloqueado)
- Revisão humana para casos border-line (confiança 50-80%)
- Audit trail de decisões (compliance legal)

**Arquitetura:**
```
Upload usuário → S3 (content-pendente)
                      │ Lambda trigger
                      ▼
                 Lambda (moderador)
                 ├── Rekognition DetectModerationLabels
                 │     (nudez, violência, drogas, etc.)
                 │
                 ├── Se score > 80% (conteúdo impróprio):
                 │     └── Mover para S3 (content-bloqueado)
                 │         DynamoDB (log: bloqueado + motivo)
                 │         SNS → Email usuário (notificação)
                 │
                 ├── Se score 20-80% (incerto):
                 │     └── A2I (Augmented AI) → Fila revisão humana
                 │         Step Functions aguarda decisão humana
                 │
                 └── Se score < 20% (aprovado):
                       └── S3 (content-aprovado) → CloudFront

Audit: DynamoDB Streams → Lambda → S3 (log compliance imutável)
```

**Rekognition para Vídeo:**
```
Vídeo upload → S3 → Lambda → Rekognition StartContentModeration (async)
                                    │ SNS callback quando completo
                                    ▼
                               Lambda (processar resultado)
                               ├── Timestamps de cenas impróprias
                               └── Decisão: aprovar/rejeitar/segmentar
```

---

## Caso 5 — Previsão de Demanda para Gestão de Estoque

**Contexto:** Rede de supermercados precisa prever demanda de 50.000 SKUs nas próximas 4 semanas para otimizar pedidos de reposição e reduzir desperdício.

**Requisitos:**
- Previsão por SKU + loja + semana
- Considerar sazonalidade, feriados, promoções
- Retreinar modelo mensalmente com dados novos
- Integrar previsões com sistema ERP de compras

**Arquitetura:**
```
DADOS HISTÓRICOS:
S3 (vendas diárias últimos 3 anos, por SKU/loja)
S3 (dados externos: feriados, promoções, clima)

AMAZON FORECAST:
├── Dataset Group (vendas + metadados + itens relacionados)
├── Predictor (algoritmo: AutoML seleciona melhor)
│     (opções: DeepAR+, Prophet, NPTS, CNN-QR)
└── Forecast (gera P10/P50/P90 para 28 dias)

INFERÊNCIA + INTEGRAÇÃO:
EventBridge (1x/semana) → Lambda
     │
     ├── CreateForecastExportJob → S3 (previsoes.csv)
     ├── Lambda processa CSV → DynamoDB (tabela compras_sugeridas)
     └── API REST ERP → lê DynamoDB → gera pedidos de compra

QuickSight (dashboard: previsão vs realizado, acurácia MAPE)
SNS → Alert se MAPE > 15% (modelo degradando)
```

**Métricas de Acurácia:**
| Métrica | Valor Típico | O que significa |
|---------|-------------|----------------|
| MAPE | < 15% | Mean Absolute Percentage Error |
| WAPE | < 10% | Weighted APE (considera volume) |
| P50 | previsão mediana | 50% confiança |
| P90 | limite otimista | Para safety stock |
