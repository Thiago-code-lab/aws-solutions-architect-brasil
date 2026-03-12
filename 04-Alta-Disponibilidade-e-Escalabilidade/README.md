# 04 Alta Disponibilidade e Escalabilidade

## 📋 Índice
- [Objetivos do Módulo](#-objetivos-do-módulo)
- [Conceitos Fundamentais](#-conceitos-fundamentais)
- [Arquitetura e Componentes](#-arquitetura-e-componentes)
- [Configurações Importantes para o Exame](#-configurações-importantes-para-o-exame)
- [Comparativo de Serviços](#-comparativo-de-serviços)
- [Dicas e Armadilhas do Exame](#-dicas-e-armadilhas-do-exame)
- [Para o Exame](#-para-o-exame)
- [Links Relacionados](#-links-relacionados)

## 🎯 Objetivos do Módulo

Ao terminar este módulo, você deve conseguir:

- selecionar o tipo correto de ELB (ALB, NLB, GWLB) a partir do protocolo e requisito do cenário
- diferenciar as políticas de scaling do ASG e aplicá-las ao perfil de tráfego da questão
- descrever o papel dos Lifecycle Hooks em operações de deploy e warm-up de instâncias
- explicar cross-zone load balancing e seu impacto em custo e distribuição
- configurar health checks corretos para um ambiente com ELB e ASG

## 📚 Conceitos Fundamentais

### Elastic Load Balancing

O ELB é um serviço gerenciado de distribuição de tráfego que disponibiliza um único ponto de entrada com alta disponibilidade automática. Existem quatro tipos, e o exame cobra fortemente as diferenças entre eles.

#### ALB — Application Load Balancer

Opera na camada 7 (HTTP/HTTPS). É o load balancer mais funcional para aplicações modernas. Suporte a HTTP, HTTPS, HTTP/2 e gRPC. O roteamento é baseado em regras configuráveis: path-based (prefix ou exact match na URL), host-based (virtual hosting por hostname), query string, HTTP headers e IP de origem.

**Target Groups**: o ALB distribui tráfego para target groups, que podem ser EC2 instances, ECS tasks, Lambda functions ou IPs (incluindo on-premises via Direct Connect/VPN). Cada regra de roteamento aponta para um target group. Um listener pode ter várias regras com prioridade, e a regra padrão (default action) atende o tráfego não coberto.

**Weighted Target Groups**: suporta distribuição ponderada de tráfego entre dois target groups. Útil para canary releases e blue/green deployments: envie 95% ao grupo antigo e 5% ao novo gradualmente.

**Connection termination**: o ALB termina a conexão SSL/TLS e se comunica com os targets numa nova conexão (com ou sem HTTPS). O header `X-Forwarded-For` carrega o IP original do cliente. Se precisar do IP real no backend, configure o X-Forwarded-For ou use o Proxy Protocol no ALB.

**Sticky sessions (Session Affinity)**: o ALB pode enviar requests de um mesmo cliente sempre para o mesmo target. O cookie pode ser gerado pelo ALB (AWSALB) ou pelo target (cookie de aplicação). Útil quando o estado de sessão está na memória do servidor — mas a prática recomendada é externalizar sessões num cache como ElastiCache.

**Lambda como target**: o ALB pode invocar Lambda diretamente. Isso permite expor um Lambda como endpoint HTTP sem API Gateway.

#### NLB — Network Load Balancer

Opera na camada 4 (TCP, UDP, TLS). É extremamente eficiente: dezenas de milhões de requests por segundo com latência de milissegundos uniconexão. Principal característica: **IP estático por AZ**. Pode ter um Elastic IP por AZ, o que é essencial para clientes que precisam de allowlist de IP no firewall.

**Preserve source IP**: ao contrário do ALB, o NLB preserva o IP de origem por padrão no target. Isso é importante em cenários de compliance e firewalling.

**TLS Offload**: o NLB pode terminar TLS e usar certificados ACM, se desejado. Se não terminar, passa tráfego TCP puro para o target.

**PrivateLink**: NLBs são a base do AWS PrivateLink. Para expor um serviço para outras contas ou VPCs de forma privada, cria-se um NLB e um VPC Endpoint Service.

**Static IP para firewall allowlist**: se a questão menciona "cliente precisa fazer allowlist de IPs no firewall corporativo" ou "IP fixo por AZ", a resposta é NLB.

#### GWLB — Gateway Load Balancer

Opera na camada 3 (IP) usando o protocolo GENEVE (encapsulamento de pacotes). Funciona como ponto de inserção de appliances virtuais de segurança (firewalls, IDS/IPS, DPI) no caminho do tráfego sem que a aplicação perceba. O tráfego é desviado para o GWLB → appliance → retorna ao GWLB → segue ao destino.

Configuração típica: tráfego entra via Internet Gateway, é interceptado por GWLB (usando VPC routing), processado pelos appliances no GWLB endpoint no Target Group, e depois roteado para o destino final. Permite escalar appliances horizontalmente com o ASG.

**Quando aparece no exame**: cenário que menciona inspeção de tráfego com dispositivos de segurança de terceiros (appliances, firewalls next-gen como Palo Alto ou Fortinet) ou centralização de segurança em uma conta de segurança dedicada.

#### CLB — Classic Load Balancer

Legado. Opera em camadas 4 e 7 de forma limitada. O exame pode mencioná-lo em contexto histórico, mas nunca como resposta correta para novos desenhos.

#### Cross-Zone Load Balancing

Por padrão, cada nó do ELB distribui tráfego apenas para as instâncias na sua AZ. Com cross-zone habilitado, cada nó do ELB distribui uniformemente para todos os targets em todas as AZs. ALB habilita por padrão sem custo extra. NLB desabilita por padrão; quando habilitado, cobra pela transferência inter-AZ. O GWLB também desabilita por padrão.

**Connection Draining (Deregistration Delay)**: período durante o qual o ELB para de enviar novas conexões para um target que está sendo removido, mas mantém as conexões ativas até o timeout. O padrão é 300 segundos. Para instâncias que têm muitas conexões de longa duração, ajuste esse valor.

### Auto Scaling Groups

O ASG gerencia uma frota de instâncias EC2, garantindo que a capacidade desejada seja mantida. Está intimamente ligado ao ELB: as instâncias do ASG são regularmente registradas e desregistradas nos target groups.

#### Launch Templates vs Launch Configurations

**Launch Configurations**: legadas, imutáveis. Para fazer qualquer mudança, é necessário criar uma nova. Não suportam múltiplos tipos de instância, Spot + On-Demand mixados, nem ARM. Devem ser evitadas em novos projetos.

**Launch Templates**: suporte a versionamento, múltiplos tipos de instância, mistura de Spot e On-Demand, famílias de instância via $Latest e $Default, e herança de templates base. São a resposta preferida no exame quando a questão menciona diversificação de tipos ou flexibilidade de Spot.

#### Políticas de Scaling

**Target Tracking Scaling**: define uma métrica-alvo (como CPUUtilization = 60% ou RequestCountPerTarget = 1000) e o ASG ajusta automaticamente a capacidade para manter essa métrica no nível definido. É a política mais simples e eficiente para workloads com crescimento proporcional. O ASG cria e gerencia automaticamente os CloudWatch Alarms.

**Step Scaling**: define faixas de alarme com ações diferentes. Exemplo: se CPU > 60%, adicione 1 instância; se CPU > 80%, adicione 3; se CPU < 30%, remova 1. Permite resposta escalonada a variações de intensidade diferentes. Requer criação manual dos alarmes do CloudWatch.

**Scheduled Scaling**: ajusta a capacidade em horários pré-definidos. Útil para padrões de tráfego previsíveis: aumento às 8h, redução às 20h, pico às sextas. Pode coexistir com outras políticas.

**Predictive Scaling**: usa machine learning para analisar o histórico de tráfego e provisionar capacidade antecipadamente. Vantagem sobre Target Tracking: já está pronto antes do pico chegar, em vez de reagir depois. Funciona melhor com histórico de pelo menos 14 dias. Para o exame, é a resposta ao cenário "evitar atraso de scale-out antes do pico diário".

#### Lifecycle Hooks

Lifecycle Hooks permitem executar ação personalizada durante as transições de estado de uma instância antes que ela complete a transição.

**EC2_INSTANCE_LAUNCHING (Pending:Wait)**: a instância foi iniciada, mas ainda não está InService. Nesse período, você pode executar scripts de aquecimento, instalar agentes de monitoramento, registrar em sistemas externos. Ao terminar, chama `CompleteLifecycleAction(CONTINUE)` ou deixa o heartbeat timeout expirar.

**EC2_INSTANCE_TERMINATING (Terminating:Wait)**: a instância vai ser terminada. Antes da remoção, pode-se coletar logs, fazer backup do estado, deregistrar de IPAM externo. Ao terminar, chama `CompleteLifecycleAction(CONTINUE)` ou `ABANDON`.

A notificação do lifecycle hook pode ser enviada para SNS, SQS ou EventBridge, que então dispara Lambda para execução da automação.

#### Health Checks

O ASG determina se uma instância está saudável via dois tipos de health check:

**EC2 health check** (padrão): baseia-se no status do hardware e rede da instância reportado pela AWS. Se a instância está em qualquer estado diferente de "running" ou tem impaired instance check, o ASG a substitui.

**ELB health check**: baseia-se nos health checks configurados no target group (GET numa URL específica, código de resposta esperado). Se a instância retorna resposta inesperada ao ALB, é marcada como unhealthy. O ASG com ELB health check habilitado substitui instâncias que falham no health check do ELB — muito mais completo que apenas o status EC2.

**Recomendação**: sempre habilite ELB health check no ASG quando há Load Balancer. Uma instância que está "running" mas com aplicação travada passa no EC2 health check mas falha no ELB health check.

### Warm-Up e Cooldown

**Instance Warmup**: período após scale-out durante o qual a nova instância contribui gradualmente para as métricas de scaling. Evita que o ASG continue adicionando instâncias enquanto as novas ainda estão inicializando. Suporte no Target Tracking.

**Cooldown do ASG**: período de espera após qualquer atividade de scaling antes de iniciar outra. Evita oscilações rápidas. Para step scaling, o cooldown é separado por ação de scale-out e scale-in.

### Termination Policy

O ASG segue uma lógica de seleção para terminar instâncias no scale-in: primeiro seleciona a AZ com mais instâncias, depois dentro dessa AZ escolhe com base na política configurada (default: instância mais antiga pelo launch template/config, depois a próxima a ser cobrada na hora). Políticas customizadas disponíveis: OldestInstance, NewestInstance, OldestLaunchTemplate, ClosestToNextInstanceHour e AllocationStrategy.

## 🏗️ Arquitetura e Componentes

```text
Internet / Clientes
       │
       ▼
  ┌──────────────────────────────────────────────────┐
  │          Elastic Load Balancer                   │
  │   ALB (HTTP/gRPC) | NLB (TCP/UDP) | GWLB (IP)   │
  │   Listeners → Rules → Target Groups              │
  └──────────────┬───────────────────────────────────┘
                 │  registra / desregistra
                 ▼
  ┌──────────────────────────────────────────────────┐
  │         Auto Scaling Group                       │
  │                                                  │
  │  Launch Template                                 │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
  │  │  EC2 AZ-a│  │  EC2 AZ-b│  │  EC2 AZ-c│       │
  │  └──────────┘  └──────────┘  └──────────┘       │
  │                                                  │
  │  Scaling Policies: Target | Step | Scheduled     │
  │  Predictive: provisiona antes do pico            │
  │  Lifecycle Hooks: Pending:Wait / Terminating:Wait│
  └──────────────────────────────────────────────────┘
                 │
                 ▼
         CloudWatch Alarms
                 │
                 ▼
         SNS / Lambda / EventBridge
```

## ⚙️ Configurações Importantes para o Exame

| Item | Valor padrão / máximo | Observação |
|---|---|---|
| Connection Draining (deregistration delay) | 300 segundos | Ajuste para conexões longas |
| ALB idle timeout | 60 segundos | Conexão fecha se inativa |
| NLB IP estático | 1 por AZ | Principal diferencial vs ALB |
| ASG min/desired/max capacity | 0/0/0 por padrão | Configurar conforme workload |
| ASG default cooldown | 300 segundos | Reduzir para scaling mais agile |
| Lifecycle Hook heartbeat timeout | 3.600 segundos | Ajustável conforme operação |
| ELB health check interval padrão | 30 segundos | Configurável por target group |
| Healthy threshold | 5 checks | Consecutivos para marcar healthy |
| Unhealthy threshold | 2 checks | Consecutivos para marcar unhealthy |

## 🔄 Comparativo de Serviços

| Característica | ALB | NLB | GWLB |
|---|---|---|---|
| Camada OSI | 7 (HTTP) | 4 (TCP/UDP) | 3 (IP) |
| Protocolos | HTTP, HTTPS, gRPC | TCP, UDP, TLS | GENEVE |
| IP estático por AZ | ❌ (DNS) | ✅ (Elastic IP) | ✅ |
| Preserve source IP | Via X-Forwarded-For | ✅ Nativo | ✅ |
| Path/host based routing | ✅ | ❌ | ❌ |
| Lambda como target | ✅ | ❌ | ❌ |
| PrivateLink | ❌ | ✅ (base) | ✅ |
| Inspecção de tráfego com appliances | ❌ | ❌ | ✅ |
| Cross-zone padrão | ✅ (sem custo) | ❌ (cobra inter-AZ) | ❌ |

| Política de Scaling | Quando usar | Vantagem |
|---|---|---|
| Target Tracking | Workload proporcional e contínuo | Simples, auto-managed alarms |
| Step Scaling | Resposta diferenciada por intensidade | Controle fino por faixa |
| Scheduled | Padrão de tráfego previsível | Evita lag no scale-out |
| Predictive | Picos recorrentes e previsíveis | Provisiona antes do pico |

## 💡 Dicas e Armadilhas do Exame

- ALB tem IP dinâmico (DNS). Se o cliente precisa de IP fixo no firewall, a resposta é NLB com Elastic IP.
- GWLB não distribui tráfego de aplicação — ele inspeciona e encaminha. Confundir com ALB é uma armadilha.
- ASG com apenas EC2 health check vai manter instâncias com aplicação travada, pois a instância está "running". Habilite ELB health check.
- Target Tracking cria alarmes CloudWatch automaticamente. Não apague esses alarmes — são gerenciados pelo serviço.
- Launch Configuration é legado e imutável. Em qualquer questão nova, Launch Template é a resposta preferida.
- Lifecycle Hook no scale-out serve para warmup antes de entrar em serviço; no scale-in, para drenagem de estado antes de encerrar.
- Cross-zone balancing no NLB tem custo de transferência inter-AZ. Se a questão menciona custo, avalie se cross-zone está necessariamente habilitado.
- Predictive Scaling requer histórico de 14 dias para funcionar bem. Para novos environments, Target Tracking or Scheduled é mais adequado.

## 💡 Para o Exame

- HTTP/gRPC, path routing, Lambda target → ALB. TCP/UDP, IP fixo, PrivateLink → NLB. Appliance de segurança de terceiro → GWLB.
- Pico previsível = Scheduled + Predictive. Workload proporcional = Target Tracking. Resposta escalonada = Step Scaling.
- Sempre habilite ELB health check no ASG em produção.
- Launch Template supera Launch Configuration em todos os cenários novos.
- Lifecycle Hooks = automação antes de entrar em serviço ou antes de sair.

## 📎 Links Relacionados

- [Questões do módulo](./questoes.md)
- [Flashcards do módulo](./flashcards.md)
- [Cheatsheet do módulo](./cheatsheet.md)
- [Casos de uso do módulo](./casos-de-uso.md)
- [Lab prático do módulo](./lab.md)
- [Links oficiais](./links.md)
- [Módulo 03: Computação EC2](../03-Computacao-EC2/)
- [Módulo 07: VPC e Redes](../07-VPC-e-Redes/)
- [Módulo 08: DNS, Route 53 e CloudFront](../08-DNS-Route53-e-CloudFront/)
