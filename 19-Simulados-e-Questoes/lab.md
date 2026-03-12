# Lab Prático — Simulados e Revisão Final: Setup Ambiente de Estudos (Módulo 19)

> **Região:** us-east-1 | **Custo estimado:** ~$0.01 (DynamoDB on-demand, Lambda minimal)  
> **Pré-requisitos:** AWS CLI configurado, Python 3.8+, pip

---

## Objetivo

Construir um ambiente personalizado de revisão: baralho de flashcards no DynamoDB, script de quiz interativo, e CloudWatch dashboard para rastrear progresso nos estudos.

---

## Parte 1 — DynamoDB como Banco de Flashcards

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
TABELA_FLASH="saa-flashcards"
TABELA_PROGRESSO="saa-progresso"

# Criar tabela de flashcards
aws dynamodb create-table \
  --table-name "$TABELA_FLASH" \
  --attribute-definitions \
    "AttributeName=modulo,AttributeType=S" \
    "AttributeName=card_id,AttributeType=S" \
  --key-schema \
    "AttributeName=modulo,KeyType=HASH" \
    "AttributeName=card_id,KeyType=RANGE" \
  --billing-mode PAY_PER_REQUEST \
  --global-secondary-indexes '[{
    "IndexName": "dificuldade-index",
    "KeySchema": [
      {"AttributeName": "modulo", "KeyType": "HASH"},
      {"AttributeName": "card_id", "KeyType": "RANGE"}
    ],
    "Projection": {"ProjectionType": "ALL"}
  }]'

# Criar tabela de progresso
aws dynamodb create-table \
  --table-name "$TABELA_PROGRESSO" \
  --attribute-definitions \
    "AttributeName=usuario,AttributeType=S" \
    "AttributeName=data_sessao,AttributeType=S" \
  --key-schema \
    "AttributeName=usuario,KeyType=HASH" \
    "AttributeName=data_sessao,KeyType=RANGE" \
  --billing-mode PAY_PER_REQUEST

aws dynamodb wait table-exists --table-name "$TABELA_FLASH"
aws dynamodb wait table-exists --table-name "$TABELA_PROGRESSO"
echo "Tabelas criadas!"

# Populando flashcards de revisão final
python3 << 'PYTHON'
import boto3
import uuid

dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
tabela = dynamodb.Table('saa-flashcards')

flashcards = [
    # Módulo EC2
    {"modulo": "ec2", "card_id": str(uuid.uuid4()),
     "pergunta": "Qual tipo de instância é mais adequado para bancos de dados na memória?",
     "resposta": "R (Memory Optimized) — ex: r6g, r5. Alta proporção RAM/vCPU para Redis, SAP HANA, in-memory databases.",
     "dificuldade": "medio"},
    {"modulo": "ec2", "card_id": str(uuid.uuid4()),
     "pergunta": "O que é Spot Instance Fleet?",
     "resposta": "Conjunto de Spot Instances de múltiplos tipos/AZs configurados para manter capacidade alvo. Usa substituição automática.",
     "dificuldade": "dificil"},

    # Módulo S3
    {"modulo": "s3", "card_id": str(uuid.uuid4()),
     "pergunta": "Qual classe S3 tem taxa de recuperação e custo mínimo de armazenamento de 90 dias?",
     "resposta": "S3 Glacier Flexible Retrieval — mín 90 dias de cobrança, retrieval em minutos a 12h.",
     "dificuldade": "medio"},
    {"modulo": "s3", "card_id": str(uuid.uuid4()),
     "pergunta": "Como funciona o S3 Transfer Acceleration?",
     "resposta": "Usa edge locations CloudFront para otimizar upload via TCP long-haul. Endpoint: bucket.s3-accelerate.amazonaws.com",
     "dificuldade": "facil"},

    # Módulo VPC
    {"modulo": "vpc", "card_id": str(uuid.uuid4()),
     "pergunta": "Diferença entre Security Group e NACL?",
     "resposta": "SG: stateful, nível instância, só ALLOW. NACL: stateless, nível subnet, ALLOW e DENY, regras numeradas.",
     "dificuldade": "facil"},
    {"modulo": "vpc", "card_id": str(uuid.uuid4()),
     "pergunta": "O que é um VPC Peering e qual sua limitação principal?",
     "resposta": "Conexão direta entre VPCs sem transitividade. A→B e B→C NÃO permite A→C. Alternativa: Transit Gateway.",
     "dificuldade": "medio"},

    # Módulo IAM
    {"modulo": "iam", "card_id": str(uuid.uuid4()),
     "pergunta": "O que é IAM Permission Boundary?",
     "resposta": "Política que define o MÁXIMO de permissão que uma entidade pode ter. Não concede — apenas limita o teto.",
     "dificuldade": "dificil"},

    # Módulo RDS
    {"modulo": "rds", "card_id": str(uuid.uuid4()),
     "pergunta": "Multi-AZ RDS vs Read Replica: qual serve para DR e qual para performance?",
     "resposta": "Multi-AZ = DR/HA (failover automático, sincrônico, mesmo região). Read Replica = leitura escalada (assíncrono, pode ser cross-region).",
     "dificuldade": "facil"},

    # Módulo Serverless
    {"modulo": "serverless", "card_id": str(uuid.uuid4()),
     "pergunta": "Lambda tem timeout máximo de quanto?",
     "resposta": "15 minutos (900 segundos). Para processos mais longos: SQS → Lambda, Step Functions, ECS/Fargate.",
     "dificuldade": "facil"},

    # Segurança
    {"modulo": "seguranca", "card_id": str(uuid.uuid4()),
     "pergunta": "Qual serviço protege contra DDoS Layer 3/4 sem custo adicional?",
     "resposta": "AWS Shield Standard — ativado automaticamente para todos os clientes. Shield Advanced: Layer 7 + suporte 24/7.",
     "dificuldade": "facil"},
    {"modulo": "seguranca", "card_id": str(uuid.uuid4()),
     "pergunta": "Difference: GuardDuty vs Macie vs Inspector vs WAF?",
     "resposta": "GuardDuty=ameaças conta/rede/S3. Macie=dados sensíveis S3 (PII). Inspector=vulnerabilidades EC2/Lambda/ECR. WAF=proteção HTTP Layer 7.",
     "dificuldade": "dificil"},
]

with tabela.batch_writer() as batch:
    for card in flashcards:
        batch.put_item(Item=card)

print(f"{len(flashcards)} flashcards inseridos!")
PYTHON
```

---

## Parte 2 — Script de Quiz Interativo

```python
# quiz.py — Salvar e executar com: python3 quiz.py
import boto3
import random
import json
from datetime import datetime

dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
tabela_flash = dynamodb.Table('saa-flashcards')
tabela_prog  = dynamodb.Table('saa-progresso')

def buscar_cards(modulo=None, dificuldade=None):
    if modulo:
        resp = tabela_flash.query(
            KeyConditionExpression=boto3.dynamodb.conditions.Key('modulo').eq(modulo)
        )
    else:
        resp = tabela_flash.scan()

    cards = resp['Items']
    if dificuldade:
        cards = [c for c in cards if c.get('dificuldade') == dificuldade]
    return cards

def executar_quiz():
    print("\n=== Quiz SAA-C03 ===")
    print("Módulos: ec2, s3, vpc, iam, rds, serverless, seguranca (ou ENTER para todos)")
    modulo = input("Módulo: ").strip() or None

    cards = buscar_cards(modulo=modulo)
    if not cards:
        print("Nenhum card encontrado!")
        return

    random.shuffle(cards)
    cards = cards[:10]  # máximo 10 por sessão

    acertos = 0
    total = len(cards)
    erros = []

    for i, card in enumerate(cards, 1):
        print(f"\n[{i}/{total}] Módulo: {card['modulo'].upper()}")
        print(f"P: {card['pergunta']}")
        input("Pressione ENTER para ver a resposta...")
        print(f"R: {card['resposta']}")
        acertou = input("Você acertou? (s/n): ").strip().lower() == 's'

        if acertou:
            acertos += 1
        else:
            erros.append(card)

    porcentagem = (acertos / total) * 100
    print(f"\n=== Resultado ===")
    print(f"Acertos: {acertos}/{total} ({porcentagem:.0f}%)")

    if erros:
        print(f"\nCards para revisar:")
        for c in erros:
            print(f"  [{c['modulo']}] {c['pergunta'][:60]}...")

    # Salvar progresso
    tabela_prog.put_item(Item={
        'usuario': 'estudante',
        'data_sessao': datetime.utcnow().isoformat(),
        'modulo': modulo or 'todos',
        'acertos': acertos,
        'total': total,
        'porcentagem': str(round(porcentagem, 1))
    })
    print("\nProgresso salvo no DynamoDB!")

if __name__ == '__main__':
    executar_quiz()
```

```bash
# Executar o quiz
python3 quiz.py
```

---

## Parte 3 — CloudWatch Dashboard de Estudos

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Publicar métricas customizadas de progresso
aws cloudwatch put-metric-data \
  --namespace "SAA/Estudos" \
  --metric-data '[
    {"MetricName": "FlashcardsRevisados", "Value": 10, "Unit": "Count",
     "Dimensions": [{"Name": "Modulo", "Value": "ec2"}]},
    {"MetricName": "PorcentagemAcerto", "Value": 80, "Unit": "Percent",
     "Dimensions": [{"Name": "Modulo", "Value": "ec2"}]},
    {"MetricName": "DiasEstudados", "Value": 1, "Unit": "Count",
     "Dimensions": [{"Name": "Semana", "Value": "semana-1"}]}
  ]'

# Criar dashboard visual
aws cloudwatch put-dashboard \
  --dashboard-name "SAA-C03-Progresso" \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "x": 0, "y": 0, "width": 12, "height": 6,
        "properties": {
          "title": "Flashcards Revisados por Módulo",
          "namespace": "SAA/Estudos",
          "metrics": [
            ["SAA/Estudos", "FlashcardsRevisados", "Modulo", "ec2"],
            ["SAA/Estudos", "FlashcardsRevisados", "Modulo", "s3"],
            ["SAA/Estudos", "FlashcardsRevisados", "Modulo", "vpc"]
          ],
          "period": 86400,
          "stat": "Sum",
          "view": "bar"
        }
      },
      {
        "type": "metric",
        "x": 12, "y": 0, "width": 12, "height": 6,
        "properties": {
          "title": "Taxa de Acerto por Módulo (%)",
          "namespace": "SAA/Estudos",
          "metrics": [
            ["SAA/Estudos", "PorcentagemAcerto", "Modulo", "ec2"],
            ["SAA/Estudos", "PorcentagemAcerto", "Modulo", "s3"],
            ["SAA/Estudos", "PorcentagemAcerto", "Modulo", "vpc"]
          ],
          "period": 86400,
          "stat": "Average",
          "view": "bar",
          "yAxis": {"left": {"min": 0, "max": 100}}
        }
      },
      {
        "type": "text",
        "x": 0, "y": 6, "width": 24, "height": 3,
        "properties": {
          "markdown": "## Metas SAA-C03\\n- 80%+ em todos os simulados\\n- Completar 6 semanas de estudo\\n- Revisar todos os flashcards 3x"
        }
      }
    ]
  }'

echo "Dashboard criado: SAA-C03-Progresso"
echo "Acesse: https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#dashboards:name=SAA-C03-Progresso"
```

---

## Limpeza

```bash
# Excluir tabelas DynamoDB
aws dynamodb delete-table --table-name "$TABELA_FLASH"
aws dynamodb delete-table --table-name "$TABELA_PROGRESSO"

# Excluir dashboard
aws cloudwatch delete-dashboards --dashboard-names "SAA-C03-Progresso"

# Remover arquivos locais
rm -f quiz.py
```

---

## O Que Você Aprendeu

- DynamoDB como backend key-value para apps simples — setup em < 1 min com CLI
- Métricas customizadas CloudWatch (`put-metric-data`) para rastrear qualquer coisa
- CloudWatch Dashboards via CLI com JSON body — codificável, versionável em Git
- Padrão de estudo active recall: exposição → ocultação → tentativa → feedback
- No exame: **70%+ em simulados Tutorials Dojo = alta probabilidade de aprovação no SAA-C03**
