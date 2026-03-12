# Módulo 13 — Machine Learning e Inteligência Artificial

## Visão Geral dos Serviços AWS de ML/IA

A AWS oferece 3 camadas de serviços de ML:
```
Camada 1 — AI Services (prontos para uso, sem expertise ML)
  Rekognition, Comprehend, Polly, Transcribe, Translate, Lex, Textract,
  Forecast, Personalize, Kendra, CodeWhisperer, Bedrock

Camada 2 — ML Services (plataforma gerenciada)
  Amazon SageMaker (treinar, otimizar, deployar modelos customizados)

Camada 3 — ML Frameworks + Infrastructure
  EC2 (GPU instances p3/p4), EKS + Karpenter, Deep Learning AMIs
```

---

## Amazon SageMaker

Plataforma completa de ML gerenciada:

### Fluxo ML com SageMaker

```
Dados (S3) → SageMaker Studio (IDE web)
                ├── Data Wrangler: preparação visual de dados
                ├── Experiments: rastreamento de experimentos
                ├── Training Job: treina modelo em instâncias gerenciadas
                │     ├── Built-in algorithms (XGBoost, Linear Learner, etc.)
                │     ├── Custom container (seu código Docker)
                │     └── Spot instances (até 90% desconto, Managed Spot Training)
                ├── Hyperparameter Tuning: otimiza hiperparâmetros automaticamente
                ├── Model Registry: versiona e governa modelos
                └── Deployment:
                      ├── Real-time Endpoint (latência ms, escala automática)
                      ├── Serverless Inference (sem manter instância ativa)
                      ├── Batch Transform (inferência em lote sobre S3)
                      └── Async Inference (jobs longos, resultado em S3)
```

### SageMaker — Recursos Importantes

| Feature | Descrição |
|---|---|
| **SageMaker Studio** | IDE Jupyter-based para todo o ciclo ML |
| **Autopilot** | AutoML — encontra o melhor modelo automaticamente |
| **Canvas** | AutoML com interface no-code (sem código Python) |
| **Ground Truth** | Labeling de dados com humanos (workforce pública + privada) |
| **Clarify** | Detecta bias nos dados e explainability dos modelos |
| **Feature Store** | Banco de features compartilhado entre treinamento e inferência |
| **Pipelines** | CI/CD para ML (MLOps) |
| **JumpStart** | Modelos pré-treinados e soluções ML prontas |

---

## Amazon Rekognition

Visão computacional gerenciada (imagens e vídeos):
- **Object/Scene Detection**: identifica objetos, atividades, cenas
- **Facial Analysis**: detecção, análise de atributos, comparação, reconhecimento
- **Text Detection**: OCR em imagens (placas, documentos)
- **Content Moderation**: detecta conteúdo inapropriado (violência, nudez)
- **PPE Detection**: Equipamento de Proteção Individual (capacete, máscara)
- **Celebrity Recognition**: identifica famosos em imagens
- **Video Analysis**: análise de streaming de vídeo em tempo real via Kinesis Video Streams

---

## Amazon Comprehend

NLP (Natural Language Processing) gerenciado:
- **Entidade/Sentiment Analysis**: detecta entidades (pessoas, locais, organizações) e sentimento (positivo, negativo, neutro, misto)
- **Topic Modeling**: descobre tópicos em coleções de documentos
- **Key Phrase Extraction**: frases-chave do texto
- **Language Detection**: identifica idioma do texto
- **PII Detection/Redaction**: detecta e redige Informações Pessoalmente Identificáveis (CPF, email, telefone)
- **Custom Models**: treina classificadores e entity recognizers customizados

---

## Amazon Polly

Text-to-Speech (TTS):
- Converte texto em fala natural em múltiplos idiomas e vozes
- **Neural TTS (NTTS)**: vozes mais naturais e expressivas
- **SSML** (Speech Synthesis Markup Language): controle fino de pronúncia, pausas, ênfase
- **Speech Marks**: metadata com timestamps de palavras/frases (útil para karaokê ou legendas sincronizadas)
- **Lexicon**: especializa pronúncia de palavras específicas (siglas, termos técnicos)

---

## Amazon Transcribe

Speech-to-Text (STT):
- Média de acurácia com modelos customizados por domínio
- **Custom Vocabulary**: adiciona termos específicos (nomes de produtos, jargões)
- **Custom Language Model**: modelo de linguagem customizado para domínio específico
- **Speaker Diarization**: identifica quem está falando em cada trecho
- **Medical Transcribe**: vocabulário médico especializado (HIPAA eligible)
- **Call Analytics**: análise de chamadas de call center (sentimento, silêncios, interrupções)
- **Streaming**: transcrição em tempo real

---

## Amazon Translate

Tradução automática neural:
- Tradução de alta qualidade em 75+ idiomas
- **Custom Terminology**: glossário de termos que não devem ser traduzidos ou têm tradução específica
- **Active Custom Translation**: adapta o modelo com exemplos de tradução fornecidos
- **Parallel Data**: treina com pares de frases originais e traduzidas
- Integra com S3 para batch translation de documentos

---

## Amazon Lex

Chatbots conversacionais (mesmo motor do Alexa):
- **Intents**: ações que o usuário quer realizar (ex: `BookFlight`, `CheckBalance`)
- **Slots**: variáveis a coletar (ex: `data_viagem`, `destino`)
- **Utterances**: frases que disparam o intent
- **Fulfillment**: Lambda processa o intent e retorna resposta
- **Channels**: integra com Facebook, Slack, Twitch, Kik
- **Amazon Connect**: Lex pode ser o bot da central de atendimento

---

## Amazon Bedrock

Foundation Models (FM) via API — IA Generativa:
- Modelos de: Anthropic (Claude), Amazon (Titan), Stability AI, Cohere, AI21, Meta (Llama)
- **Serverless**: sem infraestrutura para gerenciar; paga por token
- **RAG com Knowledge Bases**: gerencia embeddings (OpenSearch Serverless) + retrieval automaticamente
- **Agents**: define ações e Lambda functions que o modelo pode executar autonomamente
- **Fine-tuning**: adapta modelos base com seus próprios dados (S3)
- **Model Evaluation**: compara outputs de diferentes modelos com métricas automáticas e humanas
- **Guardrails**: aplica filtros de conteúdo, PII redaction, topic denial às respostas dos modelos

---

## Outros Serviços AI

| Serviço | Função |
|---|---|
| **Amazon Textract** | OCR avançado: extrai texto, tabelas e formulários de documentos PDF/imagens |
| **Amazon Kendra** | Enterprise search com NLP (encontra respostas em documentos corporativos) |
| **Amazon Forecast** | Previsão de séries temporais (demanda, vendas, uso de recursos) |
| **Amazon Personalize** | Recomendações personalizadas (como Netflix/Amazon.com) — sem ML expertise |
| **Amazon Fraud Detector** | Detecção de fraude (compras, contas falsas) com ML |
| **Amazon Lookout for Metrics** | Detecta anomalias em métricas de negócio automaticamente |
| **Amazon CodeWhisperer** | Geração de código por IA no IDE (baseado em Bedrock) |

---

## Dicas de Prova

- **Rekognition**: visão (imagem/vídeo) → Transcribe: fala→texto → Polly: texto→fala
- **Comprehend**: análise de texto (sentimento, entidades) → diferente de Rekognition (imagens)
- **Kendra**: enterprise search (busca respondendo perguntas em documentos) → diferente de OpenSearch (analytics)
- **Forecast vs Personalize**: Forecast = previsão de séries temporais (demanda, inventário); Personalize = recomendações user-item
- SageMaker Ground Truth: labeling humano → **Rekognition Custom Labels** também treina com imagens labeleadas (mais simples)
- **Textract** reconhece **tabelas e formulários** (não só texto simples como OCR tradicional)
- Bedrock: **Foundation Models via API** sem treinar modelo zero — diferente de SageMaker (treinar modelo customizado)
- SageMaker Managed Spot Training: usa Spot instances para training jobs → até 90% redução de custo; checkpointing necessário para retomar após interrupção
- **Comprehend PII**: detecta/redige dados sensíveis em texto (ex: conformidade LGPD/GDPR)
- **Amazon Connect + Lex**: call center automatizado; sem infra de telefonia para gerenciar
