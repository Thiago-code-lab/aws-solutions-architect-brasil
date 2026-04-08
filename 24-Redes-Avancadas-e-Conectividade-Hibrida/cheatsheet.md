# Cheatsheet - Redes Avancadas e Conectividade Hibrida

## Escolha rapida

| Cenario | Solucao recomendada |
|---|---|
| 2 VPCs com troca simples de trafego | VPC Peering |
| dezenas de VPCs e contas | Transit Gateway |
| publicar servico interno para varias contas | PrivateLink |
| conectar filial rapidamente | Site-to-Site VPN |
| throughput alto e latencia previsivel | Direct Connect |

## Regras importantes

- VPC Peering: sem transitive routing
- TGW: roteamento central com route tables separadas
- PrivateLink: conecta consumidor ao endpoint de servico privado
- DX: usar link redundante e VPN como contingencia
- Resolver inbound/outbound endpoints: DNS hibrido

## Armadilhas de prova

- usar peering em malha grande (nao escala)
- esquecer overlapping CIDR (bloqueia peering/TGW)
- assumir que Security Group filtra trafego de internet de subnet publica sem rota correta
- confundir NACL stateful (na verdade e stateless)

## Frases gatilho

- "multiple VPCs across accounts" => Transit Gateway
- "privately access service" => PrivateLink
- "consistent network performance" => Direct Connect
- "encrypted connection over internet" => VPN