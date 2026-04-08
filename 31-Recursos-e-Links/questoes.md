# Questões — Módulo 21: Recursos e Links

> **Quiz de Orientação de Estudos** — Conheça suas fontes e estratégias  
> **Dificuldade**: Básica-Média

---

**1.** Qual é o recurso oficial da AWS para acessar questões práticas gratuitas para o SAA-C03?

- A) AWS Certification Website → Practice Exam (pago)
- B) AWS Skill Builder — Oficial Exam Prep module (gratuito, inclui questões de sample)
- C) Amazon.com → AWS Books
- D) GitHub AWS → practice-exams

<details><summary>Resposta</summary>

**B** — AWS Skill Builder (skillbuilder.aws) oferece: (1) Sample Questions gratuitas (20 questões) para calibrar o nível; (2) Official Practice Exam (pago, 65 questões) com feedback detalhado; (3) Courses gratuitos e pagos (Individual e Team). É a fonte mais confiável porque as questões são criadas pela própria AWS.

</details>

---

**2.** Qual whitepaper AWS cobre os 6 pilares da arquitetura bem construída?

- A) "AWS Security Best Practices"
- B) "AWS Well-Architected Framework" (disponível em aws.amazon.com/architecture/well-architected)
- C) "Overview of AWS Services"
- D) "AWS Migration Acceleration Program"

<details><summary>Resposta</summary>

**B** — O whitepaper "AWS Well-Architected Framework" descreve os 6 pilares: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization e Sustainability. É leitura obrigatória para o SAA-C03 pois questões do exame frequentemente citam os princípios diretamente.

</details>

---

**3.** Qual é a vantagem do Tutorials Dojo como recurso de estudo para o SAA-C03?

- A) É o recurso oficial AWS mais completo
- B) Oferece simulados com explicações detalhadas por opção (inclusive por que cada resposta errada está errada) e alta correlação com o exame real
- C) É gratuito sem limitações
- D) Criado pelos próprios autores do exame AWS

<details><summary>Resposta</summary>

**B** — Tutorials Dojo (tutorialsdojo.com): renomado pela qualidade das explicações. Para cada questão, explica por que cada opção alternativa está incorreta — não apenas a correta. Cheat sheets por serviço. Simulados em timed e review mode. Alta correlação reportada com o exame real. Criado por Jon Bonso (arquiteto AWS certificado).

</details>

---

**4.** Qual é o domínio a priorizar nos últimos 3 dias antes do exame SAA-C03?

- A) Networking (VPC, Direct Connect, Route 53) — alta complexidade
- B) Design de Aplicações Seguras (30% do exame) — maior peso no exame
- C) Machine Learning — novidade frequente
- D) Billing e Pricing — fácil de memorizar

<details><summary>Resposta</summary>

**B** — Design de Aplicações Seguras tem 30% do exame — o maior domínio. Revisar nos últimos dias: IAM policies (conditions, boundaries), KMS vs CloudHSM, VPC security (SG stateful vs NACL stateless), S3 security (BPA, policies, ACLs, OAC), Cognito (User Pools vs Identity Pools), GuardDuty, Shield vs WAF, Secrets Manager vs Parameter Store.

</details>

---

**5.** Qual o FAQ mais importante de ler antes do exame SAA-C03?

- A) FAQ do EC2 e S3 apenas (serviços mais testados)
- B) FAQ dos 3-5 serviços core: EC2, S3, RDS, IAM, VPC — são os mais detalhados e cobrem nuances testadas
- C) FAQ de todos os 200+ serviços AWS
- D) FAQs não são úteis — focar nos cursos

<details><summary>Resposta</summary>

**B** — FAQs da AWS descrevem limites, casos especiais e nuances que aparecem no exame. Prioridade: EC2 (tipos de instância, pricing, HA), S3 (classes de armazenamento, limites, segurança), RDS (Multi-AZ vs Read Replicas, backups, encryption), IAM (policy evaluation, roles, SCP), VPC (peering, endpoints, routing). FAQs de serviços específicos como Lambda, DynamoDB e ELB também são valiosos.

</details>

---

**6.** Qual ferramenta AWS serve para estimar o custo de uma arquitetura antes de implementá-la?

- A) AWS Cost Explorer (analisa custos históricos)
- B) AWS Pricing Calculator (calculator.aws) — estima custos de serviços antes de usar
- C) AWS Trusted Advisor (recomendações de otimização)
- D) AWS Budgets (define limites de gasto)

<details><summary>Resposta</summary>

**B** — AWS Pricing Calculator (calculator.aws): ferramenta gratuita para construir estimativas de arquitetura. Adicione serviços, configure parâmetros (instância type, region, uso esperado) e veja estimativa mensal. Diferente do Cost Explorer (analisa gastos já ocorridos). No exame, quando a questão pede "como estimar custos antes de deploy" → Pricing Calculator.

</details>

---

**7.** Você precisa entender como o tráfego entra e sai de uma VPC. Qual recurso AWS recomendado oferece diagramas e exemplos de arquitetura de rede?

- A) AWS EC2 User Guide
- B) AWS VPC User Guide (docs.aws.amazon.com/vpc/latest/userguide) + AWS Architecture Center
- C) AWS CLI Reference
- D) AWS SDK for Python (Boto3)

<details><summary>Resposta</summary>

**B** — AWS VPC User Guide é a documentação mais completa sobre networking. O AWS Architecture Center (aws.amazon.com/architecture) tem diagramas de arquiteturas de referência para casos de uso comuns — excelente para visualizar patterns de HA multi-AZ, hybrid connectivity e microservices. Ambos são gratuitos e oficiais.

</details>

---

**8.** Um candidato a SAA-C03 tem 4 semanas para estudar. Qual é a distribuição mais efetiva?

- A) Semana 1-4: assistir todo o curso do Stephane Maarek sem simulados
- B) Semana 1-2: curso teórico; Semana 3: simulados com revisão das respostas erradas; Semana 4: foco em pontos fracos + simulado final
- C) Semana 1: simulado para baseline; Semanas 2-3: estudar conteúdo; Semana 4: simulado
- D) Estudar todos os serviços AWS existentes (200+) uniformemente

<details><summary>Resposta</summary>

**B** — Metologia comprovada: (1) Teoria sólida nos serviços core; (2) Simulados para identificar gaps — o feedback das questões erradas é o estudo mais eficiente; (3) Revisão focada nos domínios fracos; (4) Simulado final para medir readiness. Não adianta fazer teoria sem prática; simulados sem revisão das erradas são perda de tempo.

</details>

---

**9.** Qual é o recurso ideal para entender o modelo de preços do EC2 (On-Demand vs Reserved vs Spot)?

- A) AWS Billing Console → Bills
- B) AWS EC2 Pricing page (aws.amazon.com/ec2/pricing) + AWS Compute Optimizer
- C) AWS Trusted Advisor → Cost Optimization
- D) CUR (Cost and Usage Report)

<details><summary>Resposta</summary>

**B** — A página de preços do EC2 tem todos os modelos com calculadora interativa. AWS Compute Optimizer analisa utilização real e sugere o tipo certo de instância. Para o exame, conheça: On-Demand (máxima flexibilidade, maior preço), Reserved (1 ou 3 anos, até 72% off), Savings Plans (compromisso de gasto, mais flexível), Spot (até 90% off, interruptível).

</details>

---

**10.** O que é o AWS Architecture Center e como usá-lo para estudar?

- A) Centro de suporte para arquitetos AWS certificados
- B) Biblioteca de arquiteturas de referência, diagramas e whitepapers em aws.amazon.com/architecture — estudar os patterns de cada indústria
- C) Ferramenta de design de diagramas concorrente ao draw.io
- D) Parte do AWS Management Console para visualizar infraestrutura

<details><summary>Resposta</summary>

**B** — AWS Architecture Center (aws.amazon.com/architecture): biblioteca gratuita de "Reference Architectures" por caso de uso (web app 3-tier, serverless API, data lake, etc.) e indústria (fintech, saúde, mídia). Para estudar: leia arquiteturas de referência relevantes ao SAA-C03 para entender como serviços se combinam em patterns reais de produção.

</details>

---

**11.** Qual o canal do YouTube mais recomendado para vídeos gratuitos de preparação AWS?

- A) Canal oficial AWS → "AWS re:Invent" (técnico mas não focado em certificação)
- B) Stephane Maarek, Neal Davis (Digital Cloud Training), Be a Better Dev — conteúdo focado em certificação, gratuito no YouTube
- C) Linux Foundation YouTube
- D) Microsoft Azure YouTube (para comparação)

<details><summary>Resposta</summary>

**B** — Para SAA-C03: Stephane Maarek (snippets gratuitos de seu curso Udemy), Digital Cloud Training de Neal Davis (tutoriais detalhados), Be a Better Dev (demos práticas). O canal oficial AWS tem conteúdo excelente (re:Invent, re:Inforce) mas não é focado em exames — é mais técnico/aprofundado em serviços específicos.

</details>

---

**12.** Onde encontrar a versão atualizada do Exam Guide oficial do SAA-C03?

- A) Amazon.com → AWS Study Guides
- B) aws.amazon.com/certification/certified-solutions-architect-associate → "Exam Prep" → "Exam Guide PDF"
- C) AWS Skill Builder → My Courses
- D) AWS Console → Certification → Exam Guide

<details><summary>Resposta</summary>

**B** — O Exam Guide PDF oficial fica na página da certificação no aws.amazon.com. Ele descreve: (1) domínios e pesos, (2) tarefas esperadas por domínio, (3) serviços incluídos no escopo, (4) serviços fora do escopo. Leitura obrigatória antes de começar a estudar — orienta o que priorizar baseado nos pesos oficiais.

</details>

---

**13.** Qual é o FlashCard mais importante para memorizar sobre S3 classes de armazenamento?

- A) S3 Standard → S3 Intelligent-Tiering → S3 Standard-IA → S3 One Zone-IA → S3 Glacier Instant → S3 Glacier Flexible → S3 Glacier Deep Archive (ordem decrescente de custo de armazenamento)
- B) S3 só tem 2 classes: Standard e Glacier
- C) S3 Glacier é para backups mensais; S3 Standard para backups diários
- D) S3 Intelligent-Tiering só funciona para objetos > 1 TB

<details><summary>Resposta</summary>

**A** — As 7 classes S3 em ordem de custo de storage (do mais caro ao mais barato): Standard → Intelligent-Tiering → Standard-IA → One Zone-IA → Glacier Instant → Glacier Flexible → Glacier Deep Archive. Inversamente proporcional: quanto mais barato o storage, maior o custo de retrieval e a latência. Exame testa: qual classe para qual caso de uso (acesso frequente, infrequente, arquivo).

</details>

---

**14.** Quando usar o recurso AWS Free Tier para praticar antes do exame?

- A) Nunca — pode gerar cobranças inesperadas
- B) Para labs práticos dos módulos de estudo: criar VPC, Lambda, S3, DynamoDB — serviços core têm nível gratuito generoso; sempre definir alertas de billing
- C) Apenas para serviços serverless (Lambda, DynamoDB)
- D) O Free Tier é apenas para contas corporativas

<details><summary>Resposta</summary>

**B** — O Free Tier (aws.amazon.com/free) oferece: 12 meses de t2.micro/t3.micro EC2, 5 GB S3, 25 GB DynamoDB, 1M requisições Lambda/mês, etc. Para praticar: criar conta nova, habilitar AWS Budgets com alerta em $1-5, fazer os labs práticos do estudo. Prática real supera estudo puramente teórico. Sempre limpar recursos após os labs.

</details>

---

**15.** Após passar no exame SAA-C03, qual é a próxima certificação mais recomendada para seguir a trilha?

- A) AWS Cloud Practitioner (é pré-requisito obrigatório)
- B) AWS Solutions Architect Professional (SAP-C02) ou AWS Developer Associate (DVA-C02) dependendo do foco de carreira
- C) Google Cloud Professional Architect
- D) Kubernetes CKA — complementar ao AWS

<details><summary>Resposta</summary>

**B** — Trilhas pós-SAA-C03: (1) Aprofundar arquitetura: SAP-C02 (Professional, mais difícil, foco em enterprise); (2) Foco em desenvolvimento: DVA-C02 (Developer Associate, serverless, CI/CD, SDK); (3) Foco em operações: SOA-C02 (SysOps Administrator Associate); (4) Especialidades: AWS Security Specialty, Data Analytics Specialty. CCP (Cloud Practitioner) é pré-requisito opcional, não obrigatório.

</details>
