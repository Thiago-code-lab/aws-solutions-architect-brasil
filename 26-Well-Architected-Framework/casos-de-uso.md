# Casos de Uso Reais — Well-Architected Framework (Módulo 16)

## Caso 1 — Revisão Well-Architected de Startup em Crescimento

**Contexto:** Startup de healthtech com 150K usuários ativos cresceu de 5 para 150 funcionários em 2 anos. Arquitetura original foi feita rapidamente e nunca revisada. Investidores exigem relatório de maturidade técnica antes de novo round.

**Processo de Well-Architected Review:**
```
SETUP:
AWS Well-Architected Tool (habilitado no console)
├── Criar Workload: "Plataforma Healthtech Principal"
├── Definir: owner, ambiente, tecnologias
└── Convidar parceiro APN (opcional, para revisão facilitada)

REVISÃO POR PILAR (entrevistas com equipes):

Pilar 1 — Excelência Operacional:
├── Pergunta: "Você usa IaC para todos os recursos?"
│   Resposta: "Não, muitos recursos criados manualmente no console"
│   → HIGH RISK: Criar Terraform/CDK para todos os recursos
└── Pergunta: "Você pratica runbooks para operações comuns?"
    Resposta: "Temos apenas documentos Word desatualizados"
    → MEDIUM RISK: Criar runbooks no SSM + playbooks

Pilar 2 — Segurança:
├── "Credenciais root usadas regularmente?" → SIM → CRITICAL
├── "MFA habilitado para todos os usuários?" → NÃO → HIGH
└── "Dados de pacientes criptografados em repouso?" → PARCIALMENTE → HIGH

Pilar 3 — Confiabilidade:
├── "RDS tem Multi-AZ?" → NÃO → HIGH
├── "Auto Scaling configurado?" → NÃO → HIGH
└── "DR foi testado?" → NUNCA → HIGH

Pilar 4 — Performance:
├── "Cache implementado?" → NÃO → MEDIUM
└── "Métricas de latência p99 monitoradas?" → NÃO → MEDIUM

Pilar 5 — Otimização de Custo:
├── "Instâncias ociosas?" → 40% com CPU < 5% → HIGH
└── "Reserved Instances para workloads estáveis?" → NÃO → MEDIUM

Pilar 6 — Sustentabilidade:
└── "Regiões escolhidas levam em conta % de energia renovável?" → NÃO → LOW
```

**Plano de Remediação Priorizado:**
| Prioridade | Item | Pilar | Esforço |
|-----------|------|-------|---------|
| CRÍTICO | Desativar uso root, habilitar MFA | Segurança | 1 dia |
| CRÍTICO | Habilitar RDS Multi-AZ | Confiabilidade | 4h |
| HIGH | Migrar recursos para IaC | Op. Excelência | 3 semanas |
| HIGH | Criptografar dados PHI (KMS) | Segurança | 1 semana |
| HIGH | Auto Scaling para EC2 | Confiabilidade | 2 dias |
| MEDIUM | Implementar ElastiCache | Performance | 3 dias |
| MEDIUM | Reserved Instances (1 ano) | Custo | 1 hora |

---

## Caso 2 — Implementação de DR Multi-Região

**Contexto:** Banco digital precisa garantir continuidade de operações mesmo em caso de falha completa de uma região AWS. Regulação exige RPO ≤ 15 minutos e RTO ≤ 1 hora.

**Análise de Estratégias DR:**

| Estratégia | RPO | RTO | Custo/mês | Adequada? |
|------------|-----|-----|-----------|----------|
| Backup & Restore | Horas | 24h | $1K | ❌ RTO não atende |
| Pilot Light | 15 min | 1-2h | $5K | ⚠️ RTO no limite |
| Warm Standby | 5 min | 15 min | $25K | ✅ |
| Active-Active | ~0 | ~0 | $60K | ✅ Overkill (custo) |

**Arquitetura Warm Standby Escolhida:**
```
REGIÃO PRIMÁRIA (us-east-1):
ALB → EC2 Auto Scaling (min:4, max:20)
         └── Aurora MySQL (multi-AZ, writer)
         └── ElastiCache Redis (cluster mode)
Route 53 Health Check → ALB us-east-1

REGIÃO SECUNDÁRIA (us-west-2) — Warm Standby:
ALB → EC2 Auto Scaling (min:1, max:20) ← instâncias sempre ligadas
         └── Aurora Global Database (read replica us-west-2)
         └── ElastiCache Redis (standby pequeno)
Route 53 Health Check → ALB us-west-2

REPLICAÇÃO:
Aurora Global Database → lag < 1s entre regiões
S3 Cross-Region Replication (arquivos de usuário)
DynamoDB Global Tables (dados de sessão)

FAILOVER AUTOMÁTICO:
Route 53 Failover Policy:
├── Primary: ALB us-east-1 (health check a cada 10s)
└── Secondary: ALB us-west-2 (ativado automaticamente se primary falha)

Recovery Steps (RTO: 15-30 min):
1. Route 53 detecta falha (30-60s)
2. DNS propaga para us-west-2 (30-60s) 
3. Aurora Global: promote secondary to writer (< 1 min)
4. ASG us-west-2: scale out para capacidade completa (5-10 min)
5. App totalmente restaurada (~15 min total)
```

---

## Caso 3 — Otimização de Custo: De $80K para $35K/mês

**Contexto:** SaaS B2B com workload previsível (uso intenso 8h-18h, baixo à noite e fins de semana) estava pagando $80K/mês. Auditoria identificou oportunidades significativas.

**Análise Antes (problemas identificados):**
```
INFRA ORIGINAL:
50x EC2 m5.xlarge On-Demand (24/7): $5.400/mês
5x RDS db.r5.2xlarge On-Demand: $6.000/mês
10x NAT Gateways (redundância excessiva): $4.500/mês
S3: 200 TB em Standard (dados raramente acessados depois de 30 dias): $4.600/mês
EBS: 500 volumes gp2 não utilizados: $2.500/mês
Data Transfer: $8.000/mês (processamento em múltiplas regiões)
CloudFront não utilizado (tráfego direto para EC2): $0 → $8K potencial economia
```

**Plano de Otimização:**
```
AÇÃO 1 — Reserved Instances EC2 (1 ano):
Antes: 50 × m5.xlarge On-Demand = $10.800/mês
Depois: 30 × m5.xlarge Reserved (1 ano, no upfront) = $4.800/mês
        20 × m5.xlarge Spot (tarefas tolerantes a falha) = $1.200/mês
Economia: $4.800/mês

AÇÃO 2 — Savings Plans RDS:
Antes: 5 × db.r5.2xlarge On-Demand = $6.000/mês  
Depois: Compute Savings Plans (1 ano) + right-size db.r5.xlarge = $2.800/mês
Economia: $3.200/mês

AÇÃO 3 — S3 Lifecycle Policies:
Antes: 200 TB tudo em S3 Standard = $4.600/mês
Depois:
  0-30 dias: Standard ($0.023/GB) = $460/mês
  30-90 dias: Standard-IA ($0.0125/GB) = $250/mês
  > 90 dias: Glacier Flexible ($0.004/GB) = $820/mês
Economia: $3.070/mês

AÇÃO 4 — Consolidar NAT Gateways:
Antes: 10 NAT GWs × $0.045/h = $4.500/mês + dados
Depois: 2 NAT GWs (1 por AZ de uso) = $900/mês
Economia: $3.600/mês

AÇÃO 5 — Limpar EBS não utilizados:
Antes: 500 volumes gp2 desnachados = $2.500/mês
Depois: Automação Lambda + AWS Config rule = $0
Economia: $2.500/mês

RESULTADO:
Antes: $80.000/mês
Depois: $35.000/mês
Economia: $45.000/mês (56%)
```

---

## Caso 4 — Arquitetura de Alta Disponibilidade para E-commerce Multi-AZ

**Contexto:** Varejista online espera $90 milhões em vendas durante Black Friday. Uma hora de downtime = ~$500K em receita perdida. Precisam de arquitetura tolerante a falhas.

**Arquitetura Multi-AZ Resiliente:**
```
                    Route 53 (Health Checks)
                           │
                    CloudFront (CDN global)
                           │
                    WAF + Shield Advanced
                           │
                    ALB (Multi-AZ automático)
                    ├─────────────────────────┐
                    │                         │
              AZ us-east-1a             AZ us-east-1b
              EC2 Auto Scaling          EC2 Auto Scaling
              (min:10, des:20, max:50)  (min:10, des:20, max:50)
                    │                         │
              ElastiCache Redis         ElastiCache Redis
              (node cache AZ-a)         (node cache AZ-b)
                    │                         │
              Aurora MySQL             Aurora MySQL  
              (Writer)                 (Reader — failover automático)

S3 + CloudFront (assets estáticos — imagens, CSS, JS)
SQS → Lambda (processar pedidos async — desacopla do fluxo principal)
EventBridge → Lambda (monitorar estoque em tempo real)

Auto Scaling Policies:
├── Target Tracking: CPU 60% → escalar
├── Step Scaling: SQS > 500 msgs → +5 EC2
└── Schedule: 08h-24h na BF = min:20

Circuit Breaker Pattern:
└── Se pagamento externo falha: aceitar pedido + processar async
    → Melhor UX que erro 500
```

---

## Caso 5 — Sustentabilidade: Reduzindo Pegada de Carbono da Workload

**Contexto:** Empresa de tecnologia com compromisso ESG precisa quantificar e reduzir emissões de carbono das workloads AWS. Meta: 30% de redução em 12 meses.

**Análise com Customer Carbon Footprint Tool:**
```
DIAGNÓSTICO (AWS Console → Billing → Carbon Footprint):
Região us-east-1: 45% das emissões (datacenter coal-heavy)
Instâncias ociosas (CPU < 5%): 30% do parque
On-Demand vs Serverless: 70% EC2, 30% serverless

AÇÕES DE SUSTENTABILIDADE:

1. Migrar workloads para regiões com mais energia renovável:
   us-east-1 (43% renovável) → eu-west-1 Irlanda (96% renovável)
   us-east-1 → us-west-2 Oregon (62% renovável)
   Impacto: redução direta na intensidade de carbono

2. Serverless > EC2 (recursos usados apenas quando necessário):
   API Gateway + Lambda: zero consumo quando sem tráfego
   EC2 24/7: consome mesmo ocioso
   
3. Graviton3 no lugar de x86:
   Graviton3 (40% mais eficiente energeticamente que x86)
   c7g vs c6i: mesmo custo, menor emissão

4. Auto Scaling agressivo (scale to zero onde possível):
   Schedule: desligar dev/staging à noite e fins de semana
   70% das instâncias dev ocioso 16h/dia = wastage desnecessário

5. S3 Intelligent-Tiering (menos reprocessamento de dados frios):
   Dados frios em Glacier = menor energia de storage ativo

RESULTADO ESTIMADO:
Antes: 100 toneladas CO2e/ano
Depois: 65 toneladas CO2e/ano
Redução: 35% (supera meta de 30%)
ROI Financeiro: -$15K/mês em custos operacionais (Graviton é mais barato)
```
