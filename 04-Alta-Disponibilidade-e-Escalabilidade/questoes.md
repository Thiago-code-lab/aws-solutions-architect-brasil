# Questões — Alta Disponibilidade e Escalabilidade

## Questão 1
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Uma empresa tem uma API REST usando EC2 com Auto Scaling. Clientes corporativos precisam adicionar os IPs da API ao whitelist do firewall deles. Qual tipo de load balancer resolve isso com IP fixo?

- A) ALB com IP estático
- B) NLB com Elastic IP por AZ
- C) CLB com IP estático
- D) ALB com wildcard SSL

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

NLB permite associar um Elastic IP por Availability Zone, fornecendo endereços IP estáticos e previsíveis. ALB usa DNS e o IP pode mudar. CLB é legado e não tem IP estático confiável. Wildcard SSL não resolve o problema de IP.

**Conceito-chave:** NLB com Elastic IP para allowlist de IP no cliente
</details>

## Questão 2
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Uma plataforma SaaS com múltiplos tenants roteia tráfego para diferentes target groups baseados no hostname da requisição: `tenant-a.app.com` vai para TG-A e `tenant-b.app.com` vai para TG-B. Qual recurso do ALB implementa isso?

- A) Path-based routing
- B) Host-based routing
- C) IP-based routing
- D) Weighted target groups

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Host-based routing usa o header HTTP Host para selecionar o target group de acordo com o hostname da requisição. Path-based usa o caminho da URL (/api, /admin). IP-based routing não existe como regra de listener no ALB. Weighted target groups dividem tráfego proporcional entre grupos, não por hostname.

**Conceito-chave:** host-based routing no ALB para multi-tenant por hostname
</details>

## Questão 3
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Um time quer usar ALB para um canary release, enviando 5% do tráfego para a nova versão e 95% para a versão atual. Qual recurso do ALB permite isso?

- A) Host-based routing com dois listeners
- B) Weighted target groups em uma mesma regra de listener
- C) NLB com cross-zone balancing
- D) Auto Scaling Group com Launch Template ponderado

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

ALB suporta weighted target groups em uma regra de listener, distribuindo tráfego de forma proporcional entre dois ou mais target groups na mesma ação. É o mecanismo padrão para canary e blue/green gradual. Host-based routing roteia por hostname, não por peso. NLB não tem esse recurso. ASG não controla proporção de tráfego no nível do load balancer.

**Conceito-chave:** weighted target groups no ALB para canary release
</details>

## Questão 4
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Difícil

Uma empresa quer inspecionar todo tráfego de entrada da internet antes de chegar à aplicação, usando um firewall next-gen de terceiro que roda em EC2. Qual tipo de load balancer suporta esse modelo de inserção de appliance transparentemente?

- A) ALB com Lambda authorizer
- B) NLB com TLS pass-through
- C) GWLB com VPC routing para appliances no target group
- D) CLB com proxy protocol

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

GWLB opera na camada 3 com o protocolo GENEVE, permitindo inserção transparente de appliances de segurança no path do tráfego. Os pacotes são enviados ao GWLB endpoint, processados pelos appliances no target group e devolvidos ao GWLB, que os redireciona ao destino. ALB, NLB e CLB não suportam esse modelo de encapsulamento de pacotes IP.

**Conceito-chave:** GWLB para inserção transparente de appliances de segurança
</details>

## Questão 5
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Um ASG está configurado com health check do tipo EC2 apenas. Uma instância está em execução mas o processo da aplicação travou e retorna HTTP 500 para qualquer requisição. O ASG vai substituí-la automaticamente?

- A) Sim, o EC2 health check detecta qualquer falha de aplicação
- B) Não, EC2 health check valida apenas status da instância, não da aplicação
- C) Sim, o ALB notifica o ASG automaticamente sem configuração adicional
- D) Não, é necessário reiniciar a instância manualmente

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

O EC2 health check detecta apenas falhas de status do hardware/SO (instance status check e system status check). Uma aplicação travada retornando HTTP 500 ainda está em estado "running" para o EC2. Para o ASG substituir instâncias com aplicação falha, é necessário habilitar o ELB health check no ASG, que avalia a saúde no target group do ALB.

**Conceito-chave:** ELB health check no ASG para detectar falhas de aplicação
</details>

## Questão 6
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Uma aplicação recebe picos previsíveis toda segunda-feira às 8h quando funcionários chegam ao trabalho. A equipe quer garantir que as instâncias já estejam disponíveis antes do pico, evitando latência de scale-out. Qual política de scaling é mais adequada?

- A) Target Tracking com CPUUtilization
- B) Scheduled Scaling aumentando capacidade às 7h45 toda segunda
- C) Step Scaling com alarm em CPU > 70%
- D) Manual scaling toda segunda pela manhã

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Scheduled Scaling antecipa o aumento de capacidade para antes do pico esperado, evitando o lag inerente a políticas reativas. Target Tracking e Step Scaling são reativas: esperame a métrica subir para agir — chegando tarde para picos abruptos. Manual scaling depende de intervenção humana. Para picos previsíveis, Scheduled (ou Predictive com histórico) é a resposta.

**Conceito-chave:** Scheduled Scaling para provisionar antes de picos previsíveis
</details>

## Questão 7
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Uma empresa quer garantir que ao adicionar novas instâncias EC2 via ASG, o agente de monitoramento seja instalado e configurado antes que a instância comece a receber tráfego do load balancer. Qual mecanismo deve ser usado?

- A) User Data com script de instalação
- B) Lifecycle Hook em EC2_INSTANCE_LAUNCHING com Lambda que instala o agente
- C) Golden AMI com agente pré-instalado
- D) AWS Config rule para verificar o agente

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Lifecycle Hook em `Pending:Wait` mantém a instância fora do target group do ELB enquanto a automação corre. Uma Lambda (acionada via SNS/SQS/EventBridge) instala e configura o agente e depois chama `CompleteLifecycleAction(CONTINUE)`. User Data executa na boot mas não garante que a instância só entre no load balancer depois — o hook sim. Golden AMI é bom para o agente pré-instalado, mas não para configurações dinâmicas.

**Conceito-chave:** Lifecycle Hook Pending:Wait para custom initialization antes do ELB
</details>

## Questão 8
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Um ALB tem dois target groups em duas AZs: TG-A com 10 instâncias na AZ1 e TG-B com 2 instâncias na AZ2. Com cross-zone load balancing habilitado no ALB, como o tráfego é distribuído?

- A) 50% para AZ1 e 50% para AZ2
- B) Uniformemente entre as 12 instâncias (~8,3% cada)
- C) 83% para TG-A e 17% para TG-B
- D) 100% para TG-A por ter mais instâncias

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Com cross-zone load balancing habilitado, cada nó do ALB distribui tráfego igualmente entre todas as instâncias saudáveis em todas as AZs, independentemente de quantas estão por AZ. 12 instâncias = ~8,3% cada. Sem cross-zone, cada nó distribui apenas para instâncias na sua AZ (50% para cada AZ, então AZ1 com 10 instâncias cada uma recebe 5% e AZ2 com 2 instâncias cada uma recebe 25%).

**Conceito-chave:** cross-zone load balancing distribui uniformemente entre todas as instâncias
</details>

## Questão 9
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Difícil

Um ASG está executando scale-in. A política de termination está como default. A AZ1 tem 4 instâncias e a AZ2 tem 2. Qual instância é selecionada para terminação primeiro?

- A) A instância mais nova em qualquer AZ
- B) Uma instância da AZ2, pois tem menos e está desbalanceada
- C) Uma instância da AZ1, por ser a AZ com mais instâncias, seguido pela mais antiga pelo launch template
- D) Aleatório entre todas as instâncias

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

A política de terminação padrão do ASG: (1) seleciona a AZ com mais instâncias para rebalancear; (2) dentro dessa AZ, seleciona a instância com o launch template ou configuration mais antigo; (3) se empate, a mais próxima da próxima hora chargeable. AZ1 com 4 instâncias é selecionada primeiro. A mais antiga pelo template seria encerrada.

**Conceito-chave:** termination policy padrão: AZ com mais instâncias → mais antiga pelo template
</details>

## Questão 10
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Um serviço precisa de acesso a uma API interna que roda numa VPC de outra conta, sem expor tráfego à internet e sem criar VPC Peering. Qual arquitetura atende a esse requisito?

- A) Internet Gateway com security group restrito
- B) NLB com VPC Endpoint Service (PrivateLink) na conta produtora
- C) ALB multi-region com CloudFront
- D) VPN site-to-site entre as contas

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

AWS PrivateLink usa um NLB como backend de um VPC Endpoint Service. O consumidor cria um Interface Endpoint (VPC Endpoint) que aparece como um ENI na sua VPC com IP privado. O tráfego nunca sai da rede AWS e não requer peering, routing, ou exposição à internet. VPN é mais complexa e não soluciona a questão de forma elegante.

**Conceito-chave:** NLB + PrivateLink para exposição privada de serviço cross-account sem peering
</details>

## Questão 11
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Uma instância EC2 do ASG está processando uma tarefa longa quando o scale-in é disparado. Qual mecanismo garante que a tarefa seja concluída antes da instância ser terminada?

- A) Cooldown period de 3600 segundos
- B) Connection Draining no ELB
- C) Lifecycle Hook em EC2_INSTANCE_TERMINATING
- D) Suspended scaling para scale-in

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Lifecycle Hook em `Terminating:Wait` mantém a instância em estado de espera antes da terminação efetiva. A automação pode monitorar a conclusão da tarefa e chamar `CompleteLifecycleAction(CONTINUE)` quando terminar. Connection Draining apenas drena conexões HTTP ativas do ELB, não aguarda tarefas em segundo plano. Cooldown afeta quando o próximo scaling ocorre, não o timing de um já iniciado. Suspend interromperia o scaling.

**Conceito-chave:** Lifecycle Hook Terminating:Wait para tarefas de longa duração no scale-in
</details>

## Questão 12
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Um arquiteto precisa criar uma frota de instâncias EC2 usando múltiplos tipos (m5.large, m5.xlarge, m6i.large) com mix de On-Demand e Spot para reduzir custo. Qual recurso permite isso?

- A) Launch Configuration com múltiplos tipos
- B) Launch Template com overrides de tipo + Mixed Instance Policy no ASG
- C) Múltiplos ASGs separados por tipo
- D) Dedicated Host com multiplex de tipos

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Launch Template suporta overrides de tipo de instância. No ASG, a Mixed Instance Policy permite definir múltiplos tipos e a distribuição On-Demand/Spot. O ASG escolhe o mais barato disponível do pool de tipos definidos. Launch Configuration não suporta múltiplos tipos. Múltiplos ASGs aumentam complexidade operacional. Dedicated Host não tem a ver com diversificação de tipos Spot.

**Conceito-chave:** Launch Template com Mixed Instance Policy para frota diversificada
</details>

## Questão 13
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Fácil

Uma empresa quer garantir que mesmo se uma instância do ASG parecer saudável para o EC2, ela será substituída se a aplicação não retornar HTTP 200 nas requisições de health check. Qual configuração implementa isso?

- A) Criar um alarm CloudWatch para response code
- B) Habilitar ELB health check no ASG e configurar health check no target group
- C) Usar enhanced monitoring no EC2
- D) Configurar CloudTrail para detectar erros HTTP

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Para o ASG considerar health checks de aplicação, é preciso: (1) configurar o health check no ALB target group com o caminho e código de sucesso esperados; (2) habilitar o ELB health check no ASG. Assim, instâncias que falham no health check do ALB são marcadas como unhealthy no ASG e substituídas. Alarm CloudWatch não age no ASG automaticamente dessa forma. Enhanced monitoring traz métricas mas não substitui instâncias.

**Conceito-chave:** habilitar ELB health check no ASG + configurar TG health check
</details>

## Questão 14
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Um time quer escalar o ASG baseado no número de mensagens visíveis na fila SQS (ApproximateNumberOfMessagesVisible). Qual política de scaling suporta métricas customizadas do CloudWatch diretamente?

- A) Scheduled Scaling
- B) Predictive Scaling
- C) Target Tracking Scaling com métrica customizada do CloudWatch
- D) EC2 Auto Recovery

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Target Tracking suporta métricas customizadas do CloudWatch além das métricas predefinidas. É possível configurar APproximateNumberOfMessagesVisible/number-of-instances como target para manter uma carga de mensagens por instância constante. Scheduled é baseado em horário. Predictive usa histórico de métricas padrão. Auto Recovery é para falhas de hardware, não scaling.

**Conceito-chave:** Target Tracking com métrica customizada do CloudWatch para escalar por SQS
</details>

## Questão 15
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Difícil

Uma empresa tem um ALB com listener HTTPS na porta 443. O certificado SSL está configurado no listener. A aplicação backend roda HTTP na porta 8080. Um arquiteto precisa garantir que a comunicação ALB → EC2 também seja criptografada. Qual configuração implementa isso?

- A) ALB não pode estabelecer HTTPS para targets, apenas HTTP
- B) Configurar o target group protocol como HTTPS na porta 443 e instalar certificado nas instâncias
- C) Usar NLB com TLS pass-through
- D) Configurar SSL policy no ALB para criptografar automaticamente o backend

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

O ALB suporta HTTPS como protocolo de target group. Configure o target group com protocolo HTTPS e instale certificados nas instâncias (self-signed ou ACM via Private CA). O ALB estabelece HTTPS até cada instância. A alternativa A está errada — ALB suporta sim HTTPS backend. NLB com TLS pass-through é uma solução diferente. SSL policy no listener controla apenas a negociação TLS do cliente ao ALB.

**Conceito-chave:** target group HTTPS para criptografia end-to-end no ALB
</details>
