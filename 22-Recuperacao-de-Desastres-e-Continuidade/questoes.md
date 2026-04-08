# Questoes - Recuperacao de Desastres e Continuidade

## 1)
Uma empresa exige RTO de minutos e RPO de segundos para aplicacao global. Qual estrategia e mais adequada?

A. Backup and Restore
B. Pilot Light
C. Warm Standby
D. Multi-Site Active/Active

Resposta: D
Motivo: menor RTO/RPO, apesar de maior custo.

## 2)
A aplicacao roda em uma unica regiao com RDS Multi-AZ. Qual risco permanece?

A. Falha de instancia
B. Falha de AZ
C. Desastre regional
D. Falha de disco

Resposta: C
Motivo: Multi-AZ nao cobre perda total da regiao.

## 3)
Qual servico simplifica replicacao continua de servidores para DR com baixa mudanca na aplicacao?

A. AWS Backup
B. AWS Elastic Disaster Recovery
C. AWS DataSync
D. CloudEndure Migration

Resposta: B
Motivo: servico especifico para DR de servidores.

## 4)
Para objetos S3 criticos, qual combinacao aumenta resiliencia e recuperacao de versoes apagadas acidentalmente?

A. S3 Standard + Glacier
B. Versioning + CRR
C. Intelligent-Tiering + MFA
D. Transfer Acceleration + ACL

Resposta: B
Motivo: versioning protege contra delete/logica e CRR protege contra falha regional.

## 5)
Em prova, a frase "least operational overhead" normalmente favorece:

A. Solucao self-managed em EC2
B. Processo manual de restore
C. Servicos gerenciados com automacao
D. Ferramentas third-party sem integracao

Resposta: C
Motivo: AWS privilegia operacao simplificada quando explicitado.