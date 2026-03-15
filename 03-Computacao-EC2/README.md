# 03 Computação EC2

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

- identificar a família de instância EC2 mais adequada para um workload dado
- comparar os modelos de compra e selecionar o mais econômico para cada perfil de uso
- diferenciar EBS gp3, io2, st1 e sc1 por características de performance e custo
- explicar envelope de Instance Store vs EBS e suas implicações em falha
- escolher o tipo de placement group correto para cada requisito de latência ou isolamento
- descrever IMDSv2 e User Data e aplicá-los em questões de automação de instâncias

## 📚 Conceitos Fundamentais

### Famílias de Instância

A AWS organiza os tipos de instância EC2 em famílias, e cada família é otimizada para uma característica de hardware. Reconhecer a família pela letra inicial é essencial no exame.

**M — General Purpose (balanced)**: proporção equilibrada de CPU e memória. São as mais comuns para aplicações web, bancos de dados de médio porte e ambientes de desenvolvimento. Exemplo: m7g.xlarge.

**C — Compute Optimized**: alto desempenho de CPU. Indicada para processamento batch, encoding de vídeo, servers HPC, servidores de jogos e microserviços com muita computação. Exemplo: c7g.large.

**R — Memory Optimized**: grande quantidade de RAM em relação à CPU. Usada para bancos de dados in-memory como Redis e Memcached, SAP HANA, análise em tempo real. Exemplo: r7g.2xlarge.

**I — Storage Optimized**: alta IOPS e baixa latência de I/O local. Indicada para bancos NoSQL locais, data warehousing, sistemas de arquivos distribuídos como HDFS. Use quando o dado temporário precisa de acesso ultra-rápido. Exemplo: i4i.xlarge.

**P/G — Accelerated Computing**: GPUs para machine learning, renderização 3D, simulações científicas. P é voltado para treinamento de ML e G para inferência e gráficos. Exemplo: p4de.24xlarge.

**T — Burstable**: performance de baseline com créditos acumuláveis para picos. Ideal para cargas intermitentes como ambientes dev/test, microserviços de baixo tráfego. T3 e T4g têm modo unlimited para créditos ilimitados com custo adicional. Cuidado: workloads contínuos de alta CPU zeram os créditos e a performance cai ao baseline.

### Modelos de Compra

O modelo de compra é um dos tópicos mais cobrados em questões de otimização de custo.

**On-Demand**: pague pelo segundo ou hora sem compromisso. Maior custo unitário, zero planejamento necessário. Indicado para workloads imprevisíveis, desenvolvimento e testes.

**Reserved Instances (RI)**: compromisso de 1 ou 3 anos com desconto de até 72% vs On-Demand. Há três variantes: Standard RI (maior desconto, tipo e região fixos), Convertible RI (desconto menor, pode trocar tipo/SO dentro do período), e Zonal RI (reserva capacidade na AZ específica). Scheduled RI foi descontinuado.

**Savings Plans**: mais flexível que Reserved Instances. Você compromete um gasto horário em dólar por 1 ou 3 anos. Compute Savings Plans (desconto menor, cobre EC2, Lambda e Fargate, qualquer tipo, região e SO) vs EC2 Savings Plans (desconto maior, mas preso ao tipo e região). Não há conceito de "instância específica"; o desconto aplica automaticamente.

**Spot Instances**: compra de capacidade ociosa da AWS com desconto de até 90%. A AWS pode interromper a instância com 2 minutos de aviso quando precisar da capacidade de volta. Ideal para workloads tolerantes a falhas: batch, procesamento de dados, renderização, CI/CD. Não deve ser usada para bancos de dados stateful ou aplicações sem checkpoint.

**Spot Fleet**: coleção de instâncias Spot e opcionalmente On-Demand com uma capacidade alvo. Você define múltiplos launch pools (tipo, AZ) e a Fleet seleciona o mais barato. Estratégias: lowestPrice, diversified, capacityOptimized, priceCapacityOptimized (recomendada pela AWS).

**Dedicated Hosts**: servidor físico dedicado. Você tem visibilidade do socket/core, útil para licenciamento por socket (SQL Server, Oracle, Windows Server). Pode ser On-Demand ou Reserved.

**Dedicated Instances**: instâncias que rodam em hardware dedicado, mas sem controle do host específico. Cobre compliance que exige isolamento físico sem necessidade de visibilidade de sockets.

**Capacity Reservations**: reserva capacidade em uma AZ específica sem compromisso de desconto. Combina com Savings Plans ou Regional RI para cobrir custo.

### EC2 User Data e Instance Metadata

**User Data** é um script que executa uma única vez na primeira inicialização da instância (pelo cloud-init). Serve para instalar pacotes, configurar serviços e baixar artefatos. Roda como root. Para executar em toda inicialização é necessário configurar explicitamente pelo cloud-init ou usar MIME multi-part.

**Instance Metadata Service (IMDS)** expõe informações sobre a instância no endpoint especial `http://169.254.169.254`. A segunda versão, IMDSv2, é session-oriented e exige uma chamada prévia para obter um token com TTL. O token é então usado no header `X-aws-ec2-metadata-token` nas chamadas subsequentes. IMDSv2 protege contra SSRF, pois o token não pode ser capturado por um atacante que conseguir executar um request via SSRF. O exame cobra quando usar IMDSv2: sempre que o cenário mencionar segurança aprimorada de metadados ou proteção contra SSRF em instâncias EC2.

### AMIs

Amazon Machine Images são templates que definem o sistema operacional, configurações de software e mapeamento de dispositivos de uma instância. Podem ser criadas a partir de instâncias em execução (snapshot dos volumes EBS) ou de volumes EBS diretamente. 

Uma AMI pode ser copiada entre regiões, facilitando deployment multi-region. Pode ser compartilhada com outras contas ou tornada pública. A cópia de uma AMI cria um novo snapshot EBS na região destino. AMIs criptografadas só podem ser copiadas para contas que têm acesso à CMK usada.

Para o exame, AMIs costumam aparecer em contextos de launch templates, golden images para bootstrap consistente e estratégias de recuperação rápida.

### Amazon EBS — Elastic Block Store

EBS fornece volumes de bloco persistentes para instâncias EC2. O volume existe independentemente da instância — mesmo se a instância for terminada, o volume pode ser preservado.

**gp3**: geração atual de SSD de uso geral. Baseline de 3.000 IOPS e 125 MBps independentemente do tamanho. IOPS e throughput podem ser escalados separadamente do tamanho até 16.000 IOPS e 1.000 MBps. Custo menor que gp2 para IOPS adicionais.

**gp2**: geração anterior. IOPS vinculados ao tamanho: 3 IOPS/GB, mínimo 100, máximo 16.000 (requer volume de 5.333 GB). Possibilidade de burst para 3.000 IOPS usando créditos. Prefira gp3 para novos volumes.

**io2 Block Express**: IOPS provisionado de altíssima performance. Durabilidade de 99.999%, latência consistente, até 256.000 IOPS e 4.000 MBps. Suporte a multi-attach (múltiplas instâncias no mesmo volume — cluster-aware FS necessário).

**io1**: geração anterior de IOPS provisionado. Até 64.000 IOPS. Preferir io2 para novos casos.

**st1**: HDD otimizado para throughput. Bom para grandes volumes de dados sequenciais: logs, data lake, Hadoop. Até 500 MBps. Não pode ser volume de boot.

**sc1**: HDD frio (cold). Menor custo de EBS. Para dados raramente acessados. Até 250 MBps. Não pode ser volume de boot.

Snapshots EBS são incrementais e armazenados no S3 (transparente ao usuário). O primeiro snapshot copia tudo; os subsequentes apenas os blocos alterados. Podem ser copiados entre regiões. A criptografia é por volume: se a CMK é especificada ao criar o volume, todos os snapshots herdam a criptografia.

### Instance Store

Instance Store é armazenamento temporário diretamente nos discos físicos do servidor que hospeda a instância. Baixíssima latência e altíssimo IOPS (NVMe). A desvantagem é que os dados são perdidos quando a instância é parada, hibernada ou terminada. Use para cache, buffers e dados temporários que podem ser recreados — nunca para dados permanentes.

### Placement Groups

**Cluster Placement Group**: empacota instâncias fisicamente próximas em um rack ou grupo de racks na mesma AZ. Minimiza latência de rede (10 Gbps entre instâncias). Risco: uma falha de hardware pode afetar todas as instâncias. Indicado para HPC, aplicações de baixa latência e tráfego intenso entre nós.

**Spread Placement Group**: distribui instâncias em hardware distinto. Máximo de 7 instâncias por AZ por placement group. Maximiza disponibilidade: uma falha de rack afeta no máximo uma instância. Indicado para aplicações críticas onde cada instância não pode compartilhar ponto de falha.

**Partition Placement Group**: divide instâncias em partições, cada uma em rack separado. Até 7 partições por AZ e centenas de instâncias por grupo. A aplicação pode saber em qual partição cada instância está (via metadata). Indicado para HDFS, HBase, Cassandra, Kafka — sistemas distribuídos que têm awareness de topologia.

### Ciclo de Vida da Instância

**Stop/Start**: para instâncias com EBS como root. Os dados no EBS persistem. A instância pode ser movida para hardware diferente ao reiniciar. O IP público muda (a menos que use Elastic IP).

**Hibernate**: salva o conteúdo da RAM no volume EBS root (que deve ter espaço suficiente e ser criptografado). A instância pode retomar de onde parou, com o estado de memória intacto. Mais rápido que reboot completo. Não permanece em hibernação por mais de 60 dias.

**Terminate**: deleta a instância e, por padrão, o volume EBS root. Volumes adicionais podem ser configurados para persistir com `DeleteOnTermination=false`.

## 🏗️ Arquitetura e Componentes

```text
┌─────────────────────────────────────────────────────┐
│                  EC2 Instance                        │
│                                                      │
│  AMI (boot)                                          │
│  ┌──────────────┐    ┌────────────────────────────┐  │
│  │  EBS Root    │    │       Volumes EBS           │  │
│  │  (gp3/io2)   │    │  data, gp3, io2, st1, sc1  │  │
│  └──────────────┘    └────────────────────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │   Instance Store (NVMe) — temporário         │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  User Data (cloud-init)                              │
│  IMDS v2 → 169.254.169.254                           │
│  IAM Instance Profile → Role → STS temp credentials │
│                                                      │
│  Security Group (stateful)                           │
└─────────────────────────────────────────────────────┘

Placement Groups:
  Cluster:    [ i1 ][ i2 ][ i3 ]  ← 1 rack, low latency
  Spread:     [ i1 ] ... [ i2 ] ... [ i3 ]  ← racks distintos
  Partition:  P1[ i1,i2 ] P2[ i3,i4 ] P3[ i5,i6 ]  ← racks por partição
```

## ⚙️ Configurações Importantes para o Exame

| Item | Valor | Observação |
|---|---|---|
| Desconto máximo Spot | ~90% vs On-Demand | Workload tolerante a interrupção |
| Desconto máximo Standard RI 3 anos | ~72% | All Upfront, mesmo tipo e região |
| gp3 IOPS base | 3.000 | Independente do tamanho (diferente do gp2) |
| gp3 IOPS máximo | 16.000 | Com custo adicional por IOPS |
| io2 Block Express IOPS máximo | 256.000 | Volumes de alto desempenho |
| io2 durabilidade | 99,999% | Maior do EBS |
| Spread placement max por AZ | 7 instâncias | Por placement group |
| Hibernation duração máxima | 60 dias | RAM salva em EBS criptografado |
| IMDSv2 TTL padrão do token | 6 horas | Configurável na criação |
| Aviso de interrupção Spot | 2 minutos | Via metadata e EventBridge |

## 🔄 Comparativo de Serviços

| Modelo de Compra | Desconto | Compromisso | Ideal para |
|---|---|---|---|
| On-Demand | — | Nenhum | Workload imprevisível, dev/test |
| Reserved (Standard) | até 72% | 1 ou 3 anos | Produção estável, mesmo tipo |
| Reserved (Convertible) | até 66% | 1 ou 3 anos | Produção que pode mudar de tipo |
| Savings Plans (Compute) | até 66% | 1 ou 3 anos ($ horário) | EC2 + Lambda + Fargate, qualquer tipo |
| Savings Plans (EC2) | até 72% | 1 ou 3 anos ($ horário) | EC2 mesmo tipo e região |
| Spot | até 90% | Nenhum | Batch, CI/CD, processamento tolerante |
| Dedicated Host | variável | Pode ser RI | Licença por socket, compliance |
| Dedicated Instance | variável + taxa | Nenhum | Isolamento físico sem controle do host |

| EBS Type | Use Case | IOPS máximo | Throughput máx | Boot? |
|---|---|---:|---|---|
| gp3 | General purpose | 16.000 | 1.000 MBps | ✅ |
| gp2 | General (legacy) | 16.000 | 250 MBps | ✅ |
| io2 Block Express | OLTP crítico, baixa latência | 256.000 | 4.000 MBps | ✅ |
| io1 | IOPS provisionado (legacy) | 64.000 | 1.000 MBps | ✅ |
| st1 | Big data, logs, sequencial | 500 | 500 MBps | ❌ |
| sc1 | Cold, arquivamento barato | 250 | 250 MBps | ❌ |

## 💡 Dicas e Armadilhas do Exame

- T3 burstable em workload contínuo de CPU alta vai esgotar créditos e operar abaixo do esperado — Cuidado com questões que descrevem degradação de performance inesperada.
- Instance Store não persiste após stop ou terminate. Se a questão menciona "dados temporários de cache de alta performance", Instance Store pode ser a resposta; se menciona persistência, não.
- gp2 vincula IOPS ao tamanho. Se quiser IOPS altos sem volume grande, use gp3 (independência entre tamanho e IOPS).
- Spot Fleet com estratégia `capacityOptimized` reduz chance de interrupção; `lowestPrice` maximiza economia mas aumenta risco de interrupção.
- Dedicated Host é para licença por socket/core. Dedicated Instance é para compliance de isolamento físico sem visibilidade de host.
- O IP público de uma EC2 muda a cada stop/start. Para IP fixo, use Elastic IP.
- Placement Groups Cluster ficam em uma AZ. Se a questão pede distribuição multi-AZ, Cluster não serve.
- IMDSv2 é mais seguro que IMDSv1. A AWS recomenda só IMDSv2; em questões de segurança da infraestrutura, prefira IMDSv2 como resposta.

## 💡 Para o Exame

- Família M = balanced, C = compute, R = memory, I = storage, T = burstable. Se o workload é descrito, mapeie para a família.
- Produção estável de longo prazo = Reserved ou Savings Plans. Burst imprevisível = On-Demand. Tolerante a interrupção = Spot.
- IOPS alto e latência consistente = io2. Performance adequada e custo menor = gp3. Big data sequencial = st1. Custo mínimo sem performance = sc1.
- HPC entre nós = Cluster PG. Alta disponibilidade crítica por instância = Spread PG. HDFS/Cassandra/Kafka com awareness de rack = Partition PG.

## 📎 Links Relacionados

- [Questões do módulo](./questoes.md)
- [Flashcards do módulo](./flashcards.md)
- [Cheatsheet do módulo](./cheatsheet.md)
- [Casos de uso do módulo](./casos-de-uso.md)
- [Lab prático do módulo](./lab.md)
- [Links oficiais](./links.md)
- [Módulo 04: Alta Disponibilidade e Escalabilidade](../04-Alta-Disponibilidade-e-Escalabilidade/)
- [Módulo 05: Amazon S3 e Armazenamento](../05-Amazon-S3-e-Armazenamento/)
