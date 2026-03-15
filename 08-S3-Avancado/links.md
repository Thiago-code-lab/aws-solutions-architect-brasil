# Links e Recursos — S3 Avançado (Módulo 08)

## Documentação Oficial AWS

- [Amazon S3 — Documentação Oficial](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [S3 — Versionamento](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [S3 — Políticas de Ciclo de Vida](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [S3 — Replicação Cross-Region (CRR)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [S3 — Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html)
- [S3 — Notificações de Eventos](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html)
- [S3 — Multipart Upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [S3 Transfer Acceleration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html)
- [S3 — Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [S3 Object Lock e WORM](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)

## FAQs

- [Amazon S3 FAQ](https://aws.amazon.com/s3/faqs/)

## Whitepapers

- [Security Best Practices for Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)

## Artigos de Blog AWS

- [Optimizing S3 costs with lifecycle policies](https://aws.amazon.com/blogs/storage/optimizing-amazon-s3-with-storage-class-analysis/)
- [How to use presigned URLs for accessing S3](https://aws.amazon.com/blogs/developer/generating-amazon-s3-presigned-urls-with-sse-kms/)
- [S3 Cross-Region Replication use cases](https://aws.amazon.com/blogs/storage/use-amazon-s3-replication-to-help-improve-data-resiliency/)

## Ferramentas de Estudo

- [Tutorials Dojo — Amazon S3 Cheat Sheet](https://tutorialsdojo.com/amazon-s3/)
- [AWS S3 Storage Classes Calculator](https://aws.amazon.com/s3/storage-classes/)

## Comparativo Classes de Armazenamento S3

| Classe | Acesso | Disponibilidade | Durabilidade | Min. Duração | Custo |
|--------|--------|-----------------|-------------|--------------|-------|
| S3 Standard | Frequente | 99.99% | 11 9s | Nenhuma | Alto |
| S3 Standard-IA | Infrequente | 99.9% | 11 9s | 30 dias | Médio |
| S3 One Zone-IA | Infrequente | 99.5% | 11 9s (1 AZ) | 30 dias | Baixo |
| S3 Glacier IR | Arquivo | 99.9% | 11 9s | 90 dias | Muito baixo |
| S3 Glacier Flexible | Arquivo | 99.99% | 11 9s | 90 dias | Baixíssimo |
| S3 Glacier Deep | Arquivo frio | 99.99% | 11 9s | 180 dias | Mínimo |
| S3 Intelligent | Variável | 99.9% | 11 9s | Nenhuma | Variável |
