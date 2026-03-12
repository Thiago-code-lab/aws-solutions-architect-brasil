# Questões — Módulo 13: Machine Learning e Inteligência Artificial

> **Domínio SAA-C03**: Design de Arquiteturas de Alta Performance  
> **Dificuldade**: Média

---

**1.** Uma empresa de e-commerce quer exibir produtos recomendados personalizados para cada usuário com base em seu histórico de compras, sem ter uma equipe de ML. Qual serviço AWS usar?

- A) Amazon SageMaker com algoritmo de collaborative filtering
- B) Amazon Personalize (serviço gerenciado de recomendações)
- C) Amazon Forecast
- D) Amazon Comprehend

<details><summary>Resposta</summary>

**B** — Amazon Personalize: recomendações personalizadas totalmente gerenciadas sem expertise em ML. Você fornece dados históricos de interações (visualizações, compras), o serviço treina o modelo e fornece API de recomendação em tempo real. Mesmo motor do Amazon.com.

</details>

---

**2.** Uma empresa de mídia precisa moderar automaticamente imagens enviadas por usuários, identificando conteúdo adulto e violento. Qual serviço AWS usar?

- A) Amazon Rekognition Content Moderation
- B) Amazon Comprehend para análise de imagens
- C) Amazon Textract para extrair texto das imagens
- D) Amazon SageMaker com modelo customizado de classificação

<details><summary>Resposta</summary>

**A** — Amazon Rekognition detecta conteúdo inapropriado (nudez, violência, drogas) em imagens e vídeos com um API call. Retorna labels com confidence scores e categorias hierárquicas de moderação. Sem necessidade de treinar modelo customizado.

</details>

---

**3.** Uma central de atendimento recebe 10.000 chamadas por dia. A empresa quer analisar o sentimento dos clientes e detectar palavras-chave em todas as chamadas. Qual combinação de serviços AWS usar?

- A) Amazon Rekognition + Amazon Comprehend
- B) Amazon Transcribe Call Analytics + Amazon Comprehend
- C) Amazon Lex + Amazon Polly
- D) Amazon Connect + Amazon Kendra

<details><summary>Resposta</summary>

**B** — Transcribe Call Analytics: STT especializado para call center (identifica locutor, silêncios, interrupções, sentimento por speaker). A transcrição é enviada ao Comprehend para análise de entidades, sentimento e palavras-chave. Pipeline completo de análise de chamadas.

</details>

---

**4.** Uma empresa financeira precisa prever a demanda de um produto nos próximos 30 dias com base em 3 anos de dados históricos de vendas, incluindo variáveis externas (feriados, promoções). Qual serviço usar?

- A) Amazon Personalize
- B) Amazon Forecast
- C) Amazon SageMaker com Autopilot
- D) Amazon Lookout for Metrics

<details><summary>Resposta</summary>

**B** — Amazon Forecast: previsão de séries temporais gerenciada. Aceita dados históricos e "related time series" (variáveis externas como feriados, promoções, temperatura). Mais simples que treinar um modelo no SageMaker para casos de previsão de demanda/inventário.

</details>

---

**5.** Uma empresa quer implementar um chatbot no seu site para atender perguntas frequentes. O chatbot deve entender linguagem natural e integrar com sistemas back-end via Lambda. Qual serviço usar?

- A) Amazon Polly
- B) Amazon Lex v2 com Lambda fulfillment
- C) Amazon Comprehend
- D) Amazon Kendra

<details><summary>Resposta</summary>

**B** — Amazon Lex v2: cria chatbots conversacionais (mesma tecnologia do Alexa). Define Intents (ações), Slots (dados a coletar), Utterances (frases). Para cada intent, uma Lambda fulfillment executa a lógica de negócio (consultar BD, criar pedido, etc.) e retorna a resposta.

</details>

---

**6.** Uma empresa de saúde precisa extrair automaticamente dados estruturados (diagnósticos, medicamentos, dosagens) de relatórios médicos em PDF. Qual serviço usar?

- A) Amazon Rekognition
- B) Amazon Comprehend Medical + Amazon Textract
- C) Amazon Transcribe Medical
- D) Amazon Kendra

<details><summary>Resposta</summary>

**B** — Textract: extrai texto e estrutura (tabelas, forms) de PDFs/imagens. Comprehend Medical: analisa texto médico extraído e identifica entidades clínicas (diagnósticos, medicamentos, dosagens), condições e relações entre elas. Pipeline: Textract → texto → Comprehend Medical → dados estruturados.

</details>

---

**7.** Uma empresa de logística tem uma frota de veículos e quer detectar automaticamente quando um motorista está com sinais de fadiga (olhos fechados, bocejo) usando câmeras a bordo. Qual serviço usar?

- A) Amazon Rekognition Video com análise de faces em streaming
- B) Amazon Lookout for Vision
- C) Amazon SageMaker com modelo customizado de detecção
- D) Amazon Comprehend

<details><summary>Resposta</summary>

**A** — Rekognition Video: analisa streams de vídeo em tempo real (via Kinesis Video Streams). Face Analysis detecta atributos como olhos abertos/fechados, estado emocional. Para detecção de fadiga em tempo real, Rekognition Video + alertas SNS é a solução mais simples.

</details>

---

**8.** Um time de Data Science precisa treinar um modelo de classificação com 100 GB de dados no S3, otimizando automaticamente os hiperparâmetros, sem gerenciar a infraestrutura de treinamento. Qual serviço e feature usar?

- A) SageMaker Training Job com Hyperparameter Tuning Job
- B) Amazon Forecast com AutoML
- C) SageMaker Autopilot
- D) AWS Glue ML Transforms

<details><summary>Resposta</summary>

**A** — SageMaker Training Job: treina modelo em instâncias gerenciadas com seus dados S3. Hyperparameter Tuning Job (HPO): executa múltiplos training jobs em paralelo com diferentes combinações de hiperparâmetros usando Bayesian Optimization. Autopilot (C) é mais automático mas menos controlável.

</details>

---

**9.** Uma empresa quer usar um modelo de linguagem grande (LLM) para gerar sumários de documentos, sem treinar o modelo do zero e sem gerenciar GPUs. Qual serviço usar?

- A) Amazon SageMaker com modelo HuggingFace
- B) Amazon Bedrock com Foundation Models via API
- C) Amazon Comprehend com análise de sentimento
- D) Amazon Kendra

<details><summary>Resposta</summary>

**B** — Amazon Bedrock: acesso a Foundation Models (Claude, Titan, etc.) via API serverless, sem provisionar ou gerenciar instâncias GPU. Paga por token processado. Para Sumarização com LLM, é o caminho mais simples e custo-efetivo comparado a gerenciar SageMaker endpoints com modelos grandes.

</details>

---

**10.** Uma empresa quer implementar um sistema de busca empresarial em que funcionários possam fazer perguntas em linguagem natural e receber respostas precisas de documentos internos (PDFs, Word, wikis). Qual serviço usar?

- A) Amazon OpenSearch Service
- B) Amazon Kendra com conectores de conteúdo
- C) Amazon Comprehend Entity Recognition
- D) Amazon Lex

<details><summary>Resposta</summary>

**B** — Amazon Kendra: enterprise search com NLP que responde perguntas em linguagem natural a partir de documentos corporativos. Conectores nativos para SharePoint, S3, Confluence, ServiceNow, etc. Diferente do OpenSearch (search por keywords/relevância), Kendra entende a intenção da pergunta.

</details>

---

**11.** Uma empresa quer detectar anomalias automaticamente em suas métricas de negócio (receita, conversão, NPS) sem configurar thresholds manuais. Qual serviço usar?

- A) CloudWatch Anomaly Detection
- B) Amazon Lookout for Metrics
- C) Amazon Forecast
- D) Amazon Comprehend

<details><summary>Resposta</summary>

**B** — Amazon Lookout for Metrics: detecta anomalias em métricas de negócio (não só infraestrutura como CloudWatch) usando ML. Conecta a RDS, Redshift, S3, CloudWatch, SaaS (Salesforce, Marketo). Envia alertas com contexto (por que a anomalia ocorreu, quais dimensões foram afetadas). CloudWatch Anomaly Detection (A) é para métricas técnicas de infraestrutura.

</details>

---

**12.** Um desenvolvedor quer garantir que imagens viesadas para treinamento de ML (sub-representação de gênero ou etnia) sejam identificadas antes do treinamento do modelo. Qual serviço usar?

- A) Amazon Rekognition Face Analysis
- B) SageMaker Clarify para detecção de bias nos dados
- C) Amazon Macie para análise de dados sensíveis
- D) Amazon GuardDuty

<details><summary>Resposta</summary>

**B** — SageMaker Clarify: analisa datasets e modelos para detectar bias (pré-treino e pós-treino). Mede métricas de bias como DPPL (Difference in Positive Proportions in Labels) e CI (Class Imbalance), e gera relatórios. Também oferece explainability (SHAP values) para explicar previsões do modelo.

</details>

---

**13.** Uma empresa quer transcrever automaticamente reuniões gravadas em vídeo, identificando quem falou cada trecho. Qual serviço e feature usar?

- A) Amazon Transcribe com Speaker Diarization habilitado
- B) Amazon Rekognition para identificar rostos nos vídeos
- C) Amazon Connect para gravações de reuniões
- D) Amazon Comprehend para identificar locutores

<details><summary>Resposta</summary>

**A** — Amazon Transcribe com Speaker Diarization (ShowSpeakerLabels = true): identifica automaticamente diferentes locutores no áudio e etiqueta cada trecho transcrito com o ID do locutor (Speaker_0, Speaker_1, etc.). Suporta até 10 locutores por transcrição.

</details>

---

**14.** Uma empresa quer traduzir automaticamente documentos do inglês para 20 idiomas diferentes, respeitando terminologia técnica específica do setor. Qual serviço e feature usar?

- A) Amazon Polly com Multi-Language support
- B) Amazon Translate com Custom Terminology
- C) Amazon Comprehend com Detect Language
- D) Amazon Bedrock com Claude para tradução contextual

<details><summary>Resposta</summary>

**B** — Amazon Translate com Custom Terminology: você fornece um glossário de termos que devem ser traduzidos de forma específica (ex: "load balancer" → sempre "balanceador de carga", nunca "equilibrador"). Garante consistência de terminologia técnica em todas as traduções.

</details>

---

**15.** Uma empresa treina modelos SageMaker com jobs de treinamento que levam 8 horas cada. Os custos estão altos. Qual é a estratégia mais efetiva para reduzir custos de treinamento?

- A) Usar instâncias menores para reduzir custo horário
- B) Habilitar SageMaker Managed Spot Training com checkpoints em S3
- C) Treinar localmente no AWS Cloud9
- D) Usar SageMaker Serverless Inference ao invés de training

<details><summary>Resposta</summary>

**B** — Spot Training: SageMaker pode usar Spot Instances (até 90% de desconto) para training jobs. Como Spot pode ser interrompida, checkpoints salvos em S3 permitem retomar o treinamento de onde parou. Para jobs longos de 8 horas, a economia pode ser de 70-90% com pequena complexidade adicional.

</details>
