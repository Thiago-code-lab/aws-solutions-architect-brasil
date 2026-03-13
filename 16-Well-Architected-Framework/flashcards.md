# Flashcards — Módulo 16: Well-Architected Framework

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** Quais são os 6 pilares do AWS Well-Architected Framework?  
**R:** **(1) Operational Excellence** — operar e monitorar sistemas. **(2) Security** — proteger dados e sistemas. **(3) Reliability** — recuperar de falhas, escalar. **(4) Performance Efficiency** — usar recursos eficientemente. **(5) Cost Optimization** — evitar custos desnecessários. **(6) Sustainability** — minimizar impacto ambiental.

---

**P:** Quais são as 4 estratégias de Disaster Recovery em ordem de menor para maior custo/capacidade?  
**R:** **(1) Backup & Restore** (RPO/RTO: horas) → **(2) Pilot Light** (core mínimo sempre ligado, RPO/RTO: dezenas de min) → **(3) Warm Standby** (versão reduzida sempre ativa, RTO: minutos) → **(4) Active-Active Multi-Site** (tráfego em múltiplas regiões simultaneamente, RTO: segundos/zero).

---

**P:** O que é RPO vs RTO e como afetam a escolha de estratégia DR?  
**R:** **RPO** (Recovery Point Objective): quanto de dados posso perder. **RTO** (Recovery Time Objective): quanto tempo posso ficar indisponível. Menor RPO/RTO = estratégia mais cara: Backup&Restore (RPO/RTO horas) → Pilot Light → Warm Standby → Active-Active (RPO/RTO ≈ 0, mais caro).

---

**P:** O que é o pilar Operational Excellence e seus princípios de design?  
**R:** Foco em operar e melhorar processos continuamente. Princípios: **Perform operations as code** (IaC), **Annotate documentation** (auto-gerada), **Make frequent, small, reversible changes** (sem big-bang deployments), **Refine operations procedures frequently** (Game Days), **Anticipate failure** (pre-mortem), **Learn from failures** (blameless post-mortems).

---

**P:** O que é o pilar Security e seu princípio de "base of identity"?  
**R:** **Identity is the foundation of security.** Tudo começa com "quem pode fazer o quê". Princípios: Strong identity foundation (IAM, Organizations SCP), Enable traceability (CloudTrail, CloudWatch), Apply security at all layers (SG, WAF, KMS, encryption), Protect data in transit and at rest, Keep people away from data, Prepare for security events.

---

**P:** O que é o pilar Reliability e como Auto Scaling contribui?  
**R:** Reliability = capacidade de se recuperar de falhas e crescer. Auto Scaling contribui via: escalar horizontalmente (mais instâncias, não maiores), recuperar automaticamente de falhas (substituir instâncias unhealthy), distribuir em múltiplas AZs. Princípios: Test recovery procedures, Stop guessing capacity, Manage change with automation.

---

**P:** O que é o pilar Performance Efficiency?  
**R:** Usar recursos de computação eficientemente. Princípios: **Democratize advanced technologies** (usar serviços gerenciados em vez de construir), **Go global in minutes** (regiões, CloudFront), **Use serverless architectures** (sem gerenciar servers), **Experiment more often** (facilidade de testar tipos de instâncias), **Consider mechanical sympathy** (usar o serviço correto para o workload).

---

**P:** Quais são os princípios do pilar Cost Optimization?  
**R:** Evitar gastos desnecessários. Princípios: **Implement cloud financial management** (processo organizacional), **Adopt a consumption model** (pagar pelo que usar, desligar ocioso), **Measure overall efficiency** (output por custo), **Stop spending on undifferentiated heavy lifting** (usar serviços gerenciados), **Analyze and attribute expenditure** (tags, cost allocation).

---

**P:** O que é o pilar Sustainability e como reduz impacto ambiental?  
**R:** Minimizar impacto ambiental de workloads. Práticas: usar regiões com menor carbon footprint, otimizar utilização (instâncias não ociosas), usar serviços gerenciados (AWS otimiza hardware compartilhado), serverless (recursos sob demanda), scaling down em períodos de baixo uso, modernizar hardware via serviços novos (Graviton — melhor eficiência energética).

---

**P:** O que é o AWS Well-Architected Tool (WAT)?  
**R:** Ferramenta no console AWS para revisar workloads contra os 6 pilares. Processo: criar review → responder perguntas por pilar → receber "High Risk Issues" (HRI) e "Medium Risk Issues" (MRI) com recomendações. Cria plano de melhoria (improvement plan). Gratuito. Pode ser criado por parceiros AWS via Well-Architected Partner Program.

---

**P:** O que significa "Pilot Light" na estratégia de DR?  
**R:** Manter o **core mínimo** da aplicação sempre rodando na região DR. O core: banco de dados (replicado) e configurações básicas. Em caso de desastre: "acender o piloto" = escalar os componentes restantes (EC2, ELB) na região DR. RTO: minutos a dezenas de minutos. Custo menor que Warm Standby (só o essencial roda continuamente).

---

**P:** O que é "Warm Standby" e como difere de "Pilot Light"?  
**R:** **Warm Standby:** versão completa (mas reduzida/sub-capacidade) da aplicação rodando na região DR. **Pilot Light:** apenas o core mínimo. Diferença: Warm Standby tem todos os componentes prontos (EC2 pequenos, RDS réplica), apenas escala no failover. Pilot Light precisa provisionar EC2 etc no failover. Warm Standby = RTO menor, custo maior.

---

**P:** Qual é a tabela de disponibilidade para 99%, 99.9%, 99.99%, 99.999%?  
**R:** **99%** = ~87,6h downtime/ano (~7,3h/mês). **99.9%** = ~8,7h/ano (~43,8 min/mês). **99.99%** = ~52 min/ano (~4,4 min/mês). **99.999% (Five Nines)** = ~5,26 min/ano (~26 seg/mês). Five Nines requer arquitetura ativa-ativa com zero tolerância a falha única.

---

**P:** Como comparar On-Demand vs Reserved vs Savings Plans no contexto de Cost Optimization?  
**R:** **On-Demand:** máxima flexibilidade, máximo preço. Use para: cargas imprevisíveis, picos. **Reserved (1 ou 3 anos):** até 72% desconto. Use para: carga estável e previsível 24/7. **Savings Plans:** compromisso de $ por hora (Compute ou EC2/Fargate/Lambda). Mais flexível que Reserved. **Spot:** até 90% off, interruptível. Use para: batch, tolerante a falhas.

---

**P:** O que é o Well-Architected Framework Lens?  
**R:** Extensão do WAF para domains específicos. Disponíveis: **Serverless Lens**, **SaaS Lens**, **Machine Learning Lens**, **Data Analytics Lens**, **Games Industry Lens**, **SAP Lens**, **High Performance Computing Lens** etc. Cada Lens tem perguntas específicas e boas práticas para aquele domínio além dos 6 pilares base.

---

**P:** Qual princípio do Well-Architected diz "design for failure"?  
**R:** Pilar **Reliability**: "Test recovery procedures" e "Design for failure". AWS: design assumindo que componentes VÃO falhar. Tudo deve ter redundância. Evitar SPOFs (Single Points of Failure). Multi-AZ, Auto Scaling, circuit breakers, dead letter queues — arquitetura resiliente assume falhas como esperadas, não excepcionais.

---

**P:** Como o princípio "Infrastructure as Code" se encaixa no Well-Architected?  
**R:** Pilar **Operational Excellence**: "Perform operations as code". IaC (CloudFormation, Terraform, CDK): (1) reprodutível — mesmo estado toda vez; (2) auditável — mudanças em controle de versão; (3) reversível — rollback para versão anterior; (4) testável — validar antes de produção. Elimina configuração manual inconsistente.

---

**P:** O que é o princípio "Loose Coupling" no Well-Architected?  
**R:** Pilar **Reliability**: componentes com dependências fracas falham de forma isolada. Implementação: SQS entre serviços (mensagens bufferizadas se downstream falhar), SNS para fan-out, API Gateway desacopla clientes de backends, ELB desacopla clientes de instâncias. Contrária: tight coupling (serviços chamam diretamente → falha em um cascadeia).

---

**P:** O que são Game Days no contexto de Operational Excellence?  
**R:** Simulações planejadas de falhas e incidentes para testar procedimentos de resposta. "Pre-mortems": imaginar que o sistema falhou e identificar causas. Objetivo: descobrir vulnerabilidades e gaps nos runbooks antes que aconteçam em produção. Cultura de aprendizado por simulação. Relacionado ao Chaos Engineering (Netflix Chaos Monkey).

---

**P:** Qual a diferença entre "vertical scaling" e "horizontal scaling" no Well-Architected?  
**R:** **Vertical (Scale Up):** aumentar o tamanho da instância (m5.large → m5.xlarge). Limite físico. Requer downtime (geralmente). SPOF. **Horizontal (Scale Out):** adicionar mais instâncias do mesmo tipo. Sem limite prático. Sem downtime (gradual). Elimina SPOF. Well-Architected favorece horizontal: "Scale horizontally to increase aggregate system availability."

---

**P:** O que é o "consumption model" no pilar Cost Optimization?  
**R:** Pagar apenas pelo que usar — contrário do modelo on-premises (pagar por capacidade pico, mesmo quando ociosa). AWS: escalar para baixo quando demanda cai. Desligar ambientes de dev à noite/fim de semana. Lambda/Fargate/DynamoDB On-Demand: pagam apenas por execução real. Meta: custo proporcional ao uso real, não ao pico teórico.

---

**P:** Quais são os 3 tipos de controles de segurança segundo o Well-Architected?  
**R:** **(1) Preventive:** impede problemas antes de acontecerem (IAM policies, SCP, SGs, encryption). **(2) Detective:** detecta quando algo acontece (CloudTrail, Config Rules, GuardDuty, Security Hub). **(3) Responsive:** responde após detecção (runbooks, automação SNS/Lambda, incident response plans). Arquitetura segura precisa dos 3 tipos.

---

**P:** O que é "blast radius reduction" no pilar Security?  
**R:** Limitar o impacto potencial de uma falha de segurança. Técnicas: **Least privilege** (conta comprometida tem acesso mínimo), **Accounts separation** (AWS Organizations — workloads em contas separadas, falha em uma não afeta outras), **Network segmentation** (VPCs separadas, subnets privadas), **Encryption per-resource** (chave comprometida afeta apenas dados daquela chave).

---

**P:** Como o Sustainability pilar se aplica praticamente na AWS?  
**R:** Práticas: **(1)** Escolher região com fator de emissão de carbono menor. **(2)** Usar instâncias Graviton (ARM — melhor performance/watt). **(3)** Otimizar utilização: evitar over-provisioning (Compute Optimizer). **(4)** Serverless: recursos liberados quando não usados. **(5)** S3 Lifecycle: mover dados frios para Glacier (menos energia). **(6)** Eliminar workloads desnecessários.

---

**P:** O que são Shared Responsibility Areas no Well-Architected?  
**R:** Well-Architected reconhece que algumas responsabilidades são compartilhadas entre equipes: Dev, Ops, Security e Cloud Center of Excellence (CCoE). Revisões WAT idealmente envolvem: arquiteto de soluções + engenheiro de segurança + representante de negócio. Não é responsabilidade apenas de ops ou apenas de dev. Cultura de arquitetura compartilhada.
