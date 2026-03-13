# Lab Prático — Machine Learning: Comprehend + Rekognition + Textract (Módulo 13)

> **Região:** us-east-1 | **Custo estimado:** < $1.00 (Free Tier inclui 5K unidades/mês Comprehend, 5K imagens Rekognition)  
> **Pré-requisitos:** AWS CLI configurado, Python 3.12, boto3

---

## Objetivo

Explorar os três principais serviços de IA "pré-treinada" da AWS — sem treinar modelos, só consumindo APIs.

---

## Parte 1 — Comprehend: Análise de Texto

```python
#!/usr/bin/env python3
# comprehend_demo.py

import boto3
import json

client = boto3.client('comprehend', region_name='us-east-1')

textos = [
    "O produto chegou rápido e a qualidade é excelente! Recomendo muito.",
    "Péssimo atendimento, demorou 30 dias para entregar e veio quebrado.",
    "Entrega normal, produto conforme descrito. Nada de especial.",
    "O CPF do cliente é 123.456.789-00 e o email é joao@exemplo.com.br",
    "A Amazon Web Services lançou novos serviços de inteligência artificial em São Paulo.",
]

print("=" * 60)
print("ANÁLISE DE SENTIMENTO")
print("=" * 60)

for texto in textos[:3]:
    response = client.detect_sentiment(
        Text=texto,
        LanguageCode='pt'
    )
    sentiment = response['Sentiment']
    scores = response['SentimentScore']
    print(f"\nTexto: {texto[:50]}...")
    print(f"Sentimento: {sentiment}")
    print(f"Confiança: POSITIVE={scores['Positive']:.2%}, NEGATIVE={scores['Negative']:.2%}")

print("\n" + "=" * 60)
print("DETECÇÃO DE PII (Dados Pessoais)")
print("=" * 60)

response = client.detect_pii_entities(
    Text=textos[3],
    LanguageCode='pt'
)

print(f"\nTexto: {textos[3]}")
print("Entidades PII encontradas:")
for entity in response['Entities']:
    inicio = entity['BeginOffset']
    fim = entity['EndOffset']
    valor = textos[3][inicio:fim]
    print(f"  Tipo: {entity['Type']} | Valor detectado: {valor} | Score: {entity['Score']:.2%}")

print("\n" + "=" * 60)
print("EXTRAÇÃO DE ENTIDADES")
print("=" * 60)

response = client.detect_entities(
    Text=textos[4],
    LanguageCode='pt'
)

print(f"\nTexto: {textos[4]}")
print("Entidades encontradas:")
for entity in response['Entities']:
    print(f"  {entity['Type']}: {entity['Text']} (score: {entity['Score']:.2%})")

print("\n" + "=" * 60)
print("DETECÇÃO DE IDIOMA")
print("=" * 60)

textos_idioma = [
    "This is an English sentence about cloud computing.",
    "Esta frase está em português sobre computação em nuvem.",
    "Este es un texto en español sobre servicios en la nube.",
]

for texto in textos_idioma:
    response = client.detect_dominant_language(Text=texto)
    lang = response['Languages'][0]
    print(f"  '{texto[:40]}...' → {lang['LanguageCode']} ({lang['Score']:.2%})")
```

```bash
# Executar
python3 comprehend_demo.py
```

---

## Parte 2 — Rekognition: Análise de Imagens

```python
#!/usr/bin/env python3
# rekognition_demo.py

import boto3
import urllib.request
import json

client = boto3.client('rekognition', region_name='us-east-1')
s3 = boto3.client('s3', region_name='us-east-1')

# Usar imagem pública via URL (Rekognition pode receber bytes diretamente)
# Para o lab, vamos criar uma imagem de teste via S3

import boto3, os

ACCOUNT_ID = boto3.client('sts').get_caller_identity()['Account']
BUCKET = f"lab-rekognition-{ACCOUNT_ID}"

# Criar bucket temporário
s3_resource = boto3.resource('s3', region_name='us-east-1')
try:
    s3_resource.create_bucket(Bucket=BUCKET)
except Exception as e:
    print(f"Bucket já existe ou erro: {e}")

# Download de imagem pública de teste (foto de pessoas em paisagem)
test_image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/4/43/Cute_dog.jpg/320px-Cute_dog.jpg"
urllib.request.urlretrieve(test_image_url, "/tmp/test_image.jpg")

# Upload para S3
s3.upload_file("/tmp/test_image.jpg", BUCKET, "test_image.jpg")

print("=" * 60)
print("DETECÇÃO DE LABELS (objetos/cenas)")
print("=" * 60)

response = client.detect_labels(
    Image={'S3Object': {'Bucket': BUCKET, 'Name': 'test_image.jpg'}},
    MaxLabels=10,
    MinConfidence=80.0
)

for label in response['Labels']:
    print(f"  {label['Name']:30} Confiança: {label['Confidence']:.1f}%")

print("\n" + "=" * 60)
print("MODERAÇÃO DE CONTEÚDO")
print("=" * 60)

response = client.detect_moderation_labels(
    Image={'S3Object': {'Bucket': BUCKET, 'Name': 'test_image.jpg'}},
    MinConfidence=50.0
)

if response['ModerationLabels']:
    for label in response['ModerationLabels']:
        print(f"  ALERTA: {label['Name']} (Pai: {label['ParentName']}) — {label['Confidence']:.1f}%")
else:
    print("  Nenhum conteúdo impróprio detectado ✓")

print("\n" + "=" * 60)
print("COMPARAÇÃO DE FACES (simulação com mesma imagem)")
print("=" * 60)

# Simular com mesma imagem (mesma face)
response = client.compare_faces(
    SourceImage={'S3Object': {'Bucket': BUCKET, 'Name': 'test_image.jpg'}},
    TargetImage={'S3Object': {'Bucket': BUCKET, 'Name': 'test_image.jpg'}},
    SimilarityThreshold=80
)

print(f"  Faces encontradas na fonte: {len(response.get('SourceImageFace', {})) > 0}")
print(f"  Comparações: {len(response.get('FaceMatches', []))} matches encontrados")

# Limpeza S3
s3.delete_object(Bucket=BUCKET, Key='test_image.jpg')
s3.delete_bucket(Bucket=BUCKET)
print("\nBucket de teste removido.")
```

```bash
python3 rekognition_demo.py
```

---

## Parte 3 — Textract: Extração de Documentos

```python
#!/usr/bin/env python3
# textract_demo.py

import boto3
import json

client = boto3.client('textract', region_name='us-east-1')

# Criar um documento PDF simples para Textract analisar
# (Textract aceita PNG/JPEG/PDF/TIFF)
# Para o lab, criamos uma imagem PNG com texto

from PIL import Image, ImageDraw, ImageFont
import io

# Criar imagem com texto simulando formulário
img = Image.new('RGB', (600, 400), color='white')
draw = ImageDraw.Draw(img)

draw.text((50, 30), "FORMULÁRIO DE CADASTRO", fill='black')
draw.text((50, 80), "Nome: João Silva Santos", fill='black')
draw.text((50, 120), "CPF: 000.000.000-00", fill='black')
draw.text((50, 160), "Data de Nascimento: 15/06/1985", fill='black')
draw.text((50, 200), "Email: joao.silva@empresa.com.br", fill='black')
draw.text((50, 240), "Telefone: (11) 99999-0000", fill='black')
draw.text((50, 300), "Valor Total: R$ 1.500,00", fill='black')
draw.text((50, 340), "Status: Aprovado", fill='black')

img_bytes = io.BytesIO()
img.save(img_bytes, format='PNG')
img_bytes.seek(0)

print("=" * 60)
print("EXTRAÇÃO DE TEXTO (DetectDocumentText)")
print("=" * 60)

response = client.detect_document_text(
    Document={'Bytes': img_bytes.getvalue()}
)

print("Texto extraído linha por linha:")
for block in response['Blocks']:
    if block['BlockType'] == 'LINE':
        print(f"  [{block['Confidence']:.1f}%] {block['Text']}")

print("\n" + "=" * 60)
print("ANÁLISE DE FORMULÁRIO (AnalyzeDocument FORMS)")
print("=" * 60)

img_bytes.seek(0)
response = client.analyze_document(
    Document={'Bytes': img_bytes.getvalue()},
    FeatureTypes=['FORMS']
)

key_blocks = {}
value_blocks = {}

for block in response['Blocks']:
    if block['BlockType'] == 'KEY_VALUE_SET':
        if 'KEY' in block['EntityTypes']:
            key_blocks[block['Id']] = block
        else:
            value_blocks[block['Id']] = block

print("Pares chave-valor detectados:")
for key_id, key_block in key_blocks.items():
    # Extrair texto da chave
    key_text = ' '.join([
        w['Text'] for w in [
            next((b for b in response['Blocks'] if b['Id'] == rel['Ids'][0]), None)
            for rel in key_block.get('Relationships', []) if rel['Type'] == 'CHILD'
        ] if w and w.get('BlockType') == 'WORD'
    ])
    
    # Extrair texto do valor
    value_text = ''
    for rel in key_block.get('Relationships', []):
        if rel['Type'] == 'VALUE':
            value_block = value_blocks.get(rel['Ids'][0])
            if value_block:
                for vrel in value_block.get('Relationships', []):
                    if vrel['Type'] == 'CHILD':
                        for word_id in vrel['Ids']:
                            word = next((b for b in response['Blocks'] if b['Id'] == word_id), None)
                            if word and word['BlockType'] == 'WORD':
                                value_text += word['Text'] + ' '
    
    if key_text and value_text:
        print(f"  {key_text.strip()}: {value_text.strip()}")
```

```bash
# Instalar dependências
pip3 install boto3 Pillow

python3 textract_demo.py
```

---

## Parte 4 — Pipeline Completo: Moderação Automática com Lambda

```bash
# Criar Lambda que modera imagens automaticamente
cat > moderacao_lambda.py << 'EOF'
import boto3
import json
import os

rekognition = boto3.client('rekognition')
s3 = boto3.client('s3')

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        response = rekognition.detect_moderation_labels(
            Image={'S3Object': {'Bucket': bucket, 'Name': key}},
            MinConfidence=75
        )
        
        labels = response['ModerationLabels']
        
        if labels:
            print(f"CONTEÚDO IMPRÓPRIO em s3://{bucket}/{key}")
            for label in labels:
                print(f"  - {label['Name']}: {label['Confidence']:.1f}%")
            
            # Mover para bucket quarentena
            quarantine_bucket = os.environ.get('QUARANTINE_BUCKET', bucket + '-quarantine')
            s3.copy_object(
                Bucket=quarantine_bucket,
                CopySource={'Bucket': bucket, 'Key': key},
                Key=f"quarantine/{key}"
            )
            s3.delete_object(Bucket=bucket, Key=key)
            return {'status': 'blocked', 'labels': [l['Name'] for l in labels]}
        else:
            print(f"Conteúdo aprovado: s3://{bucket}/{key}")
            return {'status': 'approved'}
EOF

zip moderacao_lambda.zip moderacao_lambda.py

# Deploy via CLI
aws lambda create-function \
  --function-name lab-moderacao-imagens \
  --runtime python3.12 \
  --role arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/lab-lambda-rekognition-role \
  --handler moderacao_lambda.lambda_handler \
  --zip-file fileb://moderacao_lambda.zip \
  --timeout 30
```

---

## Limpeza

```bash
# Remover Lambda
aws lambda delete-function --function-name lab-moderacao-imagens

# Remover arquivos locais
rm -f *.zip *.py /tmp/test_image.jpg
```

---

## O Que Você Aprendeu

- Comprehend detecta sentimento, PII, entidades e idioma sem treinar modelos
- Rekognition detecta objetos, rostos, celebridades e conteúdo impróprio
- Textract extrai texto e pares chave-valor de formulários (muito além de OCR simples)
- Todos os 3 serviços: pay-per-use, sem provisionamento, integram nativamente com S3
- Free Tier permite experimentar sem custo: Comprehend 5K unidades/mês, Rekognition 5K imagens
