# Lab - DR Multi-Region Basico

## Objetivo
Implementar um failover simples com Route 53 e replicacao de dados em S3.

## Passos

1. Crie dois buckets S3 em regioes diferentes.
2. Habilite versioning em ambos.
3. Configure CRR do bucket primario para o secundario.
4. Publique uma pagina estatica em duas regioes (S3 static website ou EC2 minima).
5. Configure Route 53 Failover Record:
- Primary apontando para workload principal
- Secondary apontando para workload secundaria
6. Crie Health Check para o endpoint primario.
7. Simule falha (interromper endpoint primario) e valide failover.
8. Documente RTO observado.

## Validacao

- Objetos replicam para segunda regiao
- DNS muda para endpoint secundario apos falha
- Aplicacao volta ao primario apos recuperacao

## Limpeza

- remover records/health checks Route 53
- excluir recursos e buckets para evitar custo