# Questões de Prova — Módulo 07: VPC e Redes

<!-- Domínio SAA-C03: Design Secure Architectures / Design Resilient Architectures -->

---

**1.** Uma empresa tem VPCs em 10 regiões AWS e precisa que todas comuniquem entre si. Qual é a solução mais escalável?

- A) VPC Peering entre todos os pares (N*(N-1)/2 conexões)
- B) Transit Gateway com peering entre regiões
- C) VPN Site-to-Site conectando todas as VPCs
- D) Direct Connect com hosted connection para cada VPC

<details>
<summary>Resposta</summary>
**B — Transit Gateway**
Com 10 VPCs, VPC Peering exigiria 45 conexões (10×9/2) e **não é transitivo** (cada VPC precisa de conexão direta com cada outra). Transit Gateway centraliza o roteamento em hub-and-spoke: cada VPC tem apenas 1 attachment ao TGW, e o TGW é transitivo. TGW Peering permite conectar TGWs entre regiões.
</details>

---

**2.** Qual a diferença fundamental entre Security Groups e Network ACLs (NACLs)?

- A) Security Groups atuam na subnet; NACLs atuam na instância
- B) Security Groups são stateful; NACLs são stateless
- C) NACLs permitem regras Allow; Security Groups só fazem Deny
- D) Security Groups processam todas as regras; NACLs param na primeira que corresponde

<details>
<summary>Resposta</summary>
**B — Security Groups são stateful; NACLs são stateless**
SG stateful: se tráfego de entrada é permitido, a resposta de saída é automaticamente permitida (não é necessária regra de saída). NACL stateless: tráfego de entrada e saída são avaliados independentemente — você precisa de regras explícitas para ambas as direções. SGs atuam na **instância/ENI**; NACLs atuam na **subnet**.
</details>

---

**3.** Uma instância EC2 em subnet privada precisa fazer downloads de atualizações de software da internet. Qual componente é necessário?

- A) Internet Gateway attached à subnet privada
- B) NAT Gateway na subnet pública com rota na subnet privada
- C) Egress-Only Internet Gateway para IPv4
- D) VPC Endpoint para o repositório de software

<details>
<summary>Resposta</summary>
**B — NAT Gateway na subnet pública**
NAT Gateway permite que instâncias em subnet privada iniciem conexões para a internet (saída), mas bloqueia conexões iniciadas da internet (entrada). Deve ser criado na subnet **pública** (com IGW) e a rota 0.0.0.0/0 da subnet privada aponta para o NAT GW. Egress-Only IGW é para **IPv6** apenas.
</details>

---

**4.** Uma empresa quer que instâncias EC2 em uma VPC acessem o Amazon S3 sem tráfego passar pela internet. Qual solução mais simples e econômica?

- A) VPC Interface Endpoint para S3
- B) NAT Gateway + rota para S3
- C) VPC Gateway Endpoint para S3
- D) AWS PrivateLink para S3

<details>
<summary>Resposta</summary>
**C — VPC Gateway Endpoint para S3**
Gateway Endpoints para S3 e DynamoDB são **gratuitos** e são adicionados como entrada na route table (não usam ENI). Interface Endpoints (PrivateLink) custam por hora + por GB. Para S3, o Gateway Endpoint é a solução mais simples e econômica. Interface Endpoint S3 é necessário apenas quando precisa de acesso de on-prem via DX/VPN.
</details>

---

**5.** VPC Peering não permite tráfego transitivo. O que isso significa na prática?

- A) VPC A faz peering com B, e B com C: A não pode comunicar com C via B
- B) Tráfego no peering usa criptografia assimétrica
- C) Peering não funciona entre regiões diferentes
- D) Peering não permite tráfego UDP, apenas TCP

<details>
<summary>Resposta</summary>
**A — Correto**
Não-transitivo: A↔B e B↔C existem, mas A não alcança C através de B. Para A↔C, precisa de um peering direto A↔C. Para roteamento transitivo, use **Transit Gateway**. Peering pode ser cross-region (inter-region peering). Não há restrição de protocolo (TCP, UDP, ICMP funcionam).
</details>

---

**6.** Uma empresa quer conectar seu datacenter à AWS com largura de banda dedicada de 10 Gbps e baixa latência consistente. Qual serviço?

- A) Site-to-Site VPN via internet pública
- B) AWS Direct Connect (Dedicated Connection)
- C) AWS Direct Connect (Hosted Connection)
- D) Accelerated VPN com Global Accelerator

<details>
<summary>Resposta</summary>
**B — Direct Connect Dedicated Connection**
Dedicated Connection: 1 Gbps, 10 Gbps ou 100 Gbps por fiber conectada diretamente ao AWS DX location. Alta capacidade, baixa latência consistente, sem internet pública. Hosted Connection (via parceiro) oferece capacidades menores (50 Mbps a 10 Gbps via parceiro). VPN usa internet pública — latência variável e menor throughput.
</details>

---

**7.** Qual é a diferença entre Public VIF, Private VIF e Transit VIF no Direct Connect?

- A) Public VIF → endpoints públicos AWS; Private VIF → VPC via VGW; Transit VIF → múltiplas VPCs via TGW
- B) Private VIF é criptografado; os outros não são
- C) Transit VIF permite acesso à internet; outros não
- D) Todos são equivalentes, apenas nomes diferentes para billing

<details>
<summary>Resposta</summary>
**A — Correto**
- **Private VIF**: acessa recursos privados em uma VPC (via Virtual Private Gateway ou Direct Connect Gateway)
- **Public VIF**: acessa endpoints públicos AWS (S3, EC2 public IPs, API endpoints) sem internet
- **Transit VIF**: conecta ao AWS Transit Gateway para acessar múltiplas VPCs em uma ou mais regiões
Nenhum VIF é criptografado por padrão — adicione IPSec VPN sobre DX para criptografia.
</details>

---

**8.** Uma empresa precisa de alta disponibilidade para Direct Connect. Em qual configuração o failover é automático?

- A) Uma única conexão DX com Link Aggregation Group (LAG)
- B) Duas conexões DX em dois DX Locations diferentes + Site-to-Site VPN como backup
- C) DX Hosted Connection com múltiplos VIFs
- D) DX com CloudFront como CDN de backup

<details>
<summary>Resposta</summary>
**B — Dois DX Locations + VPN Backup**
Para máxima resiliência: 2 conexões físicas em **2 locais DX diferentes** (proteção contra falha física). VPN Site-to-Site como backup para falha de DX (failover automático via BGP). LAG agrupa múltiplas conexões no mesmo local — protege contra falha de porta/fibra mas não de local.
</details>

---

**9.** Um servidor on-premises precisa acessar serviços internos na VPC via Interface Endpoint (PrivateLink). O que é necessário habilitar?

- A) Nada — Interface Endpoints são acessíveis de on-prem por padrão
- B) Private DNS resolution e conexão via Direct Connect/VPN
- C) Gateway Endpoint com BGP routing
- D) VPC Peering entre on-prem e VPC

<details>
<summary>Resposta</summary>
**B — Private DNS + DX ou VPN**
Interface Endpoints com Private DNS habilitado resolvem o nome do serviço para o IP privado do ENI do endpoint. On-premises pode acessar via Direct Connect ou VPN. Gateway Endpoints (S3/DynamoDB) NÃO são acessíveis de on-prem — só Interface Endpoints. VPC Peering não conecta on-prem.
</details>

---

**10.** O que o VPC Flow Logs captura?

- A) Conteúdo dos pacotes (payload) de todas as conexões
- B) Metadados de tráfego IP (endereço, porta, protocolo, bytes, ACCEPT/REJECT)
- C) Logs de DNS queries feitas dentro da VPC
- D) CloudTrail API calls de recursos dentro da VPC

<details>
<summary>Resposta</summary>
**B — Metadados de tráfego IP**
VPC Flow Logs captura metadados: source/destination IP, port, protocol, bytes, packets, duration, action (ACCEPT/REJECT). **Não captura payload** (conteúdo do pacote). Pode ser enviado para CloudWatch Logs ou S3. DNS queries são capturadas pelo Route 53 Resolver DNS Logs. CloudTrail é API calls, não tráfego de rede.
</details>

---

**11.** Uma instância EC2 na sua VPC precisa acessar um serviço SaaS de um parceiro hospedado em VPC diferente (conta diferente), sem expor o serviço à internet. Qual solução?

- A) VPC Peering entre as contas
- B) AWS PrivateLink (Interface Endpoint para o serviço do parceiro)
- C) Transit Gateway com RAM (Resource Access Manager) entre contas
- D) Site-to-Site VPN entre as duas VPCs

<details>
<summary>Resposta</summary>
**B — AWS PrivateLink**
PrivateLink permite que um provedor exponha seu serviço via NLB e os consumidores criam Interface Endpoints na própria VPC para acessar. O tráfego não passa pela internet e não há risco de CIDR overlap (ao contrário de VPC Peering). Ideal para serviços SaaS multi-tenant entre contas.
</details>

---

**12.** Uma empresa quer permitir comunicação bidirecional IPv6 entre instâncias na VPC e a internet. Qual componente precisa adicionar?

- A) NAT Gateway com IPv6 habilitado
- B) Internet Gateway (IGW) — já suporta IPv6 bidirecionalmente
- C) Egress-Only Internet Gateway apenas
- D) NAT64 Gateway

<details>
<summary>Resposta</summary>
**B — Internet Gateway**
O IGW suporta IPv6 natively e permite **tráfego bidirecional** (entrada e saída de internet para IPv6). Egress-Only IGW é para **saída apenas** IPv6 (como NAT para IPv6 — bloqueando entrada de internet). Não existe NAT Gateway para IPv6 (NAT não é necessário pois IPv6 tem espaço de endereçamento abundante).
</details>

---

**13.** Qual é o número máximo de IPs disponíveis em uma subnet /28 na AWS?

- A) 16
- B) 14
- C) 11
- D) 13

<details>
<summary>Resposta</summary>
**C — 11**
Subnet /28 = 16 endereços totais. AWS reserva **5 IPs** por subnet: .0 (network), .1 (VPC router), .2 (DNS), .3 (futuro), .255 (broadcast). 16 - 5 = **11 IPs utilizáveis**.
</details>

---

**14.** Uma empresa usa Site-to-Site VPN para conectar on-prem à AWS. Precisa de baixíssima latência e máximo throughput no VPN. Qual recurso habilitar?

- A) BGP dynamic routing
- B) Accelerated VPN com Global Accelerator
- C) VPN com Dead Peer Detection (DPD)
- D) Multiple VPN tunnels em LAG

<details>
<summary>Resposta</summary>
**B — Accelerated VPN**
Accelerated Site-to-Site VPN usa AWS Global Accelerator para rotear tráfego VPN pelos edge locations da AWS (rede backbone AWS) até o VGW, reduzindo latência em até 60% vs. internet pública. DPD é para detectar falha de peer, não para performance. LAG não se aplica a VPN tunnels.
</details>

---

**15.** Uma organização criou 3 VPCs isoladas: Dev, Staging e Prod. Elas precisam comunicar com um VPC de serviços compartilhados (DNS, monitoramento, etc) mas NÃO entre si. Qual a arquitetura mais simples?

- A) Transit Gateway com route tables separadas bloqueando comunicação entre Dev/Staging/Prod
- B) 3 VPC Peerings: Dev↔Shared, Staging↔Shared, Prod↔Shared
- C) VPN Site-to-Site entre cada VPC e Shared
- D) Interface Endpoints nos VPCs Dev/Staging/Prod para serviços do Shared

<details>
<summary>Resposta</summary>
**B — VPC Peerings diretos com Shared**
3 peerings diretos é simples para este caso: Dev↔Shared, Staging↔Shared, Prod↔Shared. Como peering não é transitivo, Dev, Staging e Prod automaticamente ficam isolados entre si (não há rota Dev→Prod via Shared). Transit Gateway funcionaria mas é mais complexo e tem custo maior para apenas 4 VPCs com roteamento simples.
</details>
