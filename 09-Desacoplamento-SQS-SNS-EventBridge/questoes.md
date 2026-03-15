# Questões de Prova — Módulo 09: Desacoplamento (SQS, SNS, EventBridge)

<!-- Domínio SAA-C03: Design Resilient Architectures / High-Performing Architectures -->

---

**1.** Uma aplicação processa pedidos e quer garantir que cada pedido seja processado exatamente uma vez, na ordem de chegada. Qual fila SQS usar?

- A) SQS Standard com Dead Letter Queue
- B) SQS FIFO com deduplication ID
- C) SQS Standard com VisibilityTimeout longo
- D) SQS FIFO com Message Group ID por tipo de pedido

<details>
<summary>Resposta</summary>
**B — SQS FIFO com deduplication ID**
SQS FIFO garante: **ordenação** (First-In-First-Out) e **deduplicação** (exatamente uma entrega). Message Deduplication ID previne processamento duplicado dentro de janela de 5 minutos. Standard Queue oferece at-least-once delivery (pode duplicar) e best-effort ordering (pode reordenar). Para exatamente uma vez + ordem: FIFO é obrigatório.
</details>

---

**2.** Uma Lambda função demora em média 90 segundos para processar uma mensagem SQS. Qual configuração é crítica para evitar processamento duplicado?

- A) Aumentar Message Retention Period para > 90 segundos
- B) Configurar VisibilityTimeout > 90 segundos (ex: 120s)
- C) Usar FIFO queue para evitar duplicação
- D) Habilitar Long Polling no consumer

<details>
<summary>Resposta</summary>
**B — VisibilityTimeout > tempo de processamento**
VisibilityTimeout define por quanto tempo a mensagem fica "invisível" para outros consumers após ser lida. Se o processamento demora 90s e VisibilityTimeout é o padrão de 30s, a mensagem volta à fila antes de ser confirmada (deleted), causando reprocessamento. Regra: VisibilityTimeout ≥ timeout máximo de processamento.
</details>

---

**3.** Uma aplicação processa mensagens mas após 3 tentativas, mensagens continuam falhando. Como evitar que fiquem em loop na fila infinitamente?

- A) Configurar Message Retention Period de 1 hora
- B) Usar SQS FIFO com deduplication
- C) Configurar Dead Letter Queue (DLQ) com maxReceiveCount = 3
- D) Diminuir VisibilityTimeout para detectar falha mais rápido

<details>
<summary>Resposta</summary>
**C — DLQ com maxReceiveCount**
Dead Letter Queue recebe mensagens que excederam o `maxReceiveCount` de tentativas de processamento. Após 3 falhas, a mensagem é movida para a DLQ para análise sem impactar o fluxo normal. Message Retention é quanto tempo a mensagem existe na fila (não relacionado a tentativas). DLQ é a solução padrão para poison pills.
</details>

---

**4.** Qual a diferença entre SQS Long Polling e Short Polling?

- A) Long Polling retorna imediatamente mesmo sem mensagens; Short Polling espera
- B) Long Polling espera até mensagens chegarem (waitTimeSeconds > 0) ou timeout; Short Polling retorna imediatamente
- C) Short Polling é recomendado para produção; Long Polling para desenvolvimento
- D) Long Polling funciona apenas com FIFO queues

<details>
<summary>Resposta</summary>
**B — Correto**
Short Polling: retorna imediatamente (mesmo sem mensagens) → múltiplos requests vazios = custo desnecessário. Long Polling: Consumer espera até `waitTimeSeconds` (máximo 20 segundos) por mensagens — reduz número de requests, custo e CPU do consumer. Long Polling é recomendado para produção.
</details>

---

**5.** Um sistema precisa enviar a mesma notificação de "novo pedido" para 3 sistemas diferentes: estoque, faturamento e notificação por email. Qual arquitetura usar?

- A) 3 filas SQS separadas com cada sistema fazendo polling
- B) SNS Topic → 3 SQS Queues (fan-out pattern)
- C) EventBridge com 3 regras diferentes
- D) Kinesis Data Stream com 3 consumers

<details>
<summary>Resposta</summary>
**B — Fan-out com SNS → SQS**
Fan-out clássico: publicar uma mensagem no SNS e ter múltiplas filas SQS como subscriptions. Cada sistema consome de sua própria fila independentemente. Vantagens: desacoplamento completo, os sistemas podem ser lentos sem afetar o publisher, DLQ por sistema, retry independente. EventBridge também funciona mas SNS→SQS é mais simples para este caso.
</details>

---

**6.** Uma aplicação publica eventos em SNS com vários subscribers: Lambda A, Lambda B, SQS C. Apenas Lambda A deve receber eventos do tipo `order.completed`. Como filtrar?

- A) Lambda A precisa filtrar internamente ao receber todos os eventos
- B) SNS Message Filtering na subscription de Lambda A com FilterPolicy baseada em atributos
- C) Criar um SNS Topic separado apenas para order.completed
- D) EventBridge rule com event pattern

<details>
<summary>Resposta</summary>
**B — SNS Message Filtering**
SNS Message Filtering (FilterPolicy) permite que cada subscription especifique um JSON com condições para receber apenas mensagens que correspondam aos atributos. Ex: `{"eventType": ["order.completed"]}`. Lambda A recebe apenas `order.completed`; outros subscribers recebem seus próprios filtros. Evita processamento desnecessário.
</details>

---

**7.** Uma empresa usa vários serviços AWS e SaaS e quer criar automações (ex: "quando RDS CPU > 90%, criar snapshot e notificar") sem escrever código de polling. Qual serviço usar?

- A) CloudWatch Events (descontinuado)
- B) Amazon EventBridge
- C) SNS com CloudWatch Alarm
- D) Step Functions com pooling Lambda

<details>
<summary>Resposta</summary>
**B — Amazon EventBridge**
EventBridge é a evolução do CloudWatch Events. Recebe eventos de mais de 200 serviços AWS, parceiros SaaS (Salesforce, Datadog, etc.) e custom apps. Rules com Event Patterns filtram e roteiam para targets (Lambda, Step Functions, SQS, SNS, etc.). CloudWatch Alarm + SNS funciona para métricas específicas, mas EventBridge é mais flexível para eventos de estado.
</details>

---

**8.** Qual é a diferença principal entre Kinesis Data Streams e SQS?

- A) Kinesis é para dados de streaming em tempo real com retenção e replay; SQS é para filas de mensagens com delete após consumo
- B) SQS suporta mais consumidores simultâneos que Kinesis
- C) Kinesis é mais barato que SQS para todos os casos de uso
- D) SQS tem menor latência que Kinesis

<details>
<summary>Resposta</summary>
**A — Correto**
Kinesis Data Streams: dados retidos por 1-365 dias, múltiplos consumers leem o mesmo dado (replay), ordenados por shard, base64 bytes. SQS: mensagem é deletada após confirmação de consumo (at-most-once com MessageVisibility, at-least-once padrão), sem replay nativo, order apenas no FIFO. Kinesis para analytics pipelines; SQS para task queues.
</details>

---

**9.** Uma empresa processa 1 milhão de transações por segundo para análise de fraude em tempo real. Qual serviço de streaming usar?

- A) SQS FIFO (3.000 TPS por default)
- B) SNS com filtros por tipo de transação
- C) Kinesis Data Streams com shards suficientes
- D) Amazon MQ (ActiveMQ) cluster

<details>
<summary>Resposta</summary>
**C — Kinesis Data Streams**
Kinesis escala por **shards**: cada shard suporta 1 MB/s de entrada e 2 MB/s de saída. Para 1M TPS, adicione shards suficientes. Kinesis Data Analytics pode processar em tempo real. SQS FIFO tem limite de 3.000 TPS (com batching). MQ é para migração de sistemas legacy com protocolos AMQP/STOMP/MQTT.
</details>

---

**10.** O que é "Enhanced Fan-Out" no Kinesis Data Streams?

- A) Permite enviar dados de múltiplas fontes para um único stream
- B) Permite que cada consumer registrado receba 2 MB/s de saída dedicado por shard
- C) Habilita múltiplas partições por shard para maior throughput de entrada
- D) Distribui automaticamente dados entre múltiplos Kinesis Firehose deliveries

<details>
<summary>Resposta</summary>
**B — 2 MB/s dedicado por consumer por shard**
Sem Enhanced Fan-Out: saída é compartilhada entre todos os consumers (2 MB/s total por shard dividido por N consumers). Com Enhanced Fan-Out: cada consumer registrado via `SubscribeToShard` recebe **2 MB/s dedicado por shard** via HTTP/2 push (ao invés de polling). Ideal para múltiplos consumers processando o mesmo stream em paralelo com alta velocidade.
</details>

---

**11.** Qual a diferença entre Kinesis Data Streams e Kinesis Data Firehose?

- A) Firehose precisa de consumers customizados (polling); Streams entrega diretamente para destinos
- B) Streams requer gestão de shards e consumers; Firehose é totalmente gerenciado e entrega para destinos (S3, Redshift, OpenSearch)
- C) Firehose suporta replay de dados; Streams não
- D) Streams é serverless; Firehose requer provisionamento de capacidade

<details>
<summary>Resposta</summary>
**B — Correto**
Kinesis Data Streams: você gerencia shards, escreve consumers (KCL, Lambda), dados ficam no stream por 1-365 dias (replay possível). Kinesis Firehose: totalmente managed, sem shards para gerenciar, entrega automática para S3/Redshift/OpenSearch/Splunk/HTTP endpoints, near real-time (buffer de 60s ou 1 MB), sem replay. Firehose para pipeline simples de ingestão→destino.
</details>

---

**12.** Uma empresa migrou de RabbitMQ on-premises para AWS. Querem manter o código da aplicação usando protocolo AMQP sem mudanças. Qual serviço AWS?

- A) Amazon SQS com AMQP adapter
- B) Amazon MQ (RabbitMQ ou ActiveMQ gerenciado)
- C) Amazon Kinesis com AMQP SDK
- D) SNS com FIFO topics

<details>
<summary>Resposta</summary>
**B — Amazon MQ**
Amazon MQ oferece ActiveMQ e RabbitMQ gerenciados, compatíveis com protocolos padrão: AMQP, MQTT, STOMP, OpenWire, WebSocket. Permite lift-and-shift de aplicações com brokers legados sem reescrever. SQS/SNS são APIs proprietárias AWS. Use Amazon MQ quando o código existente precisa do protocolo AMQP/MQTT nativo.
</details>

---

**13.** EventBridge Archive e Replay servem para que fim?

- A) Comprimir eventos antigos para reduzir custo de armazenamento
- B) Arquivar eventos do event bus para retransmiti-los (replay) em caso de bug no consumer
- C) Criar snapshots do event bus para disaster recovery
- D) Replicar eventos entre regiões automaticamente

<details>
<summary>Resposta</summary>
**B — Arquivar e retransmitir eventos**
Archive: salva eventos de um event bus com retenção configurável (indefinida ou N dias). Replay: retransmite eventos arquivados para o event bus (ou outro target), escolhendo janela de tempo. Útil para: corrigir bug em consumer e reprocessar eventos históricos, testar com dados de produção reais, auditar sequência de eventos.
</details>

---

**14.** Qual serviço usar quando você precisa de garantia de entrega ordered, exatamente-uma-vez, para mensagens entre microserviços, com throughput de 300 mensagens/segundo?

- A) SQS Standard Queue
- B) SNS FIFO Topic → SQS FIFO Queue
- C) EventBridge com retry policy
- D) Kinesis Data Streams (1 shard)

<details>
<summary>Resposta</summary>
**B — SNS FIFO → SQS FIFO**
SQS FIFO garante ordem e deduplicação, 300 TPS (ou 3.000 TPS com batching). Para fan-out com FIFO semântics: SNS FIFO Topic subscreve SQS FIFO Queues. EventBridge não garante ordem. Standard SQS não garante ordem. Kinesis poderia funcionar com 1 shard e consumer único, mas a semântica de queue (delete após consumo) não existe no Kinesis.
</details>

---

**15.** Uma aplicação precisa de delay de 5 minutos antes que mensagens entrem na fila para processamento. Qual configuração SQS usar?

- A) VisibilityTimeout de 300 segundos
- B) Message Timer (delay por mensagem) ou Delay Queue (delay na criação da fila)
- C) Dead Letter Queue com maxReceiveCount = 0
- D) Long Polling com waitTimeSeconds = 300

<details>
<summary>Resposta</summary>
**B — Delay Queue ou Message Timer**
**Delay Queue**: delay configurado na fila (padrão 0, máximo 15 minutos) — todas as mensagens aguardam antes de aparecer para consumers. **Message Timer**: delay configurado por mensagem individual sobrescreve o delay da fila. VisibilityTimeout esconde uma mensagem **já recebida**, não antes de entrar. Long Polling máximo é 20 segundos (não 300).
</details>
