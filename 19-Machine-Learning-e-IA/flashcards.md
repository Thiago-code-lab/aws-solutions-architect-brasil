# Flashcards — Módulo 13: Machine Learning e IA

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** Quais são as 3 camadas do framework AWS AI/ML?  
**R:** **(1) AI Services (topo):** APIs prontas sem ML knowledge — Rekognition, Comprehend, Polly, Transcribe, Translate, Lex, Textract, Kendra, Bedrock. **(2) ML Services (meio):** SageMaker (build-train-deploy personalizado). **(3) Infrastructure (base):** EC2 GPU, Trainium, Inferentia para training/inference em escala.

---

**P:** O que é Amazon Bedrock e o que diferencia do SageMaker?  
**R:** **Bedrock:** acesso a Foundation Models (LLMs) de terceiros via API (Anthropic Claude, Amazon Titan, Meta Llama, Mistral, Cohere, Stability AI) sem treinar modelo. **SageMaker:** plataforma para treinar, tunar e deployar modelos ML próprios. Bedrock = usar modelos prontos; SageMaker = construir modelos customizados.

---

**P:** O que é RAG no contexto do Amazon Bedrock?  
**R:** Retrieval Augmented Generation: técnica que conecta LLMs a uma base de conhecimento externa. Fluxo: usuário pergunta → sistema busca contexto relevante na base → envia contexto + pergunta para o LLM → LLM gera resposta fundamentada nos dados atuais. Bedrock Knowledge Bases implementa RAG com Amazon OpenSearch Serverless como vector store.

---

**P:** Para qual caso de uso usar Amazon Rekognition?  
**R:** **Análise de imagens e vídeos:** detecção de objetos/cenas/texto, reconhecimento facial (comparação, pesquisa em coleção), detecção de conteúdo inapropriado (moderation), análise de PPE (EPI), detecção de celebridades, análise de vídeo em streaming ou batch. Não requer ML knowledge.

---

**P:** Qual é a diferença entre Amazon Transcribe e Amazon Polly?  
**R:** **Transcribe:** Speech-to-Text (STT) — converte áudio em texto. Features: diarização (identificar quem fala), vocabulário customizado, redação de PII, identificação de idioma automática. **Polly:** Text-to-Speech (TTS) — converte texto em áudio. Features: vozes neurais (NTTS), SSML para controle de pronúncia, lexicons customizados.

---

**P:** O que é Amazon Comprehend e quais são suas principais análises?  
**R:** NLP service para análise de texto sem código ML. Análises: **Entities** (pessoas, organizações, locais), **Key Phrases** (expressões-chave), **Sentiment** (positivo/negativo/neutro/misto), **Language Detection**, **PII Detection/Redaction**, **Topic Modeling** (agrupar documentos por temas latentes). Comprehend Medical: versão especializada para textos clínicos.

---

**P:** O que é o Amazon Lex v2 e para que serve?  
**R:** Serviço gerenciado para criar **chatbots e assistentes virtuais** de voz e texto. Usa ASR (Automatic Speech Recognition) + NLU (Natural Language Understanding). Intents = ações; Slots = parâmetros a coletar. Base da Alexa. Use case: IVR (URA), bots de atendimento, assistentes de help desk. Nenhum ML knowledge necessário.

---

**P:** Qual é a diferença entre Amazon Kendra e Amazon OpenSearch?  
**R:** **Kendra:** search inteligente para documentos corporativos usando ML e NLP. Entende linguagem natural, busca semântica, responde perguntas diretamente. Input: PDFs, Word, SharePoint, Salesforce, S3. **OpenSearch:** search e analytics técnico (logs, métricas, e-commerce search, full-text search). Kendra = inteligente para humanos; OpenSearch = técnico/full-text.

---

**P:** Para qual cenário usar Amazon SageMaker Ground Truth?  
**R:** **Labeling de dados** para treino de modelos ML. Fluxo: humans (crowdworkers via Amazon Mechanical Turk ou força de trabalho privada) anotam/rotulam imagens, textos, vídeos. Ativo Learning: modelos parcialmente treinados rotulam automaticamente dados de alta confiança, reduzindo custo de labeling humano em até 70%.

---

**P:** O que é Amazon Textract e como difere de um OCR simples?  
**R:** **Textract:** extrai texto + **estrutura** de documentos escaneados. Vai além de OCR: detecta Tables (extrai cabeçalhos e células), Forms (extrai campo:valor de formulários), Queries (responde perguntas específicas sobre o documento). Suporta PDFs multi-página, IDs (passaportes, carteiras de habilitação), documentos médicos.

---

**P:** O que é Amazon Fraud Detector?  
**R:** Serviço gerenciado de detecção de fraudes usando ML. Modelos pré-treinados em dados de fraude AWS + dados históricos do cliente. Detection events: new account registrations, online payments, guest checkout. Retorna um fraud score. Sem necessidade de expertise em ML.

---

**P:** Quais são as etapas do SageMaker ML pipeline?  
**R:** **(1) Data prep:** SageMaker Data Wrangler / Processing Jobs. **(2) Training:** Training Jobs (instâncias EC2). **(3) Tuning:** Hyperparameter Optimization (HPO). **(4) Evaluation:** Experiments. **(5) Registry:** Model Registry. **(6) Deploy:** SageMaker Endpoints (online) / Batch Transform (offline). **(7) Monitor:** Model Monitor.

---

**P:** O que é o Amazon SageMaker Feature Store?  
**R:** Repositório centralizado de features (variáveis de input) para modelos ML. Permite: compartilhar features entre equipes e modelos, reutilizar features computadas, consistência entre training e inference (mesmo cálculo). **Online Store:** baixa latência para inference em real-time. **Offline Store:** histórico via S3 para training.

---

**P:** O que é Amazon Forecast e qual problema resolve?  
**R:** Serviço gerenciado de **previsão de séries temporais** usando DeepAR+ (algoritmo proprietário Amazon). Input: dados históricos de séries temporais (vendas, demanda, tráfego) + metadados (promoções, feriados). Output: previsões futuras com intervalos de confiança. Casos: planejamento de demanda, supply chain, capacity planning.

---

**P:** O que é Amazon Personalize e para qual tipo de recomendação serve?  
**R:** Serviço gerenciado de **recomendações personalizadas** (mesmo que usa em Amazon.com/Prime Video). Input: dados de interação usuário-item (clicks, purchases, plays), metadados de itens/usuários. Output: recomendações em tempo real by userId. Use cases: "outros usuários também compraram", "continuar assistindo", "produtos recomendados".

---

**P:** O que é Amazon Lookout for Metrics?  
**R:** Detecção automática de **anomalias em série temporal** usando ML. Input: métricas de negócio (revenue, DAUs, conversion rate) de S3, RDS, Redshift, Salesforce, etc. Detecta desvios significativos automaticamente. Envia alertas via SNS. Sem ML knowledge necessário. Diferente de CloudWatch Anomaly Detection (métricas técnicas de infraestrutura).

---

**P:** O que é o Amazon SageMaker Jumpstart?  
**R:** Hub de modelos pré-treinados e soluções ML prontas para deploy em um clique. Contém: Foundation Models (Llama, Mistral, Falcon), modelos de visão (ResNet, EfficientNet), NLP (BERT, RoBERTa), soluções industriais (demand forecast, fraud). Acelera início de projetos ML sem partir do zero.

---

**P:** O que é Amazon Bedrock Guardrails?  
**R:** Controles de segurança para aplicações com Foundation Models. Funcionalidades: filtros de conteúdo (violence, hate speech, sexual), topic denial (bloquear tópicos proibidos — ex: concorrência), PII redaction, word filters. Aplicados automaticamente no input do usuário E na resposta do modelo.

---

**P:** Para qual cenário usar Amazon Translate com Custom Terminology?  
**R:** Quando há termos específicos da empresa/produto que não devem ser traduzidos automaticamente (nomes de produto, gírias corporativas, siglas). Custom Terminology: CSV com par origem→destino. Ex: "Amazon Aurora" → "Amazon Aurora" (não traduzir), "order" → "pedido" (tradução específica). Mantém consistência terminológica em traduções.

---

**P:** O que é Amazon SageMaker Canvas?  
**R:** Interface **no-code** para ML. Profissionais de negócio sem experiência em programação podem treinar modelos de ML (classificação, regressão, previsão de séries temporais, análise de imagens) via interface visual. Input: uploads de CSV, S3. Usa AutoML sob o capô. Compartilha modelos com cientistas de dados no SageMaker Studio.

---

**P:** O que é Amazon Rekognition Content Moderation?  
**R:** API que detecta conteúdo inapropriado ou adulto em imagens e vídeos. Retorna labels hierárquicos com confidence score (ex: "Explicit Nudity" > "Nudity", confidence: 99.5%). Casos: plataformas UGC (User Generated Content), redes sociais, marketplaces. Empresas definem threshold de confidence para moderação automática.

---

**P:** O que é Amazon Transcribe Call Analytics?  
**R:** Versão especializada do Transcribe para análise de ligações de call center. Além de transcrição: detecta o **sentimento** de cada turno de conversa (cliente vs. agente), identifica categorias personalizadas (reclamação, cancelamento), extrai entidades (números de conta, datas), mede tempo de silêncio e interrupções.

---

**P:** Quando usar Amazon Comprehend Medical?  
**R:** Extrai entidades médicas de texto clínico: **diagnósticos** (ICD-10 codes), **medicamentos** (dosagem, frequência, via de administração), **procedimentos médicos**, **anatomia**, **sinais vitais**. Detecta relacionamentos (medicamento → dosagem). HIPAA eligible. Casos: NLP de prontuários eletrônicos, codificação médica, pesquisa clínica.

---

**P:** O que é Amazon Polly Neural Text-to-Speech (NTTS)?  
**R:** Vozes sintéticas de altíssima qualidade usando deep learning — praticamente indistinguíveis de voz humana. Mais natural que vozes padrão (concatenation synthesis). Suporta **SSML** (Speech Synthesis Markup Language) para controlar pausas, ênfase, pronúncia. Casos: IVR, audiobooks, acessibilidade, e-learning.

---

**P:** O que é o modelo de responsabilidade no Amazon Bedrock (Security)?  
**R:** **AWS responsável por:** infraestrutura, modelo base, dados de training do FM. **Cliente responsável por:** dados enviados ao modelo (prompts/contexto), dados de fine-tuning, aplicação e respostas geradas. Os prompts e respostas **não são usados para treinar modelos** da AWS (dados privados do cliente). Bedrock é HIPAA/SOC/ISO elegível.
