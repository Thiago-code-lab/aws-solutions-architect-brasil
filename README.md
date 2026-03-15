# 📚 AWS Certified Solutions Architect – Associate (SAA-C03)

> Repositório completo de estudos em Português do Brasil para quem quer passar no exame com entendimento real de arquitetura na AWS.

Portal publicado: https://thiago-code-lab.github.io/aws-solutions-architect-brasil/

## 🎯 Sobre este Repositório

Este material foi estruturado para um estudante de nível intermediário, que já usa AWS no dia a dia e já domina os fundamentos de cloud. O foco não é memorização cega: o objetivo é desenvolver o raciocínio esperado de um arquiteto de soluções, entendendo trade-offs entre resiliência, desempenho, segurança e custo.

O repositório foi pensado para funcionar bem no GitHub e no GitHub Pages. Cada módulo contém teoria, questões no estilo da prova, flashcards, cheatsheet, cenários arquiteturais, labs e curadoria de links oficiais. A ordem dos módulos acompanha os temas mais cobrados no exame e uma progressão de complexidade que faz sentido para revisão em 8 semanas.

## 📊 Estrutura e Domínios do Exame

| Domínio | Peso | O que cai na prática | Módulos principais |
|---|---:|---|---|
| Design Resilient Architectures | 30% | Alta disponibilidade, disaster recovery, desacoplamento, multi-AZ e multi-region | 04, 05, 06, 07, 08, 09, 16, 17, 18 |
| Design High-Performing Architectures | 28% | Seleção de compute, caching, performance de storage, redes e bancos | 03, 04, 05, 06, 07, 08, 10, 11, 12 |
| Design Secure Applications and Architectures | 24% | IAM, KMS, isolamento de rede, criptografia, trilhas de auditoria e governança | 02, 05, 07, 08, 10, 11, 14, 15, 16 |
| Design Cost-Optimized Architectures | 18% | Rightsizing, classes de storage, modelos de compra, serverless e automação | 03, 04, 05, 06, 11, 12, 15, 16 |

## 🗺️ Mapa de Estudos

### Semana 1
- Módulo 01: visão do exame, estratégia, pesos e serviços mais cobrados
- Módulo 02: IAM, KMS, Organizations, Identity Center e políticas

### Semana 2
- Módulo 03: EC2, EBS, placement groups e modelos de compra
- Módulo 04: ELB, ASG, health checks e escalabilidade

### Semana 3
- Módulo 05: S3, lifecycle, replicação, classes e segurança
- Módulo 06: RDS, Aurora, DynamoDB, ElastiCache e Redshift

### Semana 4
- Módulo 07: VPC, subnets, endpoints, TGW, VPN e Direct Connect
- Módulo 08: Route 53, CloudFront, OAC, WAF e Global Accelerator

### Semana 5
- Módulo 09: SQS, SNS, EventBridge e Step Functions
- Módulo 10: ECS, EKS, Fargate, ECR e App Runner

### Semana 6
- Módulo 11: Lambda, API Gateway, authorizers e observabilidade serverless
- Módulo 12: Kinesis, Firehose, Glue, Athena, EMR e analytics

### Semana 7
- Módulo 13: serviços gerenciados de IA e ML mais cobrados
- Módulo 14: CloudWatch, CloudTrail, Config e X-Ray
- Módulo 15: DMS, MGN, DataSync, Snow Family e Transfer Family

### Semana 8
- Módulo 16: Well-Architected Framework e DR patterns
- Módulo 17: estudos de caso reais
- Módulo 18: labs end-to-end
- Módulo 19: simulados completos
- Módulo 20 e 21: glossário e revisão de links oficiais

## 📁 Módulos

| # | Módulo | Domínio predominante | Status | Link |
|---|---|---|---|---|
| 01 | Introdução SAA-C03 | Foundation / Estratégia | ✅ Completo | [README](./01-Introducao-SAA-C03/README.md) |
| 02 | IAM e Segurança | Secure | ✅ Completo | [README](./02-IAM-e-Seguranca/README.md) |
| 03 | Computação EC2 | High-Performing | ✅ Completo | [README](./03-Computacao-EC2/README.md) |
| 04 | Alta Disponibilidade e Escalabilidade | Resilient | ✅ Completo | [README](./04-Alta-Disponibilidade-e-Escalabilidade/README.md) |
| 05 | Amazon S3 e Armazenamento | Resilient / Cost | ✅ Completo | [README](./05-Amazon-S3-e-Armazenamento/README.md) |
| 06 | Banco de Dados | High-Performing | ✅ Completo | [README](./06-Banco-de-Dados/README.md) |
| 07 | VPC e Redes | Secure / Resilient | ✅ Completo | [README](./07-VPC-e-Redes/README.md) |
| 08 | DNS, Route 53 e CloudFront | Resilient / Performance | ✅ Completo | [README](./08-DNS-Route53-e-CloudFront/README.md) |
| 09 | Desacoplamento: SQS, SNS, EventBridge | Resilient | ✅ Completo | [README](./09-Desacoplamento-SQS-SNS-EventBridge/README.md) |
| 10 | Containers: ECS, EKS, Fargate | High-Performing | ✅ Completo | [README](./10-Containers-ECS-EKS-Fargate/README.md) |
| 11 | Serverless: Lambda e API Gateway | Cost / Secure | ✅ Completo | [README](./11-Serverless-Lambda-API-Gateway/README.md) |
| 12 | Dados e Analytics | High-Performing | ✅ Completo | [README](./12-Dados-e-Analytics/README.md) |
| 13 | Machine Learning e IA | Serviços gerenciados | ✅ Completo | [README](./13-Machine-Learning-e-IA/README.md) |
| 14 | Monitoramento: CloudWatch e CloudTrail | Secure / Ops | ✅ Completo | [README](./14-Monitoramento-CloudWatch-CloudTrail/README.md) |
| 15 | Migração e Transferência | Resilient / Cost | ✅ Completo | [README](./15-Migracao-e-Transferencia/README.md) |
| 16 | Well-Architected Framework | Todos os domínios | ✅ Completo | [README](./16-Well-Architected-Framework/README.md) |
| 17 | Casos de Uso Reais | Todos os domínios | ✅ Completo | [README](./17-Casos-de-Uso-Reais/README.md) |
| 18 | Labs Práticos | Hands-on | ✅ Completo | [README](./18-Labs-Praticos/README.md) |
| 19 | Simulados e Questões | Todos os domínios | ✅ Completo | [README](./19-Simulados-e-Questoes/README.md) |
| 20 | Glossário | Revisão | ✅ Completo | [README](./20-Glossario/README.md) |
| 21 | Recursos e Links | Revisão | ✅ Completo | [README](./21-Recursos-e-Links/README.md) |

## 🚀 Como Usar Este Repositório

1. Comece pelo módulo 01 para alinhar expectativa, pesos e estratégia.
2. Estude os módulos 02 a 16 em ordem ou reorganize pelo seu ponto fraco no exame.
3. Ao final de cada módulo, resolva as questões antes de abrir o gabarito.
4. Use os flashcards para revisão espaçada e o cheatsheet para revisão rápida pré-simulado.
5. Rode os labs em us-east-1 sempre que possível para manter consistência com o material.
6. Feche a preparação com os estudos de caso, labs end-to-end e os três simulados completos.

## 🤝 Contribuições

Contribuições são bem-vindas, especialmente para:

- corrigir detalhes técnicos e limites de serviço
- melhorar cenários de questões e explicações de alternativas
- atualizar mudanças recentes dos serviços AWS
- adicionar diagramas ASCII mais claros e labs mais econômicos

Ao contribuir, prefira mudanças pequenas, tecnicamente justificadas e com links oficiais da AWS quando o assunto envolver comportamento específico de serviço.

## 📄 Licença

MIT
