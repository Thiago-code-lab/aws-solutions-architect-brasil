# Casos de Uso Reais — Monitoramento: CloudWatch, CloudTrail e Config (Módulo 14)

## Caso 1 — Observabilidade Full Stack de Aplicação Crítica

**Contexto:** Plataforma de pagamentos precisa de monitoramento 360° — infraestrutura, aplicação e negócio — com alertas proativos e dashboards executivos. SLA de 99,95% de disponibilidade.

**Requisitos:**
- Alertas em < 5 minutos para qualquer degradação
- Correlacionar erros de aplicação com métricas de infraestrutura
- Dashboard em tempo real para NOC (Network Operations Center)
- Post-mortem automatizado após cada incidente

**Arquitetura:**
```
CAMADA INFRAESTRUTURA:
EC2/ECS → CloudWatch Agent → CW Metrics
  (CPU, mem, disco, conexões ativas)

CAMADA APLICAÇÃO:
Lambda/ECS → CloudWatch Logs
           → Embedded Metrics Format (métricas a partir de logs)
           → X-Ray (rastreamento distribuído)

CAMADA NEGÓCIO:
Lambda → PutMetricData → CW Custom Metrics
  (pedidos/min, valor transacionado, taxa de aprovação)

ALERTAS:
CW Alarms (composite) → SNS → PagerDuty + Slack + Email
  └── Composite Alarm: CPU > 80% AND Latência > 2s AND Error Rate > 5%

DASHBOARDS:
CloudWatch Dashboard (NOC — métricas técnicas)
QuickSight (executivos — métricas de negócio)

POST-MORTEM:
CloudTrail → S3 → Athena (quem fez o quê durante o incidente)
X-Ray Service Map (onde a latência apareceu primeiro)
CloudWatch Logs Insights (query logs do período do incidente)
```

**Composite Alarm (não aciona sem coocorrência de condições):**
```
ALARM IF:
  (CPUAlarm = ALARM)
  AND (LatencyAlarm = ALARM)
  → Evita falsos positivos isolados
```

---

## Caso 2 — Auditoria de Segurança e Compliance com CloudTrail

**Contexto:** Empresa de saúde (HIPAA) precisa rastrear todas as ações de usuários e serviços sobre dados de pacientes. Auditoria externa anual exige evidências de 365 dias de logs imutáveis.

**Requisitos:**
- Log de 100% das ações de API AWS
- Logs imutáveis (não podem ser alterados nem deletados)
- Alertas em tempo real para ações de alto risco (deletar EC2, alterar IAM)
- Query de logs para investigações: "quem deletou o bucket S3 na sexta?"

**Arquitetura:**
```
FONTE:
CloudTrail (trail multi-região habilitado)
├── Management Events (ações de controle — sempre habilitado)
├── Data Events (S3 GetObject/PutObject para buckets PHI)
└── Insights Events (atividade incomum de API)

ARMAZENAMENTO IMUTÁVEL:
CloudTrail → S3 (cloudtrail-logs-hipaa)
               └── Object Lock (COMPLIANCE mode, 365 dias)
               └── SSE-KMS (criptografia com CMK)
               └── MFA Delete habilitado

ANÁLISE:
S3 (logs CloudTrail) → Athena (queries de investigação)

ALERTAS TEMPO REAL:
CloudTrail → CloudWatch Logs → CW Metric Filters
  ├── Filter: "DeleteBucket" → CW Alarm → SNS → PagerDuty
  ├── Filter: "CreateUser" ou "AttachUserPolicy" → Alarm
  └── Filter: "ConsoleLoginFailures" > 5/min → Alarm

EXEMPLO DE QUERY ATHENA (investigação):
SELECT eventtime, useridentity.arn, sourceIPAddress, requestParameters
FROM cloudtrail_logs
WHERE eventname = 'DeleteBucket'
  AND DATE(eventtime) = '2024-01-15'
ORDER BY eventtime DESC;
```

---

## Caso 3 — Governança de EC2 com AWS Config + Systems Manager

**Contexto:** Empresa com 500 instâncias EC2 em múltiplas contas precisa garantir que todas as instâncias seguem padrões de segurança: sem IP público direto, patches aplicados, AMI aprovada, tags obrigatórias.

**Requisitos:**
- Detectar automaticamente instâncias não-conformes
- Remediar automaticamente quando possível (ex: aplicar patch)
- Relatório mensal de compliance para o CISO
- Impedir criação de recursos não conformes (preventivo)

**Arquitetura:**
```
AWS Config (habilitado em todas as contas via Organizations)
  Managed Rules:
  ├── ec2-no-amazon-key-pair ✓ (sem key pair = SSH hardcoded)
  ├── ec2-instance-no-public-ip ✓ (sem IP público direto)
  ├── approved-amis-by-id ✓ (apenas AMIs do catálogo interno)
  ├── required-tags ✓ (Environment, Owner, CostCenter)
  └── ec2-managedinstance-patch-compliance-status-check ✓

  NON-COMPLIANT:
  Config → EventBridge → Lambda (remediação automática)
    ├── Tag faltando → Lambda adiciona tag padrão
    └── Patch desatualizado → SSM Automation (patch group)

  SSM Patch Manager:
  ├── Patch Baseline (aprovação automática patches críticos em 7 dias)
  ├── Maintenance Window (janela: domingos 02h-06h)
  └── Patch Group por tag: Environment=Production

RELATÓRIO:
AWS Config → Conformance Pack → S3 (relatório conformidade)
Security Hub (agrega findings de Config + GuardDuty + Inspector)
```

---

## Caso 4 — Detecção de Ameaças com GuardDuty + Security Hub

**Contexto:** Startup atacada por crypto mining em instâncias EC2 comprometidas. Precisam de sistema proativo que detecte e responda automaticamente a ameaças.

**Requisitos:**
- Detectar comportamento anômalo (exfiltração, mineração, comunicação C2)
- Resposta automática em < 1 minuto (isolar instância comprometida)
- Centralizar findings de múltiplas fontes de segurança
- Notificar time de segurança imediatamente

**Arquitetura:**
```
DETECÇÃO:
GuardDuty (analisa VPC Flow Logs + DNS Logs + CloudTrail)
  Findings de alto risco:
  ├── CryptoCurrency:EC2/BitcoinTool.B (comunicação para pool mining)
  ├── Backdoor:EC2/Spambot (envio de spam)
  ├── Trojan:EC2/BlackholeTraffic (conexão a domínio malicioso)
  └── UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B (login suspeito)

RESPOSTA AUTOMÁTICA:
GuardDuty Finding → EventBridge Rule (severity >= HIGH)
       │
       ▼
  Lambda (incident-responder)
  ├── EC2: RemoveFromLoadBalancer (tira do tráfego)
  ├── EC2: CreateSecurityGroup (bloqueia toda entrada/saída)
  ├── EC2: ModifyInstanceAttribute (isola na SG quarentena)
  ├── IAM: DetachUserPolicies (se usuário IAM comprometido)
  └── SNS → PagerDuty (alerta time segurança)

CENTRALIZAÇÃO:
Security Hub (agrega GuardDuty + Config + Inspector + Macie)
     └── Findings padronizados (ASFF format)
     └── Dashboard executivo (compliance score por conta)
```

---

## Caso 5 — Rastreamento de Requisições com X-Ray em Microsserviços

**Contexto:** Plataforma de e-commerce com 12 microsserviços nota que algumas compras levam 8 segundos (objetivo: < 2s). Sem visibilidade de qual serviço está causando a lentidão.

**Requisitos:**
- Identificar gargalo exato (qual serviço, qual chamada)
- Ver dependências entre serviços
- Alertar quando p99 latência > 3s
- Correlacionar com erros (quais requests lentos também têm erro)

**Arquitetura:**
```
API Gateway (X-Ray habilitado)
     │ X-Ray Trace ID propagado nos headers
     ▼
Lambda order-service (X-Ray SDK)
  Segments/Subsegments:
  ├── DynamoDB GetItem (products) — 15ms ✓
  ├── DynamoDB PutItem (order) — 12ms ✓
  ├── HTTP call → payment-service — 6.800ms ⚠️ GARGALO
  └── SNS Publish — 8ms ✓

payment-service (ECS/Lambda com X-Ray)
  ├── HTTP call → fraud-checker — 200ms
  ├── HTTP call → external-card-processor — 6.400ms ← ROOT CAUSE
  └── DynamoDB PutItem — 10ms

X-Ray Service Map:
  → Visualização gráfica de todas as dependências
  → Latência média por conexão destacada

CloudWatch Insights (correlação):
  filter @duration > 5000
  | stats avg(@duration), count(*) by service
```

**Descoberta:** `external-card-processor` com timeout de 6s sem retry configurado e sem circuit breaker. Solução: cache de resultados + timeout reduzido + fallback local.
