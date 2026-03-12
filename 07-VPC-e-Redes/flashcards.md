# Flashcards — Módulo 07: VPC e Redes

---

**P:** Quantos IPs a AWS reserva em cada subnet e quais são eles?
**R:** 5 IPs: .0 (network address), .1 (VPC router), .2 (DNS — Amazon DNS), .3 (reservado para uso futuro), .255 (broadcast — não usado mas reservado)

---

**P:** Security Groups são stateful ou stateless? E NACLs?
**R:** Security Groups: **stateful** (resposta de tráfego permitido entra/sai automaticamente). NACLs: **stateless** (precisam de regras explícitas para entrada E saída)

---

**P:** Qual é o tamanho máximo de CIDR block que pode ser atribuído a uma VPC?
**R:** /16 (65.536 endereços). Mínimo: /28 (16 endereços, mas apenas 11 utilizáveis). Uma VPC pode ter até 5 CIDR blocks associados

---

**P:** O que é um Internet Gateway (IGW) e o que é necessário para uma instância ser acessível pela internet?
**R:** IGW é componente horizontalmente escalável e redundante que conecta a VPC à internet. Para instância ser acessível: 1) subnet com rota 0.0.0.0/0 → IGW; 2) instância com IP público/EIP; 3) SG permitindo tráfego de entrada

---

**P:** Qual a diferença entre NAT Gateway e NAT Instance?
**R:** NAT Gateway: gerenciado pela AWS, altamente disponível na AZ, 45 Gbps, sem gerenciamento. NAT Instance: EC2 que você gerencia, pode ser usado como bastion também, mais barato para tráfego baixo, você gerencia HA

---

**P:** Para HA com NAT Gateway, quantos gateways criar?
**R:** Um NAT Gateway **por AZ**. NAT GW é zonal — se a AZ falhar, instâncias em outras AZs que dependem desse NAT GW perdem conectividade. Cada subnet privada por AZ deve ter rota para o NAT GW da sua própria AZ

---

**P:** VPC Peering é transitivo? Como fazer comunicação transitiva?
**R:** Peering é **não-transitivo**. A↔B e B↔C não permite A↔C via B. Para comunicação transitiva: use **Transit Gateway** (hub-and-spoke, cada VPC tem 1 attachment ao TGW, TGW roteia entre todas)

---

**P:** Qual a diferença entre Gateway Endpoint e Interface Endpoint (PrivateLink)?
**R:** Gateway Endpoint: gratuito, S3 e DynamoDB apenas, adicionado como rota na route table, não acessível de on-prem. Interface Endpoint: cobrado ($/hora + $/GB), cria ENI com IP privado, suporta todos serviços, acessível de on-prem

---

**P:** Quais são os 3 tipos de Virtual Interface (VIF) no Direct Connect?
**R:** Private VIF: recursos privados em uma VPC. Public VIF: endpoints públicos AWS (S3, EC2 public). Transit VIF: conecta ao Transit Gateway para múltiplas VPCs. (Hosted VIF: Virtual Interface com capacidade ≤ 1 Gbps compartilhada via parceiro)

---

**P:** O que é Link Aggregation Group (LAG) no Direct Connect?
**R:** Agrupa múltiplas conexões DX físicas (1 Gbps ou 10 Gbps) como link único lógico, aumentando bandwidth e redundância (failover automático entre as conexões do grupo). Todas as conexões devem ser no mesmo DX location e mesma velocidade

---

**P:** O que o VPC Flow Logs captura? E o que NÃO captura?
**R:** Captura: source/dest IP, port, protocol, bytes, packets, action (ACCEPT/REJECT), duration. NÃO captura: payload (conteúdo), tráfego DHCP, tráfego DNS para Route 53 Resolver, tráfego da instância para IMDS (169.254.169.254)

---

**P:** Qual é a função do Egress-Only Internet Gateway?
**R:** Permite que instâncias com IPv6 iniciem conexões para a internet (saída), mas bloqueia conexões iniciadas da internet (entrada). É o equivalente ao NAT Gateway mas para IPv6. IPv6 endereços são públicos por natureza, então o Egress-Only IGW provê privacidade de entrada

---

**P:** O que é AWS PrivateLink?
**R:** Tecnologia que permite expor serviços (SaaS, internos) via VPC Endpoint (Interface Endpoint) para outras VPCs sem peering, sem tráfego pela internet. Provedor cria NLB; consumidor cria Interface Endpoint. Não há restrição de CIDR overlap

---

**P:** Qual a diferença entre Site-to-Site VPN e Direct Connect?
**R:** Site-to-Site VPN: internet pública encriptada (IPSec), minutos para provisionar, throughput variável (~1,25 Gbps por túnel). Direct Connect: link físico dedicado, semanas para provisionar, latência consistente, throughput de 1/10/100 Gbps

---

**P:** O que é Accelerated Site-to-Site VPN?
**R:** Site-to-Site VPN que usa AWS Global Accelerator: tráfego VPN vai ao edge location AWS mais próximo pelo backbone privado da AWS (não internet pública). Reduz latência e melhora estabilidade da VPN

---

**P:** Em uma VPC, o que é o CIDR secundário e por que adicionar?
**R:** VPCs podem ter até 5 CIDRs associados (1 primário + 4 secundários). Útil quando o espaço de IPs originais se esgota sem precisar recriar a VPC. Limitações: não deve sobreposição com CIDRs de VPCs peered ou on-prem

---

**P:** Qual componente de rede isola security boundaries dentro de uma VPC?
**R:** A combinação de: **Subnets** (segmentação L3), **Security Groups** (firewall stateful na ENI), **NACLs** (firewall stateless na subnet) e **Route Tables** (controle de tráfego). Subnets públicas/privadas separam recursos expostos dos internos

---

**P:** Qual o tamanho máximo de uma VPC CIDR block?
**R:** /16 (65.536 endereços). Prefira usar CIDRs da faixa RFC 1918 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) para evitar conflitos com outros CIDRs da empresa

---

**P:** O que é o Transit Gateway Route Table?
**R:** Cada TGW tem uma ou mais route tables que controlam como o tráfego é distribuído entre os attachments. Permite isolamento: ex, VPCs de produção só comunicam entre si (route table separada) e não com VPCs de desenvolvimento

---

**P:** O que acontece se dois VPCs com CIDRs sobrepostos tentam fazer peering?
**R:** O peering falha — VPC Peering requer que os CIDRs dos dois VPCs **não se sobreponham**. Mesmo CIDR idêntico ou parcialmente sobreposto impede a criação do peering. Solução: re-endereçar uma das VPCs ou usar AWS PrivateLink (que não requer routeability entre CIDRs)
