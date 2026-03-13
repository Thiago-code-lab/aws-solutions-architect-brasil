# Cheatsheet — Módulo 07: VPC e Redes

## Security Groups vs NACLs

| | Security Group | NACL |
|---|---|---|
| Nível | Instância/ENI | Subnet |
| Estado | **Stateful** (resposta automática) | **Stateless** (regras de entrada E saída separadas) |
| Regras | Apenas Allow | Allow e **Deny** |
| Processamento | Todas as regras avaliadas | Ordem numérica; para na 1ª que corresponde |
| Default | Bloqueia tudo de entrada; permite tudo de saída | Allow tudo (entrada e saída) |
| Associação | N SGs por ENI, 1 SG por múltiplas ENIs | 1 NACL por subnet; 1 subnet por NACL |

## Componentes de Rede VPC

| Componente | Função |
|---|---|
| Internet Gateway (IGW) | Saída + entrada da internet para subnet pública (IPv4/IPv6) |
| NAT Gateway | Saída de internet para subnet privada (IPv4 apenas); zonal |
| Egress-Only IGW | Saída de internet para subnet privada (IPv6 apenas) |
| Virtual Private Gateway (VGW) | Endpoint AWS para Site-to-Site VPN e DX Private VIF |
| Customer Gateway (CGW) | Representa dispositivo on-prem na VPN configuration |
| Transit Gateway (TGW) | Hub central para múltiplas VPCs e conexões on-prem (transitivo) |

## VPC Endpoints

| | Gateway Endpoint | Interface Endpoint (PrivateLink) |
|---|---|---|
| Serviços | S3, DynamoDB (apenas) | ~200+ serviços AWS + custom |
| Como funciona | Entrada na route table | ENI com IP privado da VPC |
| Custo | Gratuito | Cobrado ($/hora + $/GB) |
| On-premises | Não acessível | Acessível via DX/VPN |
| DNS | Não altera DNS | Private DNS option → resolve para IP ENI |

## Direct Connect (DX)

| Tipo | Capacidade | Provisioning | Via |
|---|---|---|---|
| Dedicated Connection | 1/10/100 Gbps | Semanas | Direto com AWS |
| Hosted Connection | 50 Mbps a 10 Gbps | Mais rápido | Via parceiro APN |

**Virtual Interfaces (VIF):**
- **Private VIF** → VPC via VGW ou DX Gateway
- **Public VIF** → Endpoints públicos AWS (S3, EC2 IPs)
- **Transit VIF** → Transit Gateway (múltiplas VPCs)

**DX + VPN:** adicione IPSec VPN sobre DX para criptografia de dados em trânsito (DX por si só não criptografa)

## Site-to-Site VPN

| | Site-to-Site VPN | Accelerated VPN |
|---|---|---|
| Caminho | Internet pública (HTTPS/IPSec) | Edge AWS → backbone privado |
| Latência | Variável | Reduzida |
| Custo | Menor | SGA + por horas ou transferência adicional |
| Throughput | ~1,25 Gbps por túnel | ~1,25 Gbps por túnel |

- Cada conexão VPN Google tem **2 túneis** (redundância; recomenda-se HA via 2 customer gateways)
- VGW = lado AWS; CGW = lado on-premises

## Roteamento: Peering vs Transit Gateway

| | VPC Peering | Transit Gateway |
|---|---|---|
| Trânsito | Não transitivo | **Transitivo** |
| Escala | N*(N-1)/2 conexões para N VPCs | N attachments por TGW |
| Cross-region | Sim (peering inter-região) | Sim (TGW Peering) |
| Custo | Sem custo de conexão | Custo por attachment + por GB |
| CIDR overlap | Impede peering | Não permite comunicação (mas attachment possível) |

## IPs Reservados por Subnet AWS
```
10.0.0.0/24 (256 IPs) → 251 disponíveis:
  10.0.0.0  → Network address
  10.0.0.1  → VPC router (default gateway)
  10.0.0.2  → DNS (Amazon DNS = VPC_CIDR_base + 2)
  10.0.0.3  → Reservado pela AWS (futuro)
  10.0.0.255 → Broadcast
```

## VPC Flow Logs — Campos Principais
```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
```
- Destinos: CloudWatch Logs ou S3
- NÃO captura: DNS (Route 53), DHCP, IMDS (169.254.x.x), Windows license activation

## Dicas Rápidas de Prova
- NAT Gateway: criar **1 por AZ** para HA; subnet privada de cada AZ roteia para o NAT GW da sua AZ
- VPC Peering: CIDRs não podem se sobrepor; não é transitivo; funciona cross-region
- Transit Gateway: substitui malha de peerings para topologias grandes; suporta multicast
- Gateway Endpoint: adiciona rota na route table; não precisa de IP público para acessar S3/DynamoDB
- Interface Endpoint com Private DNS: resolve nome público do serviço para IP privado na VPC
- PrivateLink: ideal para expor serviços SaaS ou internos para outras VPCs sem peering
- Direct Connect: NÃO criptografa por padrão; adicione VPN sobre DX para criptografia
- Egress-Only IGW: para IPv6 saída apenas (NAT não existe para IPv6)
- Route Tables: uma por subnet; subnets sem associação explícita usam a main route table
