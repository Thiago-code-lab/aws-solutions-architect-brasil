# Questões — Módulo 14: Monitoramento CloudWatch, CloudTrail e Config

> **Domínio SAA-C03**: Design de Aplicações Seguras | Excelência Operacional  
> **Dificuldade**: Média

---

**1.** Uma empresa nota que suas instâncias EC2 têm métricas de CPU e rede no CloudWatch, mas não conseguem monitorar o uso de memória RAM. O que precisam configurar?

- A) Habilitar Detailed Monitoring nas instâncias EC2
- B) Instalar o CloudWatch Agent nas instâncias e configurar coleta de métricas do SO
- C) Usar AWS Trusted Advisor para métricas de memória
- D) Criar métricas customizadas via CloudWatch API no user data das instâncias

<details><summary>Resposta</summary>

**B** — Métricas padrão do EC2 não incluem memória RAM, uso de disco e processos (são métricas do SO, não de hipervisor). O CloudWatch Agent instalado nas instâncias coleta essas métricas do sistema operacional e as publica como Custom Metrics no CloudWatch. Detailed Monitoring (A) aumenta granularidade de 5 min para 1 min, mas ainda sem memória.

</details>

---

**2.** Uma auditoria de segurança descobriu que recursos AWS foram deletados sem autorização. O analista precisa identificar quem deletou, quando e de qual IP. Qual serviço tem essa informação?

- A) Amazon CloudWatch Logs
- B) AWS CloudTrail (Management Events)
- C) AWS Config (Configuration History)
- D) Amazon GuardDuty

<details><summary>Resposta</summary>

**B** — CloudTrail registra todas as API calls incluindo: quem fez (userIdentity), quando (eventTime), de onde (sourceIPAddress), qual ação (eventName como DeleteInstance, DeleteBucket). É o "audit log" da AWS. CloudWatch Logs tem logs de aplicação, não de API calls.

</details>

---

**3.** Uma empresa precisa ser alertada quando qualquer S3 bucket na conta se tornar publicamente acessível. Qual é a solução mais eficiente?

- A) Lambda que verifica todos os bucket ACLs a cada hora
- B) AWS Config Rule `s3-bucket-public-read-prohibited` com notificação SNS ao detectar non-compliance
- C) CloudWatch Alarm monitorando métricas de acesso S3
- D) Amazon Macie escaneando todos os buckets

<details><summary>Resposta</summary>

**B** — AWS Config Rule `s3-bucket-public-read-prohibited`: avalia continuamente se buckets têm permissão de leitura pública. Quando um bucket fica public, o Config marca como NON_COMPLIANT e pode notificar via SNS ou executar remediação automática via SSM. Proativo e sem código customizado.

</details>

---

**4.** Um time de DevOps quer rastrear como as configurações de seus Security Groups mudaram ao longo do tempo (quem adicionou regra, qual regra, quando). Qual serviço fornece essa visão?

- A) VPC Flow Logs
- B) AWS Config com Configuration Timeline do recurso
- C) CloudTrail com filtro para `AuthorizeSecurityGroupIngress`
- D) B e C fornecem visões complementares

<details><summary>Resposta</summary>

**D** — AWS Config: mostra o histórico de configuração do Security Group (estado antes e depois de cada mudança). CloudTrail: mostra qual usuário/role fez a mudança via API. Juntos: "o que mudou" (Config) + "quem mudou" (CloudTrail). No exame, Config é para "o quê" e CloudTrail é para "quem/quando".

</details>

---

**5.** Uma aplicação de microserviços tem latência alta intermitente. O time quer identificar qual serviço específico (Lambda, DynamoDB, API Gateway) está causando o atraso. Qual serviço de observabilidade usar?

- A) CloudWatch Logs com parsing manual de timestamps
- B) AWS X-Ray com tracing distribuído
- C) CloudWatch Metrics com dashboards
- D) CloudTrail para rastrear chamadas de API

<details><summary>Resposta</summary>

**B** — X-Ray: instrumenta a aplicação para criar traces end-to-end. O Service Map visualiza o tempo gasto em cada componente (API GW → Lambda → DynamoDB). Subsegments mostram exatamente onde a latência ocorre. Identifica gargalos em sistemas distribuídos complexos.

</details>

---

**6.** Uma empresa precisa manter logs do CloudTrail por 7 anos por requisito regulatório, mas o custo de armazenamento precisa ser minimizado. Qual é a configuração ideal?

- A) CloudTrail → CloudWatch Logs com retenção de 7 anos (2.555 dias)
- B) CloudTrail → S3 com lifecycle policy: Standard (90 dias) → Glacier (90-365 dias) → Glacier Deep Archive (após 1 ano)
- C) CloudTrail Lake com Event Data Store de 7 anos
- D) CloudTrail → S3 com Intelligent-Tiering

<details><summary>Resposta</summary>

**B** — Para armazenamento de longo prazo com custo mínimo: CloudTrail entrega logs para S3, lifecycle policy move para camadas mais baratas ao longo do tempo. Glacier Deep Archive ($0.00099/GB/mês) é a opção mais barata para dados raramente acessados. CloudTrail Lake (C) é excelente para queries mas custa mais.

</details>

---

**7.** Uma empresa quer detectar automaticamente atividades suspeitas na AWS como: reconhecimento de instâncias, chamadas incomuns de API às 3h, acesso de IPs de países desconhecidos. Qual serviço usar?

- A) AWS Config
- B) Amazon GuardDuty
- C) CloudTrail Insights
- D) AWS Security Hub

<details><summary>Resposta</summary>

**B** — GuardDuty: análise de Machine Learning sobre CloudTrail, VPC Flow Logs e DNS Logs. Detecta threats como reconhecimento, comprometimento de credenciais, atividade de mineração de criptomoedas, acesso de IPs de C&C (Command & Control). Sem configuração de regras manual.

</details>

---

**8.** Um desenvolvedor precisa executar comandos em instâncias EC2 em uma subnet privada sem abrir portas SSH (port 22). Qual solução implementar?

- A) Adicionar uma regra no SG para o IP do dev
- B) Criar um Bastion Host na subnet pública
- C) Usar AWS Systems Manager Session Manager
- D) Usar VPN Client AWS

<details><summary>Resposta</summary>

**C** — SSM Session Manager: acesso ao shell da instância via console AWS ou CLI sem portas abertas, sem key pairs, sem bastion host. O SSM Agent na instância + IAM role com `AmazonSSMManagedInstanceCore` são os únicos requisitos. Sessões são auditadas (CloudTrail) e podem ser gravadas (S3/CloudWatch).

</details>

---

**9.** Uma empresa recebe uma notificação que suas instâncias EC2 estão sem a tag obrigatória `CostCenter`. Eles querem que instâncias sem essa tag sejam automaticamente paradas. Qual configuração implementa isso?

- A) CloudWatch Alarm → SNS → Email para o time de operações
- B) AWS Config Rule para `required-tags` + Remediation Action via SSM Automation (StopInstances)
- C) Lambda agendada pelo EventBridge verificando tags via API
- D) Trusted Advisor para checar tags ausentes

<details><summary>Resposta</summary>

**B** — AWS Config `required-tags` rule detecta recursos sem as tags obrigatórias. Config Remediation com SSM Automation document para StopInstances executa a ação automaticamente quando a rule detecta Non-Compliant. Solução totalmente gerenciada sem código Lambda customizado.

</details>

---

**10.** Uma empresa tem 50 contas AWS na sua organização. O time de segurança precisa ter visibilidade centralizada de todos os findings de GuardDuty, Inspector, Macie e Config. Qual serviço consolida essa visão?

- A) AWS CloudTrail com multi-account trail
- B) AWS Security Hub com aggregação central na management account
- C) Amazon Detective para investigação consolidada
- D) VPC Flow Logs centralizados em S3

<details><summary>Resposta</summary>

**B** — AWS Security Hub: consolida findings de segurança de GuardDuty, Inspector, Macie, Config, IAM Access Analyzer e ferramentas de terceiros em um único painel. Com Organizations, delega automaticamente para a management account ou conta de segurança designada. Prioriza findings com score de severidade.

</details>

---

**11.** Uma aplicação Lambda gera logs estruturados em JSON. O time quer monitorar a taxa de erros e criar um alarme quando ultrapassar 1% das invocações. Qual é a solução?

- A) Usar as métricas padrão Lambda `Errors` e `Invocations` do CloudWatch
- B) CloudWatch Logs Metric Filter para extrair erros dos logs → CloudWatch Metric → Alarm
- C) CloudWatch Logs Insights com alertas
- D) X-Ray com fault detection

<details><summary>Resposta</summary>

**A** — As métricas padrão Lambda (`Errors`, `Invocations`) no CloudWatch já registram erros de execução. Crie um Alarm com a métrica `sum(Errors)/sum(Invocations) * 100 > 1`. Métrica Filter (B) é necessária apenas para erros customizados nos logs de aplicação que não são capturados pelas métricas padrão Lambda.

</details>

---

**12.** Uma empresa precisa manter um registro de todas as mudanças de configuração de recursos AWS por 2 anos para auditoria de conformidade. Qual configuração implementar?

- A) CloudTrail habilitado em todas as regiões com entrega para S3 com Object Lock Compliance
- B) AWS Config com delivery channel para S3 + retenção de histórico de configuração de 2 anos
- C) CloudWatch Logs com retenção de 730 dias para todos os logs de serviços
- D) A e B são complementares e ambas necessárias para auditoria completa

<details><summary>Resposta</summary>

**D** — Config registra "o que estava configurado" (snapshots de recursos); CloudTrail registra "o que mudou via API call". Para auditoria completa de conformidade, ambos são necessários. Config → S3 para retenção de longo prazo + CloudTrail → S3 com Object Lock para imutabilidade.

</details>

---

**13.** Uma empresa usa Parameter Store do Systems Manager para armazenar strings de conexão de banco de dados. Eles precisam garantir que esses parâmetros estejam criptografados com uma chave KMS específica e que apenas a aplicação possa descriptografar. Como implementar?

- A) Usar parâmetros tipo `String` criptografados com SHA256
- B) Usar parâmetros tipo `SecureString` com KMS CMK específico; IAM role da aplicação tem `kms:Decrypt` apenas para essa chave
- C) Armazenar em S3 criptografado com KMS
- D) Usar Secrets Manager ao invés de Parameter Store

<details><summary>Resposta</summary>

**B** — SecureString no Parameter Store usa KMS para criptografar o valor. Com CMK (Customer Managed Key), você controla a key policy: apenas a IAM role da aplicação tem `kms:Decrypt` para essa chave específica. Outros services/users não podem ler o valor mesmo con `ssm:GetParameter`.

</details>

---

**14.** Um time de operações quer saber quando qualquer instância EC2 entra em estado "impaired" (falha de status check) e automaticamente reiniciar a instância. Qual configuração implementar?

- A) CloudWatch Alarm na métrica `StatusCheckFailed_System` → Ação: EC2 Instance Recovery
- B) Auto Scaling Group com health check que substitui instâncias com falha
- C) Lambda agendada verificando status das instâncias
- D) A para single instances; B para instâncias em ASG

<details><summary>Resposta</summary>

**D** — Para instâncias únicas fora de ASG: EC2 Instance Recovery (via CloudWatch Alarm) move a instância para hardware saudável mantendo mesmo IP, EIP e instance ID. Para instâncias em ASG: o próprio ASG substitui instâncias unhealthy automaticamente (health checks). A escolha depende da arquitetura.

</details>

---

**15.** Uma empresa de segurança financeira precisa garantir que todos os logs de CloudTrail sejam imutáveis (ninguém possa deletar ou modificar, nem o root account). Qual configuração implementar?

- A) CloudTrail → S3 com S3 Object Lock em modo Compliance + Bucket Policy negando delete
- B) CloudTrail com CloudWatch Logs (logs ficam em CloudWatch que tem retenção automática)
- C) CloudTrail com MFA Delete no bucket S3
- D) S3 Versioning no bucket de logs do CloudTrail

<details><summary>Resposta</summary>

**A** — S3 Object Lock em modo Compliance com retention period: **nem o root account** pode deletar ou sobrescrever objetos durante o período de retenção. É a única forma de garantir imutabilidade absoluta. MFA Delete (C) e Versioning (D) ainda permitem deleção por admins autorizados.

</details>
