# Questões — Módulo 20: Glossário

> **Quiz de Definições** — Teste seu conhecimento dos termos AWS  
> **Dificuldade**: Básica-Média

---

**1.** O que é um ARN (Amazon Resource Name) e qual é seu formato?

- A) Identificador de região AWS. Formato: `aws:region:nome`
- B) Identificador único de qualquer recurso AWS. Formato: `arn:aws:service:region:account-id:resource`
- C) Nome automático atribuído a instâncias EC2. Formato: `ec2-ip-address.region.compute.internal`
- D) Identificador de usuários IAM. Formato: `iam::user:nome`

<details><summary>Resposta</summary>

**B** — ARN identifica univocamente qualquer recurso AWS. Exemplo: `arn:aws:s3:::meu-bucket`, `arn:aws:ec2:us-east-1:123456:instance/i-xxx`, `arn:aws:iam::123456:role/minha-role`. Usado em IAM policies, resource policies e APIs para referenciar recursos específicos.

</details>

---

**2.** Qual é a diferença entre RTO e RPO em estratégias de Disaster Recovery?

- A) RTO = quanto dados podem ser perdidos; RPO = quanto tempo o sistema pode ficar offline
- B) RTO = tempo máximo aceitável de downtime; RPO = quantidade máxima de dados que podem ser perdidos
- C) RTO e RPO são sinônimos para "Recovery Time"
- D) RTO é para bancos de dados; RPO é para aplicações

<details><summary>Resposta</summary>

**B** — RTO (Recovery Time Objective): quanto tempo posso levar para restaurar o serviço? → define a estratégia DR (menor RTO = estratégia mais cara). RPO (Recovery Point Objective): quanto de dados posso perder? → define a frequência de backup/replicação. Por exemplo: RTO 1h, RPO 15 min = max 1h offline + max 15 min de dados perdidos.

</details>

---

**3.** O que significa IOPS no contexto de armazenamento AWS?

- A) Internet Operations Per Second — velocidade de download
- B) Input/Output Operations Per Second — medida de performance de leitura/escrita de storage
- C) Instance Operations Per Second — throughput de EC2
- D) Infrastructure Operations Per Service — métrica de SLA

<details><summary>Resposta</summary>

**B** — IOPS = Input/Output Operations Per Second. Quanto mais IOPS, mais operações de leitura/escrita por segundo o storage suporta. EBS gp3 baseline = 3.000 IOPS; io2 Block Express = até 256.000 IOPS. Relevante para bancos de dados que precisam de muitas operações I/O por segundo.

</details>

---

**4.** Qual é a diferença entre um Security Group e uma NACL no VPC?

- A) Security Group: nível de subnet (stateless); NACL: nível de instância (stateful)
- B) Security Group: nível de instância/ENI (stateful, apenas Allow); NACL: nível de subnet (stateless, Allow e Deny)
- C) São funcionalidades equivalentes; Security Group é o termo AWS e NACL é o padrão de mercado
- D) NACL é para IPv4; Security Group é para IPv6

<details><summary>Resposta</summary>

**B** — SG: nível de ENI (instância), stateful (retorno automático permitido), regras apenas de ALLOW, referencia outros SGs. NACL: nível de subnet, stateless (deve permitir explicitamente outbound também), tem regras ALLOW e DENY numeradas avaliadas em ordem.

</details>

---

**5.** O que é WORM no contexto do S3 Object Lock?

- A) Write Once Read Many — dados escritos não podem ser modificados ou deletados durante o período de retenção
- B) Write Once Replicate Many — objetos são replicados automaticamente
- C) Web Object Routing Mechanism — como objetos são roteados via CloudFront
- D) Workload Object Retention Method — política de lifecycle do S3

<details><summary>Resposta</summary>

**A** — WORM = Write Once Read Many. S3 Object Lock Compliance mode: uma vez escrito, o objeto não pode ser deletado ou sobrescrito por ninguém (nem root account) durante o retention period. Ideal para: regulatórios financeiros, auditoria, compliance, evidência forense.

</details>

---

**6.** O que é um VPC Endpoint e quais são os dois tipos?

- A) Endpoint de DNS para resolver nomes em uma VPC. Tipos: Route 53 Inbound e Outbound
- B) Conexão privada entre VPC e serviços AWS sem sair para internet. Tipos: Gateway (S3, DynamoDB) e Interface (PrivateLink para outros serviços)
- C) IP público fixo para serviços dentro de uma VPC. Tipos: Elastic IP e NAT Gateway
- D) Ponto de acesso para VPN. Tipos: Site-to-Site e Client VPN

<details><summary>Resposta</summary>

**B** — VPC Endpoint: acesso a serviços AWS sem internet (tráfego fica na rede privada AWS). Gateway Endpoint: gratuito, para S3 e DynamoDB, adicionado à route table. Interface Endpoint (PrivateLink): pago (~$0.01/hora + dados), cria ENI na VPC, suporta 150+ serviços AWS e serviços privados de terceiros.

</details>

---

**7.** O que significa "stateful" em um Security Group?

- A) O Security Group guarda logs de todas as conexões
- B) O retorno de uma conexão permitida é automaticamente permitido sem regra de saída explícita
- C) O Security Group mantém histórico de todas as regras aplicadas
- D) Apenas conexões com estado ESTABLISHED são permitidas

<details><summary>Resposta</summary>

**B** — Stateful: o Security Group rastreia o estado das conexões. Se você permite tráfego inbound na porta 443 (HTTPS), o tráfego de retorno (saída da resposta) é automaticamente permitido sem precisar de regra outbound. Diferente de NACL (stateless), onde você precisa de regras explícitas para inbound E outbound.

</details>

---

**8.** O que é DAX e para qual serviço ele é específico?

- A) AWS Data Aggregation Exchange — ferramenta de consolidação multi-conta
- B) DynamoDB Accelerator — cache in-memory específico para DynamoDB com latência de microssegundos
- C) Direct Access eXchange — protocolo para Direct Connect
- D) Data Analytics eXecutor — motor de análise serverless

<details><summary>Resposta</summary>

**B** — DAX = DynamoDB Accelerator: cache in-memory gerenciado e compatível com a API DynamoDB. Sem mudança de código na aplicação (drop-in replacement). Leituras: de milissegundos para microssegundos. Ideal para: leituras muito frequentes dos mesmos itens (hot items), gaming leaderboards, sessões de usuário.

</details>

---

**9.** O que é um IAM Role e como difere de um IAM User?

- A) Role é para serviços AWS; User é para humanos. Role não tem credenciais permanentes (usa credenciais temporárias do STS)
- B) Role é mais segura que User porque tem mais permissões
- C) User é mais moderno que Role e deve ser usado em novos projetos
- D) Role e User são idênticos; a diferença é apenas no console

<details><summary>Resposta</summary>

**A** — IAM Role: identidade sem credenciais permanentes; assume-se temporariamente (STS AssumeRole → credenciais de curta duração). Para: serviços AWS (EC2, Lambda), usuários federados (SSO), cross-account access. IAM User: tem credenciais permanentes (password + access keys). Melhores práticas: usar Roles sempre que possível, evitar access keys para humans.

</details>

---

**10.** O que é SPICE no Amazon QuickSight?

- A) Security Protocol for Intelligence and Cloud Events — protocolo de segurança
- B) Super-fast Parallel In-memory Calculation Engine — mecanismo de cache in-memory do QuickSight
- C) Standard Protocol for Inter-Cloud Exchange
- D) Scalable Processing of Intelligent Content Engine

<details><summary>Resposta</summary>

**B** — SPICE: cache in-memory do QuickSight. Dados importados no SPICE ficam em memória para dashboards ultra-rápidos (independente de queries ao banco ou S3 em real-time). Cada usuário QuickSight tem uma quota de SPICE (GB). Refresh agendado ou manual atualiza os dados do SPICE.

</details>

---

**11.** O que diferencia CRR de SRR no Amazon S3?

- A) CRR = replicação automática entre regiões; SRR = replicação na mesma região. Ambos requerem versioning habilitado no bucket origem
- B) CRR = backup de objetos grandes; SRR = backup de metadados
- C) SRR é mais barato que CRR; CRR tem latência menor
- D) CRR é para objetos < 5 GB; SRR para objetos maiores

<details><summary>Resposta</summary>

**A** — CRR (Cross-Region Replication): replica objetos para bucket em outra região (DR, compliance, latência global). SRR (Same-Region Replication): replica para bucket na mesma região (aggregação de logs, compliance de cópia, testes sem afetar produção). Ambos: versioning obrigatório, replicam apenas novos objetos (não retroativos), suportam filtro por prefixo/tag.

</details>

---

**12.** O que significa "serverless" no contexto AWS?

- A) A aplicação não usa servidores — roda diretamente no hardware
- B) A AWS gerencia toda a infraestrutura de servidores; você só define o código/configuração e paga pelo uso real
- C) Os servidores são vituais (como EC2) sem hardware dedicado
- D) A aplicação não precisa de código — é configurada declarativamente

<details><summary>Resposta</summary>

**B** — Serverless: sem provisionamento ou gerenciamento de servidores pelo usuário. A AWS gerencia: instâncias, patches, escalabilidade, HA. Você define apenas: código (Lambda), tabelas (DynamoDB), queries (Athena), configuração (S3). Paga pelo consumo real (invocações, GB processados, objetos). Exemplos: Lambda, Fargate, DynamoDB, S3, Athena, API Gateway.

</details>

---

**13.** O que é o padrão de arquitetura "fan-out" na AWS?

- A) Escalar horizontalmente adicionando mais servidores
- B) Enviar uma mensagem para múltiplos consumidores em paralelo usando SNS → múltiplas SQS queues
- C) Distribuir tráfego entre múltiplas instâncias usando um Load Balancer
- D) Replicar dados para múltiplas regiões

<details><summary>Resposta</summary>

**B** — Fan-out: uma mensagem publicada em um tópico SNS é entregue simultaneamente para múltiplos subscribers (SQS queues, Lambda functions, HTTP endpoints). Cada subscriber processa independentemente. Exemplo: uma order_created vai para: queue de email, queue de analytics, queue de fulfillment, queue de auditoria — tudo em paralelo.

</details>

---

**14.** O que é um Placement Group Cluster no EC2?

- A) Grupo de instâncias distribuídas em diferentes racks para alta disponibilidade
- B) Grupo de instâncias no mesmo rack físico/hardware para máxima performance de rede (baixa latência, alta largura de banda entre instâncias)
- C) Grupo de instâncias reservadas antecipadamente
- D) Conjunto de instâncias Spot com preço consistente

<details><summary>Resposta</summary>

**B** — Cluster Placement Group: instâncias muito próximas fisicamente (mesmo rack/hardware). Resultado: latência de rede ultra-baixa e até 100 Gbps de throughput entre instâncias. Sacrifica disponibilidade (falha de rack afeta todas). Use para: HPC, aplicações MPI, computação intensiva, simulações. Contrast: Spread PG (máxima HA, instâncias em racks separados).

</details>

---

**15.** O que é o AWS Shared Responsibility Model?

- A) A AWS é responsável por toda a segurança da nuvem; o cliente não tem responsabilidades
- B) A AWS é responsável pela segurança DA cloud (infraestrutura); o cliente é responsável pela segurança NA cloud (dados, IAM, configuração do OS, aplicação)
- C) A responsabilidade é igual entre AWS e cliente em todas as áreas
- D) O cliente é responsável pela segurança apenas em ambientes de produção

<details><summary>Resposta</summary>

**B** — AWS: hardware, data centers, rede física, hipervisor, serviços gerenciados. Cliente: IAM (who can access), configuração de SG/NACL, criptografia de dados (em repouso e trânsito), patches do SO em EC2, código da aplicação, configurações de compliance. Em serviços gerenciados (RDS, Lambda), o cliente tem menos responsabilidades de infraestrutura.

</details>
