# Questões — Módulo 19: Simulados e Questões

> Estratégia de estudo para a prova

---

**Sobre as questões do simulado**

O README deste módulo contém o simulado completo com 65 questões. Este arquivo contém questões adicionais de estratégia de prova e meta-perguntas sobre o formato do exame.

---

**1.** No exame SAA-C03, uma questão pede para escolher a solução "mais custo-efetiva". Qual abordagem de eliminação usar?

- A) Escolher sempre a opção com Lambda (serverless é sempre mais barato)
- B) Eliminar opções com recursos ociosos (EC2 sempre ligado, instâncias sobre-provisionadas), depois comparar custo das restantes
- C) Escolher sempre Reserved Instances (maior desconto)
- D) Escolher a opção com menos serviços AWS (menos serviços = menos custo)

<details><summary>Resposta</summary>

**B** — "Mais custo-efetiva" varia pelo contexto: para cargas intermitentes, Lambda/Fargate/Spot são mais baratos. Para cargas 24/7 estáveis, Reserved Instances. Elimine primeiro: (1) over-provisioning, (2) recursos idle, (3) redundância desnecessária. Então compare os serviços restantes pelo modelo de precificação.

</details>

---

**2.** Uma questão do exame pede para escolher 2 respostas corretas de 5 opções. Qual é a melhor estratégia?

- A) Marcar as 2 primeiras opções que parecem corretas
- B) Identificar as 2 incorretas mais óbvias, eliminar, e das 3 restantes escolher as 2 mais corretas
- C) Marcar como incerto e voltar depois
- D) Procurar por palavras-chave como "sem servidor" ou "gerenciado" nas opções

<details><summary>Resposta</summary>

**B** — Estratégia de eliminação: remova primeiro as opções claramente erradas (serviço errado para o contexto, viola princípio de segurança óbvio). Das restantes, escolha as 2 que melhor atendem TODOS os requisitos da questão. Em questões de múltipla seleção, todas as opções corretas devem ser escolhidas para pontuar.

</details>

---

**3.** O exame pede uma arquitetura "altamente disponível". Qual é o critério mínimo que a resposta deve atender?

- A) Usar o serviço mais novo da AWS
- B) Ter recursos em pelo menos 2 Availability Zones com failover automático
- C) Usar Reserved Instances em 3 regiões
- D) Ter backup diário dos dados

<details><summary>Resposta</summary>

**B** — Alta disponibilidade = tolerância a falha de uma AZ. Mínimo: recursos em 2+ AZs com failover automático (ELB + ASG multi-AZ, RDS Multi-AZ, etc.). Backup é para recovery (não HA). Multi-region é para disaster recovery ou alcance global (diferente de HA simples).

</details>

---

**4.** Uma questão descreve uma aplicação que "às vezes tem picos de tráfego imprevisíveis" e precisa de uma solução de compute. Qual serviço é a resposta mais provável?

- A) EC2 On-Demand fixo
- B) EC2 Reserved Instances
- C) Lambda ou ECS Fargate com Auto Scaling
- D) EC2 Spot Instances

<details><summary>Resposta</summary>

**C** — "Imprevisível" e "picos" = serverless/auto-scaling. Lambda escala para zero quando não há carga e para milhares quando há pico, sem configuração manual. Fargate com ASG baseado em métricas também. Reserved Instances são para carga previsível; Spot pode ser interrompida durante picos.

</details>

---

**5.** No exame, quando a questão menciona "menor mudança na aplicação existente" (lift-and-shift), qual categoria de resposta é mais provável?

- A) Lambda e DynamoDB (serverless nativo)
- B) EC2 + RDS (equivalente de VM + banco relacional)
- C) EKS + S3 (containers + armazenamento)
- D) CloudFormation + CodePipeline (automação)

<details><summary>Resposta</summary>

**B** — "Menor mudança possível" = lift-and-shift = EC2 (VM equivalente), RDS (banco gerenciado, mesma engine), ELB (load balancer). A aplicação não precisa ser reescrita. Lambda/DynamoDB requerem refactoring. EKS requer containerização. A questão está testando reconhecer o padrão "Rehost" dos 7Rs.

</details>

---

**6.** Qual é a dica mais importante ao ler uma questão do exame SAA-C03?

- A) Ler a última frase da questão primeiro para entender o que está sendo perguntado
- B) Identificar todos os requisitos (disponibilidade, custo, segurança, performance) explicitamente mencionados antes de olhar as opções
- C) Eliminar as opções mais longas (geralmente complexidade desnecessária)
- D) Sempre escolher a opção com o serviço mais gerenciado

<details><summary>Resposta</summary>

**B** — Leia o cenário completo e anote os requisitos explícitos (ex: "RTO < 1 hora", "sem código customizado", "menor custo"). Cada requisito pode eliminar opções. Algumas questões têm distractors que atendem parte dos requisitos mas violam outros. Identificar TODOS os requisitos antes de avaliar as respostas é a estratégia mais efetiva.

</details>

---

**7.** No dia do exame, quanto tempo dedicar para a primeira passagem pelas 65 questões?

- A) Resolver todas as questões meticulosamente na primeira passagem (sem pular)
- B) ~1,5 minuto por questão na primeira passagem (~97 min); marcar questões difíceis; usar tempo restante para revisar
- C) Pular as primeiras 20 questões (geralmente mais difíceis)
- D) Responder as mais fáceis primeiro (últimas 30 questões)

<details><summary>Resposta</summary>

**B** — Gestão de tempo: 130 minutos / 65 questões = 2 min/questão. Reserve ~97 min para primeira passagem (~1,5 min/questão). Questões que você está confiante: responder e seguir. Questões difíceis: marcar e pular. Com os 33 min restantes, revise as marcadas. Nunca fique preso em uma única questão por 10 minutos.

</details>

---

**8.** Uma questão do exame tem as seguintes opções e pede a resposta "mais segura":
- A) Usar access keys IAM hardcoded no código da aplicação
- B) Usar IAM Role associada ao EC2 instance profile
- C) Armazenar access keys em variáveis de ambiente do EC2
- D) Usar um arquivo ~/.aws/credentials no servidor

<details><summary>Resposta</summary>

**B** — IAM Role via Instance Profile: credenciais temporárias rotacionadas automaticamente pelo STS; nunca expostas em código, env vars ou arquivos. Access keys hardcoded (A), env vars (C) e arquivo de credenciais (D) são todos riscos de vazamento. A regra geral: nunca use long-lived credentials quando IAM Roles são possíveis.

</details>

---

**9.** Qual domínio do exame SAA-C03 tem o maior peso?

- A) Design de Arquiteturas Resilientes (26%)
- B) Design de Arquiteturas de Alta Performance (24%)
- C) Design de Aplicações Seguras (30%)
- D) Design de Arquiteturas Otimizadas em Custo (20%)

<details><summary>Resposta</summary>

**C** — Design de Aplicações Seguras tem 30% do exame — é o domínio mais pesado. Foque em: IAM (roles, policies, conditions), KMS, VPC security (SG vs NACL), S3 (Block Public Access, OAC, pre-signed), Cognito, SecretsManager, GuardDuty, Shield, WAF. 30% das questões são sobre segurança.

</details>

---

**10.** Qual é a pontuação mínima para aprovação no SAA-C03?

- A) 65% (700/1000)
- B) 72% (720/1000)
- C) 75% (750/1000)
- D) 80% (800/1000)

<details><summary>Resposta</summary>

**B** — Score mínimo: **720/1000** (numa escala de 100-1000). Não é 72% das questões corretas — a pontuação usa modelo psicométrico (questões de pruning não pontuam). Na prática, precisar acertar aproximadamente 43-47 das 65 questões pontuadas (existem questões de teste não pontuadas que aparecem aleatórias).

</details>

---

**11.** Uma questão descreve: "a empresa precisa de uma solução que processe dados **em lote** uma vez por dia". Qual serviço/combinação é candidato a resposta correta?

- A) Kinesis Data Streams + Lambda
- B) EventBridge Scheduler + Lambda (ou ECS/Batch)
- C) SQS FIFO + Lambda
- D) Kinesis Data Firehose

<details><summary>Resposta</summary>

**B** — "Uma vez por dia" = processamento agendado. EventBridge Scheduler (ou Rule do tipo `schedule`) + Lambda/ECS Fargate/AWS Batch é o padrão. Kinesis é para streaming contínuo em tempo real, não batch diário. AWS Batch é excelente para jobs de computação pesada em batch.

</details>

---

**12.** Você está no exame e uma questão pede para escolher entre RDS Multi-AZ e Read Replicas. O requisito diz "alta disponibilidade para escrita". O que escolher?

- A) Read Replicas (podem ser promovidas para primary em caso de falha)
- B) RDS Multi-AZ (failover automático síncrono para standy; sem perda de dados de escrita)
- C) Ambos, com Read Replicas apontando para o standby Multi-AZ
- D) Aurora em vez de RDS padrão

<details><summary>Resposta</summary>

**B** — Multi-AZ: replicação **síncrona** para standby em outra AZ. Failover automático em 1-2 minutos. Sem perda de writes (síncrono). Read Replicas: replicação **assíncrona** — podem ter lag; se o primary falhar, requer promoção manual (não automático para RDS — apenas Aurora promove automaticamente).

</details>

---

**13.** Uma questão menciona "custo de saída de dados" (data transfer out). Qual é o impacto arquitetural disso?

- A) Dados transferidos entre regiões AWS são gratuitos
- B) Upload para S3 é gratuito; transferência S3 → internet tem custo (~$0.09/GB); usar CloudFront reduz o custo de transferência
- C) VPC Peering nunca tem custo de transferência de dados
- D) Lambda não tem custo de transferência de dados

<details><summary>Resposta</summary>

**B** — Data transfer pricing: UPLOAD para AWS = gratuito. Download da AWS para internet = ~$0.09/GB. CloudFront tem custo menor por GB que S3 direto + inclui edge caching. Transferência intra-região (mesma AZ) = geralmente gratuito entre serviços. AZ para AZ na mesma região = com custo ($0.01/GB). Entre regiões = sempre tem custo.

</details>

---

**14.** No exame, quando uma questão menciona uma "startup com orçamento limitado" precisando de alta disponibilidade, qual abordagem o exame geralmente favorece?

- A) Multi-region active-active para máxima HA
- B) Multi-AZ com instâncias menores + Auto Scaling (HA sem super-provisionar)
- C) On-Demand instances em única AZ (mais barato)
- D) Reserved Instances de 3 anos imediatamente

<details><summary>Resposta</summary>

**B** — O exame frequentemente testa: alta disponibilidade custo-efetiva. Multi-AZ é suficiente para a maioria dos casos de HA. Instâncias menores + Auto Scaling evitam on-demand caro e desnecessário. Multi-region é para DR (diferente de HA) e custa muito mais. Reserve só após o crescimento estabilizar.

</details>

---

**15.** Qual é o propósito das questões "unscored" (não pontuadas) no exame SAA-C03?

- A) Não existem questões não pontuadas no SAA-C03
- B) São questões de piloting (testando novas questões para futuras versões do exame), distribuídas aleatoriamente entre as 65
- C) São as 5 questões mais fáceis para garantir pontuação mínima
- D) São questões sobre serviços novos que foram anunciados recentemente

<details><summary>Resposta</summary>

**B** — AWS inclui questões de "field trial" ou "pre-operational" — questões novas que estão sendo testadas para calibração. Você não sabe quais são pontuadas e quais não são. Implicação: não pule nenhuma questão (mesmo as que parecem muito difíceis podem ser não-pontuadas; mesmo as fáceis são pontuadas). Responda todas.

</details>
