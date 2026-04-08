# Flashcards — Alta Disponibilidade e Escalabilidade

> Revise um cartão de cada vez. Tente responder antes de revelar a resposta.

---

**P:** Qual é a diferença fundamental entre ALB e NLB?  
**R:** ALB opera na camada 7 (HTTP/HTTPS/HTTP2/gRPC), roteamento por conteúdo (path, host, header, query string). NLB opera na camada 4 (TCP/UDP/TLS), ultra-baixa latência, IP estático por AZ, preserva IP de origem do cliente por padrão.

---

**P:** O que é GWLB (Gateway Load Balancer) e quando usar?  
**R:** Camada 3, protocolo GENEVE, projetado para inserção transparente de appliances de segurança (firewalls next-gen, IDS/IPS, inspeção profunda de pacotes). Encapsula tráfego e o envia aos appliances (EC2 com NVA) no target group. Use com VPC Routing e GWLB Endpoints.

---

**P:** O que é cross-zone load balancing e qual é o padrão em cada tipo de ELB?  
**R:** Distribui tráfego igualmente entre todas as instâncias registradas, independente de AZ. ALB: habilitado por padrão, sem cobrança extra. NLB/GWLB: desabilitado por padrão, habilitação cobra por tráfego inter-AZ. CLB: habilitado por padrão via console, sem cobrança.

---

**P:** O que é Connection Draining (Deregistration Delay)?  
**R:** Período (padrão 300s, 0–3600s) em que o ELB aguarda conexões em andamento completarem antes de desregistrar/interromper uma instância. Durante esse período, o ALB para de enviar novas requisições para a instância mas mantém as existentes. Reduzir para aplicações com requisições curtas; aumentar para uploads longos.

---

**P:** Quais tipos de targets o ALB suporta?  
**R:** Instâncias EC2 (por instance ID), endereços IP (recursos on-premises, outros serviços com IP), funções Lambda (para serverless backends), e outros ALBs (para arquiteturas de microserviços encadeadas).

---

**P:** O ALB preserva o IP de origem do cliente?  
**R:** O ALB NÃO preserva o IP de origem diretamente — ele termina a conexão TCP e faz nova requisição para o target. O IP do cliente original está no header `X-Forwarded-For`. O NLB SIM preserva o IP de origem (pass-through na camada 4).

---

**P:** O que é Launch Template vs Launch Configuration?  
**R:** Launch Configuration: legado, imutável (não pode editar, só criar novo), não suporta multiple instance types, não suporta spot com on-demand mix. Launch Template: recomendado, tem versões ($Latest, $Default), suporta mixed instances policy, spot fleet, placamento em grupos, T2/T3 unlimited.

---

**P:** Quais são as 4 políticas de Auto Scaling?  
**R:** 1) Target Tracking: mantém métrica em um valor alvo (ex: CPU 50%), cria alarmes automaticamente. 2) Step Scaling: escalona em passos baseados em ranges de métricas, requer alarme CloudWatch. 3) Scheduled Scaling: baseado em data/hora, para padrões previsíveis. 4) Predictive Scaling: usa ML para prever carga e pre-provisiona.

---

**P:** O que é cooldown period no Auto Scaling?  
**R:** Período (padrão 300s) após uma atividade de scaling durante o qual o ASG não inicia outra ação de scaling. Objetivo: deixar a nova instância começar a absorver carga antes de decidir escalar mais. Target Tracking tem warmup period separado que substitui o cooldown para scale-out.

---

**P:** O que é Lifecycle Hook no ASG?  
**R:** Permite pausar o ASG durante transições do ciclo de vida da instância: `Pending:Wait` (antes de entrar em serviço) e `Terminating:Wait` (antes de ser terminada). O hook pode acionar Lambda, SNS, SQS ou EventBridge. Timeout padrão: 3600s. Para continuar: `CompleteLifecycleAction(CONTINUE)` ou `ABANDON`.

---

**P:** O que é a política de terminação padrão do ASG?  
**R:** 1) Seleciona AZ com mais instâncias (rebalancear); 2) Instância com Launch Configuration/Template mais antigo; 3) Instância mais próxima da próxima hora de cobrança; 4) Aleatório se empate. Também existem políticas: `OldestInstance`, `NewestInstance`, `ClosestToNextInstanceHour`, `AllocationStrategy`.

---

**P:** O que é Predictive Scaling e qual é o requisito?  
**R:** Usa ML para analisar histórico de métricas e pré-provisionar capacidade antes do pico esperado. Requer pelo menos 14 dias de histórico de métricas para gerar previsões. Pode ser usado combinado com Target Tracking para cobertura proativa e reativa.

---

**P:** O que é PrivateLink e como o NLB se relaciona com ele?  
**R:** AWS PrivateLink permite expor um serviço de uma VPC para consumidores de outras VPCs/contas sem peering, via Interface Endpoints. O NLB é o pré-requisito: o serviço precisa estar atrás de um NLB para ser registrado como VPC Endpoint Service. Consumidores criam Interface VPC Endpoints que apontam ao serviço.

---

**P:** Como funciona o health check do ALB vs do ASG?  
**R:** ALB verifica saúde das instâncias no target group (HTTP path, código de resposta, intervalo). ASG pode usar dois tipos: EC2 (status de sistema) ou ELB (delegado ao ELB). Na produção, SEMPRE configurar o ASG para usar ELB health check, pois detecta falhas de aplicação que EC2 health check não detecta.

---

**P:** O que é warm-up period no ASG Target Tracking?  
**R:** Período (em segundos) durante o qual uma nova instância não é considerada nas métricas de scaling, dando tempo para ela inicializar. Evita que o ASG interprete uma instância "aquecendo" como baixa performance e escale mais desnecessariamente.

---

**P:** Como implementar blue/green deployment com ALB e ASG?  
**R:** Criar dois target groups: `tg-blue` (versão atual) e `tg-green` (nova versão). Configurar listener rule com weighted target groups (ex: 95% blue, 5% green). Gradualmente aumentar peso do green. Se sucesso, redirecionar 100% ao green. Se falha, voltar ao 100% blue imediatamente.

---

**P:** O que é Sticky Sessions no ALB?  
**R:** Garante que requisições do mesmo cliente vão para a mesma instância durante um período. Usa cookie: `AWSALB` (gerado pelo ALB, duração 1s–7 dias) ou cookie definido pela aplicação. Útil para sessões com estado. Pode causar desbalanceamento se uma instância receber muitas sessões longas.

---

**P:** O que é NLB Zonal DNS e static IP?  
**R:** Todo NLB recebe um DNS host-name (ex: `my-nlb-xxxx.elb.amazonaws.com`). Cada AZ habilitada tem um nó NLB com IP estático atribuível ao Elastic IP. Para whitelisting de IP no cliente, use o Elastic IP do NLB. O DNS resolve para todos os nós; para IP fixo por AZ, use o IP diretamente.

---

**P:** Qual é a diferença entre Step Scaling e Simple Scaling?  
**R:** Simple Scaling (legado): dispara uma ação e espera o cooldown antes de qualquer outra ação. Tudo ou nada. Step Scaling: permite múltiplos passos baseados em ranges de métrica sem cooldown entre steps diferentes — reage mais rápido a mudanças graduais ou bruscas de carga.

---

**P:** Como funciona a Mixed Instance Policy no ASG com Spot?  
**R:** Define um pool de tipos de instância e a porcentagem de On-Demand vs Spot. O ASG tenta lançar Spot do tipo mais barato disponível. Se interrompido, tenta outro tipo do pool. Estratégia `capacityOptimized` minimiza interrupções escolhendo pools com mais capacidade disponível.
