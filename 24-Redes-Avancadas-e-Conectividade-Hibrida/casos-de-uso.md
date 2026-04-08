# Casos de Uso - Redes Avancadas e Conectividade Hibrida

## Caso 1: Grupo empresarial multi-conta

Cenario:
- 50 VPCs em varias contas
- necessidade de comunicacao controlada

Arquitetura:
- Transit Gateway central
- route tables separadas por dominio
- inspeção de trafego em VPC de seguranca

## Caso 2: SaaS privado para clientes enterprise

Cenario:
- clientes em contas AWS diferentes
- sem exposicao publica

Arquitetura:
- NLB + Endpoint Service
- consumo via Interface VPC Endpoint (PrivateLink)

## Caso 3: Datacenter para AWS com baixa variacao de latencia

Cenario:
- cargas sensiveis a variacao de rede
- alto volume de dados

Arquitetura:
- AWS Direct Connect principal
- Site-to-Site VPN como backup
- BGP para failover

## Caso 4: Integracao DNS hibrida

Cenario:
- aplicacoes on-premises precisam resolver zonas privadas na AWS

Arquitetura:
- Route 53 Resolver inbound endpoint
- conditional forwarding no DNS on-premises