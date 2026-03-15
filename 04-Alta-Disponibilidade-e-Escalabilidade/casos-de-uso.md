# Casos de Uso — Alta Disponibilidade e Escalabilidade

## Caso 1: E-commerce Multi-Tier com ALB e ASG

**Contexto:**  
Um e-commerce precisa de alta disponibilidade (99,9% SLA) com capacidade de escalar de 50 para 2.000 req/s em horários de pico (Black Friday), separando tráfego de API e frontend.

**Arquitetura:**

```
Route 53 (DNS Failover + Health Check)
         │
         ▼
 Internet Gateway  (VPC pública)
         │
         ▼
  ALB (multi-AZ, listener 443)
  ├── Rule 1: /api/* → TG-API (EC2 c5.xlarge, Target Tracking CPU 60%)
  ├── Rule 2: /static/* → TG-Static (S3 via redirect ou EC2)
  └── Rule 3: default → TG-Web (EC2 t3.large, Target Tracking CPU 50%)

TG-API:
  └── ASG: min=2, max=50, desired=4
        ├── Scaling: Target Tracking (CPUUtilization = 60)
        └── Health check: ELB (/health, HTTP 200, intervalo 15s)

TG-Web:
  └── ASG: min=2, max=30, desired=4
        ├── Scaling: Target Tracking + Scheduled (Black Friday pre-scale)
        └── AZ: us-east-1a, us-east-1b, us-east-1c

ElastiCache (Sessão)     RDS Multi-AZ (DB)
us-east-1a               us-east-1a (primary)
us-east-1b               us-east-1b (standby AZ)

Backup /static:
  CloudFront → S3 (assets estáticos, CDN global)
```

**Decisões de design:**
- Path-based routing no ALB separa API de frontend sem dois ALBs
- ELB health check no ASG garante substituição automática de instâncias com falha de aplicação
- Scheduled scaling pré-provisiona instâncias antes do Black Friday

**Conceitos cobrados:** ALB path-based routing, ASG Target Tracking, ELB health check, multi-AZ, escalabilidade horizontal

---

## Caso 2: API Interna com PrivateLink e NLB

**Contexto:**  
Uma fintech expõe uma API de pagamentos para parceiros externos. Os parceiros têm suas próprias VPCs e precisam acessar a API com segurança, sem tráfego pela internet, sem VPC Peering (requisito regulatório que proíbe CIDR overlap).

**Arquitetura:**

```
Conta Fintech (Provider):                    Conta Parceiro A:
┌──────────────────────────────┐             ┌─────────────────────┐
│  VPC: 10.0.0.0/16            │             │  VPC: 10.0.0.0/16   │
│                              │             │  (mesmo CIDR — ok!) │
│  EC2 (API Payment)           │             │                     │
│    └── Port 8443             │             │  EC2 (Consumer)     │
│         ↓                    │             │    └── Interface VPC │
│  NLB (internal)              │ PrivateLink │       Endpoint       │
│  └── TG: port 8443           │◀───────────▶│    IP: 10.0.x.x     │
│  └── Listener: 443           │             │    (ENI na VPC)      │
│  └── TLS: ACM cert           │             │                     │
│         ↓                    │             └─────────────────────┘
│  VPC Endpoint Service        │
│  (com whitelist de accounts) │             Conta Parceiro B:
│                              │             ┌─────────────────────┐
└──────────────────────────────┘             │  Mesmo padrão ↑     │
                                             └─────────────────────┘

Tráfego: 100% dentro da rede AWS (nunca internet)
Sem Peering: sem rota entre VPCs, sem problemas de CIDR overlap
```

**Configuração do Endpoint Service:**
```bash
# Cria o serviço a partir do NLB
aws ec2 create-vpc-endpoint-service-configuration \
  --network-load-balancer-arns arn:aws:elasticloadbalancing:... \
  --acceptance-required  # requer aprovação manual de novos consumidores

# Adiciona permissão para o account do parceiro criar endpoint
aws ec2 modify-vpc-endpoint-service-permissions \
  --service-id vpce-svc-xxxxxxxxxxxxxxxx \
  --add-allowed-principals "arn:aws:iam::PARTNER_ACCOUNT_ID:root"
```

**Conceitos cobrados:** PrivateLink, NLB como backend de VPC Endpoint Service, Interface VPC Endpoint, segurança cross-account sem internet

---

## Caso 3: Blue/Green Deployment sem Downtime

**Contexto:**  
Um time de DevOps precisa fazer deploys de nova versão da aplicação com zero downtime, com capacidade de rollback imediato se a nova versão apresentar erros.

**Arquitetura com ALB Weighted Target Groups:**

```
ALB Listener (:443)
│
└── Rule: /* → Forward (weighted)
      ├── TG-Blue (versão 1.2.0): peso 90% → 9 instâncias (m5.large)
      └── TG-Green (versão 2.0.0): peso 10% → 1 instância (m5.large)

Deploy gradual:
  Fase 1: Blue 100%, Green 0%     (nova versão disponível mas sem tráfego)
  Fase 2: Blue 95%, Green 5%      (canary: monitoramento)
  Fase 3: Blue 80%, Green 20%     (validação)
  Fase 4: Blue 50%, Green 50%     (50/50)
  Fase 5: Blue 0%, Green 100%     (deploy completo)
  
Rollback instantâneo:
  Qualquer fase: alterar pesos → Blue 100%, Green 0%
  API: aws elbv2 modify-rule (< 1 segundo de propagação)

Monitoramento (CloudWatch):
  ├── ALB: HTTPCode_Target_5XX_Count por TG
  ├── ALB: TargetResponseTime por TG
  └── Alarm → SNS → Lambda (rollback automático se error rate > 1%)
```

**Conceitos cobrados:** ALB weighted target groups, blue/green zero-downtime, canary release, rollback automatizado

---

## Caso 4: Worker Fleet com Lifecycle Hooks e SQS

**Contexto:**  
Um sistema de processamento de vídeo usa EC2 para transcodificar arquivos recebidos de uma fila SQS. Cada job demora até 30 minutos. Ao escalar-in, é crítico que o job atual termine antes da instância ser terminada.

**Arquitetura:**

```
SQS Queue (video-jobs)
    │ Long polling
    ▼
ASG Worker Fleet
  ├── Scale-out Trigger: ApproximateNumberOfMessages > 10 (Custom CloudWatch Metric)
  ├── Scale-in Trigger: ApproximateNumberOfMessages = 0 por 5 min
  │
  └── Lifecycle Hook: EC2_INSTANCE_TERMINATING
          └── Timeout: 3600s (1 hora)
          └── Notification: EventBridge Rule

EventBridge Rule (lifecycle hook event)
  └── Lambda: LifecycleHookTerminating
        ├── Consulta status do job na instância (SSM Parameter Store)
        │     └── /workers/{instance-id}/job-status
        ├── Se job running: espera (polls a cada 60s)
        ├── Se job complete: CompleteLifecycleAction(CONTINUE)
        └── Se timeout (safety): CompleteLifecycleAction(CONTINUE)

Fluxo completo:
  ASG decide terminar instância-X
    → instância-X muda para Terminating:Wait
    → EventBridge dispara Lambda
    → Lambda verifica: job em andamento
    → Lambda espera 20 minutos
    → Lambda confirma job concluído
    → CompleteLifecycleAction(CONTINUE)
    → instância-X termina normalmente
```

**Conceitos cobrados:** Lifecycle Hooks, Terminating:Wait, EventBridge, Lambda para automação de lifecycle, custom CloudWatch metrics para ASG scaling

---

## Caso 5: Inspeção de Tráfego com GWLB e Appliance de Segurança

**Contexto:**  
Uma empresa de serviços financeiros precisa inspecionar todo o tráfego de entrada e saída da aplicação com um IDS/IPS de terceiro (Palo Alto Networks) antes de alcançar os servidores de aplicação.

**Arquitetura:**

```
Internet Gateway
      │
      ▼
VPC Provider Account (Security Hub Centralizado):
┌──────────────────────────────────────────┐
│  GWLB Endpoint (na VPC do consumidor)   │
│       │                                  │
│  GWLB (Layer 3, GENEVE)                  │
│  └── Target Group: Firewall EC2s         │
│        ├── Palo Alto VM-Series (AZ-a)    │
│        └── Palo Alto VM-Series (AZ-b)    │
└──────────────────────────────────────────┘
      │ Pacotes inspecionados devolvidos
      ▼
VPC Consumidor (Application Account):
┌──────────────────────────────────────────┐
│  Ingress Routing (VPC Route Table)       │
│  Destino: 0.0.0.0/0 → GWLB Endpoint     │
│  (Todo tráfego via internet vai ao GWLB) │
│                                          │
│  ALB → ASG (App Servers)                 │
└──────────────────────────────────────────┘

Fluxo:
  Internet → IGW → GWLB Endpoint → GWLB → Palo Alto
    → (se OK) devolvido ao GWLB → GWLB Endpoint → ALB → App
  Se bloqueado pelo IDS/IPS → dropped

Benefício: Appliances gerenciados centralmente no Security Hub Account,
           isolados da aplicação, escaláveis via ASG
```

**Conceitos cobrados:** GWLB, GENEVE protocol, VPC Ingress Routing, Security Hub centralizado, appliance transparente
