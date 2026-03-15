# Cheatsheet — Módulo 13: Machine Learning e IA

## As 3 Camadas do Framework AWS AI/ML

```
┌─────────────────────────────────────────────────────────────────────┐
│       Nível 1: AI Services (sem ML knowledge necessário)            │
│  Rekognition | Comprehend | Polly | Transcribe | Translate | Lex    │
│  Textract | Kendra | Bedrock | Fraud Detector | Personalize          │
│  Forecast | Lookout for Metrics | Lookout for Vision                 │
├─────────────────────────────────────────────────────────────────────┤
│       Nível 2: ML Services (build/train/deploy seus modelos)        │
│  SageMaker Studio | Ground Truth | Experiments | Model Monitor      │
│  Training Jobs | Hyperparameter Tuning | Feature Store | Pipelines  │
├─────────────────────────────────────────────────────────────────────┤
│       Nível 3: Infrastructure (hardware especializado)              │
│  EC2 GPU (p3, p4, g4) | AWS Trainium | AWS Inferentia               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Comparativo — Serviços de Linguagem Natural (NLP)

| Serviço | Direção | Input | Output |
|---|---|---|---|
| **Amazon Transcribe** | Audio → Texto (STT) | Áudio (MP3, WAV, FLAC) | Texto + timestamps |
| **Amazon Polly** | Texto → Audio (TTS) | Texto / SSML | Áudio (MP3, PCM) |
| **Amazon Translate** | Texto → Texto | Texto em idioma X | Texto em idioma Y |
| **Amazon Comprehend** | Texto → Análise | Texto | Entities, Sentiment, PII, Topics |
| **Amazon Lex v2** | Conversa → Intent | Texto ou Áudio | Intent + Slots preenchidos |
| **Amazon Kendra** | Pergunta → Resposta | Pergunta em linguagem natural | Resposta de documentos |

---

## Comparativo — Visão Computacional e Documentos

| Serviço | Input | Output |
|---|---|---|
| **Rekognition Image** | Imagem | Objects, faces, text, moderation labels |
| **Rekognition Video** | Vídeo (S3 ou stream) | Person tracking, activity, labels ao longo do tempo |
| **Textract** | Imagem/PDF de documento | Texto + Tables + Forms + Keys-Values |
| **Lookout for Vision** | Imagens de produto (factory) | Anomalia detectada: NORMAL/ANOMALY |

---

## SageMaker — Pipeline ML

| Fase | Serviço/Feature |
|---|---|
| **Data Labeling** | Ground Truth |
| **Data Preparation** | Data Wrangler, Processing Jobs |
| **Feature Engineering** | Feature Store |
| **Model Training** | Training Jobs (EC2) |
| **Hyperparameter Tuning** | Automatic Model Tuning (HPO) |
| **Model Evaluation** | Experiments |
| **Model Registry** | Model Registry (versionamento) |
| **Deployment Online** | Endpoint (real-time inference) |
| **Deployment Batch** | Batch Transform |
| **Model Monitoring** | Model Monitor (drift detection) |
| **No-Code** | Canvas (business users) |

---

## Amazon Bedrock — Foundation Models Disponíveis

| Provedor | Modelos |
|---|---|
| **Amazon** | Titan Text, Titan Embeddings, Titan Image Generator |
| **Anthropic** | Claude 3 (Haiku, Sonnet, Opus) |
| **Meta** | Llama 3 |
| **Cohere** | Command R, Embed |
| **Mistral AI** | Mistral, Mixtral |
| **Stability AI** | Stable Diffusion (imagens) |

**RAG:** Bedrock Knowledge Bases → OpenSearch Serverless (vector store) → LLM contextualizado.

---

## Comparativo — Serviços de Recomendação e Previsão

| Serviço | Caso de Uso | Input | Output |
|---|---|---|---|
| **Personalize** | Recomendações personalizadas | Interações usuário-item + metadados | Top-N items por userId |
| **Forecast** | Previsão de séries temporais | Dados históricos + metadados | Valores futuros + intervalos |
| **Fraud Detector** | Detecção de fraude | Eventos de transação | Score de fraude |
| **Lookout for Metrics** | Anomalias em métricas de negócio | Métricas de S3/RDS/Redshift | Alertas de anomalia |

---

## Comprehend — Análises Disponíveis

| Análise | Descrição | Exemplo |
|---|---|---|
| **Entity Recognition** | Pessoas, locais, orgs, datas | "João trabalha na Amazon em São Paulo" |
| **Sentiment Analysis** | Positivo/Negativo/Neutro/Misto | Review de produto |
| **Key Phrases** | Expressões principais do texto | "aws certification", "cloud computing" |
| **Language Detection** | Identifica o idioma | pt-BR, en-US, es |
| **PII Detection** | CPF, email, telefone, etc. | Redação de dados pessoais |
| **Topic Modeling** | Agrupa documentos por tema | Análise de chamados de suporte |
| **Targeted Sentiment** | Sentimento por entidade mencionada | "camera" → positivo, "battery" → negativo |

**Comprehend Medical:** especializado em texto clínico (diagnósticos, medicamentos, procedures).

---

## Rekognition — Features por Tipo

| Feature | Imagem | Vídeo |
|---|---|---|
| Detecção de objetos/cenas | ✅ | ✅ (com timestamps) |
| Reconhecimento facial | ✅ (compare, search) | ✅ (person tracking) |
| Detecção de texto (OCR) | ✅ | ✅ |
| Content moderation | ✅ | ✅ |
| PPE detection (EPI) | ✅ | ✅ |
| Detecção de celebridades | ✅ | ✅ |
| Análise de atividades | ❌ | ✅ |

---

## SageMaker Deployment Options

| Opção | Latência | Uso |
|---|---|---|
| **Real-time Endpoint** | Milissegundos | Inference online, um request por vez |
| **Serverless Inference** | Segundos (cold start) | Tráfego intermitente, sem custo idle |
| **Async Inference** | Minutos | Payload grande, filas |
| **Batch Transform** | Horas | Inference em lote (S3 input → S3 output) |
| **Multi-Model Endpoint** | Low | Hospedar múltiplos modelos em 1 endpoint |

---

## Dicas de Prova — Qual Serviço de IA Usar

| Cenário | Serviço AWS |
|---|---|
| "Transcrever ligações de call center" | Amazon Transcribe (+ Call Analytics) |
| "Gerar voz sintética para IVR" | Amazon Polly (NTTS) |
| "Detectar conteúdo adulto em fotos de usuários" | Amazon Rekognition Content Moderation |
| "Extrair campos de formulários escaneados" | Amazon Textract |
| "Análise de sentimento de reviews" | Amazon Comprehend |
| "Chatbot com linguagem natural" | Amazon Lex v2 |
| "Busca inteligente em documentos Word/PDF" | Amazon Kendra |
| "Recomendações de produtos personalizadas" | Amazon Personalize |
| "Previsão de demanda de estoque" | Amazon Forecast |
| "Detectar fraude em transações" | Amazon Fraud Detector |
| "Detectar anomalias em métricas de receita" | Amazon Lookout for Metrics |
| "Usar modelos LLM via API sem treinar" | Amazon Bedrock |
| "Treinar modelo ML customizado" | Amazon SageMaker |
| "Anotar dados de imagem para treino" | SageMaker Ground Truth |
| "NLP em texto clínico médico" | Amazon Comprehend Medical |
| "Scanner de qualidade de produto na fábrica" | Amazon Lookout for Vision |
