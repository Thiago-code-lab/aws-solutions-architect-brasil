# Módulo 14 — Monitoramento: CloudWatch, CloudTrail, Config e Afins

## Visão Geral do Monitoring Stack AWS

```
Observabilidade = Métricas + Logs + Traces + Eventos

CloudWatch    → Métricas, Logs, Alarmes, Dashboards, Anomaly Detection
CloudTrail    → Auditoria de API calls (quem fez o quê, quando, de onde)
AWS Config    → Configuração de recursos (o que está configurado como)
X-Ray         → Tracing distribuído (latência e gargalos em microserviços)
Systems Manager → Gerenciamento de instâncias e parâmetros operacionais
```

---

## Amazon CloudWatch

### Métricas

- Dados de ponto no tempo com namespace, nome e dimensões
- **EC2 padrão**: CPUUtilization, NetworkIn/Out, StatusCheckFailed (**não inclui memória/disco**)
- **Métricas personalizadas**: via CloudWatch Agent ou `PutMetricData` API
- **Granularidade padrão**: 1 minuto (básico) ou 5 minutos (EC2 sem detailed monitoring)
- **High Resolution Metrics**: até 1 segundo (standard = 1 min; high-resolution = 1s, 5s, 10s, 30s)
- Retenção: 3h (1s), 15 dias (1min), 63 dias (5min), 15 meses (1h)

### CloudWatch Alarms

| Estado | Descrição |
|---|---|
| OK | Métrica dentro do threshold |
| ALARM | Métrica ultrapassou threshold |
| INSUFFICIENT_DATA | Dados insuficientes para avaliar |

**Ações de Alarme:**
- EC2 Actions (stop, terminate, reboot, recover)
- Auto Scaling Actions (scale out/in)
- SNS Notification (email, webhook, Lambda, SQS)
- Systems Manager OpsItem

**Composite Alarms**: um alarme que combina múltiplos alarmes com AND/OR lógico → evita "alarm storms"

### CloudWatch Logs

```
Log Group (ex: /aws/lambda/minha-funcao)
  └── Log Streams (geradas por cada execution environment ou instância)
        └── Log Events (linhas individuais de log + timestamp)
```

**Retenção**: configurável por grupo (1 dia a Never Expire); logs expiram automaticamente se configurado

**Insights**: query SQL-like nos logs; suporta pattern/parse/stats/sort/dedup
```
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)
```

**CloudWatch Agent**: coleta métricas do SO (memória, disco, processos) e logs de arquivos customizados; usa `config.json` ou SSM Parameter Store

**Subscriptions**: envio em tempo real de logs para Lambda, Kinesis Streams, Kinesis Firehose, OpenSearch

### CloudWatch Contributor Insights

Analisa logs para identificar "top contributors" a erros/latência (ex: top 10 URLs com mais erros, top IPs)

### CloudWatch Anomaly Detection

Cria banda de valores esperados para uma métrica usando ML; alarma quando valor sai da banda

---

## AWS CloudTrail

Auditoria de API calls em todos os serviços AWS:

### Tipos de Eventos

| Tipo | Exemplos | Padrão |
|---|---|---|
| **Management Events** | CreateBucket, TerminateInstances, CreateUser | Habilitado por padrão |
| **Data Events** | S3 GetObject/PutObject, Lambda Invoke, DynamoDB GetItem | Desabilitado (alto volume) |
| **Insights Events** | Atividade de API incomum (ML-based) | Habilitado separadamente |

### Trails

- **Single-region**: registra eventos somente na região onde criado
- **All-regions**: recomendado para organização; replica eventos a S3 bucket central
- **Multi-account via Organizations**: um trail cobrindo toda a org

### CloudTrail Lake

- Aggregação e query de eventos CloudTrail em SQL (EventBridge Schema)
- Retenção configurável até 7 anos
- Alternativa ao envio para S3 + Athena

---

## AWS Config

Registro contínuo de configuração de recursos e avaliação de compliance:

**Config Recording**: snapshot e histórico de configuração de recursos (o que mudou, quando, quem mudou)

**Config Rules**: avalia compliance de recursos:
| Tipo | Descrição |
|---|---|
| **AWS Managed Rules** | 100+ regras prontas (ex: s3-bucket-public-read-prohibited, required-tags) |
| **Custom Rules** | Lambda function que avalia a configuração |
| **Detective Rules** | Avalia configuração atual (when change happens) |
| **Proactive Rules** | Avalia **antes** de criar recurso via CloudFormation (evita criar non-compliant resource) |

**Remediation**: Config + SSM Automation (corrige automaticamente recursos não-conformes)

**Aggregator**: agrega dados de Config de múltiplas regiões/contas em uma única conta central

---

## AWS X-Ray

Tracing distribuído para aplicações:

```
Request trace = conjunto de segments de uma requisição end-to-end
  └── Segment: operação em um único serviço/recurso
        └── Subsegement: operação dentro do segment (ex: chamada ao DynamoDB)
              └── Annotations: key-value indexados (filtráveis)
              └── Metadata: key-value não-indexados (para depuração)
```

**Service Map**: visualização gráfica das dependências e latência entre serviços

**X-Ray Daemon**: coleta os traces enviados pelo SDK e os envia para X-Ray API (evita ter cada app chamando a API diretamente)

**Integrações nativas**: Lambda, API Gateway, ALB, ECS, EC2 (com daemon), Elastic Beanstalk, Step Functions

---

## AWS Systems Manager (SSM)

### Session Manager
- Acesso SSH/RDP sem abrir portas (sem security groups inbound), sem bastion, sem key pairs
- Sessões gravadas no S3/CloudWatch; auditadas via CloudTrail
- Requer SSM Agent no host + IAM role `AmazonSSMManaged​InstanceCore`

### Parameter Store
- Armazenamento de configuração e segredos hierárquico
- **/Standard**: gratuito; até 4KB por parâmetro; sem histórico rotação
- **/Advanced**: pago; até 8KB; TTL policies; integração com Secrets Manager

| Tipo | Criptografia | Custo |
|---|---|---|
| String | Não | Grátis (Standard) |
| StringList | Não | Grátis (Standard) |
| SecureString | AWS KMS | Grátis (Standard) + KMS costs |

### Patch Manager
- Define Patch Baselines (quais patches aprovar/rejeitar)
- Patch Groups: agrupa instâncias por tag para patch diferenciado
- Maintenance Windows: agenda patches para janelas de manutenção

### Run Command
- Executa comandos ou scripts em instâncias sem acesso SSH
- Usa SSM Documents (JSON/YAML que define ações)

---

## AWS Trusted Advisor

Recomendações em 6 categorias:
- **Cost Optimization**: recursos ociosos/subutilizados
- **Performance**: limites de serviços, throughput
- **Security**: grupos de segurança abertos, root MFA, bucket público
- **Fault Tolerance**: backups, Multi-AZ, snapshots
- **Service Quotas**: uso próximo de limites de serviços
- **Operational Excellence** (Business/Enterprise Support)

Plano **Developer**: apenas checks básicos (Service Quotas + Security essentials)
Plano **Business/Enterprise**: todos os checks + programmatic access via API

---

## Dicas de Prova

- CloudWatch **não** coleta memória RAM/uso de disco do EC2 por padrão → precisa do CloudWatch Agent
- CloudTrail **Management Events** habilitados por padrão; **Data Events** são opt-in (custam extra)
- CloudTrail Insights: detecta **atividade de API incomum** (ex: pico de chamadas a `TerminateInstances`)
- AWS Config Rule: avalia **compliance** (não bloqueia criação); para bloquear use **Service Control Policies** ou **Config Proactive Rules com CloudFormation hooks**
- **X-Ray Sampling**: por padrão, 5% das requests são tracedas (configurável para reduzir custo)
- Parameter Store vs Secrets Manager: Secrets Manager tem **rotação automática nativa** (RDS, Redshift, DocumentDB); Parameter Store não tem rotação nativa
- SSM Session Manager: **zero SSH ports** — melhor prática de segurança ao invés de bastion hosts
- CloudWatch Logs Insights suporta queries like `fields @timestamp | filter level = "ERROR" | sort @timestamp desc | limit 20`
- Composite Alarms: combinam múltiplos alarmes para reduzir notificações desnecessárias
- config:// (AWS Config) = snapshot de **recursos**; cloudtrail:// = snapshot de **API actions**
