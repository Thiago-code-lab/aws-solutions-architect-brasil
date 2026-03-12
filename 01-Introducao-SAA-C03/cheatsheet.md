# Cheatsheet — Introdução SAA-C03

## 📌 Pesos do Exame

| Domínio | Peso | Foco prático |
|---|---:|---|
| Design Resilient Architectures | 30% | HA, DR, desacoplamento, tolerância a falhas |
| Design High-Performing Architectures | 28% | Compute, performance de storage, banco, cache, rede |
| Design Secure Applications and Architectures | 24% | IAM, KMS, isolamento, auditoria, criptografia |
| Design Cost-Optimized Architectures | 18% | Rightsizing, serviços gerenciados, classes e modelos de compra |

## 🧠 Palavras-Chave do Enunciado

| Termo | O que normalmente sinaliza |
|---|---|
| least operational overhead | serviço gerenciado |
| cost-effective | menor custo total coerente com o requisito |
| highly available | multi-AZ, redundância, failover |
| fault tolerant | tolerância a falha sem indisponibilidade percebida |
| near real-time | streaming, evento, baixa latência |
| minimal changes | evitar refatoração grande |
| securely | IAM, KMS, endpoint privado, menor privilégio |
| global users | CloudFront, Route 53, Global Accelerator |

## 🧱 Serviços Mais Frequentes

| Grupo | Serviços |
|---|---|
| Compute | EC2, Auto Scaling, ELB, Lambda |
| Storage | S3, EBS, EFS |
| Database | RDS, Aurora, DynamoDB, ElastiCache |
| Network | VPC, Route 53, CloudFront, Transit Gateway |
| Security | IAM, KMS, Secrets Manager, Security Groups |
| Messaging | SQS, SNS, EventBridge |
| Observability | CloudWatch, CloudTrail, Config |

## ⚖️ Comparações que Mais Caem

| Comparação | Regra rápida |
|---|---|
| CloudFront vs Global Accelerator | cache HTTP vs aceleração global de tráfego |
| RDS vs DynamoDB | relacional/transacional vs NoSQL altamente escalável |
| SQS vs SNS | fila desacoplada vs pub/sub fan-out |
| Secrets Manager vs Parameter Store | rotação de segredos vs configuração/segredo simples |
| Lambda vs EC2 | serverless elástico vs controle granular |
| Multi-AZ vs Multi-Region | HA regional vs DR regional completo |

## ⏱️ Gestão de Tempo de Prova

| Item | Referência |
|---|---|
| Questões | 65 |
| Tempo total | 130 minutos |
| Meta média por questão | ~2 minutos |
| Questões longas | marcar, avançar e voltar |
| Revisão final | reservar 10 a 15 minutos |

## 🚫 Armadilhas Comuns

| Armadilha | Correção |
|---|---|
| escolher a solução mais complexa | escolha a mais aderente ao requisito |
| ignorar operação | inclua esforço administrativo na decisão |
| confundir HA com DR | HA não substitui plano multi-region |
| superestimar serviço raro | concentre-se no núcleo do exame |
| decorar sem contexto | pratique cenários completos |
