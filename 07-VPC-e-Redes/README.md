# Módulo 07 — VPC e Redes

## Fundamentos da VPC

Uma VPC (Virtual Private Cloud) é uma rede virtual isolada logicamente dentro da AWS. Cada conta recebe uma **Default VPC** por região.

### Componentes Principais
```
VPC (10.0.0.0/16)
├── Subnet Pública (10.0.1.0/24) — AZ-a
│     ├── Internet Gateway → rota 0.0.0.0/0
│     └── Instâncias com IP público
├── Subnet Privada (10.0.2.0/24) — AZ-a
│     ├── NAT Gateway (saída para internet, sem entrada)
│     └── Instâncias sem IP público
├── Subnet Privada (10.0.3.0/24) — AZ-b
│     └── (redundância multi-AZ)
├── Route Tables (uma por subnet ou compartilhada)
├── Network ACLs (stateless, nível de subnet)
├── Security Groups (stateful, nível de instância/ENI)
├── DHCP Options Set
├── Elastic IPs
└── ENIs (Elastic Network Interfaces)
```

### CIDRs
- **Primário**: atribuído na criação (não pode ser alterado), ex: `10.0.0.0/16`
- **Secundário**: até 5 blocos adicionais podem ser adicionados
- Tamanho de subnet: `/28` (mínimo) a `/16` (máximo)
- AWS **reserva 5 IPs** por subnet (.0 rede, .1 VPC router, .2 DNS, .3 reservado, .255 broadcast)

---

## Security Groups vs Network ACLs

| Característica | Security Group | Network ACL |
|---|---|---|
| Nível | Instância / ENI | Subnet |
| Estado | **Stateful** (resposta automática) | **Stateless** (regras de entrada e saída independentes) |
| Regras | Apenas ALLOW | ALLOW e DENY |
| Avaliação | Todas as regras avaliadas | Regras em ordem de número (1, 100, 200...) — primeira que corresponde |
| Padrão novo | Nega tudo, precisa de regras | Allow All inbound/outbound |
| Referência | Pode referenciar outro SG | Apenas CIDR |

**Ordem de avaliação de tráfego de entrada:**
1. NACL (subnet level) → 2. Security Group (instance level)

---

## NAT Gateway vs NAT Instance

| Característica | NAT Gateway | NAT Instance (legado) |
|---|---|---|
| Gerenciado | AWS gerencia | Você gerencia (EC2) |
| Escala | Automática (até 100 Gbps) | Manual (tipo de instância) |
| Disponibilidade | Altamente disponível por AZ | Ponto único de falha |
| Custo | Por hora + por GB processado | Por hora EC2 + EIP |
| Source/Dest Check | Automático | Deve desabilitar manualmente |
| Security Groups | Não (usa NACL da subnet) | Sim (SG na instância) |

**Importante:** NAT Gateway é zonal — para HA real, **uma NAT GW por AZ** com route table por AZ apontando para a NAT GW local.

---

## VPC Peering

Conexão de rede privada entre duas VPCs (mesma ou diferentes regiões/contas).

- Tráfego usa rede privada AWS (não pela internet)
- **Não transitive**: VPC A ↔ VPC B e VPC B ↔ VPC C **não** implica VPC A ↔ VPC C
- CIDRs não podem sobrepor
- Requer atualização de route tables em ambas as VPCs
- Suporta cross-account e cross-region

```
VPC A ←──── peering ────→ VPC B
VPC B ←──── peering ────→ VPC C
VPC A ←────────────────── VPC C  ← PRECISA de peering separado
```

---

## Transit Gateway (TGW)

Hub central que permite conectar múltiplas VPCs e redes on-premises:

```
VPC A ──┐
VPC B ──┤
VPC C ──┼──→ Transit Gateway ←──→ VPN / Direct Connect
VPC D ──┤
VPC E ──┘
```

- **Transitive** por padrão (A ↔ TGW ↔ C = A ↔ C, diferente do peering)
- **Route Tables** do TGW controlam qual VPC pode comunicar com qual
- Suporta **multicast** (único serviço AWS a suportar)
- TGW Resource Sharing via AWS RAM (compartilhar entre contas da Org)
- **TGW Peering**: conecta TGWs de diferentes regiões
- Ideal para hub-and-spoke com dezenas/centenas de VPCs

---

## VPC Endpoints

Permitem acesso privado a serviços AWS **sem sair da VPC** (sem IGW, NAT, VPN, DX):

### Gateway Endpoints
- Gratuitos
- Apenas para **S3 e DynamoDB**
- Adicionados como entrada na route table (prefixo do serviço → endpoint ID)
- Não podem ser extendidos para on-premises ou VPC peering

### Interface Endpoints (PrivateLink)
- ENI com IP privado na VPC
- Cobrado por hora + por GB processado
- Suporta **a maioria dos serviços AWS** e serviços de terceiros
- Acesso via DNS privado (Endpoint-specific DNS names ou Private DNS habilitado)
- Podem ser acessados de on-premises via VPN/DX ✅

---

## AWS Direct Connect (DX)

Conexão de rede dedicada entre on-premises e AWS:

| Tipo | Velocidade | SLA | Provisionamento |
|---|---|---|---|
| Dedicated Connection | 1, 10, 100 Gbps | AWS SLA | Diretamente com AWS, semanas |
| Hosted Connection | 50 Mbps – 10 Gbps | Parceiro | Através de parceiro, mais flexível |

### Virtual Interfaces (VIFs)
- **Public VIF**: acessa endpoints públicos da AWS (S3, DynamoDB, API do KMS)
- **Private VIF**: acessa IPs privados da VPC
- **Transit VIF**: acessa Transit Gateway (para múltiplas VPCs via DX)

### Direct Connect Gateway
- Conecta um DX connection a múltiplas VPCs em **diferentes regiões** (mesma conta)
- Número de Private VIFs reduzido (1 conexão → múltiplas regiões/VPCs)

### High Availability com DX
- **Backup DX**: dois circuitos em diferentes pontos de presença
- **DX + VPN como backup**: Site-to-Site VPN como failover (menor custo, maior latência)
- **LAG (Link Aggregation Group)**: múltiplos circuitos físicos agregados como um lógico

---

## Site-to-Site VPN

Conexão encriptada (IPSec) entre on-premises e VPC:
- **Virtual Private Gateway (VGW)**: termina a VPN na VPC
- **Customer Gateway (CGW)**: dispositivo físico/software on-premises
- VPN passa pela internet pública (latência variável, encriptado)
- Throughput: até ~1,25 Gbps (limitado pelo VGW)
- **Accelerated Site-to-Site VPN**: usa Global Accelerator para reduzir latência (sai da internet na PoP mais próxima)

---

## VPC Flow Logs

- Captura metadados de tráfego IP (não o payload) em VPCs, subnets ou ENIs
- Destinos: **CloudWatch Logs** ou **S3**
- Inclui: IP origem/destino, porta, protocolo, bytes, pacotes, ação (ACCEPT/REJECT), ID da ENI
- Não captura: DNS queries, DHCP, metadata (169.254.x.x), AWS link-local
- Latência: ~10–15 min após captura (near-real-time)

---

## IPv6 na VPC

- Bloco `/56` atribuído à VPC (AWS escolhe, não há ranges privados IPv6)
- Subnets recebem `/64`
- **Todos os endereços IPv6 são públicos** (não há NAT para IPv6)
- **Egress-Only Internet Gateway**: permite saída IPv6 sem receber tráfego de entrada (equivalente ao NAT GW para IPv6)

---

## Elastic Network Interface (ENI)

Interface de rede virtual que pode ser:
- Movida entre instâncias (para failover mantendo IP privado e security groups)
- Múltiplas ENIs por instância (multi-homed: ex: mgmt network + app network)
- Cada ENI pode ter múltiplos IPs privados e um Elastic IP

---

## Dicas de Prova

- **Security Group = stateful** — não precisa de regra de retorno; **NACL = stateless** — precisa de regra de entrada E saída
- **VPC Peering não é transitivo** — cada par precisa de peering próprio; para muitas VPCs, use TGW
- **NAT Gateway por AZ** para HA — se todas as subnets privadas usarem uma NAT GW em uma AZ, há single point of failure
- **Gateway Endpoint** (S3, DynamoDB) = gratuito; **Interface Endpoint** = cobrado por hora
- **Direct Connect + VPN** = dual layer of redundancy (DX primário + VPN fallback)
- Interface Endpoints acessíveis de on-prem via DX/VPN; Gateway Endpoints não
- **TGW** suporta transitividade — recomendado quando você tem muitas VPCs (>5) para conectar
- **Egress-only IGW** = NAT para IPv6 (apenas saída, sem entrada pública)
- **VPC Flow Logs** não captura conteúdo dos pacotes — apenas metadados (IP, porta, ação)
- Flow Logs com REJECT = SG ou NACL bloqueou; Flow Logs com ACCEPT mas problema de conectividade = problema na aplicação ou SO
