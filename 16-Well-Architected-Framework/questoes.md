# Questões — Módulo 16: Well-Architected Framework

> **Domínio SAA-C03**: Todos os domínios (framework transversal)  
> **Dificuldade**: Média-Alta

---

**1.** Uma empresa quer implementar os princípios do pilar de Confiabilidade do Well-Architected Framework. Qual das seguintes ações NÃO é um princípio desse pilar?

- A) Escalar horizontalmente para aumentar disponibilidade do sistema
- B) Recuperar automaticamente de falhas
- C) Minimizar o impacto ambiental das workloads
- D) Parar de adivinhar capacidade usando Auto Scaling

<details><summary>Resposta</summary>

**C** — Minimizar impacto ambiental pertence ao pilar de **Sustentabilidade** (6° pilar). Os princípios do pilar Confiabilidade são: recuperar automaticamente de falhas, testar procedimentos de recuperação, escalar horizontalmente, parar de adivinhar capacidade e gerenciar mudanças por automação.

</details>

---

**2.** Uma empresa tem uma aplicação crítica com RPO de 1 hora e RTO de 30 minutos. Qual estratégia de DR é a mais custo-efetiva para esses requisitos?

- A) Backup & Restore (snapshots a cada hora)
- B) Pilot Light com componentes críticos rodando em segunda região
- C) Warm Standby com capacidade reduzida em segunda região
- D) Multi-site Active-Active

<details><summary>Resposta</summary>

**B** — Pilot Light: mantém o BD replicado (via Aurora Global DB ou DMS CDC → RPO ~segundos) e AMIs prontas na segunda região. Em caso de desastre, em ~10-15 minutos é possível provisionar os componentes e redirecionar o DNS (Route 53 failover). RTO 30 min e RPO 1h são alcançáveis com custo menor que Warm Standby.

</details>

---

**3.** Uma empresa está revisando seu workload com o AWS Well-Architected Tool e identificou que não possui logging de todas as API calls. Sob qual pilar esse risco se enquadra?

- A) Performance Efficiency
- B) Cost Optimization
- C) Security
- D) Operational Excellence

<details><summary>Resposta</summary>

**C** — O pilar de Segurança exige "habilitar rastreabilidade" — CloudTrail para APIs, CloudWatch Logs, Config para configurações. Ausência de logging é um High Risk Item de segurança (impossibilita detecção e investigação de incidentes).

</details>

---

**4.** Uma startup usa EC2 On-Demand para executar suas cargas de trabalho de produção que ficam online 24/7 há 18 meses. O CTO quer reduzir custos imediatamente. Qual é a principal recomendação do pilar de Cost Optimization?

- A) Comprar Reserved Instances ou Savings Plans de 1 ano para a carga de trabalho estável
- B) Migrar tudo para Lambda (serverless)
- C) Mover para instâncias Spot
- D) Reduzir o número de Availability Zones

<details><summary>Resposta</summary>

**A** — Workloads 24/7 estáveis são ideais para Reserved Instances ou Savings Plans. RI/SP de 1 ano oferecem ~40% de desconto vs On-Demand. Spot (C) é interruptível (inadequado para produção 24/7 sem tolerância a falhas). Lambda nem sempre é mais barato para workloads de longa duração.

</details>

---

**5.** Uma empresa está desenvolvendo uma nova aplicação e quer aplicar o princípio de "implementar uma base de identidade forte" do pilar de Segurança. Quais práticas representam isso? (Selecione 2)

- A) Usar IAM roles para todas as aplicações (sem access keys hardcoded)
- B) Habilitar MFA para usuários humanos que acessam o Console AWS
- C) Configurar CloudFront para todos os endpoints
- D) Usar instâncias com maior capacidade de memória

<details><summary>Resposta</summary>

**A e B** — IAM roles (sem credenciais de longa duração) e MFA são os fundamentos de "base de identidade forte" do pilar Security. CloudFront é Performance Efficiency/Reliability. Memória maior é Performance Efficiency.

</details>

---

**6.** Uma empresa sofreu um incidente de segurança. Após a investigação, perceberam que não tinham logs suficientes para identificar a causa raiz. Qual princípio do Well-Architected Framework foi negligenciado e qual pilar?

- A) "Automatizar boas práticas de segurança" — Pilar: Security
- B) "Habilitar rastreabilidade" — Pilar: Security
- C) "Refinar procedimentos operacionais frequentemente" — Pilar: Operational Excellence
- D) "Antecipar falhas" — Pilar: Reliability

<details><summary>Resposta</summary>

**B** — "Habilitar rastreabilidade" é um design principle do pilar Security. Rastreabilidade = logs de CloudTrail (APIs), VPC Flow Logs (rede), CloudWatch Logs (aplicação), Config (configuração). Sem esses logs, investigação forense é impossível.

</details>

---

**7.** Uma empresa tem um ambiente de desenvolvimento que funciona 8 horas por dia, 5 dias por semana. Atualmente usa instâncias On-Demand e gasta $500/mês. O que o pilar de Cost Optimization recomenda?

- A) Converter para Reserved Instances de 1 ano
- B) Usar Spot Instances para desenvolvimento
- C) Criar automação (Lambda + EventBridge) para ligar/desligar as instâncias fora do horário de uso
- D) Migrar o ambiente de desenvolvimento para containers no Fargate

<details><summary>Resposta</summary>

**C** — Ambientes não-produção devem rodar apenas quando necessários. Ligar/desligar (8h × 5 dias = 40h/semana) vs 168h/semana On-Demand → ~76% de redução de custo. EventBridge Rules (schedule) → Lambda (start/stop EC2). Cost Optimization: adote modelo de consumo (pague pelo que usa).

</details>

---

**8.** Uma empresa usa um servidor de banco de dados com especificação muito alta (64 vCPUs) mas o uso médio de CPU é de apenas 5%. O pilar de Performance Efficiency e Cost Optimization recomendam o quê?

- A) Manter a especificação alta para garantir headroom
- B) Usar o AWS Compute Optimizer para identificar o tipo de instância RDS adequado (rightsizing)
- C) Migrar para Lambda
- D) Adicionar Read Replicas para distribuir a carga

<details><summary>Resposta</summary>

**B** — Compute Optimizer analisa métricas históricas (CPU, memória — com CloudWatch agent, rede) e recomenda o tipo de instância ideal. Over-provisioned = custo desnecessário sem benefício de performance. Rightsizing é um princípio central de Cost Optimization.

</details>

---

**9.** Uma empresa percebe que seu processo de deploy manual (SSH + rsync de código) é lento e propenso a erros, causando incidentes frequentes. Qual pilar do Well-Architected Framework endereça isso e qual tecnologia AWS?

- A) Reliability — usar Multi-AZ para evitar downtime durante deploys
- B) Operational Excellence — implementar CI/CD com CodePipeline + CodeDeploy (realizar operações como código)
- C) Performance Efficiency — usar Elastic Beanstalk para deploy automático
- D) Security — usar IAM para controlar quem pode fazer deploy

<details><summary>Resposta</summary>

**B** — Operational Excellence: "realizar operações como código" e "fazer mudanças frequentes, pequenas e reversíveis". CI/CD automatiza deploys, reduz erros humanos e permite rollback rápido. CodePipeline + CodeDeploy implementa esse princípio diretamente.

</details>

---

**10.** Uma aplicação global precisa servir conteúdo com latência mínima para usuários em todo o mundo sem manter múltiplos clusters de aplicação em cada região. Qual princípio do pilar de Performance Efficiency é aplicado?

- A) Usar a tecnologia certa para cada tarefa
- B) Ser global em minutos usando CloudFront, Global Accelerator e S3
- C) Parar de adivinhar capacidade
- D) Democratizar tecnologias avançadas

<details><summary>Resposta</summary>

**B** — "Ser global em minutos": CloudFront distribui conteúdo para 400+ edge locations sem precisar de infraestrutura em cada região. Global Accelerator roteia tráfego pela rede privada AWS para reduzir latência. Performance sem escalar a aplicação horizontalmente por região.

</details>

---

**11.** Uma empresa quer medir se suas workloads na AWS estão alinhadas com os princípios do Well-Architected Framework de forma formal e documentada. Qual é o processo correto?

- A) Comparar manualmente a arquitetura com os whitepapers
- B) Usar o AWS Well-Architected Tool (WAT) para realizar uma revisão formal por workload
- C) Contratar AWS Professional Services para uma auditoria
- D) Usar Trusted Advisor para verificar as 5 categorias

<details><summary>Resposta</summary>

**B** — AWS Well-Architected Tool: define o workload, responde perguntas sobre cada pilar, gera relatório de riscos (HRI e MRI — High/Medium Risk Items), cria Improvement Plan e rastreia progresso. Gratuito e pode ser realizado pelo próprio time de arquitetura.

</details>

---

**12.** Do ponto de vista do pilar de Sustentabilidade, qual das seguintes arquiteturas tem o menor impacto ambiental para uma carga de trabalho intermitente?

- A) EC2 On-Demand sempre ligado (24/7) com alta utilização
- B) AWS Lambda (serverless) para cargas intermitentes e Fargate para containers
- C) EC2 Reserved Instances para garantir disponibilidade
- D) EC2 com Auto Scaling com mínimo de 5 instâncias

<details><summary>Resposta</summary>

**B** — Sustentabilidade: maximizar utilização de recursos. Lambda e Fargate cobram apenas quando executando (zero consumo idle). Menos hardware sendo usado = menor pegada de carbono. Serverless alinha recursos precisamente com a demanda, eliminando recursos ociosos.

</details>

---

**13.** Uma empresa tem diferentes ambientes (dev, staging, prod) e quer implementar o princípio de "usar IaC" do pilar Operational Excellence. Qual benefício direto isso traz?

- A) Ambientes dev/staging/prod ficam idênticos, eliminando bugs de "funciona na minha máquina"
- B) Reduz o custo de EC2 automaticamente
- C) Aumenta a segurança das instâncias EC2
- D) Melhora o desempenho de queries RDS

<details><summary>Resposta</summary>

**A** — IaC (CloudFormation, CDK, Terraform) permite criar ambientes idênticos de forma reproduzível. O mesmo template provisiona dev, staging e prod com as mesmas configurações, eliminando deriva de configuração (configuration drift) e o famoso problema de "funciona no dev mas não no prod".

</details>

---

**14.** O time de segurança quer garantir que nenhuma porta SSH (22) esteja aberta para 0.0.0.0/0 em qualquer Security Group da empresa. Qual é a abordagem preventiva mais robusta do pilar Security?

- A) Auditar os SGs manualmente toda semana
- B) Config Rule `restricted-ssh` + Remediation automática que remove a regra infratora
- C) CloudTrail alert quando alguém adiciona regra no SG
- D) IAM Permission negando `AuthorizeSecurityGroupIngress` para todos os usuários

<details><summary>Resposta</summary>

**B** — Config Rule detecta continuamente SGs com porta 22 aberta para 0.0.0.0/0. Com Remediation configurada, o Config executa uma SSM Automation que remove a regra infratora automaticamente (ou notifica para ação manual). Automates security best practices — princípio do pilar Security.

</details>

---

**15.** Uma empresa está avaliando o custo total de operação (TCO) para uma carga de trabalho que consome 1.000 horas de EC2 m5.large por mês, consistentemente há 2 anos. Qual modelo de compra o pilar Cost Optimization recomenda?

- A) On-Demand (sem compromisso, máxima flexibilidade)
- B) Spot Instances (máximo desconto)
- C) Compute Savings Plans de 1 ano ($X por hora comprometido)
- D) Reserved Instances Convertible de 3 anos (máximo desconto)

<details><summary>Resposta</summary>

**C** — Compute Savings Plans de 1 ano: ~66% de desconto vs On-Demand, mais flexível que RI (pode mudar instância type, OS, região dentro do Compute SP). Para carga previsível há 2 anos, algum compromisso é claramente o caminho. 3 anos (D) dá mais desconto mas menos flexibilidade; 1 ano é o ponto de equilíbrio typical recomendado.

</details>
