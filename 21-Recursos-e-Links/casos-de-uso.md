# Casos de Uso Reais — Recursos de Estudo em Ação (Módulo 21)

## Caso 1 — Plano de Estudo para Quem Tem 8 Semanas

**Contexto:** Profissional de TI com experiência em infraestrutura on-premises (sem experiência AWS prévia) planeja obter SAA-C03 em 8 semanas, estudando 2 horas por dia.

**Plano Semana a Semana:**
```
SEMANA 1 — BASE E COMPUTAÇÃO (14h):
Recursos:
  ├── Curso Udemy Stephane Maarek: Seções 1-5 (EC2, ELB, ASG)
  ├── AWS Free Tier: criar conta + explorar console
  └── Skill Builder: "AWS Cloud Practitioner Essentials" (gratuito)
  
Prática:
  └── Criar EC2 t3.micro, conectar via SSH, instalar nginx, acessar via browser
  
Meta: entender EC2, tipos de instância, pricing, ELB, Auto Scaling

SEMANA 2 — STORAGE E BANCO DE DADOS (14h):
Recursos:
  ├── Curso Maarek: Seções 6-10 (EBS, EFS, S3, RDS, DynamoDB)
  └── AWS Docs: S3 FAQ + RDS FAQ (30min cada)
  
Prática:
  └── Criar S3 bucket, versioning, lifecycle policy
  └── Criar RDS Free Tier, conectar via DBeaver

SEMANA 3 — REDE E SEGURANÇA (14h):
Recursos:
  ├── Curso Maarek: Seções 11-14 (VPC, Route 53, CloudFront, WAF)
  └── Whitepaper: "AWS Security Best Practices" (1h)
  
Prática:
  └── Criar VPC com 2 subnets (public+private), NAT GW, bastion host

SEMANA 4 — SERVERLESS E INTEGRAÇÃO (14h):
Recursos:
  ├── Curso Maarek: Seções 15-19 (Lambda, SQS, SNS, API GW, Kinesis)
  └── Tutorial: "Build a Serverless App" (AWS Labs gratuito)
  
Prática:
  └── Lambda + API GW + DynamoDB (todo app completo)

SEMANA 5 — MICROSSERVIÇOS, MONITORING E ANALÍTICA (14h):
Recursos:
  ├── Curso Maarek: Seções 20-24 (ECS, CloudWatch, CloudTrail, Athena)
  └── AWS Architecture Center: Well-Architected examples

SEMANA 6 — MIGRAÇÃO, CUSTO, BOAS PRÁTICAS (14h):
Recursos:
  ├── Curso Maarek: Seções finais
  └── Whitepaper: "AWS Well-Architected Framework"
  
SEMANA 7 — PRIMEIRO SIMULADO (14h):
Recursos:
  ├── Tutorials Dojo: 1 simulado completo (65 questões)
  └── Revisar TODAS as questões erradas (ler explicação com cuidado)
  
Meta: score > 65% no primeiro simulado

SEMANA 8 — REVISÃO INTENSIVA + EXAME (14h):
Recursos:
  ├── Tutorials Dojo: segundo simulado completo
  ├── Revisão de cheatsheets dos serviços mais errados
  └── Flashcards nos dias anteriores ao exame
  
AGENDAR EXAME: no início da semana 7 (motivação para estudar!)
```

---

## Caso 2 — Usando Skill Builder para Laboratórios Práticos

**Contexto:** Candidato sabe teoria mas nunca operou na AWS. Skill Builder oferece labs práticos com créditos incluídos.

**Plano de Labs no Skill Builder:**
```
AWS Skill Builder Individual ($29/mês ou $299/ano): 
  Acesso a labs em sandbox real (jamais paga custos de AWS nos labs)
  
LABS PRIORITÁRIOS PARA SAA-C03:
├── "Introduction to Amazon EC2" (grátis no Skill Builder básico)
├── "Introduction to Amazon S3" 
├── "Creating an Amazon Virtual Private Cloud (VPC)"
├── "Introduction to Amazon RDS"
├── "Introduction to AWS Lambda"
├── "Introduction to Amazon DynamoDB"
├── "Introduction to Amazon CloudWatch"
├── "Introduction to AWS IAM"
└── "AWS Well-Architected Best Practices" (não é lab, é módulo teórico)

EXAM READINESS: "AWS Certified Solutions Architect - Associate" 
  (curso oficial gratuito, 3h, cobre todos os domínios do exame)
  Disponível em: skillbuilder.aws → pesquisar "SAA Exam Readiness"

GRATUITO NO SKILL BUILDER BÁSICO (sem pagar):
├── Cloud Practitioner Essentials
├── AWS Technical Essentials  
├── Exam Readiness SAA-C03
└── Vários labs de nível introdutório
```

---

## Caso 3 — Estratégia para Questões de Diagrama de Arquitetura

**Contexto:** Exame SAA-C03 apresenta cenários complexos de múltiplos serviços. Como estruturar o raciocínio.

**Framework de Análise:**
```
MÉTODO EM 4 PASSOS:

PASSO 1 — LER O ÚLTIMO PARÁGRAFO PRIMEIRO:
(geralmente contém a constraint principal)
"A empresa precisa da solução com MENOR CUSTO que atenda os requisitos."
→ Isso elimina imediatamente todas as soluções mais caras

PASSO 2 — IDENTIFICAR PALAVRAS-CHAVE:
┌─────────────────────────────────────────────────────────┐
│ "serverless"       → Lambda, Fargate, Aurora Serverless  │
│ "menor overhead"   → Serviços gerenciados                │
│ "real-time"        → Kinesis, EventBridge                │
│ "desacoplar"       → SQS, SNS                            │
│ "global"           → CloudFront, Route 53, DynamoDB GT   │
│ "failover auto"    → Multi-AZ, Route 53 Failover         │
│ "escalar leituras" → Read Replica (NÃO Multi-AZ)         │
│ "sem perda dados"  → Multi-AZ, síncrono                  │
│ "compliance/audit" → CloudTrail + S3 + Object Lock       │
│ "dados sensíveis"  → Macie, KMS, Secrets Manager         │
└─────────────────────────────────────────────────────────┘

PASSO 3 — ELIMINAR OPÇÕES CLARAMENTE ERRADAS:
"Instalar software em instâncias EC2" → flag de alerta (mais overhead)
"Gerenciar manualmente" em questão sobre HA → quase sempre errado
"Compartilhar credenciais root" → sempre errado

PASSO 4 — ESCOLHER ENTRE AS 2 RESTANTES:
Aplicar critério da questão (custo? performance? disponibilidade?)
Se ainda em dúvida: serviço managed > self-managed na AWS
```

---

## Caso 4 — Usando Tutorials Dojo vs Exame Real

**Contexto:** Candidato tem score 78% no Tutorials Dojo. Estará pronto para o exame real?

**Calibração de Simulados:**
```
CORRELAÇÃO SCORE SIMULADO × EXAME REAL (empírico):
Tutorials Dojo 75% → Exame real provavelmente 70-75% (aprovado: 72%)
Tutorials Dojo 80% → Exame real provavelmente 75-82% (aprovação confortável)
Tutorials Dojo 85% → Exame real provavelmente 80-88% (aprovação boa)

POR QUÊ SIMULADOS PODEM SER MAIS DIFÍCEIS:
Tutorials Dojo e ExamTopics têm questões mais técnicas/detalhadas
Exame real tem mais questões de cenário do que questões técnicas puras
Exame real: 50 válidas + 15 experimentais (novas questões não contam)

INTERPRETANDO SCORE POR DOMÍNIO:
Domínio 1 (Segurança 30%): score < 70% → rever IAM, KMS, VPC
Domínio 2 (Resiliência 26%): score < 70% → rever Multi-AZ, Auto Scaling, DR
Domínio 3 (Performance 24%): score < 70% → rever ElastiCache, Kinesis, CloudFront
Domínio 4 (Custo 20%): score < 70% → rever S3 tiers, Reserved, Spot

META RECOMENDADA:
Fazer 2-3 simulados completos de 65 questões
Score médio > 78% nos últimos 2 simulados → agendar exame
Não fazer simulado no dia anterior ao exame (ansiedade)
```

---

## Caso 5 — Pós-Certificação: Próximos Passos na Carreira

**Contexto:** Candidato acabou de passar no SAA-C03. O que fazer a seguir para maximizar o retorno do investimento.

**Plano de Carreira:**
```
IMEDIATO (primeiros 7 dias):
├── Atualizar LinkedIn:
│   Seção Licenças e Certificações:
│   Nome: "AWS Certified Solutions Architect - Associate"
│   Emissor: Amazon Web Services (AWS)
│   Link verificação: badge Credly
├── Aceitar badge no Credly (email da AWS)
└── Atualizar currículo/perfil em vagas

PRÓXIMAS CERTIFICAÇÕES (recomendação de ordem):
OPÇÃO A — Ampliar (outras Associate antes do Professional):
SAA-C03 → DVA-C02 (Developer) → SOA-C02 (SysOps) → SAP-C02 (Pro)

OPÇÃO B — Aprofundar direto:
SAA-C03 → SAP-C02 (Professional — nível máximo Architect)
           (SAA renova automaticamente ao passar no SAP)

OPÇÃO C — Especialização:
SAA-C03 → DOP-C02 (DevOps Pro) ou ANS-C01 (Advanced Network)
           ou SCS-C02 (Security Specialty)

PARA QUEM BUSCA EMPREGO:
AWS Partner Network (registrar conta gratuita) → encontrar vagas
LinkedIn: "AWS Solutions Architect" → alertas de vagas
GitHub: criar repositório com projetos AWS (evidência prática)
Blog: escrever sobre o que aprendeu → visibilidade

PARA FREELANCER:
Upwork/Toptal: certificação aumenta taxa/hora
AWS Marketplace: empresas buscam SAs certificados para consultorias
AWS Partner: empresa pode se tornar parceira com 2+ SAs certificados

MANTER RELEVÂNCIA:
AWS re:Invent (novembro): conferência principal, novidades do ano
AWS Blogs (aws.amazon.com/blogs): novidades semanais dos serviços
re:Post: responder perguntas da comunidade (solidificar conhecimento)
```
