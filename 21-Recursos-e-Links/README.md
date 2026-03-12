# Módulo 21 — Recursos e Links para Estudo

## Documentação Oficial AWS

### Guias do Usuário Essenciais

| Serviço | Link |
|---|---|
| **EC2 User Guide** | https://docs.aws.amazon.com/ec2/latest/userguide/ |
| **S3 User Guide** | https://docs.aws.amazon.com/AmazonS3/latest/userguide/ |
| **VPC User Guide** | https://docs.aws.amazon.com/vpc/latest/userguide/ |
| **RDS User Guide** | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/ |
| **DynamoDB Developer Guide** | https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ |
| **Lambda Developer Guide** | https://docs.aws.amazon.com/lambda/latest/dg/ |
| **IAM User Guide** | https://docs.aws.amazon.com/IAM/latest/UserGuide/ |
| **CloudFormation User Guide** | https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/ |
| **ECS Developer Guide** | https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ |
| **EKS User Guide** | https://docs.aws.amazon.com/eks/latest/userguide/ |

---

## Whitepapers Oficiais AWS (Essenciais para o Exame)

| Documento | Relevância |
|---|---|
| **AWS Well-Architected Framework** | ALTA — todo o módulo 16 é baseado neste |
| **Architecting for the Cloud: Best Practices** | ALTA — padrões arquiteturais |
| **AWS Security Best Practices** | ALTA — domínio de segurança (30%) |
| **Running Containerized Microservices on AWS** | MEDIA — ECS, EKS, microserviços |
| **Practicing Continuous Integration and Delivery on AWS** | MEDIA — CodePipeline, CodeBuild, deploy |
| **AWS Storage Services Overview** | ALTA — S3, EBS, EFS, FSx, Storage Gateway |
| **Building Big Data Storage Solutions (Data Lakes) for Maximum Flexibility** | MEDIA — S3, Glue, Athena, Lake Formation |
| **Overview of Deployment Options on AWS** | MEDIA — Blue/green, canary, rolling |

Portal: https://aws.amazon.com/whitepapers/

---

## FAQs Importantes dos Serviços AWS

As FAQs têm perguntas que frequentemente aparecem no exame:

| Serviço | FAQ URL |
|---|---|
| **Amazon EC2** | https://aws.amazon.com/ec2/faqs/ |
| **Amazon S3** | https://aws.amazon.com/s3/faqs/ |
| **Amazon RDS** | https://aws.amazon.com/rds/faqs/ |
| **Amazon DynamoDB** | https://aws.amazon.com/dynamodb/faqs/ |
| **Amazon VPC** | https://aws.amazon.com/vpc/faqs/ |
| **Amazon Route 53** | https://aws.amazon.com/route53/faqs/ |
| **AWS Lambda** | https://aws.amazon.com/lambda/faqs/ |
| **Amazon SQS** | https://aws.amazon.com/sqs/faqs/ |
| **Amazon SNS** | https://aws.amazon.com/sns/faqs/ |
| **Amazon CloudFront** | https://aws.amazon.com/cloudfront/faqs/ |

---

## Cursos e Treinamentos

### Cursos Oficiais AWS (AWS Training & Certification)
- **AWS Cloud Practitioner Essentials** — Fundamentos (gratuito): https://explore.skillbuilder.aws
- **Architecting on AWS** — Curso presencial/virtual de 3 dias da AWS
- **AWS Solutions Architect - Associate (Learning Path)** — SkillBuilder: https://skillbuilder.aws/learning_plan/view/78/solutions-architect-associate-learning-plan

### Cursos de Terceiros Recomendados (PT-BR e EN)

| Plataforma | Instrutor | Observação |
|---|---|---|
| Udemy | **Stephane Maarek** (SAA-C03) | Mais popular, atualizado, EN |
| Udemy | **Neal Davis** (Adrian Cantrill material) | Muito técnico, EN |
| CloudGuru / Pluralsight | Vários | Laboratórios práticos |
| YouTube | **FreeCodeCamp** | Crash courses gratuitos |

---

## Simulados e Questões de Prática

| Plataforma | Tipo | Observação |
|---|---|---|
| **Tutorials Dojo** (Jon Bonso) | Simulados SAA-C03 | Melhor banco de simulados para o exame; explicações detalhadas |
| **AWS Skill Builder** (Official Practice) | Exames oficiais | Exame prático oficial: 20 questões ($0 com subscription) |
| **ExamPro** | Neal Davis simulados | Alta qualidade, explicações |
| **Whizlabs** | Banco de questões | Opção adicional |
| **Udemy Practice Tests** | Maarek / Neal Davis | Geralmente vem com o curso |

---

## Ferramentas de Estudo

| Ferramenta | Uso |
|---|---|
| **AWS Free Tier** | Conta gratuita para laboratorios: https://aws.amazon.com/free/ |
| **AWS Architecture Center** | Diagramas e referências de arquitetura: https://aws.amazon.com/architecture/ |
| **AWS This Is My Architecture** | Vídeos de arquiteturas reais de clientes AWS: https://aws.amazon.com/this-is-my-architecture/ |
| **AWS Blog** | Novidades e deep dives técnicos: https://aws.amazon.com/blogs/aws/ |
| **AWS re:Invent YouTube** | Talks técnicos gratuitos: https://youtube.com/@AWSEventsChannel |
| **AWS Calculator** | Estimativa de custos: https://calculator.aws/ |
| **AWS Exam Guide** | Guia oficial do exame SAA-C03: https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf |

---

## Recursos por Domínio do Exame

### Domínio 1 — Arquiteturas Resilientes (26%)
- [AWS Fault Isolation Boundaries](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/abstract-and-introduction.html)
- [Disaster Recovery of Workloads on AWS: Recovery in the Cloud](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)
- [Amazon RDS Multi-AZ](https://aws.amazon.com/rds/features/multi-az/)
- [SQS Developer Guide — Dead Letter Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)

### Domínio 2 — Arquiteturas de Alta Performance (24%)
- [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Caching Overview (ElastiCache)](https://aws.amazon.com/caching/aws-caching/)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [Amazon Aurora Features](https://aws.amazon.com/rds/aurora/features/)

### Domínio 3 — Aplicações Seguras (30%)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Security Best Practices (Whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/aws-security-best-practices/aws-security-best-practices.html)
- [VPC Security Groups and NACLs](https://docs.aws.amazon.com/vpc/latest/userguide/infrastructure-security.html)
- [KMS Key Policies](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)

### Domínio 4 — Arquiteturas Otimizadas em Custo (20%)
- [AWS Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
- [Spot Instances](https://aws.amazon.com/ec2/spot/)
- [AWS Savings Plans](https://aws.amazon.com/savingsplans/)
- [S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)

---

## Canais YouTube Recomendados

| Canal | Conteúdo |
|---|---|
| **AWS Events** | re:Invent, re:Inforce talks |
| **TechWorld with Nana** | Kubernetes, Docker, DevOps (EN) |
| **AWS Online Tech Talks** | Demos técnicos de serviços específicos |
| **Rampup programs (AWS)** | Séries de aprendizado estruturado |

---

## Linha do Tempo de Estudo Sugerida

```
Semana 1-2:  Módulos 01-04 (Cloud, EC2, IAM, S3, Storage)
Semana 3-4:  Módulos 05-07 (S3 avançado, Bancos, VPC/Redes)
Semana 5-6:  Módulos 08-10 (Route53/CloudFront, SQS/SNS, Containers)
Semana 7-8:  Módulos 11-13 (Lambda/API GW, Analytics, ML/IA)
Semana 9:    Módulos 14-16 (Monitoramento, Migração, Well-Architected)
Semana 10:   Simulados (Módulo 19) + revisão de pontos fracos
Semana 11:   Flashcards intensivos + Cheatsheets de todos os módulos
Semana 12:   Simulados completos + revisão final + agenda o exame
```

---

## Registro do Exame

1. Acesse: https://aws.amazon.com/certification/certified-solutions-architect-associate/
2. Crie conta em: https://www.aws.training/ ou https://home.pearsonvue.com/aws
3. Escolha modalidade: Presencial (centro de testes) ou Online Proctored
4. Custo: USD $150
5. Validade: 3 anos (renovar com exame recertification ou exame superior)

**ID do exame**: SAA-C03
