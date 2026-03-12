# Lab Prático — SQS e SNS: Filas, DLQ e Fan-out (Módulo 10)

> **Região:** us-east-1 | **Custo estimado:** ~$0 (primeiro milhão de mensagens gratuito/mês)  
> **Pré-requisitos:** AWS CLI configurado, Python 3.8+

---

## Objetivo

Praticar SQS Standard e FIFO, Dead Letter Queue, visibility timeout, SNS fan-out e integração SQS → Lambda.

---

## Parte 1 — SQS Standard + DLQ

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Criar DLQ primeiro
DLQ_URL=$(aws sqs create-queue \
  --queue-name "lab-pedidos-dlq" \
  --attributes "MessageRetentionPeriod=1209600" \
  --query 'QueueUrl' --output text)

DLQ_ARN=$(aws sqs get-queue-attributes \
  --queue-url "$DLQ_URL" \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' --output text)

echo "DLQ criada: $DLQ_ARN"

# Criar fila principal com redrive policy
QUEUE_URL=$(aws sqs create-queue \
  --queue-name "lab-pedidos" \
  --attributes "{
    \"VisibilityTimeout\": \"30\",
    \"ReceiveMessageWaitTimeSeconds\": \"20\",
    \"MessageRetentionPeriod\": \"345600\",
    \"RedrivePolicy\": \"{\\\"deadLetterTargetArn\\\":\\\"${DLQ_ARN}\\\",\\\"maxReceiveCount\\\":\\\"3\\\"}\"
  }" \
  --query 'QueueUrl' --output text)

echo "Fila principal criada: $QUEUE_URL"

# Verificar atributos
aws sqs get-queue-attributes \
  --queue-url "$QUEUE_URL" \
  --attribute-names All \
  --query 'Attributes.[VisibilityTimeout, ReceiveMessageWaitTimeSeconds, RedrivePolicy]' \
  --output json | python3 -m json.tool
```

---

## Parte 2 — Produzir e Consumir Mensagens

```python
# sqs_lab.py
import boto3
import json
import time

sqs = boto3.client('sqs', region_name='us-east-1')
QUEUE_URL = "SUBSTITUIR_PELA_URL_DA_FILA"  # terraform output ou CLI

# ──────────────────────────────────────────
# 1. Enviar mensagens em batch
# ──────────────────────────────────────────
pedidos = [
    {"pedido_id": f"PED-{i:03}", "cliente": f"CLI-{i % 5:03}",
     "produto": f"Produto {i}", "valor": i * 50.0}
    for i in range(10)
]

entries = [
    {
        'Id': str(i),
        'MessageBody': json.dumps(p),
        'MessageAttributes': {
            'prioridade': {
                'DataType': 'String',
                'StringValue': 'alta' if p['valor'] > 200 else 'normal'
            }
        }
    }
    for i, p in enumerate(pedidos)
]

# Batch de 10 (máximo por chamada)
resp = sqs.send_message_batch(QueueUrl=QUEUE_URL, Entries=entries)
print(f"Enviadas: {len(resp['Successful'])} mensagens")

# ──────────────────────────────────────────
# 2. Consumir mensagens (long polling)
# ──────────────────────────────────────────
processadas = 0
MAX = 10

print("\nConsumindo mensagens...")
while processadas < MAX:
    resp = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=5,
        WaitTimeSeconds=5,
        MessageAttributeNames=['All']
    )
    
    mensagens = resp.get('Messages', [])
    if not mensagens:
        print("Fila vazia.")
        break
    
    for msg in mensagens:
        corpo = json.loads(msg['Body'])
        prioridade = msg.get('MessageAttributes', {}).get('prioridade', {}).get('StringValue', 'normal')
        
        print(f"  Processando: {corpo['pedido_id']} | prioridade={prioridade}")
        time.sleep(0.1)  # simular processamento
        
        # Deletar após processar
        sqs.delete_message(QueueUrl=QUEUE_URL, ReceiptHandle=msg['ReceiptHandle'])
        processadas += 1

print(f"\nTotal processadas: {processadas}")

# ──────────────────────────────────────────
# 3. Simular mensagem que falha (vai para DLQ)
# ──────────────────────────────────────────
sqs.send_message(
    QueueUrl=QUEUE_URL,
    MessageBody=json.dumps({"pedido_id": "PED-FALHA", "tipo": "invalido"})
)
print("\nMensagem de falha enviada (maxReceive=3 → DLQ)")
print("Recepcionar sem deletar 3x para enviar à DLQ...")

DLQ_URL = QUEUE_URL.replace('lab-pedidos', 'lab-pedidos-dlq')
for tentativa in range(3):
    msgs = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=1,
        WaitTimeSeconds=1
    ).get('Messages', [])
    
    for msg in msgs:
        corpo = json.loads(msg['Body'])
        if corpo.get('tipo') == 'invalido':
            print(f"  Tentativa {tentativa+1}: fingindo falha (NÃO deletando)")
            # Aguardar visibility timeout expirar (30s = aguardar ou usar ChangeMessageVisibility)
            sqs.change_message_visibility(
                QueueUrl=QUEUE_URL,
                ReceiptHandle=msg['ReceiptHandle'],
                VisibilityTimeout=0  # torna visível imediatamente para próxima tentativa
            )
```

```bash
# Substituir URL e executar
QUEUE_URL=$(aws sqs get-queue-url --queue-name "lab-pedidos" --query 'QueueUrl' --output text)
sed -i "s|SUBSTITUIR_PELA_URL_DA_FILA|$QUEUE_URL|" sqs_lab.py
python3 sqs_lab.py
```

---

## Parte 3 — SQS FIFO

```bash
# Criar fila FIFO (nome deve terminar em .fifo)
FIFO_URL=$(aws sqs create-queue \
  --queue-name "lab-pagamentos.fifo" \
  --attributes "{
    \"FifoQueue\": \"true\",
    \"ContentBasedDeduplication\": \"false\",
    \"VisibilityTimeout\": \"30\",
    \"ReceiveMessageWaitTimeSeconds\": \"10\",
    \"DeduplicationScope\": \"messageGroup\",
    \"FifoThroughputLimit\": \"perMessageGroupId\"
  }" \
  --query 'QueueUrl' --output text)

echo "FIFO criada: $FIFO_URL"

# Enviar mensagens com MessageGroupId e MessageDeduplicationId
for i in 1 2 3 4 5; do
  aws sqs send-message \
    --queue-url "$FIFO_URL" \
    --message-body "{\"pagamento_id\": \"PAG-$i\", \"valor\": $((i*100))}" \
    --message-group-id "cliente-001" \
    --message-deduplication-id "pay-cli001-seq-$i"
done

# Mensagens chegam NA ORDEM de envio para o mesmo MessageGroupId
echo "5 pagamentos enviados em ordem para cliente-001"

# Consumir FIFO
aws sqs receive-message \
  --queue-url "$FIFO_URL" \
  --max-number-of-messages 5 \
  --query 'Messages[*].Body' \
  --output text
```

---

## Parte 4 — SNS Fan-out para SQS

```bash
# Criar SNS Topic
SNS_ARN=$(aws sns create-topic \
  --name "lab-eventos-compra" \
  --query 'TopicArn' --output text)

# Criar filas para subscrever
FILA_ESTOQUE_URL=$(aws sqs create-queue \
  --queue-name "lab-fila-estoque" \
  --query 'QueueUrl' --output text)
FILA_ESTOQUE_ARN=$(aws sqs get-queue-attributes \
  --queue-url "$FILA_ESTOQUE_URL" \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' --output text)

FILA_EMAIL_URL=$(aws sqs create-queue \
  --queue-name "lab-fila-email" \
  --query 'QueueUrl' --output text)
FILA_EMAIL_ARN=$(aws sqs get-queue-attributes \
  --queue-url "$FILA_EMAIL_URL" \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' --output text)

# Policy para SQS aceitar mensagens do SNS
for ARN in "$FILA_ESTOQUE_ARN" "$FILA_EMAIL_ARN"; do
  URL=$(aws sqs get-queue-url --queue-name "$(basename "${ARN%:*}" )" 2>/dev/null \
        || echo "$FILA_ESTOQUE_URL")
done

aws sqs set-queue-attributes \
  --queue-url "$FILA_ESTOQUE_URL" \
  --attributes "{
    \"Policy\": \"{\\\"Statement\\\":[{\\\"Effect\\\":\\\"Allow\\\",\\\"Principal\\\":{\\\"Service\\\":\\\"sns.amazonaws.com\\\"},\\\"Action\\\":\\\"sqs:SendMessage\\\",\\\"Resource\\\":\\\"${FILA_ESTOQUE_ARN}\\\",\\\"Condition\\\":{\\\"ArnEquals\\\":{\\\"aws:SourceArn\\\":\\\"${SNS_ARN}\\\"}}}]}\"
  }"

aws sqs set-queue-attributes \
  --queue-url "$FILA_EMAIL_URL" \
  --attributes "{
    \"Policy\": \"{\\\"Statement\\\":[{\\\"Effect\\\":\\\"Allow\\\",\\\"Principal\\\":{\\\"Service\\\":\\\"sns.amazonaws.com\\\"},\\\"Action\\\":\\\"sqs:SendMessage\\\",\\\"Resource\\\":\\\"${FILA_EMAIL_ARN}\\\",\\\"Condition\\\":{\\\"ArnEquals\\\":{\\\"aws:SourceArn\\\":\\\"${SNS_ARN}\\\"}}}]}\"
  }"

# Subscrever filas ao topic
aws sns subscribe \
  --topic-arn "$SNS_ARN" \
  --protocol sqs \
  --notification-endpoint "$FILA_ESTOQUE_ARN"

aws sns subscribe \
  --topic-arn "$SNS_ARN" \
  --protocol sqs \
  --notification-endpoint "$FILA_EMAIL_ARN"

# Publicar mensagem — vai para AMBAS as filas
aws sns publish \
  --topic-arn "$SNS_ARN" \
  --message '{"compra_id":"C001","cliente":"joao@ex.com","total":350.00}' \
  --subject "Nova Compra Confirmada"

sleep 5

# Verificar mensagens nas filas
echo "Fila Estoque:"
aws sqs receive-message --queue-url "$FILA_ESTOQUE_URL" \
  --query 'Messages[0].Body' --output text | python3 -m json.tool

echo "Fila Email:"
aws sqs receive-message --queue-url "$FILA_EMAIL_URL" \
  --query 'Messages[0].Body' --output text | python3 -m json.tool
```

---

## Limpeza

```bash
# Deletar subscrições e SNS
aws sns delete-topic --topic-arn "$SNS_ARN"
aws sqs delete-queue --queue-url "$FILA_ESTOQUE_URL"
aws sqs delete-queue --queue-url "$FILA_EMAIL_URL"
aws sqs delete-queue --queue-url "$QUEUE_URL"
aws sqs delete-queue --queue-url "$DLQ_URL"
aws sqs delete-queue --queue-url "$FIFO_URL"

rm -f sqs_lab.py
```

---

## O Que Você Aprendeu

- **Visibility Timeout:** durante processamento a mensagem fica "invisível"; se não deletar, volta à fila
- **Long Polling (WaitTimeSeconds=20):** reduz chamadas vazias em 99% vs short polling — SEMPRE preferir
- **DLQ:** após `maxReceiveCount` falhas, mensagem desviada para DLQ; alarmar sobre mensagens na DLQ
- **FIFO:** garante ordem (FIFO rigoroso), exactly-once via MessageDeduplicationId; máx 300 TPS sem batching
- **Standard:** at-least-once, out-of-order, throughput praticamente ilimitado — para maioria dos casos
- **SNS fan-out:** publicar uma vez → múltiplos consumidores independentes; DLQ no SQS protege mensagens perdidas
