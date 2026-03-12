# Flashcards — Módulo 09: Desacoplamento (SQS, SNS, EventBridge)

---

**P:** Qual é o VisibilityTimeout padrão do SQS e por que é importante?
**R:** Padrão: **30 segundos** (range: 0s a 12h). Após uma mensagem ser lida, fica "invisível" por esse tempo. Se não for deletada dentro do timeout, reaparece na fila. Deve ser ≥ tempo máximo de processamento para evitar reprocessamento duplicado

---

**P:** Qual é o throughput máximo de uma fila SQS FIFO?
**R:** 300 TPS (transactions per second) sem batching. Com batching de até 10 mensagens: **3.000 TPS**. Sem batching de alta escala, se necessitar mais → use SQS Standard (quase ilimitado) mas perde garantias de ordem

---

**P:** Qual é o período máximo de retenção de mensagens no SQS?
**R:** **14 dias** (padrão: 4 dias). Mínimo: 1 minuto. Após o período, mensagens são automaticamente apagadas mesmo sem serem consumidas

---

**P:** O que é SQS Long Polling e por que usar?
**R:** Consumer aguarda até `waitTimeSeconds` (máximo **20 segundos**) por mensagens disponíveis antes de retornar. Reduz requests vazios (custo), reduz latência. Short Polling retorna imediatamente (mesmo sem mensagens) — mais caro e ineficiente

---

**P:** O que é uma Dead Letter Queue (DLQ) e quando uma mensagem vai para ela?
**R:** Fila separada para mensagens que falharam no processamento. Uma mensagem vai para a DLQ quando o `ReceiveCount` excede o `maxReceiveCount` configurado na redrive policy. Serve para isolar "poison pills" (mensagens irreprocessáveis) para análise

---

**P:** Qual é o tamanho máximo de mensagem no SQS?
**R:** **256 KB** por mensagem. Para mensagens maiores, use o padrão "S3 + SQS": armazene o payload no S3 e coloque apenas o S3 object reference na mensagem SQS (via SQS Extended Client Library)

---

**P:** O que é Delay Queue no SQS?
**R:** Fila com delivery delay configurado (0 a **15 minutos**). Mensagens publicadas ficam invisíveis por esse período antes de aparecerem para consumers. Message Timer permite delay por mensagem individual (sobrescreve o delay da fila)

---

**P:** O que é SNS Message Filtering e como funciona?
**R:** Cada subscription SNS pode ter uma FilterPolicy (JSON com condições sobre message attributes). Only mensagens que correspondem ao filtro são entregues àquela subscription. Exemplo: `{"eventType": ["order.completed", "order.refunded"]}` — subscription só recebe esses tipos

---

**P:** O que é SNS FIFO Topic? Quais são as limitações?
**R:** SNS FIFO garante ordem e deduplicação. Apenas SQS FIFO Queues podem assinar SNS FIFO Topics. Throughput limitado a 300 TPS (3.000 com batching) — mesmo limite do SQS FIFO. Não integra com Lambda, email, HTTP diretamente (apenas SQS FIFO)

---

**P:** Qual é a diferença entre EventBridge Default Bus, Custom Bus e Partner Bus?
**R:** Default Bus: eventos de serviços AWS (CloudTrail, EC2, RDS, etc.). Custom Bus: eventos da sua própria aplicação (PutEvents API). Partner Bus: eventos de parceiros SaaS (Datadog, Salesforce, Zendesk, etc.) diretamente no EventBridge sem necessidade de API polling

---

**P:** O que é EventBridge Schema Registry?
**R:** Descobre e armazena schemas de eventos automaticamente. Permite gerar código de binding (Java, Python, TypeScript) para serializar/deserializar eventos type-safely. Facilita desenvolvimento orientado a eventos com type safety

---

**P:** Qual é a capacidade por shard no Kinesis Data Streams?
**R:** Entrada: **1 MB/s** ou 1.000 records/s por shard. Saída: **2 MB/s** por shard (shared entre consumers) ou 2 MB/s por consumer com Enhanced Fan-Out (HTTP/2 push dedicado)

---

**P:** Por quanto tempo Kinesis Data Streams retém dados?
**R:** Padrão: **24 horas**. Pode ser estendido para até **365 dias** (com custo adicional). Permite que múltiplos consumers leiam o mesmo stream em momentos diferentes (replay)

---

**P:** Qual é a diferença de latência entre Kinesis Data Streams e Kinesis Firehose?
**R:** Kinesis Data Streams: latência de ~200ms (real-time). Kinesis Firehose: **near real-time** — buffer de 60 segundos (mínimo) ou 1 MB antes de entregar ao destino (S3, Redshift, OpenSearch, Splunk). Firehose não permite acesso direto ao stream nem replay

---

**P:** O que é Kinesis Producer Library (KPL) e qual o benefício?
**R:** Biblioteca de alto nível para produtores Kinesis. Implementa batching (PutRecords), retry com backoff, compression e monitoramento integrado. Aumenta throughput e eficiência comparado ao SDK direto. Requer descompressão no consumer via KCL

---

**P:** Para qual caso de uso Amazon MQ é o serviço correto?
**R:** Migração lift-and-shift de aplicações existentes que usam brokers de mensagens com protocolos padrão: **AMQP, MQTT, STOMP, OpenWire**. Suporta ActiveMQ e RabbitMQ gerenciados. Se está criando uma nova aplicação, prefira SQS/SNS (mais escaláveis e gerenciados)

---

**P:** Qual o padrão de arquitetura "fan-out" com SNS e SQS?
**R:** Publicar 1 mensagem em um SNS Topic → múltiplas filas SQS (ou Lambda, HTTP) assinam. Cada subscriber processa independentemente, pode ter DLQ própria, retry próprio. Desacopla o publisher de múltiplos consumers

---

**P:** O que é "at-least-once delivery" no SQS Standard?
**R:** SQS Standard garante que a mensagem seja entregue **pelo menos uma vez** — pode ser entregue mais de uma vez (duplicação rara). Aplicações devem ser **idempotentes** (processar a mesma mensagem múltiplas vezes sem efeito colateral). Para "exatamente uma vez": use SQS FIFO com deduplication ID

---

**P:** Como integrar SQS com Auto Scaling para escalar consumers baseado na fila?
**R:** CloudWatch Alarm baseado em `ApproximateNumberOfMessagesVisible` (ou `NumberOfMessagesSent` / instâncias) → Step Scaling ou Target Tracking no Auto Scaling Group. Target: ex: manter ≤ 100 mensagens por instância. ECS também pode escalar tasks com essa métrica customizada

---

**P:** O que é EventBridge Pipe?
**R:** Conexão point-to-point entre uma source (SQS, Kinesis, DynamoDB Streams, Kafka) e um target (Lambda, Step Functions, EventBridge Bus), com filtering e enrichment opcionais no meio. Simplifica pipelines de eventos sem Lambda intermediário apenas para roteamento
