# Cheatsheet - Recuperacao de Desastres e Continuidade

## Mapa rapido de decisao

| Requisito | Melhor padrao | Observacao de prova |
|---|---|---|
| Menor custo, aceita horas de indisponibilidade | Backup and Restore | Foco em snapshots, backups e restore automatizado |
| RTO de minutos com custo controlado | Pilot Light | Banco e dados replicados, compute sobe no desastre |
| Baixo downtime com operacao simples | Warm Standby | Ambiente reduzido sempre ativo em segunda regiao |
| Quase zero downtime e perda de dados | Active/Active | Usa roteamento global e dados replicados em tempo quase real |

## Servicos chave

- Route 53: failover, health checks, latency routing
- AWS Backup: backup centralizado, vault lock, cross-account
- Elastic Disaster Recovery: replicacao continua de servidores
- S3: versioning, CRR, lifecycle, object lock
- RDS/Aurora: backups automaticos, snapshots, global database
- DynamoDB: PITR, global tables

## Armadilhas comuns

- confundir alta disponibilidade (multi-AZ) com DR regional
- ignorar custo de replicacao cross-region
- escolher arquitetura complexa sem necessidade de negocio
- esquecer de testar runbooks de failover/failback

## Frases gatilho do exame

- "regional disaster" => multi-region
- "least operational overhead" => servico gerenciado
- "recover quickly" => warm standby ou active/active
- "cost-effective" => backup and restore ou pilot light