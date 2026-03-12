# Flashcards — Módulo 21: Recursos e Links

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 20 flashcards — recursos, estratégias e orientações de estudo

---

**P:** Qual é o site oficial da AWS para todas as certificações e onde agendar o exame?  
**R:** **aws.amazon.com/certification/** — página principal de certificações. Para agendar: botão "Schedule Exam" → redireciona para **Pearson VUE** (centro de testes presenciais ou online proctored). Alternativa: registro pelo próprio site da certificação SAA-C03. Lembrar: identificação válida necessária (CPF + RG ou passaporte para proctored online).

---

**P:** Quais são os 3 principais cursos pagos recomendados para SAA-C03?  
**R:** **(1) Stephane Maarek (Udemy):** Ultimate AWS Certified Solutions Architect Associate — mais popular, 30h+, atualizado, foco em exame. **(2) Neal Davis (Digital Cloud Training):** AWS SAA-C03 — explicações claras, bom para iniciantes. **(3) Adrian Cantrill:** profundidade técnica superior, mais difícil, anime style — para quem quer entender de verdade. Todos disponíveis na Udemy frequentemente com desconto.

---

**P:** Qual é o recurso oficial AWS mais importante para questões de prática reais?  
**R:** **AWS Skill Builder** (skillbuilder.aws): **(1)** "AWS Certified Solutions Architect – Associate Official Practice Question Set" — 20 questões oficiais gratuitas. **(2)** "Exam Prep Official Practice Exam: AWS SAA-C03" — 65 questões pagas, com feedback detalhado. Credenciais: conta free no Skill Builder. As questões oficiais dão o nível real de dificuldade do exame.

---

**P:** O que é Tutorials Dojo e por que é altamente recomendado?  
**R:** **tutorialsdojo.com** — plataforma de simulados criada por Jon Bonso. Destaques: explicações para CADA opção de cada questão (por que certa + por que cada errada está errada); cheat sheets por serviço; modos timed e review; correlação alta com exame real. Custo: ~$15-30 por exame de prática. Amplamente citado como melhor recurso de prática além do oficial.

---

**P:** Qual whitepaper AWS é mais importante para o SAA-C03 e onde encontrá-lo?  
**R:** **"AWS Well-Architected Framework"** — aws.amazon.com/architecture/well-architected. Descreve os 6 pilares com design principles. Outros essenciais: **(1)** "Architecting for the Cloud: AWS Best Practices". **(2)** "AWS Security Best Practices". **(3)** "Disaster Recovery of Workloads on AWS". Todos gratuitos em aws.amazon.com/whitepapers.

---

**P:** Como usar o AWS Architecture Center para estudar?  
**R:** **aws.amazon.com/architecture** — biblioteca de Reference Architectures por caso de uso (web apps, serverless, data lake, microservices) e indústria. Para estudar: abrir diagramas de arquiteturas relevantes ao exame (3-tier web app, serverless API, data lake), entender quais serviços são combinados e por quê. Arquiteturas de referência oficiais AWS.

---

**P:** Qual é a ferramenta oficial para estimar custos de uma arquitetura AWS?  
**R:** **AWS Pricing Calculator** (calculator.aws) — gratuito. Selecionar serviços, configurar parâmetros (região, tipo de instância, uso esperado en horas/GB/requisições), adicionar ao estimate, compartilhar estimativa via URL. Diferente do Cost Explorer (para custos já incorridos). No exame: "estimar custos antes de implementar" → Pricing Calculator.

---

**P:** Onde encontrar os FAQs dos serviços AWS e como usá-los para estudar?  
**R:** Em cada página de serviço AWS: `aws.amazon.com/[service]/faqs/`. Ex: `aws.amazon.com/s3/faqs/`, `aws.amazon.com/ec2/faqs/`. FAQs explicam limites, casos edge e comportamentos específicos que aparecem no exame. Prioridade de leitura: EC2, S3, RDS, IAM, VPC, Lambda, DynamoDB — serviços com maior peso no exam.

---

**P:** Qual é a diferença entre Pearson VUE e PSI para agendar o exame AWS?  
**R:** Ambos são provedores de exame aprovados pela AWS. **Pearson VUE:** mais popular, mais centros no Brasil, opção online proctored (OnVUE). **PSI:** alternativa, menos centros, também tem online proctored. Para online proctored: precisa de webcam, microfone, ambiente quieto sem outras pessoas, nenhum objeto na mesa. Exame em PT-BR disponível (mas algumas questões técnicas ficam em inglês).

---

**P:** O que estudar na semana final antes do exame SAA-C03?  
**R:** Última semana: **(1)** Revisar erros dos simulados (focar no que errou, não no que já sabe). **(2)** Cheatsheets por serviço (limites numéricos, comparativos). **(3)** Domain 3 (Security, 30%) — IAM conditions, encryption, network security. **(4)** Simulado completo (65q) para medir readiness. **(5)** Não estudar nada novo nos últimos 2 dias — revisão + descanso. Score 750+ nos simulados = pronto para o exame.

---

**P:** Qual canal do YouTube tem os melhores resumos gratuitos de serviços AWS para o exame?  
**R:** **(1) Be a Better Dev** — demos práticos e conceituais. **(2) Stephane Maarek** — snippets do curso. **(3) Digital Cloud Training (Neal Davis)** — tutorials detalhados. **(4) AWS Events** — re:Invent talks técnicos (avançado). **(5) TechWorld with Nana** — Docker/K8s complementares. Para PT-BR: pesquisar "SAA-C03 preparação" — comunidade brasileira crescente.

---

**P:** Para que serve o AWS Free Tier nos estudos práticos?  
**R:** Praticar labs sem custo. Recursos Always Free essenciais: Lambda 1M req/mês, DynamoDB 25GB, CloudFront 1TB/mês, SNS 1M req, API Gateway 1M req/mês. 12-month free: EC2 t2/t3.micro 750h/mês, S3 5GB, RDS db.t2.micro 750h/mês. **SEMPRE:** configurar Budget Alert no Billing Console para alertar se custo > $1. Fazer cleanup após labs (parar/deletar recursos).

---

**P:** Como usar o AWS Skill Builder gratuitamente para preparação?  
**R:** Criar conta free em skillbuilder.aws. Acessar: **(1)** "AWS Cloud Practitioner Essentials" (curso gratuito básico). **(2)** "Architecting on AWS" (conceitos). **(3)** "AWS Certified Solutions Architect - Associate" learning path (gratuito). **(4)** "Official Practice Question Set SAA-C03" (20 perguntas gratuitas). Assinatura paga ($29/mês) dá acesso a simulados completos e labs.

---

**P:** O que é o AWS Certification Discord e como se beneficiar?  
**R:** Comunidades online onde candidatos compartilham experiências de prova, recursos e dicas. AWS: join em servidores como "AWS Study Group", "r/AWSCertifications" (Reddit), "Discord AWS Brasil". Benefícios: dicas de atualização do exame, notificações quando questões mudaram, compartilhamento de recursos gratuitos, suporte de pessoas que passaram recentemente.

---

**P:** Como configurar ambiente de estudo local para labs (AWS CLI + Terraform)?  
**R:** **(1)** Instalar AWS CLI v2: `winget install Amazon.AWSCLI` (Windows) ou `brew install awscli` (Mac). **(2)** Configurar: `aws configure` (access key + secret + região + formato). **(3)** Instalar Terraform: `winget install Hashicorp.Terraform`. **(4)** Verificar: `aws sts get-caller-identity` retorna account info. Usar AWS Cloud9 como alternativa — ambiente no browser sem setup local.

---

**P:** Qual livro é recomendado para complementar o estudo do SAA-C03?  
**R:** **"AWS Certified Solutions Architect Study Guide" by Ben Piper and David Clinton** (Sybex/Wiley) — atualizado para SAA-C03, abordagem metódica, bom pré-exame complementar. **"AWS Cookbook" by John Culkin** — mais hands-on, receitas práticas. Livros são complementares (não substitutos) aos simulados. Prioridade: simulados > livros.

---

**P:** O que é o "Exam Readiness" da AWS e como interpretar?  
**R:** AWS Skill Builder tem cursos "Exam Readiness: SAA-C03" que mapeiam domínios, explicam tipos de questão e estratégias de eliminação. Além disso: score de 800+ nos simulados do Tutorials Dojo em modo timed é geralmente considerado readiness suficiente. Abaixo de 720 (80% equivalente nos simulados): continuar estudando pontos fracos identificados.

---

**P:** Onde ler sobre novos serviços e updates AWS que podem aparecer no exame?  
**R:** **(1) AWS What's New:** aws.amazon.com/new — feed de updates. **(2) AWS Blog:** aws.amazon.com/blogs/aws — posts técnicos. **(3) re:Invent announcements:** YouTube/AWS Events. **(4) AWS Certification changelog:** a AWS notifica quando versões de exame mudam com 6 meses de antecedência. Foco: serviços que entraram no Exam Guide oficial. Não estudar serviços fora do escopo.

---

**P:** Como criar um plano de estudos de 6 semanas para o SAA-C03?  
**R:** **Semana 1:** Módulos fundamentais (EC2, S3, IAM, VPC). **Semana 2:** Databases + Networking avançado (RDS, DynamoDB, Route 53, CloudFront). **Semana 3:** Serverless + Decoupling (Lambda, API GW, SQS, SNS, EventBridge). **Semana 4:** Analytics + Migração + Well-Architected. **Semana 5:** Simulados completos (2-3 full exams) + revisar erros. **Semana 6:** Focus nos pontos fracos + simulado final + descanso.

---

**P:** Como usar os cheat sheets do Tutorials Dojo para revisão final?  
**R:** Tutorialsdojo.com/aws-cheat-sheets/ — disponíveis por serviço, gratuitos. Formato: tabelas comparativas, limites numéricos importantes, diferenças entre serviços similares. Para revisão: imprimir ou salvar offline os cheat sheets dos serviços core (EC2, S3, RDS/Aurora, VPC, IAM, Lambda, DynamoDB). Revisar especialmente tabelas comparativas que o exame frequentemente testa (ex: SQS vs SNS vs EventBridge).

---

**P:** Após obter a certificação SAA-C03, como mantê-la válida?  
**R:** Validade: **3 anos**. Para recertificar: **(1)** Re-fazer o SAA-C03 antes do vencimento. **(2)** Obter uma certificação de nível superior (SAP-C02 Professional) — recertifica automaticamente o SAA. **(3)** Obter outra Associate ou specialty (não recertifica automaticamente). No perfil AWS Certification: pode baixar o certificado digital e badge Credly. Compartilhar no LinkedIn.
