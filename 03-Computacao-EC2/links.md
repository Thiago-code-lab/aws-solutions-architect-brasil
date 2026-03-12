# Links de Referência — Computação EC2

## Documentação Oficial AWS

### EC2 Fundamentals
- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/) — Guia completo do EC2
- [Instance Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) — Todos os tipos e famílias
- [EC2 Instance Type Explorer](https://aws.amazon.com/ec2/instance-explorer/) — Comparação interativa de tipos

### Purchasing Options
- [On-Demand Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html)
- [Reserved Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html) — Standard e Convertible
- [Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html) — Guia completo Spot
- [Spot Fleet](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet.html) — Configurar Spot Fleet
- [Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/) — Compute vs EC2 Instance Savings Plans
- [Dedicated Hosts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html) — BYO-L e compliance

### Instance Metadata e User Data
- [Instance Metadata (IMDS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) — IMDSv1 vs IMDSv2
- [Transitioning to Using Instance Metadata Service Version 2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) — Migrar para IMDSv2
- [User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) — Scripts de inicialização

### AMIs
- [Amazon Machine Images](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) — Criar, compartilhar, copiar
- [Copying an AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html) — Cross-region copy

### EBS
- [EBS Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html) — Comparativo detalhado gp2/gp3/io1/io2/st1/sc1
- [EBS-Optimized Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-optimized.html) — Instâncias com I/O dedicado
- [Amazon EBS io2 Block Express](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/provisioned-iops.html) — Até 256K IOPS
- [Migrating gp2 to gp3](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/modify-volume-type.html) — Migração de gp2 para gp3 sem downtime

### Placement Groups
- [Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html) — Cluster, Spread, Partition

### EC2 Lifecycle
- [Instance Lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html) — Estados e transições
- [Hibernate Your Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Hibernate.html) — Requisitos e limitações do hibernate

---

## Calculadoras e Comparadores

- [AWS Pricing Calculator](https://calculator.aws/pricing/2/home) — Comparar custos On-Demand vs RI vs Spot
- [EC2 Spot Instance Advisor](https://aws.amazon.com/ec2/spot/instance-advisor/) — Taxa de interrupção por tipo
- [EC2 Instance Comparison](https://instances.vantage.sh/) — Ferramenta não-oficial excelente para comparar tipos

---

## Whitepapers

- [Compute Pillar — Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/) — Pilar de eficiência de performance
- [Amazon EC2 Spot Best Practices](https://docs.aws.amazon.com/whitepapers/latest/cost-optimization-leveraging-ec2-spot-instances/) — Whitepaper cost optimization com Spot
- [Running Containerized Microservices on AWS](https://docs.aws.amazon.com/whitepapers/latest/running-containerized-microservices/) — Inclui EC2 launch types

---

## FAQs para o Exame

- [EC2 FAQ](https://aws.amazon.com/ec2/faqs/)
- [Reserved Instances FAQ](https://aws.amazon.com/ec2/pricing/reserved-instances/faq/)
- [Spot Instances FAQ](https://aws.amazon.com/ec2/spot/faqs/)

---

## Conceitos SAA-C03 Chave Neste Módulo

| Conceito | Frequência no Exame |
|---|---|
| Spot para batch tolerante a falhas | Alta |
| gp3 vs gp2 (gp3 mais flexível e barato) | Alta |
| io2 Block Express para >64K IOPS | Alta |
| Cluster PG para HPC/baixa latência | Alta |
| IMDSv2 obrigatório (HttpTokens: required) | Média |
| Dedicated Host para BYO-L (Oracle, SQL Server) | Alta |
| Hibernate para preservar estado de RAM | Média |
| Partition PG para HDFS/Cassandra/Kafka | Média |
| Compute Savings Plans vs EC2 Instance SP | Média |
| st1 para throughput sequencial (não boot) | Alta |
