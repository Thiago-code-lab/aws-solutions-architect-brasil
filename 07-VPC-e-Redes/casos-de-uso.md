# Casos de Uso — Módulo 07: VPC e Redes

## Caso 1: VPC Multi-Tier com Subnet Pública e Privada

**Cenário:** Aplicação web 3-tier: Web (público), App (privado), DB (privado isolado).

**Arquitetura:**
```
VPC: 10.0.0.0/16
  │
  ├── Subnet Pública us-east-1a (10.0.1.0/24)
  │     ├── ALB (Internet-facing)
  │     └── NAT Gateway (1a) ← rota para Internet Gateway
  │
  ├── Subnet Pública us-east-1b (10.0.2.0/24)
  │     ├── ALB (segundo AZ)
  │     └── NAT Gateway (1b)
  │
  ├── Subnet Privada App us-east-1a (10.0.10.0/24)
  │     └── EC2 App Servers → rota 0.0.0.0/0 → NAT GW (1a)
  │
  ├── Subnet Privada App us-east-1b (10.0.11.0/24)
  │     └── EC2 App Servers → rota 0.0.0.0/0 → NAT GW (1b)
  │
  ├── Subnet DB us-east-1a (10.0.20.0/24)
  │     └── RDS Primary (sem rota de saída para internet)
  │
  └── Subnet DB us-east-1b (10.0.21.0/24)
        └── RDS Standby (Multi-AZ)

VPC Endpoints:
  Gateway Endpoint: S3 e DynamoDB → adicionado nas route tables privadas
  Interface Endpoint: Secrets Manager, CloudWatch (para subnet privada)
```

---

## Caso 2: Conectividade Híbrida com Direct Connect + VPN Backup

**Cenário:** Empresa com datacenter on-premises precisa de conectividade privada de alta velocidade para AWS com failover automático.

**Arquitetura:**
```
                    Direct Connect (10 Gbps) ← primário
On-Premises ──────────────────────────────────── AWS (VGW)
                                │                    │
                                │                    └── VPC (Private VIF)
                    Site-to-Site VPN ← backup via internet
                    (failover automático via BGP)

DX Gateway:
  On-premises → DX Connection → DX Gateway → VGW (VPC us-east-1)
                                           → VGW (VPC us-west-2)
  [Um único DX acessa múltiplas VPCs em múltiplas regiões]
```

**Configuração de failover BGP:**
- DX usa BGP AS Path prepending menor (preferido)
- VPN usa AS Path mais longo (usado apenas se DX cair)
- BFD (Bidirectional Forwarding Detection) detecta falhas em sub-segundo

---

## Caso 3: Conectar 50 VPCs com Transit Gateway

**Cenário:** Empresa com 50 VPCs (dev, staging, prod por serviço) + VPN on-premises. Requisito: Prod VPCs não se comunicam com Dev.

**Arquitetura com TGW Route Tables:**
```
Transit Gateway
  ├── Attachment: VPC-Prod-1 → Route Table: PROD
  ├── Attachment: VPC-Prod-2 → Route Table: PROD
  ├── Attachment: VPC-Dev-1  → Route Table: DEV
  ├── Attachment: VPC-Dev-2  → Route Table: DEV
  ├── Attachment: VPC-Shared → Route Table: SHARED (propagado em PROD e DEV)
  └── Attachment: VPN (on-prem) → Route Table: HYBRID

Route Table PROD: rotas para VPC-Prod-* e VPC-Shared
Route Table DEV:  rotas para VPC-Dev-* e VPC-Shared
[Sem rotas cruzadas → Prod e Dev isolados automaticamente]
```

---

## Caso 4: Exposição de Serviço SaaS via PrivateLink

**Cenário:** Empresa A quer vender uma API para Empresa B sem usar internet pública e sem VPC Peering (evitar CIDR conflicts).

**Arquitetura:**
```
Empresa A (Provider)                    Empresa B (Consumer)
  VPC: 10.0.0.0/16                        VPC: 10.0.0.0/16 (mesmo CIDR ok!)
    └── NLB (Service Endpoint)                └── Interface VPC Endpoint
          └── API Service (EC2/ECS)                 └── DNS: api.xxxxx.vpce.amazonaws.com
                                                          → IP privado na VPC da Empresa B
Endpoint Service:
  com.amazonaws.vpce.us-east-1.xxxxxxxx
  [Empresa A aceita a conexão; Empresa B cria o Interface Endpoint]
```

**PrivateLink vs VPC Peering:**
- PrivateLink: apenas o serviço específico é acessível; tráfego unidirecional (consumidor → provedor)
- Sem conflito de CIDR possível: cada empresa usa seu próprio espaço de IPs

---

## Caso 5: VPC Flow Logs para Análise de Segurança

**Cenário:** Equipe de segurança precisa investigar tráfego suspeito e detectar comunicação com IPs maliciosos.

**Arquitetura:**
```
VPC Flow Logs → CloudWatch Log Group "/vpc/flow-logs"
                    └── CloudWatch Insights Queries:
                          - Por IP: fields srcAddr | filter srcAddr like "1.2.3.4"
                          - REJECT analysis: filter action = "REJECT" | stats count by dstPort
                          - Top talkers: stats sum(bytes) by srcAddr | sort by sum desc

VPC Flow Logs → S3 Bucket (para análise histórica)
    └── Amazon Athena → SQL queries sobre logs particionados por data
          └── QuickSight → Dashboards de segurança

GuardDuty: usa VPC Flow Logs + DNS Logs automaticamente para ML-based threat detection
```

**Análise típica com Athena:**
```sql
SELECT srcaddr, dstaddr, dstport, action, sum(bytes) as bytes
FROM vpc_flow_logs
WHERE action = 'REJECT' AND day >= '2024-01-01'
GROUP BY 1, 2, 3, 4
ORDER BY bytes DESC LIMIT 10
```
