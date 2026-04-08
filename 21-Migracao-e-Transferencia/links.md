# Links e Recursos — Migração e Transferência (Módulo 15)

## Documentação Oficial AWS

- [AWS Database Migration Service (DMS) — Documentação](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [AWS Schema Conversion Tool (SCT) — Documentação](https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP_Welcome.html)
- [AWS Migration Hub — Documentação](https://docs.aws.amazon.com/migrationhub/latest/ug/whatishub.html)
- [AWS Application Migration Service (MGN) — Documentação](https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html)
- [AWS Snowball Edge — Documentação](https://docs.aws.amazon.com/snowball/latest/developer-guide/whatisedge.html)
- [AWS Snowflake / Snowcone — Comparativo](https://aws.amazon.com/snow/)
- [AWS DataSync — Documentação](https://docs.aws.amazon.com/datasync/latest/userguide/what-is-datasync.html)
- [AWS Transfer Family (SFTP/FTP/FTPS) — Documentação](https://docs.aws.amazon.com/transfer/latest/userguide/what-is-aws-transfer.html)
- [Amazon S3 Transfer Acceleration — Documentação](https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html)

## FAQs

- [AWS Database Migration Service FAQ](https://aws.amazon.com/dms/faqs/)
- [AWS Snowball Family FAQ](https://aws.amazon.com/snowball/faqs/)
- [AWS DataSync FAQ](https://aws.amazon.com/datasync/faqs/)

## Whitepapers

- [AWS Migration Whitepaper (6 R's Strategy)](https://docs.aws.amazon.com/whitepapers/latest/aws-migration-whitepaper/welcome.html)
- [AWS Database Migration Best Practices](https://docs.aws.amazon.com/whitepapers/latest/database-migration/database-migration.html)
- [Cloud Migration — AWS CAF (Cloud Adoption Framework)](https://docs.aws.amazon.com/whitepapers/latest/overview-aws-cloud-adoption-framework/welcome.html)

## Artigos de Blog AWS

- [Migrating to AWS: The 6 R's strategy explained](https://aws.amazon.com/blogs/enterprise-strategy/6-strategies-for-migrating-applications-to-the-cloud/)
- [Zero-downtime migration with DMS and CDC](https://aws.amazon.com/blogs/database/migrating-oracle-to-amazon-aurora-postgresql-using-aws-dms/)
- [When to use AWS Snowball vs DataSync vs Storage Gateway](https://aws.amazon.com/blogs/storage/evaluating-aws-services-for-your-data-migration/)

## Vídeos re:Invent

- [Migrating Large Scale Databases to AWS (DAT307)](https://www.youtube.com/results?search_query=aws+reinvent+migrating+databases+DMS+DAT307)
- [Large-scale migration strategies (MIG201)](https://www.youtube.com/results?search_query=aws+reinvent+large+scale+migration+MIG201)
- [Moving data to the cloud with Snow Family](https://www.youtube.com/results?search_query=aws+reinvent+snow+family+data+migration)

## Ferramentas de Estudo

- [Tutorials Dojo — AWS DMS Cheat Sheet](https://tutorialsdojo.com/aws-database-migration-service/)
- [Tutorials Dojo — AWS Snowball Family Cheat Sheet](https://tutorialsdojo.com/aws-snowball-edge/)
- [AWS Skill Builder — Cloud Migration Foundations](https://explore.skillbuilder.aws/learn/course/external/view/elearning/10)

## Os 6 R's de Migração — Resumo

| Estratégia | Descrição | Esforço |
|-----------|-----------|---------|
| **Rehost** (Lift-and-Shift) | Mover sem alterar código | Baixo |
| **Replatform** (Lift-Tinker-Shift) | Pequenas otimizações | Médio |
| **Repurchase** | Mudar para SaaS | Baixo |
| **Refactor** (Re-architect) | Redesenhar arquitetura | Alto |
| **Retire** | Descomissionar | Nenhum |
| **Retain** | Deixar on-premises | Nenhum |

## Quando Usar Snowball vs DataSync vs Direct Connect

| Cenário | Ferramenta |
|---------|-----------|
| > 10 TB ou sem banda disponível | Snowball Edge |
| Transferência contínua após migração | DataSync |
| Conectividade dedicada on-premises | Direct Connect |
| Transferência S3 cross-region | S3 Replication |
| Migração de banco com zero downtime | DMS + CDC |
