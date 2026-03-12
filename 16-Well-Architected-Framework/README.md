# Módulo 16 — AWS Well-Architected Framework

## Os 6 Pilares

```
1. Operational Excellence    → Operar e melhorar sistemas continuamente
2. Security                  → Proteger dados, sistemas e ativos
3. Reliability               → Recuperar de falhas, atender demanda
4. Performance Efficiency    → Usar recursos computacionais eficientemente
5. Cost Optimization         → Evitar custos desnecessários
6. Sustainability            → Minimizar impacto ambiental
```

---

## 1. Excelência Operacional (Operational Excellence)

**Objetivo**: Executar e monitorar sistemas para entregar valor e melhorar continuamente.

### Princípios de Design
- **Realizar operações como código** (IaC: CloudFormation, CDK, Terraform)
- **Fazer mudanças frequentes, pequenas e reversíveis** (deploy granular, feature flags)
- **Refinar procedimentos operacionais frequentemente** (runbooks, playbooks atualizados)
- **Antecipar falhas** (game days, chaos engineering)
- **Aprender com todas as falhas operacionais** (post-mortems blameless)

### Serviços Chave
- AWS CloudFormation, AWS CDK (IaC)
- AWS Config (conformidade de configuração)
- AWS CloudTrail (auditoria)
- Amazon CloudWatch (monitoramento)
- AWS Systems Manager (operações centralizadas)
- AWS X-Ray (observabilidade)

---

## 2. Segurança (Security)

**Objetivo**: Proteger informações, sistemas e ativos ao mesmo tempo em que entrega valor.

### Princípios de Design
- **Implementar uma base de identidade sólida** (MFA, least privilege, IAM roles)
- **Habilitar rastreabilidade** (CloudTrail, Config, logging de acesso)
- **Aplicar segurança em todas as camadas** (VPC, SG, NACL, WAF, CDN)
- **Automatizar boas práticas de segurança** (Security Hub, GuardDuty, Macie)
- **Proteger dados em trânsito e em repouso** (TLS, KMS)
- **Manter as pessoas longe dos dados** (acesso programático, no human touch)
- **Preparar para eventos de segurança** (runbooks de resposta a incidentes, Security Hub)

### Serviços Chave
- **Identity**: IAM, AWS SSO/IAM Identity Center, Organizations, SCPs
- **Detection**: GuardDuty, Macie, SecurityHub, Inspector, Config
- **Infrastructure Security**: VPC, SG, WAF, Shield, Network Firewall, Firewall Manager
- **Data Protection**: KMS, CloudHSM, ACM, Secrets Manager
- **Incident Response**: CloudTrail, AWS IR Playbooks

---

## 3. Confiabilidade (Reliability)

**Objetivo**: Recuperar de interrupções de infraestrutura ou serviço e escalar para atender demanda.

### Princípios de Design
- **Recuperar automaticamente de falhas** (Auto Scaling, Health Checks, CloudWatch Alarms)
- **Testar procedimentos de recuperação** (Game Days, DR exercises)
- **Escalar horizontalmente** para aumentar disponibilidade (muitas instâncias pequenas)
- **Parar de adivinhar capacidade** (Auto Scaling + CloudWatch)
- **Gerenciar mudanças por automação** (CloudFormation, CodePipeline)

### Metas de Disponibilidade vs Arquitetura

| Disponibilidade | Downtime/ano | Arquitetura Mínima |
|---|---|---|
| 99% | ~3,65 dias | Single AZ backup + restore |
| 99,9% | ~8,7 horas | Multi-AZ, warm standby |
| 99,99% | ~52 minutos | Multi-AZ + Multi-Region active-active ou active-passive |
| 99,999% | ~5 minutos | Multi-Region active-active |

### RTO e RPO
- **RPO** (Recovery Point Objective): quanto de dados posso perder? → frequência de backup
- **RTO** (Recovery Time Objective): quanto tempo posso ficar offline? → estratégia de DR

### Estratégias de DR

```
Custo        Estratégia              RTO/RPO
Menor ──────── Backup & Restore ─────── Horas/Horas
       │        Pilot Light ────────── 10-15 min / Min
       │        Warm Standby ─────────  Min / Segundos
Maior ────────  Multi-site Active-Active  Segundos / Zero
```

---

## 4. Eficiência de Performance (Performance Efficiency)

**Objetivo**: Usar recursos computacionais eficientemente para atender requisitos do sistema.

### Princípios de Design
- **Democratizar tecnologias avançadas** (usar serviços gerenciados em vez de gerenciar infra)
- **Ser global em minutos** (CloudFront, Global Accelerator, multi-region)  
- **Usar arquitetura without servers** (Lambda, Fargate, DynamoDB)
- **Experimentar com mais frequência** (A/B testing, blue/green)
- **Usar a tecnologia certa para cada tarea** (não usar solução genérica)

### Tipos de Recursos
- **Compute**: EC2 (tipos certos), Lambda (serverless), Fargate, Graviton2/3 (ARM, melhor custo/performance)
- **Storage**: EBS gp3 (IOPS independente), S3 (S3 Intelligent-Tiering para acesso variável)
- **Database**: RDS (relacional), DynamoDB (NoSQL chave-valor), ElastiCache (cache), Aurora (MySQL/PostgreSQL gerenciado)
- **Network**: CloudFront (CDN), Global Accelerator, VPC Endpoints, placement groups

---

## 5. Otimização de Custos (Cost Optimization)

**Objetivo**: Evitar custos desnecessários.

### Princípios de Design
- **Implementar Cloud Financial Management** (tagging, budgets, Cost Explorer)
- **Adotar um modelo de consumo** (pague pelo que usa; desligar ambientes não-prod)
- **Medir eficiência global** (custo por unidade de trabalho)
- **Parar de gastar em trabalho indiferenciado** (usar serviços gerenciados)
- **Analisar e atribuir despesas** (cost allocation tags, AWS Cost Allocation Tags)

### Modelos de Compra EC2
| Modelo | Desconto | Compromisso |
|---|---|---|
| On-Demand | 0% (base) | Nenhum |
| Reserved (1 ano) | ~40% | 1 ano |
| Reserved (3 anos) | ~60% | 3 anos |
| Savings Plans Compute | até 66% | 1 ou 3 anos ($$/hora) |
| Savings Plans EC2 | até 72% | 1 ou 3 anos (família/região) |
| Spot | até 90% | Interruptível |

**Ferramentas de Custo:**
- **AWS Cost Explorer**: análise e previsão de gastos
- **AWS Budgets**: alertas quando gastar além do threshold
- **Cost Allocation Tags**: rastreia custos por projeto/time/ambiente
- **Compute Optimizer**: recomenda tipo de instância ideal (rightsizing)
- **Trusted Advisor**: identificar recursos subutilizados

---

## 6. Sustentabilidade (Sustainability)

**Objetivo**: Minimizar o impacto ambiental das cargas de trabalho na nuvem.

### Princípios de Design
- **Entender seu impacto** (Customer Carbon Footprint Tool)
- **Estabelecer metas de sustentabilidade** (redução de emissão de CO2)
- **Maximizar utilização** (right-sizing, evitar recursos ociosos)
- **Antecipar e adotar hardware mais eficiente** (Graviton3, instâncias de última geração)
- **Usar serviços gerenciados** (compartilham infraestrutura com outros clientes)
- **Reduzir o impacto downstream** (compressão, caching, menos transferências)

---

## AWS Well-Architected Tool (WAT)

Ferramenta gratuita para revisar workloads contra os 6 pilares:
1. Define o workload (tecnologias, ambiente)
2. Responde perguntas sobre cada pilar
3. WAT gera relatório de riscos (High/Medium Risk Items)
4. Cria Improvement Plan
5. Rastreia resolução de riscos ao longo do tempo

**AWS Well-Architected Partner Program**: parceiros certificados que conduzem revisões formais

---

## Dicas de Prova

- Perguntas WAF são frequentes no SAA-C03 — memorize os 6 pilares e princípios chave
- **Reliability ≠ Availability**: Reliability = capacidade de se recuperar de falhas; Availability = % uptime
- **Pilot Light**: componentes críticos mínimos sempre rodando na AWS (banco de dados replicado); outros suportados em AMIs prontas
- **Warm Standby**: versão menor escalonável rodando na AWS; após DR, escala para capacidade full
- Multi-site Active-Active: custo mais alto, mas RTO/RPO próximo de zero
- WAT não cobra por uso; parceiros Well-Architected podem cobrar pelo serviço de consultoria
- Sustainability: Graviton3, Serverless (Lambda/Fargate) e serviços gerenciados são padrões sustentáveis
- Para Cost Optimization: Compute Optimizer = rightsizing; Cost Explorer = análise no tempo; Budgets = alertas proativos
- Security "shift left": aplicar segurança no início do ciclo de desenvolvimento, não apenas antes do deploy
- "Least privilege" é um princípio de segurança: cada IAM entity tem apenas as permissões mínimas necessárias
