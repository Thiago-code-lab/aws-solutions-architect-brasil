# 24 Redes Avancadas e Conectividade Hibrida

## Objetivos do modulo

- dominar conectividade entre VPCs, contas e datacenter on-premises
- diferenciar VPC Peering, Transit Gateway, PrivateLink, VPN e Direct Connect
- desenhar redes com resiliencia, seguranca e baixa latencia
- resolver questoes de prova com foco em roteamento, isolamento e custo de transferencia

## Conceitos fundamentais

No SAA-C03, redes avancadas aparecem quando o cenario envolve multiplas VPCs, multiplas contas, conectividade hibrida e servicos privados.

## Componentes chave

- VPC Peering: conecta duas VPCs diretamente; nao suporta transitive routing.
- AWS Transit Gateway (TGW): hub-and-spoke para muitas VPCs e VPNs.
- AWS PrivateLink: expor/consumir servico privado sem peering completo.
- Site-to-Site VPN: tunel IPSec pela internet.
- AWS Direct Connect (DX): link dedicado de baixa variabilidade para on-premises.
- Route 53 Resolver endpoints: DNS hibrido entre AWS e on-premises.

## Quando usar cada um

- poucas VPCs e topologia simples: VPC Peering
- muitas VPCs/contas com roteamento central: TGW
- publicar servico interno para outras contas com menor superficie: PrivateLink
- conectividade rapida sem circuito dedicado: VPN
- conectividade estavel de alto throughput: Direct Connect (com VPN de backup)

## Dicas de exame

- peering nao e transitive; TGW resolve isso.
- se o enunciado pede acesso a servico privado sem abrir CIDR completo, pense em PrivateLink.
- DX costuma aparecer com requisito de latencia mais previsivel e trafego alto.
- para alta disponibilidade hibrida, combine DX + VPN backup.
- Security Group e stateful; NACL e stateless (armadilha recorrente).

## Links relacionados

- [Cheatsheet](./cheatsheet.md)
- [Casos de uso](./casos-de-uso.md)
- [Questoes](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Links oficiais](./links.md)