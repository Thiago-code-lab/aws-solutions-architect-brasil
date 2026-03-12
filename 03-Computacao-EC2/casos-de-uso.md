# Casos de Uso — Computação EC2

## Caso 1: Cluster de HPC com Placement Group e EFA

**Contexto:**  
Uma empresa de engenharia executa simulações de CFD (Computational Fluid Dynamics) que exigem comunicação MPI ultra-rápida entre instâncias. Qualquer latência de rede acima de 1ms degrada significativamente a performance.

**Arquitetura:**

```
Cluster Placement Group (mesma AZ, mesmo rack)
┌──────────────────────────────────────────────────────┐
│                                                      │
│  hpc6a.48xlarge  ←──EFA──→  hpc6a.48xlarge           │
│       │                          │                   │
│       └──────────────────────────┘                   │
│              10 Gbps / <100μs latência               │
│                                                      │
│  hpc6a.48xlarge  ←──EFA──→  hpc6a.48xlarge           │
│                                                      │
└──────────────────────────────────────────────────────┘
     ↑
EFA (Elastic Fabric Adapter) = RDMA-capable network interface
Bypass do kernel de rede para latência mínima

Storage: Lustre (FSx) montado em todas as instâncias
Orquestração: AWS ParallelCluster
```

**Configuração do Placement Group:**
```bash
aws ec2 create-placement-group \
  --group-name hpc-cluster \
  --strategy cluster

aws ec2 run-instances \
  --instance-type hpc6a.48xlarge \
  --placement "GroupName=hpc-cluster" \
  --network-interfaces '[{"InterfaceType":"efa"}]' \
  --count 8
```

**Trade-off:** Cluster PG → todos no mesmo rack → se o rack falhar, todas as instâncias caem. Para HPC a latência é prioridade sobre disponibilidade.

**Conceitos cobrados:** Cluster Placement Group, EFA, HPC, instâncias hpc6a

---

## Caso 2: Batch Processing com Spot Fleet e Tolerância a Falhas

**Contexto:**  
Uma empresa de data science executa jobs de machine learning que demoram entre 2 e 6 horas. O custo é crítico. Os jobs podem ser reiniciados do ponto de checkpoint se a instância for interrompida.

**Arquitetura:**

```
SQS Queue (1000 jobs)
       │
       ▼
Spot Fleet (ASG com Mixed Instance Policy)
┌─────────────────────────────────────────────────────┐
│  Strategy: capacityOptimized                        │
│  Tipos: m5.4xlarge, m5a.4xlarge, m4.4xlarge         │
│         c5.4xlarge, r5.2xlarge                      │
│                                                     │
│  [m5.4xl] [c5.4xl] [m5a.4xl] [r5.2xl] [m5.4xl]    │
│     ↑         ↑        ↑                            │
│  Job.py  Job.py   Job.py                            │
│  (SQS msg visibility timeout = job duration + 10%)  │
└─────────────────────────────────────────────────────┘
       │
       ▼
S3 (Checkpoint a cada hora + result final)

Se instância interrompida:
  → Job volta para fila SQS (visibility timeout expira)
  → Outra instância pega o job
  → Retoma do último checkpoint no S3
```

**Configuração do ASG com Mixed Instance:**
```json
{
  "MixedInstancesPolicy": {
    "InstancesDistribution": {
      "OnDemandPercentageAboveBaseCapacity": 0,
      "SpotAllocationStrategy": "capacity-optimized"
    },
    "LaunchTemplate": {
      "Overrides": [
        {"InstanceType": "m5.4xlarge"},
        {"InstanceType": "m5a.4xlarge"},
        {"InstanceType": "m4.4xlarge"},
        {"InstanceType": "c5.4xlarge"}
      ]
    }
  }
}
```

**Conceitos cobrados:** Spot Fleet, capacityOptimized, checkpointing, SQS para distribuição de jobs, tolerância a interrupção

---

## Caso 3: Servidor de Banco de Dados OLTP com io2 e Placement Group

**Contexto:**  
Uma empresa migra um banco de dados Oracle para EC2. O banco requer 50.000 IOPS garantidos, backup via snapshots e licença Oracle BYO-L (por socket).

**Arquitetura:**

```
Dedicated Host (r6i.16xlarge)
  ├── Licença Oracle: 2 sockets × 16 cores = 32 cores BYO-L
  ├── RAM: 512 GB (instância memory-optimized)
  └── Storage:
        ├── EBS io2 Block Express (500 GB, 50.000 IOPS provisionados)
        │     └── Multi-attach para standby? → Não neste caso
        └── EBS gp3 (logs de redo/archive: 2 TB)

Backup:
  └── EBS Snapshots automáticos via Data Lifecycle Manager
        └── Retenção: 7 dias, horário: 02:00 UTC
        └── Cross-region copy para DR

Monitoring:
  └── CloudWatch enhanced monitoring (1s granularity)
  └── CloudWatch Alarm → SNS → PagerDuty se IOPS > 45.000
```

**Por que Dedicated Host vs Dedicated Instance:**
- Dedicated Host: visibilidade de socket/núcleo → necessário para Oracle license compliance
- Dedicated Instance: hardware exclusivo mas sem contagem de socket → não conta para BYO-L Oracle

**Conceitos cobrados:** io2 Block Express, Dedicated Host para BYO-L, EBS snapshots, enhanced monitoring

---

## Caso 4: Ambiente Web com ASG, Spot e Launch Template

**Contexto:**  
Um e-commerce quer reduzir custos de EC2 em 60% usando Spot para instâncias web stateless (sessão no ElastiCache), mantendo uma base On-Demand para disponibilidade mínima.

**Arquitetura:**

```
ALB (us-east-1)
  │
  ├── Target Group: Web-TG
  │
ASG (mixed instances policy)
├── Base: 2 On-Demand (m5.large) — disponibilidade mínima garantida
├── Spot: até 10 instâncias (m5.large, m5a.large, m4.large, t3.xlarge)
│     └── Strategy: capacityOptimized
│
├── Launch Template v2:
│     ├── AMI: ami-prod-web (golden AMI)
│     ├── UserData: inicia app, registra no service discovery
│     ├── Spot interruption handler (SSM parameter)
│     └── IMDSv2: HttpTokens=required
│
├── Scale-out: CPU > 70% por 2 minutos → +2 instâncias
├── Scale-in: CPU < 30% por 5 minutos → -1 instância
└── Scheduled: +4 instâncias às 18h-22h (horário de pico)

ElastiCache Redis
  └── Sessões dos usuários (TTL 30min)
  └── Quando instância Spot é interrompida, sessão persiste no Redis
```

**Conceitos cobrados:** ASG Mixed Instances, Spot com On-Demand base, Launch Template, stateless app, Target Tracking + Scheduled scaling

---

## Caso 5: Hibernate para Workloads de Análise com Estado

**Contexto:**  
Uma cientista de dados executa notebooks Jupyter em EC2 com grandes datasets carregados em RAM (150 GB). Ao final do dia, ela desligar a instância mas quer retomar exatamente de onde parou no dia seguinte sem recarregar dados.

**Solução com Hibernate:**

```
r6i.4xlarge (128 vCPU, 128 GB RAM)
  ├── EBS root (gp3, 500 GB) — encriptado com CMK ⚠️
  │     └── Armazena: OS + dados + conteúdo da RAM ao hibernar
  └── EBS data (gp3, 2 TB) — datasets

Fluxo:
  18:00: hibernate-instância
    → RAM (150 GB) gravada no EBS root
    → Instância para de cobrar (cobra apenas EBS)
  08:00: start-instância
    → RAM restaurada do EBS root
    → Jupyter continua exatamente onde parou
    → Datasets em memória preservados

Requisitos para hibernate:
  ✅ RAM ≤ 150 GB (limite para hiberna)
  ✅ Root EBS encriptado
  ✅ Root EBS com espaço ≥ tamanho da RAM
  ✅ Instância não pode estar no Instance Store
  ✅ Prazo máximo: 60 dias hibernado
```

**Custo comparativo:**
- Stop: para de cobrar EC2, mas perde dados de RAM → recarregamento = horas de tempo da cientista
- Hibernate: para de cobrar EC2, RAM preservada → produtividade mantida

**Conceitos cobrados:** EC2 Hibernate, EBS encryption requirement, cost optimization, memory-optimized instances
