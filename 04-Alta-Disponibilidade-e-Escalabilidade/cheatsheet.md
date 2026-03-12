# Cheatsheet — Alta Disponibilidade e Escalabilidade

## ELB — Comparativo de Tipos

| Característica | ALB | NLB | GWLB | CLB |
|---|---|---|---|---|
| Camada OSI | 7 (HTTP) | 4 (TCP/UDP) | 3 (IP) | 4/7 (misto) |
| Protocolos | HTTP, HTTPS, HTTP/2, gRPC, WebSocket | TCP, UDP, TLS | IP (GENEVE) | HTTP, HTTPS, TCP |
| IP estático | ❌ (DNS dinâmico) | ✅ (Elastic IP por AZ) | N/A | ❌ |
| IP de origem preservado | ❌ (X-Forwarded-For) | ✅ (nativo) | ✅ | ❌ |
| Targets suportados | EC2, IP, Lambda, ALB | EC2, IP | EC2 (appliances) | EC2 |
| Latência | Moderada | Ultra-baixa (<1ms) | Moderada | Moderada |
| Roteamento avançado | Path, Host, Header, Query, Method | Nenhum | Nenhum | Básico |
| Weighted target groups | ✅ | ❌ | ❌ | ❌ |
| PrivateLink backend | ❌ | ✅ | ❌ | ❌ |
| SSL/TLS offload | ✅ | ✅ (TLS listener) | ❌ | ✅ |
| Status | Atual | Atual | Atual | Legado |

---

## ALB — Regras de Listener

| Tipo de Condição | Exemplo |
|---|---|
| Path-based | `/api/*` → TG-API; `/static/*` → TG-Frontend |
| Host-based | `api.example.com` → TG-API; `app.example.com` → TG-App |
| HTTP header | `X-Version: v2` → TG-V2 |
| HTTP method | `DELETE` → TG-Secure |
| Query string | `?env=staging` → TG-Staging |
| Source IP | `10.0.0.0/8` → TG-Internal |

**Ações possíveis:** Forward (com peso), Redirect (301/302), Fixed Response (200/403/404), Authenticate (Cognito/OIDC)

---

## Cross-Zone Load Balancing

| ELB | Padrão | Custo inter-AZ |
|---|---|---|
| ALB | ✅ Habilitado | Sem custo |
| NLB | ❌ Desabilitado | Cobrado se habilitado |
| GWLB | ❌ Desabilitado | Cobrado se habilitado |
| CLB | ✅ Habilitado (via console) | Sem custo |

**Sem cross-zone:** cada nó ELB distribui tráfego apenas para instâncias na SUA AZ (concentração de carga em AZs com mais instâncias).  
**Com cross-zone:** cada nó distribui uniformemente por todas as instâncias em todas as AZs.

---

## ASG — Políticas de Scaling

| Política | Tipo | Alarme necessário | Pro | Contra |
|---|---|---|---|---|
| Target Tracking | Reativa | Criado automaticamente | Simples, resposta proporcional | Menos controle granular |
| Step Scaling | Reativa | Manual (CloudWatch) | Controle por faixas de métrica | Mais configuração |
| Simple Scaling | Reativa | Manual (CloudWatch) | Simples | Cooldown inibe reação |
| Scheduled Scaling | Proativa | Não aplicável | Antecipa picos previsíveis | Não adapta a variações inesperadas |
| Predictive Scaling | Proativa | ML automático | Pré-provisiona antes do pico | Requer 14 dias de histórico |

---

## ASG — Conceitos de Tempo

| Conceito | Default | Propósito |
|---|---|---|
| Cooldown period | 300s | Após scale-out/in, aguarda estabilizar (Simple Scaling) |
| Warmup period | 300s | Instância nova não conta nas métricas enquanto aquece |
| Health check grace period | 300s | Ignora health checks logo após lançamento |
| Lifecycle Hook timeout | 3600s | Máx tempo em Pending:Wait ou Terminating:Wait |
| Connection Draining | 300s | Drena conexações ELB antes de desregistrar instância |

---

## Lifecycle Hooks — Fluxo

```
Scale-Out:                              Scale-In:
Pending                                 InService
   ↓                                       ↓
Pending:Wait ← Lifecycle Hook          Terminating:Wait ← Lifecycle Hook
   ↓                                       ↓
(Lambda / SNS / SQS / EventBridge)     (Lambda / SNS / SQS / EventBridge)
   ↓                                       ↓
CompleteLifecycleAction(CONTINUE)      CompleteLifecycleAction(CONTINUE)
   ↓                                       ↓
InService                               Terminating:Proceed → Terminated
```

---

## Termination Policy (Padrão)

```
1. AZ com mais instâncias (rebalancear)
2. Instância com Launch Config/Template mais antigo
3. Instância mais próxima da próxima hora de cobrança
4. Aleatório (desempate)
```

Outras políticas disponíveis: `OldestInstance`, `NewestInstance`, `OldestLaunchTemplate`, `ClosestToNextInstanceHour`, `AllocationStrategy` (respeita On-Demand vs Spot target).

---

## Launch Template vs Launch Configuration

| Característica | Launch Template | Launch Configuration |
|---|---|---|
| Estado | ✅ Atual (recomendado) | ⚠️ Legado |
| Versionamento | ✅ Sim ($Latest, $Default, versões numeradas) | ❌ Imutável |
| Mixed Instance Policy | ✅ Sim | ❌ Não |
| Múltiplos tipos de instância | ✅ Sim (overrides) | ❌ Não |
| Spot + On-Demand mix | ✅ Sim | ❌ Não |
| SSM Parameter Store | ✅ (AMI via parâmetro) | ❌ |
| T2/T3 Unlimited | ✅ Sim | ❌ |
| Exigência para ASG novo | A partir de 2023 AWS incentiva LT | Suportado mas legado |

---

## Sticky Sessions

| Tipo de cookie | Gerado por | Expiração | Config |
|---|---|---|---|
| `AWSALB` | ALB | 1s – 7 dias | Duration-based no TG |
| `AWSALBAPP` | Aplicação | Conforme app | Application-based no TG |

> Sticky Sessions podem causar desbalancamento. Usar apenas quando necessário (sessões com estado).

---

## Dicas de Prova (HA/Escalabilidade)

- **NLB + Elastic IP** → IP estático para whitelist de clientes corporativos
- **ALB weighted target groups** → canary release, blue/green, A/B test
- **GWLB** → único LB para inserção transparente de appliances (GENEVE protocol)
- **Target Tracking** é a política mais simples — cria e gerencia alarmes automaticamente
- **Predictive Scaling** precisa de 14 dias de histórico — não funciona em ASG novo
- **Lifecycle Hook** é a solução para "executar algo antes de entrar em serviço" ou "antes de terminar"
- **ELB health check no ASG** é obrigatório em produção — EC2 health check não detecta falha de aplicação
- **Cross-zone no NLB** é cobrado — considerar custo de data transfer inter-AZ
- **Connection Draining**: se sessões são curtas, reduza para 30s; se longas (uploads), aumente para 900s+
- **PrivateLink** sempre requer **NLB** como backend — não ALB  
- ALB pode ter targets **Lambda** — excelente para backends serverless por path
