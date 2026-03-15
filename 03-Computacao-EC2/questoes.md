# Questões — Computação EC2

## Questão 1
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Uma empresa precisa executar um job de machine learning com cálculos matriciais intensivos de GPU. Qual família de instância EC2 é mais adequada?

- A) M5 — General Purpose
- B) R6g — Memory Optimized
- C) P4de — Accelerated Computing
- D) T3 — Burstable

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Instâncias P são aceleradas com GPUs NVIDIA, desenvolvidas para treinamento de machine learning e HPC. As outras famílias não têm GPU. M5 é balanced CPU/RAM sem GPU. R6g é Memory Optimized para in-memory databases. T3 é burstable para cargas leves intermitentes.

**Conceito-chave:** família P (GPU) para ML/HPC
</details>

## Questão 2
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Uma equipe de análise roda processamento de dados em lotes de 6 horas diárias, com falhas toleráveis e capacidade de recomeçar do checkpoint. Qual modelo de compra oferece o maior desconto?

- A) On-Demand
- B) Reserved Instance Standard 1 ano
- C) Spot Instances
- D) Dedicated Host

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Spot Instances oferecem até 90% de desconto para cargas tolerantes a interrupção. O workload de batch com checkpoint satisfaz exatamente esse perfil: pode ser interrompido e retomado. Reserved e Dedicated Host são para workloads contínuos ou de compliance. On-Demand é mais caro sem compromisso.

**Conceito-chave:** Spot Instances para batch tolerante a interrupção
</details>

## Questão 3
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Uma empresa usa um banco de dados ERP que roda 24/7 sem interrupções há dois anos e precisa ficar ativo pelos próximos 3 anos. Qual modelo de compra oferece o maior desconto com a menor possibilidade de mudança de tipo de instância?

- A) Spot Instances
- B) On-Demand
- C) Reserved Instance Standard 3 anos All Upfront
- D) Savings Plans Compute 1 ano

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Standard Reserved Instance 3 anos All Upfront oferece o maior desconto possível (~72%). O workload é estável e previsível, sem necessidade de flexibilidade de tipo. All Upfront maximiza o desconto em relação a Partial ou No Upfront. Savings Plans seria adequado se a empresa quisesse flexibilidade de tipo/região, mas tem desconto ligeiramente menor.

**Conceito-chave:** Standard RI 3 anos All Upfront para workload estável de longo prazo
</details>

## Questão 4
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma EC2 rodando uma API precisa buscar o ID da instância, a região e o ARN da role associada. De onde pode obter essas informações de forma segura sem acessar serviços externos?

- A) User Data
- B) Parameter Store
- C) Instance Metadata Service (IMDSv2)
- D) Secrets Manager

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

O IMDS disponibiliza informações da instância em 169.254.169.254. Com IMDSv2 (session-oriented), obtém-se token primeiro e usa-o nos requests seguintes. Inclui instance-id, region, ami-id, IAM role credentials e muito mais. User Data apenas executa script na inicialização. Parameter Store e Secrets Manager armazenam dados externos, não metadados da instância.

**Conceito-chave:** IMDSv2 para self-discovery da instância
</details>

## Questão 5
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Uma aplicação HPC distribui cálculos entre nós que precisam de latência de rede sub-milissegundo e throughput de 100 Gbps entre si. Qual configuração EC2 otimiza isso?

- A) Spread Placement Group em múltiplas AZs
- B) Partition Placement Group na mesma AZ
- C) Cluster Placement Group na mesma AZ com instâncias Enhanced Networking
- D) Instâncias em AZs diferentes sem placement group

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Cluster Placement Group coloca instâncias fisicamente próximas, minimizando latência e maximizando banda entre elas (até 10 Gbps com instâncias Enhanced Networking / EFA). Deve estar na mesma AZ. Spread PG maximiza disponibilidade mas dispersa fisicamente, deteriorando a latência entre nós. Partition PG diferencia racks mas não minimiza latência intra-grupo. AZs diferentes introduzem latência de rede regional.

**Conceito-chave:** Cluster Placement Group para HPC de baixa latência
</details>

## Questão 6
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Fácil

Uma aplicação crítica usa EC2 com um banco de dados em memória. A equipe precisa garantir que se a instância for substituída, o banco de dados volte com o estado anterior rapidamente em vez de recarregar tudo do zero. Qual recurso EC2 resolve isso?

- A) Instance Store
- B) Hibernation
- C) EC2 Auto Recovery
- D) Elastic IP

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Hibernation salva o conteúdo da RAM no volume EBS root, permitindo que a instância retome com o estado exato. O banco em memória estaria no mesmo ponto ao sair da hibernação. Instance Store perde dados ao parar. Auto Recovery substitui a instância em novo hardware, mas não preserva estado de memória. Elastic IP é sobre IP fixo.

**Conceito-chave:** Hibernation salva RAM no EBS para retomada de estado
</details>

## Questão 7
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Um sistema de streaming de vídeo precisa de volumes EBS com 40.000 IOPS consistentes, baixíssima latência e 99,999% de durabilidade para armazenar chunks de vídeo durante processamento. Qual tipo de EBS deve ser usado?

- A) gp3
- B) st1
- C) io2 Block Express
- D) sc1

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

io2 Block Express oferece até 256.000 IOPS com durabilidade de 99,999% e latência sub-milissegundo consistente. Para 40.000 IOPS, gp3 chega apenas a 16.000. st1 e sc1 são HDD com IOPS baixíssimo, inadequados para streaming de alta performance.

**Conceito-chave:** io2 Block Express para IOPS altíssimo e durabilidade máxima
</details>

## Questão 8
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Uma empresa usa instâncias T3 para hospedar microserviços de monitoramento que recentemente passaram a fazer cálculos intensivos contínuos. Os clientes reclamam de lentidão. Qual é a causa mais provável?

- A) A instância T3 está sem Elastic IP
- B) Os créditos de CPU burstable foram esgotados e a instância opera no baseline
- C) O volume EBS está cheio
- D) A instance metadata está desabilitada

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Instâncias T3 acumulam créditos de CPU quando abaixo do baseline e os consomem em bursts. Workloads continuamente intensivos esgotam os créditos e a performance cai ao baseline, que pode ser muito baixo (ex.: t3.micro tem baseline de 10% de vCPU). A solução é migrar para uma família sem burst (M, C) ou habilitar T3 Unlimited com custo adicional.

**Conceito-chave:** créditos de burst T3 esgotados em workload contínuo
</details>

## Questão 9
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Difícil

Uma empresa precisa implantar um banco de dados Cassandra em 18 instâncias EC2, organizadas de forma que falhas de rack afetem a menor quantidade de nós possível, mas o Cassandra possa usar informações de partição para replicação inteligente. Qual placement group deve ser usado?

- A) Cluster Placement Group
- B) Spread Placement Group
- C) Partition Placement Group
- D) Sem placement group, usando múltipas AZs

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Partition Placement Group foi desenhado exatamente para sistemas como Cassandra, HDFS e Kafka. Cada partição representa um rack distinto. O Cassandra pode usar a partição como zona de disponibilidade virtual para replicação awareness. Com 18 instâncias em 6 partições (3 por partição), uma falha de rack afeta apenas 3 nós. Spread limita a 7 instâncias/AZ. Cluster é inadequado para Cassandra por concentrar hardware.

**Conceito-chave:** Partition PG para databases distribuídas com awareness de rack
</details>

## Questão 10
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma empresa precisa que instâncias EC2 em produção possam ser acessadas para troubleshooting sem abrir a porta 22 ao mundo e sem bastion host. Qual solução elimina o vetor de ataque de SSH exposto?

- A) Abrir a porta 22 no security group apenas para o IP do administrador
- B) Usar AWS Systems Manager Session Manager para acesso sem porta 22
- C) Configurar VPN site-to-site para o laptop do administrador
- D) Criar EC2 Key Pair e compartilhar entre os admins

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Session Manager permite acesso ao shell da instância via console AWS ou CLI sem abrir nenhuma porta de entrada no security group, sem key pairs e com auditoria automática no CloudTrail e session logging no S3/CloudWatch. A alternativa A ainda expõe a porta 22. A alternativa C resolv parcialmente, mas não elimina o SSH. A alternativa D compartilha key pair, péssima prática.

**Conceito-chave:** Session Manager para acesso sem porta aberta
</details>

## Questão 11
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Uma startup usa múltiplos tipos de EC2 e também Lambda e Fargate. Ela quer um único plano de desconto flexível que cubra todos esses computados. Qual é a melhor opção?

- A) Reserved Instances Standard para cada tipo de instância individualmente
- B) Compute Savings Plans cobrindo EC2, Lambda e Fargate
- C) EC2 Savings Plans restritos à família e região
- D) Dedicated Hosts com RI 3 anos

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Compute Savings Plans é o mais flexível: aplica desconto a qualquer tipo EC2, família, região, SO, Lambda (acima de 0,5 GB) e Fargate, sem compromisso de tipo específico. EC2 Savings Plans tem desconto ligeiramente maior, mas é restrito à família e região. Reserved Instances exigem especificação por tipo. Dedicated Hosts têm finalidade diferente.

**Conceito-chave:** Compute Savings Plans para cobertura multi-serviço flexível
</details>

## Questão 12
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Um banco de dados MySQL gerenciado por conta própria em EC2 precisa de volume EBS com alto throughput sequencial para escritas de log binário. Qual tipo de EBS oferece o melhor custo-benefício para esse padrão de acesso?

- A) io2 Block Express
- B) gp3 com IOPS extras
- C) st1 — HDD Throughput Optimized
- D) sc1 — Cold HDD

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

st1 é otimizado para acesso sequencial de grandes blocos com throughput de até 500 MBps e é muito mais barato que SSD para esse uso. Logs binários são escritas sequenciais grandes — perfil ideal para st1. io2 seria desperdício para log sequencial. sc1 é ainda mais lento e para cold data. gp3 é SSD de uso geral, bom mas mais caro e com throughput menor para sequencial massivo.

**Conceito-chave:** st1 para I/O sequencial de alto throughput com custo baixo
</details>

## Questão 13
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Fácil

Uma empresa quer copiar uma AMI que usa customer-managed KMS CMK para outra região. Qual pré-requisito é obrigatório?

- A) A AMI deve ser pública antes da cópia
- B) A conta destino deve ter acesso à CMK original ou uma CMK equivalente deve ser usada na cópia
- C) A AMI precisa ser desencriptada antes da cópia
- D) Não é possível copiar AMIs criptografadas entre regiões

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Ao copiar uma AMI criptografada entre regiões, é necessário especificar uma CMK na região destino. Os snapshots EBS da AMI são re-encriptados com a chave da região destino. A conta solicitante precisa ter permissão tanto na CMK de origem (para descriptografar) quanto na CMK de destino (para re-encriptar). Não é possível copiar sem acesso às chaves, mas a alternativa D está errada pois a cópia é possível com as cmKs corretas.

**Conceito-chave:** CMK de destino necessária ao copiar AMI criptografada cross-region
</details>

## Questão 14
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Difícil

Uma empresa tem 100 instâncias On-Demand e quer reduzir custo. O padrão de uso mostra carga constante de 60 instâncias ao longo do dia com picos de 40 adicionais nos horários comerciais. Qual estratégia de compra equilibra melhor custo e disponibilidade?

- A) 100 instâncias Spot
- B) 60 Reserved Instances para a base + 40 On-Demand ou Spot para os picos
- C) 100 Reserved Instances Standard 3 anos
- D) Savings Plans Compute cobrindo apenas 20% do compromisso

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

A arquitetura ideal é cobrir a base consistente com RI ou Savings Plans (maior desconto) e usar On-Demand ou Spot para a capacidade variável. 100 Spot seria arriscado para carga crítica de base. 100 RI cobrindo 40 instâncias que às vezes ficam ociosas desperdiça o compromisso. Savings Plans com 20% de compromisso deixa 80% da base no On-Demand.

**Conceito-chave:** base estável com RI + burst com On-Demand/Spot
</details>

## Questão 15
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Um ambiente de produção usa uma instância EC2 como servidor de aplicação. Após um stop/start, o IP público da instância mudou e conexões existentes falharam. Qual é a solução mais direta para garantir IP público permanente?

- A) Habilitar Enhanced Networking
- B) Associar uma Elastic IP à instância
- C) Criar uma nova AMI com IP fixo embutido
- D) Migrar para instância com Instance Store

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Elastic IP é um IP público estático que permanece associado à conta mesmo após stop/start ou falha da instância. Pode ser rapidamente remapeado para uma instância substituta em caso de falha. Enhanced Networking é sobre performance de rede. AMI não define IP. Instance Store não tem relação com IP.

**Conceito-chave:** Elastic IP para IP público permanente na EC2
</details>
