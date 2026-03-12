# Cheatsheet — Simulados e Questões (Módulo 19)

## Estrutura do Exame SAA-C03

| Domínio | Peso | Tópicos-Chave |
|---------|------|---------------|
| **1 — Design de Arquiteturas Seguras** | 30% | IAM, KMS, SCPs, VPC, WAF, Shield, Cognito |
| **2 — Design de Arquiteturas Resilientes** | 26% | Multi-AZ, Multi-Region, Auto Scaling, DR, SQS, SNS |
| **3 — Design de Arquiteturas de Alto Desempenho** | 24% | ElastiCache, CloudFront, Kinesis, RDS Read Replicas, DynamoDB |
| **4 — Design de Arquiteturas Otimizadas em Custo** | 20% | Savings Plans, Reserved, Spot, S3 tiers, Trusted Advisor |

- **Questões:** 65 (50 marcadas + 15 não marcadas / experimentais)
- **Duração:** 130 minutos (~2 min/questão)
- **Aprovação:** 720/1000 (escala 100–1000)
- **Formato:** múltipla escolha (1 resposta) + múltipla resposta (2–3 respostas)
- **Idioma disponível:** Inglês, Japonês, Coreano, Português simplificado, etc.

---

## Estratégia de Tempo

| Fase | Tempo | Ação |
|------|-------|------|
| **1ª passagem** | ~70 min | Responder questões que sabe. Marcar dúvidas. |
| **2ª passagem** | ~40 min | Revisar marcadas. Eliminar e escolher. |
| **Buffer** | ~20 min | Questões difíceis / revisão final |
| **Total** | 130 min | |

> **Regra:** Nunca deixar questão em branco. Adivinhação inteligente é melhor que zero.

---

## Matriz de Decisão de Serviços

### Computação
| Critério | Lambda | ECS/Fargate | EC2 |
|---------|--------|-------------|-----|
| Duração máxima | 15 min | ilimitado | ilimitado |
| Gerenciamento SO | zero | zero | você |
| Cold start | sim | sim (menor) | não |
| Estado local | não | sim (efêmero) | sim |
| Custo base | por req | por vCPU/mem/h | por h |
| Quando usar | event-driven, curto | containers, long-running | controle total, licenças |

### Banco de Dados
| Critério | DynamoDB | RDS / Aurora | ElastiCache |
|---------|---------|--------------|-------------|
| Modelo | NoSQL (KV/Doc) | SQL relacional | In-memory |
| Escalabilidade | horizontal automática | vertical + read replicas | em cluster |
| Latência | single-digit ms | ms–s | sub-ms |
| Transações | sim (limited) | ACID completo | não |
| Quando usar | escala massiva, KV | dados relacionais | cache L1, sessão |

### Armazenamento de Objetos (S3)
| Classe | Acesso | Custo relativo | Minimum Storage |
|--------|--------|----------------|-----------------|
| Standard | Frequente | Alto | — |
| Intelligent-Tiering | Variável (auto-move) | Médio + fee monitoramento | — |
| Standard-IA | Infrequente | Médio-baixo | 30 dias |
| One Zone-IA | Infrequente, 1 AZ | Baixo | 30 dias |
| Glacier Instant | Arquivo, acesso rápido | Muito baixo | 90 dias |
| Glacier Flexible | Arquivo, 1–5h retrieval | Ultra baixo | 90 dias |
| Glacier Deep Archive | Arquivo longo prazo, 12h | Mínimo | 180 dias |

---

## Valores Numéricos Críticos para Memorizar

| Serviço | Valor | O que é |
|---------|-------|---------|
| **Lambda** | 15 min | Timeout máximo |
| **Lambda** | 10 GB | Memória máxima |
| **Lambda** | 512 MB → 10 GB | /tmp storage |
| **SQS Standard** | 14 dias | Retenção máxima |
| **SQS FIFO** | 300/s (3.000 com batching) | Throughput máximo |
| **SQS** | 256 KB | Tamanho máximo mensagem |
| **SNS** | 256 KB | Tamanho máximo mensagem |
| **S3** | 11 noves (99.999999999%) | Durabilidade |
| **S3** | 5 TB | Tamanho máximo objeto |
| **S3 Multipart** | > 100 MB recomendado | Quando usar multipart upload |
| **DynamoDB** | 400 KB | Tamanho máximo item |
| **DynamoDB** | single-digit ms | Latência típica |
| **Kinesis KDS** | 1 MB/s ou 1.000 rec/s | Por shard (ingestion) |
| **Kinesis KDS** | 2 MB/s | Por shard (leitura) |
| **Kinesis KDS** | 7 dias | Retenção padrão (max 365) |
| **API Gateway** | 29 segundos | Timeout máximo integração |
| **CloudFront** | 1 ano | TTL máximo |
| **RDS** | 35 dias | Backup automático máximo |
| **Aurora** | 6 cópias em 3 AZs | Replicação storage |
| **EBS GP3** | 16.000 IOPS | Máximo |
| **EBS io2** | 64.000 IOPS | Máximo (io2 Block Express: 256K) |
| **EFS** | Ilimitado (elástico) | Capacidade |
| **VPC** | /16 | Maior CIDR permitido |
| **VPC** | /28 | Menor CIDR permitido |
| **Subnets reservadas** | 5 IPs | AWS reserva primeiros 4 + último |
| **IAM** | 5.000 usuários | Limite padrão por conta |
| **CloudTrail** | 90 dias | Retenção padrão (Event History) |

---

## Estratégias de Eliminação

### Dicas para Eliminar Respostas Erradas

1. **Palavras-gatilho negativas:**
   - "manualmente" → quase sempre errado (AWS = automação)
   - "requer downtime" → geralmente errado para HA
   - "compartilhar credenciais root" → sempre errado

2. **Palavras que apontam resposta correta:**
   - "menor mudança operacional" → serviço gerenciado
   - "menor custo" → checar Spot, Reserved, serverless, S3-IA
   - "maior disponibilidade" → Multi-AZ, Auto Scaling
   - "mínimo de código" → serviço managed/fully-managed

3. **Armadilhas comuns:**
   - **Multi-AZ vs Read Replica:** Multi-AZ = disponibilidade/failover, Read Replica = performance/leitura
   - **SQS vs SNS:** SQS = fila (1 consumidor por msg default), SNS = pub/sub (N consumidores)
   - **S3 Standard-IA vs Glacier:** IA = acesso esporádico mas rápido, Glacier = arquivo
   - **Security Group vs NACL:** SG = stateful (retorno automático), NACL = stateless (regra explícita)

---

## Padrões de Identificação Rápida de Serviço

| Palavra-chave | Serviço Provável |
|--------------|-----------------|
| "sem servidor" / "serverless" | Lambda, Fargate, Aurora Serverless, DynamoDB |
| "event-driven" | Lambda + EventBridge / SQS / SNS |
| "streaming em tempo real" | Kinesis Data Streams |
| "transformação/entrega streaming" | Kinesis Data Firehose |
| "fila desacoplada" | SQS |
| "pub/sub" / "broadcast" | SNS |
| "workflow / orquestração" | Step Functions |
| "CDN / conteúdo estático" | CloudFront |
| "DNS / roteamento global" | Route 53 |
| "balanceamento de carga HTTP/HTTPS" | ALB |
| "balanceamento TCP/UDP extrema performance" | NLB |
| "autenticação usuários" | Cognito |
| "permissões entre contas" | IAM Roles + STS |
| "secrets / senhas rotacionáveis" | Secrets Manager |
| "parâmetros de configuração" | SSM Parameter Store |
| "criptografia de chaves gerenciadas" | KMS |
| "proteção DDoS layer 7" | WAF + Shield |
| "análise logs SQL" | Athena |
| "ETL big data" | Glue |
| "data warehouse" | Redshift |
| "ML pré-treinado (NLP/visão)" | Rekognition / Comprehend / Textract |
| "ML treinamento customizado" | SageMaker |

---

## Políticas de Roteamento Route 53

| Política | Quando Usar | Pergunta-Chave |
|---------|-------------|----------------|
| **Simple** | 1 recurso, sem lógica | Básico |
| **Weighted** | A/B testing, deploy gradual | "% do tráfego" |
| **Latency** | Menor latência por região | "melhor performance global" |
| **Failover** | DR ativo-passivo | "failover automático" |
| **Geolocation** | Conteúdo por país/continente | "leis locais / localização" |
| **Geoproximity** | Desviar tráfego por bias | "ajuste fino de roteamento" |
| **Multivalue Answer** | Health check + múltiplos IPs | "disponibilidade simples multi-IP" |

---

## Dicas de Prova

| Pista na Questão | Resposta Esperada |
|-----------------|------------------|
| "resposta imediata, 65 questões, 130 min" | ~2 min por questão |
| "aprovação mínima SAA-C03" | 720/1000 |
| "maior peso do exame" | Domínio 1 — Arquiteturas Seguras (30%) |
| "Multi-AZ RDS — qual benefício?" | Failover automático (HA), não performance |
| "Read Replica RDS — qual benefício?" | Escalar leituras (não failover) |
| "S3 durabilidade?" | 11 noves (99.999999999%) |
| "Lambda timeout máximo?" | 15 minutos |
| "SQS retenção máxima?" | 14 dias |
| "questão pede mínimo custo + acesso raro a arquivos" | S3 Glacier Deep Archive |
| "questão pede mínimo custo + acesso imprevisível" | S3 Intelligent-Tiering |
| "VPC subnets: quantos IPs AWS reserva?" | 5 (primeiros 4 + último) |
