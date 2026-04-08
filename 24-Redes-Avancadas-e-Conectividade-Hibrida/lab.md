# Lab - TGW + VPN (conceitual guiado)

## Objetivo
Montar conectividade centralizada entre VPCs e preparar conectividade hibrida.

## Passos

1. Crie duas VPCs (App e Shared).
2. Crie um Transit Gateway.
3. Anexe as duas VPCs ao TGW.
4. Ajuste route tables das subnets para trafego inter-VPC via TGW.
5. Valide comunicacao privada entre instancias.
6. (Opcional) Crie VPN connection simulada com Customer Gateway para estudo de parametros.
7. Revise metricas e logs de conectividade.

## Validacao

- comunicacao entre VPCs via rotas do TGW
- isolamento por route tables quando necessario
- entendimento do fluxo de failover em VPN

## Limpeza

- remover attachments e TGW
- remover VPCs e recursos auxiliares