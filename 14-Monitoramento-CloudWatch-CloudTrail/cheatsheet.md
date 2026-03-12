# Cheatsheet — Módulo 14: Monitoramento, CloudWatch e CloudTrail

## Visão Geral dos Serviços de Monitoramento

| Serviço | Responde | Melhor Para |
|---|---|---|
| **CloudWatch Metrics** | "Como está a performance?" | Métricas de recursos e aplicações |
| **CloudWatch Logs** | "O que aconteceu?" | Logs de apps, Lambda, VPC Flow |
| **CloudTrail** | "Quem fez o quê?" | Auditoria de API calls |
| **AWS Config** | "Está configurado corretamente?" | Compliance e configuração |
| **X-Ray** | "Onde está o gargalo?" | Tracing distribuído |
| **GuardDuty** | "Há atividade suspeita?" | Detecção de ameaças ML |
| **Security Hub** | "Qual é a postura de segurança?" | Consolidação de findings |
| **Trusted Advisor** | "O que melhorar?" | Recomendações de boas práticas |

---

## CloudWatch — Métricas Importantes por Serviço

| Serviço | Métricas Built-in | Métricas Requerem Agente |
|---|---|---|
| EC2 | CPUUtilization, NetworkIn/Out, DiskReadOps | **MemoryUtilization**, DiskSpaceUtilization |
| ELB | RequestCount, TargetResponseTime, HTTPCode_5XX | — |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections | — |
| Lambda | Invocations, Duration, Errors, Throttles, ConcurrentExecutions | — |
| API GW | Count, Latency, 4XXError, 5XXError | — |
| SQS | ApproximateNumberOfMessagesVisible, NumberOfMessagesSent | — |
| DynamoDB | ConsumedReadCapacityUnits, SystemErrors | — |

**Memory e Disk EC2:** apenas via CloudWatch Agent (instalado na instância).

---

## CloudWatch Alarms — Estados

| Estado | Significado |
|---|---|
| **OK** | Métrica dentro do threshold |
| **ALARM** | Métrica violou o threshold pelo período configurado |
| **INSUFFICIENT_DATA** | Dados insuficientes para avaliar (início ou sem dados) |

**Ações de Alarm:** EC2 Action (reboot, stop, terminate, recover), Auto Scaling Action, SNS notification.

---

## CloudTrail — Tipos de Eventos

| Tipo | O Que Registra | Habilitado Por Padrão | Custo Extra |
|---|---|---|---|
| **Management Events** | Criar/deletar recursos, IAM changes, console login | **Sim** | Não (1 cópia free) |
| **Data Events** | S3 GetObject/PutObject, Lambda Invoke | **Não** | Sim |
| **Insights Events** | Atividade anômala de API | **Não** | Sim |

**Retenção no console:** 90 dias. Para longo prazo: habilitar trail com destino S3 (ilimitado).

---

## AWS Config — Rule Types

| Tipo | Quem Cria | Exemplos |
|---|---|---|
| **Managed Rules** | AWS (~200 disponíveis) | `s3-bucket-public-read-prohibited`, `ec2-instance-no-public-ip`, `rds-storage-encrypted` |
| **Custom Rules (Lambda)** | Você | Lógica customizada de compliance |
| **Proactive Rules** | AWS (novo) | Avalia antes de criar (CloudFormation hook) |
| **Detective Rules** | AWS/Custom | Avalia após criar/mudar |

**Remediação:** SSM Automation Document executado automaticamente quando NON_COMPLIANT.

---

## SSM Parameter Store — Hierarquia e Tipos

| Tipo de Parâmetro | Criptografia | Uso |
|---|---|---|
| **String** | Não | Configurações não-secretas |
| **StringList** | Não | Lista de valores (ex: AMI IDs por região) |
| **SecureString** | KMS | Secrets, passwords, tokens |

**Hierarquia:** `/app/prod/database/host`, `/app/prod/database/password` — permite política IAM por hierarquia.

| Tier | Limite | Custo | Max Tamanho |
|---|---|---|---|
| Standard | 10.000 parâmetros | Gratuito | 4 KB |
| Advanced | 100.000 parâmetros | $0.05/param/mês | 8 KB |

---

## X-Ray — Conceitos Chave

| Conceito | Definição |
|---|---|
| **Trace** | Conjunto de segments representando um request end-to-end |
| **Segment** | Dados de um único serviço/recurso no trace |
| **Subsegment** | Granularidade dentro de um segment (chamadas de banco, HTTP externo) |
| **Service Map** | Visualização gráfica de serviços + latências + erros |
| **Annotations** | Key-value indexados para filtrar traces (ex: `UserID`, `OrderID`) |
| **Metadata** | Key-value não indexados para dados adicionais de debug |
| **Sampling Rule** | % de requests a tracear; default 5% + 1/s |

**X-Ray Daemon:** processo leve instalado em EC2/ECS que coleta e envia traces para X-Ray API.

---

## CloudWatch vs CloudTrail vs Config — Decisão

```
Pergunta: "Quem deletou o bucket S3?"
→ CloudTrail (API call audit: quem + quando + de onde)

Pergunta: "O bucket S3 estava público ontem?"
→ AWS Config (histórico de configuração do recurso)

Pergunta: "Minha API está lenta?"
→ CloudWatch (métricas de latência do API GW)

Pergunta: "Onde está o gargalo na minha arquitetura microservices?"
→ AWS X-Ray (distributed tracing)

Pergunta: "Qual instância está usando mais CPU?"
→ CloudWatch (Top Contributor via Contributor Insights)

Pergunta: "Há login suspeito de IP incomum?"
→ GuardDuty (ameaças ML-based)
```

---

## Dicas de Prova — Padrões Comuns

| Pista no Enunciado | Resposta Provável |
|---|---|
| "Memória da EC2 no CloudWatch" | Instalar CloudWatch Agent |
| "Log imutável de auditoria CloudTrail" | S3 Object Lock + Log File Validation |
| "Compliance multi-conta centralizado" | Config Aggregator + Organization |
| "Acessar EC2 sem SSH, sem bastion" | SSM Session Manager |
| "Auto-remediar non-compliant Config" | Config Rule + SSM Automation |
| "Latência alta em microsserviços" | X-Ray + Service Map |
| "Alertar quando Lambda tem > 5 erros/min" | CloudWatch Alarm na métrica Errors |
| "Reter logs CloudTrail por 7 anos" | S3 Lifecycle + Glacier Deep Archive |
| "Alertar quando API sem MFA acessa S3 sensível" | Config Rule ou CloudTrail + EventBridge |
| "Detectar EC2 fazendo port scan suspeito" | GuardDuty |
| "Password/access key rotacionada automaticamente" | Secrets Manager (não Parameter Store) |
| "Detectar credenciais AWS expostas no GitHub" | GuardDuty FindingType: UnauthorizedAccess |
