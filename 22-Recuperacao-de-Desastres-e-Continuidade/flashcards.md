# Flashcards - Recuperacao de Desastres e Continuidade

1. RTO significa?
- Tempo maximo para restaurar o servico.

2. RPO significa?
- Janela maxima de perda de dados.

3. Multi-AZ cobre desastre regional?
- Nao.

4. Menor custo de DR?
- Backup and Restore.

5. Estrategia com ambiente minimo ativo em segunda regiao?
- Pilot Light.

6. Estrategia com ambiente reduzido sempre ativo?
- Warm Standby.

7. Menor RTO/RPO em geral?
- Active/Active multi-region.

8. Servico de DR de servidores AWS?
- AWS Elastic Disaster Recovery.

9. Protecao de objetos contra exclusao acidental?
- S3 Versioning.

10. Replicacao de objetos entre regioes?
- S3 CRR.

11. Failover DNS gerenciado?
- Route 53 com health checks.

12. RDS Multi-AZ e para?
- Alta disponibilidade regional, nao DR multi-region.

13. DynamoDB para DR global?
- Global Tables.

14. O que testar periodicamente em DR?
- Runbook de failover e failback.

15. Frase de prova: "regional disaster"?
- Exige desenho multi-region.