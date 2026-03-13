# Links e Recursos — VPC e Redes (Módulo 05)

## Documentação Oficial AWS

- [Amazon VPC — Documentação Oficial](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [VPC — Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
- [VPC — Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [VPC — NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [VPC — Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [VPC — Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [VPC Peering — Documentação](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [AWS Transit Gateway — Documentação](https://docs.aws.amazon.com/transit-gateway/latest/tgw-ug/what-is-transit-gateway.html)
- [VPC Flow Logs — Documentação](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [AWS PrivateLink (VPC Endpoints) — Documentação](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
- [AWS Direct Connect — Documentação](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html)
- [VPN Site-to-Site — Documentação](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
- [VPC Reachability Analyzer](https://docs.aws.amazon.com/vpc/latest/reachability/what-is-reachability-analyzer.html)

## FAQs

- [Amazon VPC FAQ](https://aws.amazon.com/vpc/faqs/)
- [AWS Transit Gateway FAQ](https://aws.amazon.com/transit-gateway/faqs/)
- [AWS Direct Connect FAQ](https://aws.amazon.com/directconnect/faqs/)

## Whitepapers

- [AWS VPC Connectivity Options Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/welcome.html)
- [Security in Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/security.html)

## Artigos de Blog AWS

- [One VPC vs Multiple VPCs: When to use each](https://aws.amazon.com/blogs/networking-and-content-delivery/vpc-sharing-a-new-approach-to-multiple-accounts-and-vpc-management/)
- [Transit Gateway vs VPC Peering](https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-transit-gateway/)
- [How to use VPC Flow Logs for security analysis](https://aws.amazon.com/blogs/security/how-to-use-amazon-vpc-flow-logs/)

## Ferramentas de Estudo

- [Tutorials Dojo — Amazon VPC Cheat Sheet](https://tutorialsdojo.com/amazon-vpc/)
- [CIDR Calculator](https://cidr.xyz/)
- [Subnet Calculator](https://www.subnet-calculator.com/)

## Comparativo Chave

| Conceito | Security Group | NACL |
|---------|---------------|------|
| Nível | Instância | Subnet |
| Estado | Stateful | Stateless |
| Regras | Apenas ALLOW | ALLOW e DENY |
| Avaliação | Todas as regras | Por número (ordem) |
| Retorno | Automático | Precisa de regra explícita |
