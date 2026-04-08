# Links e Recursos — RDS e Bancos Relacionais (Módulo 06)

## Documentação Oficial AWS

- [Amazon RDS — Documentação Oficial](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [Amazon Aurora — Documentação](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)
- [RDS Multi-AZ — Configuração](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [RDS Read Replicas — Documentação](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)
- [RDS Backups Automáticos e Snapshots](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
- [RDS Parameter Groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html)
- [RDS Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html)
- [Amazon ElastiCache — Documentação](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html)
- [Amazon Redshift — Documentação](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html)
- [RDS Proxy — Documentação](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)

## FAQs

- [Amazon RDS FAQ](https://aws.amazon.com/rds/faqs/)
- [Amazon Aurora FAQ](https://aws.amazon.com/rds/aurora/faqs/)
- [Amazon ElastiCache FAQ](https://aws.amazon.com/elasticache/faqs/)

## Whitepapers

- [Running MySQL on AWS](https://docs.aws.amazon.com/whitepapers/latest/best-practices-for-wordpress-on-aws/best-practices-for-wordpress-on-aws.html)
- [AWS Database Migration Best Practices](https://docs.aws.amazon.com/whitepapers/latest/database-migration/database-migration.html)

## Artigos de Blog AWS

- [Amazon Aurora vs RDS: Which one should you use?](https://aws.amazon.com/blogs/database/is-amazon-rds-for-mysql-a-better-choice-than-amazon-aurora-mysql-compatible-edition/)
- [RDS Proxy: Connection pooling for serverless](https://aws.amazon.com/blogs/compute/using-amazon-rds-proxy-with-aws-lambda/)
- [ElastiCache caching strategies](https://aws.amazon.com/blogs/database/work-with-caching-strategies-in-elasticache/)

## Ferramentas de Estudo

- [Tutorials Dojo — Amazon RDS Cheat Sheet](https://tutorialsdojo.com/amazon-relational-database-service-amazon-rds/)
- [Tutorials Dojo — Amazon Aurora Cheat Sheet](https://tutorialsdojo.com/amazon-aurora/)

## Comparativo Multi-AZ vs Read Replica

| Critério | Multi-AZ | Read Replica |
|---------|----------|-------------|
| Propósito | HA/DR | Performance (leitura) |
| Sincronização | Síncrona | Assíncrona |
| Leitura ativa? | Não (standby) | Sim |
| Failover auto? | Sim | Não |
| Cross-region? | Não (mesmo Multi-AZ) | Sim |
| Promovível? | Não | Sim (independente) |
| Downtime no failover | ~1-2 min | N/A |
