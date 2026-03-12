# Flashcards — Computação EC2

> Revise um cartão de cada vez. Tente responder antes de revelar a resposta.

---

**P:** Quais são as principais famílias de instâncias EC2 e seus casos de uso?  
**R:** M (general purpose, balanced CPU/RAM), C (compute-optimized: HPC, batch, ML inference), R (memory-optimized: in-memory DB, cache), I (storage-optimized: NVMe local, high IOPS DB), P/G (GPU: ML training, rendering), T (burstable: apps que raramente precisam de pico, mas têm baseline baixo).

---

**P:** O que é o mecanismo de CPU credits nas instâncias T?  
**R:** Instâncias T acumulam créditos quando CPU está abaixo do baseline e os gastam quando acima. T3/T3a com Unlimited mode podem usar CPU além dos créditos acumulados pagando por crédito extra. Se Standard mode e créditos zerados, CPU é limitada ao baseline.

---

**P:** Qual é a diferença entre On-Demand e Reserved Instance?  
**R:** On-Demand: paga por hora/segundo, sem compromisso, mais caro. Reserved Instance: compromisso de 1 ou 3 anos, desconto de até 72%. Standard RI não pode mudar tipo, região ou OS. Convertible RI pode trocar atributos, menor desconto (~54%). Ambos podem ser pagos up-front, parcial ou monthly.

---

**P:** Qual é a diferença entre Compute Savings Plans e EC2 Instance Savings Plans?  
**R:** Compute Savings Plans: até 66% desconto, aplica a qualquer família, região, OS e tamanho, também cobre Lambda e Fargate. EC2 Instance Savings Plans: até 72%, locked à família + região específica (ex: m5 em us-east-1), mais desconto, menos flexibilidade.

---

**P:** O que é Spot Instance e quando usá-la?  
**R:** Instâncias Spot usam capacidade ociosa da AWS a até 90% de desconto. Podem ser interrompidas com 2 minutos de aviso. Ideais para: batch jobs tolerantes a falha, ML training distribuído, renderização, testes. NÃO usar para: banco de dados primário, sessões web com estado.

---

**P:** O que é Spot Fleet e como funciona?  
**R:** Spot Fleet gerencia um conjunto de instâncias Spot (e opcionalmente On-Demand). Você define um pool de tipos/regiões e a estratégia: `lowestPrice` (escolhe o mais barato), `capacityOptimized` (escolhe o pool com mais capacidade disponível — recomendado para reduzir interrupções), `diversified` (distribui por todos os pools).

---

**P:** Qual é a diferença entre Dedicated Host e Dedicated Instance?  
**R:** Dedicated Host: você aluga um servidor físico específico, tem visibilidade do socket/núcleo, pode usar licenças BYO-L (por socket/core como SQL Server, Windows Server). Dedicated Instance: instância no HW dedicado à sua conta mas sem controle do host físico. Host é mais caro mas necessário para compliance de licenciamento.

---

**P:** O que é IMDSv2 e por que é preferível ao IMDSv1?  
**R:** IMDSv2 (Instance Metadata Service v2) usa um modelo baseado em token: primeiro você faz PUT para obter token com TTL, depois usa o token no header `X-aws-ec2-metadata-token`. Protege contra ataques SSRF onde um processo na instância poderia capturar metadata sem o token. Para desabilitar IMDSv1, configure `HttpTokens: required`.

---

**P:** Qual é a diferença entre gp2 e gp3 no EBS?  
**R:** gp2: IOPS vinculados ao tamanho (3 IOPS/GB, máx 16.000), burst. gp3: baseline de 3.000 IOPS e 125 MB/s independente do tamanho, pode provisionar até 16.000 IOPS e 1.000 MB/s separadamente. gp3 é mais barato (~20%) e flexível — use por padrão.

---

**P:** Quando usar io1/io2 vs gp3?  
**R:** io2: até 64.000 IOPS por volume, io2 Block Express até 256.000 IOPS, 99,999% durabilidade (vs 99,8-99,9% do gp). Use para bancos de dados de alto desempenho (Oracle, SQL Server, workloads que precisam >16.000 IOPS garantidos). io1: versão anterior, durabilidade 99,8%.

---

**P:** O que é Instance Store e quais são suas características?  
**R:** Armazenamento NVMe efêmero diretamente no host físico da instância. IOPS muito altos (ex: i3.large tem ~100K IOPS). Dados PERDIDOS em: stop, terminate ou falha de hardware. Persistem em: restart. Ideal para cache de alto desempenho, buffer temporário, scratch disk.

---

**P:** Quais são os três tipos de Placement Group?  
**R:** Cluster: todas as instâncias num mesmo rack e AZ, latência <1ms entre instâncias, ideal para HPC e MPI. Spread: cada instância num rack separado, máximo 7 por AZ, para alta disponibilidade crítica. Partition: grupos de instâncias por partição (cada partição = rack separado), até 7 partições por AZ, para HDFS/Cassandra/Kafka.

---

**P:** O que acontece a uma instância EC2 quando fazemos Stop (vs Hibernate)?  
**R:** Stop: instância para, RAM é perdida, EBS root preservado, cobrado pelo EBS não pela instância. Hibernate: RAM é gravada no EBS root volume (precisa ter espaço e estar encriptado), ao iniciar o SO retoma do ponto onde parou. Limite de hibernate: 60 dias.

---

**P:** O que são AMIs e quais são as categorias de virtualização?  
**R:** Amazon Machine Image é um template de instância (OS, dados, configurações). HVM (Hardware Virtual Machine): emulação completa de hardware — única suportada por tipos modernos. PV (Paravirtual): legado. AMIs podem ser compartilhadas entre contas, copiadas entre regiões. Para copiar uma AMI não encriptada para encriptada, especifique uma CMK na cópia.

---

**P:** O que é EC2 User Data?  
**R:** Script (shell ou cloud-init) executado uma única vez no primeiro boot da instância com privilégios de root. Acessível em `http://169.254.169.254/latest/user-data`. Útil para instalar software, configurar aplicação. Não é re-executado em reinicializações (a menos que configurado explicitamente).

---

**P:** Como consultar os metadados de uma instância EC2 de dentro dela com IMDSv2?  
**R:** 
```bash
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```
Sem o token (IMDSv1), apenas: `curl http://169.254.169.254/latest/meta-data/instance-id`

---

**P:** O que é Capacity Reservation e quando usar?  
**R:** Reserva capacidade em uma AZ específica sem desconto de preço (paga On-Demand). Garante que a capacidade estará disponível quando precisar. Pode combinar com Savings Plans ou Regional RI para obter desconto. Útil para DR, compliance ou workloads críticos que não podem ficar sem capacidade.

---

**P:** Qual é o máximo de IOPS do io2 Block Express e st1 vs sc1?  
**R:** io2 Block Express: até 256.000 IOPS, 4.000 MB/s. st1 (throughput): 40 MB/s por TB baseline, 250 MB/s por TB burst, máx 500 MB/s — acesso sequencial (logs, Kafka, data warehouse). sc1 (cold HDD): 12 MB/s por TB baseline, mais barato — dados raramente acessados.

---

**P:** O que é Elastic IP e quando deve ser evitado?  
**R:** Elastic IP é um IPv4 estático que pode ser associado/reassociado a instâncias ou ENIs. Cobrado quando NÃO está em uso (0,005 USD/hora). Evitar em arquiteturas escaláveis — use DNS (Route 53) em vez de depender de IP fixo. Use apenas quando cliente precisa de IP na allowlist.

---

**P:** O que é AWS Systems Manager Session Manager?  
**R:** Permite acesso interativo a instâncias EC2 sem abrir porta 22 (SSH) ou 3389 (RDP). A instância precisa ter o SSM Agent instalado e role IAM com `AmazonSSMManagedInstanceCore`. Acesso via Console ou CLI. Auditado pelo CloudTrail e sessões podem ser logadas no S3/CloudWatch Logs.
