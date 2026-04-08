# 22 Recuperacao de Desastres e Continuidade

## Objetivos do modulo

- dominar RTO e RPO e mapear cada estrategia de DR ao requisito de negocio
- diferenciar Backup and Restore, Pilot Light, Warm Standby e Multi-Site Active/Active
- selecionar servicos AWS para continuidade: AWS Backup, Elastic Disaster Recovery, Route 53, Aurora Global Database, DynamoDB Global Tables e S3 CRR
- responder questoes de prova equilibrando resiliencia, custo e complexidade operacional

## Conceitos fundamentais

Recuperacao de desastres no SAA-C03 nao e apenas backup. A prova cobra desenho arquitetural que continue operando quando houver falha de AZ, regiao, servico gerenciado ou erro humano.

- RTO (Recovery Time Objective): tempo maximo aceitavel para restaurar o servico.
- RPO (Recovery Point Objective): perda maxima aceitavel de dados em tempo.

Quanto menor o RTO e RPO, maior tende a ser o custo e a complexidade.

## Estrategias de DR

1. Backup and Restore
- custo mais baixo
- restauracao mais lenta
- RTO alto, RPO moderado/alto
- ideal para workloads nao criticas

2. Pilot Light
- componentes criticos minimos ficam ativos na regiao secundaria
- aplicacao e capacidade escalam apenas no desastre
- RTO medio, RPO baixo/medio

3. Warm Standby
- ambiente reduzido sempre ativo em outra regiao
- rapido scale-up no desastre
- RTO baixo, RPO baixo

4. Multi-Site Active/Active
- duas regioes ativas atendendo trafego
- failover quase imediato
- RTO e RPO muito baixos
- maior custo e operacao mais complexa

## Servicos e padroes mais cobrados

- Route 53 Failover + Health Checks para redirecionamento entre regioes.
- AWS Elastic Disaster Recovery para replicacao continua de servidores e cutover rapido.
- AWS Backup para politica central de backup e cofres cross-account.
- S3 Versioning + CRR para objetos e resiliencia regional.
- RDS/Aurora: snapshots, read replicas cross-region e Aurora Global Database.
- DynamoDB: PITR e Global Tables para baixa latencia e continuidade.
- EBS snapshots cross-region para restauracao de EC2.

## Dicas de exame

- Multi-AZ nao resolve desastre regional; para isso a resposta precisa de multi-region.
- Se o enunciado destaca "menor custo", Backup and Restore ou Pilot Light tende a vencer.
- Se pede "minimo downtime", Warm Standby ou Active/Active geralmente e melhor.
- Se diz "menor esforco operacional", prefira servicos gerenciados e automacao nativa.
- Se pede preservar DNS durante failover, pense em Route 53 com health checks.

## Links relacionados

- [Cheatsheet](./cheatsheet.md)
- [Casos de uso](./casos-de-uso.md)
- [Questoes](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Links oficiais](./links.md)