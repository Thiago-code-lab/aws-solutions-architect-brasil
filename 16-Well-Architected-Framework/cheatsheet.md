# Cheatsheet — Módulo 16: Well-Architected Framework

## Os 6 Pilares — Resumo

| Pilar | Foco | Princípio-Chave |
|---|---|---|
| **Operational Excellence** | Operar e melhorar | IaC, mudanças pequenas e reversíveis |
| **Security** | Proteger | Identity como base, encrypt tudo |
| **Reliability** | Recuperar de falhas | Design for failure, horizontal scaling |
| **Performance Efficiency** | Usar recursos bem | Serverless, serviços gerenciados |
| **Cost Optimization** | Evitar gastos | Pay-per-use, eliminar ociosidade |
| **Sustainability** | Impacto ambiental | Graviton, serverless, utilização alta |

---

## Estratégias de Disaster Recovery — Comparativo

| Estratégia | RPO | RTO | Custo | Como Funciona |
|---|---|---|---|---|
| **Backup & Restore** | Horas | Horas | $ | Backups periódicos; restore quando necessário |
| **Pilot Light** | Minutos | 10-30 min | $$ | Core mínimo rodando; escalar na DR |
| **Warm Standby** | Segundos/Minutos | Minutos | $$$ | Versão reduzida sempre ativa |
| **Active-Active** | Zero | Segundos | $$$$ | Multi-region com tráfego ativo em ambas |

**RPO** = Recovery Point Objective = perda máxima de dados  
**RTO** = Recovery Time Objective = tempo máximo de downtime

---

## Tabela de Disponibilidade

| SLA | Downtime/Ano | Downtime/Mês | Arquitetura Típica |
|---|---|---|---|
| **99%** | ~87,6 horas | ~7,3 horas | Única AZ, sem redundância |
| **99.9%** | ~8,76 horas | ~43,8 minutos | Multi-AZ básico |
| **99.99%** | ~52,6 minutos | ~4,4 minutos | Multi-AZ + Auto Scaling |
| **99.999%** | ~5,26 minutos | ~26 segundos | Active-Active, redundância total |

---

## EC2 Pricing Models — Comparativo

| Modelo | Desconto vs On-Demand | Comprometimento | Melhor Para |
|---|---|---|---|
| **On-Demand** | — | Nenhum | Cargas imprevisíveis, teste |
| **Savings Plans (Compute)** | Até 66% | $ por hora (1 ou 3 anos) | Qualquer EC2/Fargate/Lambda, flexível |
| **Savings Plans (EC2)** | Até 72% | Família + região (1 ou 3 anos) | Família específica, maior desconto |
| **Reserved (Standard)** | Até 72% | Específico (tipo+região+OS) | Carga estável e previsível |
| **Reserved (Convertible)** | Até 54% | Pode trocar tipo/OS | Carga estável, mais flexibilidade |
| **Spot** | Até 90% | Nenhum (interruptível!) | Batch, fault-tolerant, dev/test |
| **Dedicated Host** | Varia | Por hora ou 1/3 anos | Licenças BYOL, compliance regulatório |

---

## Well-Architected Pillars — Ferramentas AWS

| Pilar | Ferramentas AWS Associadas |
|---|---|
| **Operational Excellence** | CloudFormation, CodePipeline, CloudWatch, Systems Manager |
| **Security** | IAM, KMS, CloudTrail, GuardDuty, Config, Shield, WAF, Cognito |
| **Reliability** | ELB + ASG, Route 53, RDS Multi-AZ, Aurora, S3, Backup |
| **Performance Efficiency** | CloudFront, ElastiCache, Kinesis, Lambda, DynamoDB, EMR |
| **Cost Optimization** | Cost Explorer, Budgets, Trusted Advisor, Compute Optimizer, S3 Intelligent-Tiering |
| **Sustainability** | Graviton, Serverless, Compute Optimizer, |

---

## Pilares — Design Principles Resumidos

### Operational Excellence
- Perform operations as code (IaC)
- Make frequent, small, reversible changes
- Refine operations procedures frequently
- Learn from all operational failures

### Security
- Implement a strong identity foundation
- Enable traceability
- Apply security at all layers
- Protect data in transit and at rest
- Keep people away from data

### Reliability
- Test recovery procedures regularly
- Scale horizontally
- Stop guessing capacity
- Manage change through automation
- Design for failure

### Performance Efficiency
- Democratize advanced technologies
- Go global in minutes
- Use serverless architectures
- Experiment more often
- Consider mechanical sympathy

### Cost Optimization
- Implement cloud financial management
- Adopt a consumption model
- Measure overall efficiency
- Stop spending on undifferentiated heavy lifting
- Analyze and attribute expenditure

### Sustainability
- Understand your impact
- Establish sustainability goals
- Maximize utilization
- Use managed services
- Reduce downstream impact

---

## Well-Architected Tool — Processo

```
1. CRIAR Workload → nome, região, tipo
2. RESPONDER perguntas por pilar (cada pilar: ~10-20 perguntasS)
3. VER resultados: High Risk Issues (HRI) e Medium Risk Issues (MRI)
4. CRIAR Improvement Plan → priorizar e atribuir items
5. REVISAR periodicamente (a cada 6-12 meses ou após mudanças grandes)
```

---

## Dicas de Prova — Well-Architected

| Pista no Enunciado | Pilar Relevante | Resposta Típica |
|---|---|---|
| "Aplicação sem downtime durante deploys" | Reliability | Blue/Green Deploy, Rolling Update |
| "Reduzir custo de EC2 sempre ligado" | Cost Optimization | Reserved ou Savings Plans |
| "Ambiente de dev ligado 8h por dia" | Cost Optimization | Scheduled On/Off via Lambda+EventBridge |
| "Acesso a dados de auditoria" | Security | CloudTrail + S3 + Object Lock |
| "Detectar e corrigir drift de configuração" | Operational Excellence | AWS Config + SSM Remediation |
| "Instância parou de responder, recuperar mesmo IP" | Reliability | EC2 Recovery Alarm (StatusCheckFailed_System) |
| "Menor carbono na arquitetura" | Sustainability | Graviton, Lambda, gerenciar scheduling |
| "Revisar arquitetura contra boas práticas" | Todos os pilares | AWS Well-Architected Tool (WAT) |
| "RTO < 1 minuto multi-região" | Reliability | Active-Active + Aurora Global Database |
| "RTO de horas é aceitável para DR" | Reliability | Backup & Restore strategy |
