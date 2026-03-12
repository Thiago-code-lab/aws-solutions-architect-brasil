# Flashcards — Módulo 14: Monitoramento, CloudWatch e CloudTrail

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** Como o CloudWatch Agent permite monitorar memória e disco no EC2?  
**R:** Por padrão, CloudWatch **não** coleta métricas de memória RAM, uso de disco, processos. Instalar o **CloudWatch Agent** na instância permite enviar métricas customizadas ao CloudWatch (ex: `mem_used_percent`, `disk_used_percent`). O Agent também coleta logs e os envia ao CloudWatch Logs.

---

**P:** O que é CloudWatch Logs Insights?  
**R:** Engine de **query interativa** para CloudWatch Logs. Sintaxe própria (similar SQL). Permite: filtrar, agregar, visualizar logs. Ex: `fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 20`. Paga por dados escaneados. Resultados visuais (tabelas, gráficos de séries temporais).

---

**P:** O que é CloudTrail e quais eventos ele registra?  
**R:** Serviço de **auditoria de API calls** na AWS. Registra: quem chamou qual API, quando, de qual IP, com qual resultado. Tipos: **Management Events** (criar/deletar recursos — CloudFormation, IAM, EC2 start/stop), **Data Events** (operações sobre dados — S3 GetObject, Lambda Invoke; desabilitado por padrão), **Insights Events** (atividade incomum de API).

---

**P:** Qual é a diferença entre CloudTrail e AWS Config?  
**R:** **CloudTrail:** "quem fez o quê e quando?" — histórico de API calls. Auditoria de ações. **AWS Config:** "como está configurado agora e como mudou?" — histórico de configurações de recursos. Compliance: avaliar recursos contra regras. CloudTrail registra eventos; Config registra estados de configuração.

---

**P:** O que são AWS Config Rules e como funcionam?  
**R:** Regras que avaliam se recursos AWS estão conformes com políticas. **Managed Rules:** pré-construídas pela AWS (ex: `s3-bucket-public-read-prohibited`). **Custom Rules:** código Lambda que avalia conformidade. Avaliação: triggered por changes ou periódica. Status: COMPLIANT / NON_COMPLIANT. Remediações automáticas via SSM.

---

**P:** O que é o AWS X-Ray e quais conceitos são centrais?  
**R:** Serviço de **distributed tracing** para aplicações. Conceitos: **Trace** = request end-to-end; **Segment** = work feito por um serviço; **Subsegment** = granular sub-work; **Annotations** = key-value para filtrar traces; **Sampling** = % de requests a tracear (padrão: 5%). Service Map: visualização das dependências e latências.

---

**P:** O que é CloudWatch Anomaly Detection?  
**R:** Usa ML para criar um modelo de comportamento esperado para uma métrica (baseado em histórico). Cria uma banda dinâmica de "normal". Alertas quando a métrica sai da banda. Adapta-se a sazonalidade (dias da semana, horários). Mais sofisticado que alarme de threshold estático.

---

**P:** O que é AWS Systems Manager Session Manager e por que é mais seguro que SSH?  
**R:** Acesso a instâncias EC2 (e on-premises gerenciados) via browser ou CLI **sem abrir porta 22 (SSH) e sem SSH keys**. Auditável: todas as sessões são logadas no CloudWatch/S3. Funciona em instâncias em subnets privadas sem bastion host. Requer SSM Agent na instância + IAM Role com permissão `ssm:StartSession`.

---

**P:** Quais são os dois tiers do SSM Parameter Store?  
**R:** **Standard:** gratuito, até 10.000 parâmetros, valor até 4 KB, sem Advanced Policies, sem TTL. **Advanced:** pago ($0.05/mês por parâmetro), até 100.000 parâmetros, valor até 8 KB, suporta Parameter Policies (TTL automático, notificações de expiração). Para secrets rotativos: usar Secrets Manager.

---

**P:** O que é CloudWatch Contributor Insights?  
**R:** Analisa logs e métricas para identificar os **top contributors** de latência, erro ou custo. Ex: "quais 10 IPs geram mais erros 5XX?" ou "quais clientes têm maior latência de API?". Cria regras com padrões de log. Útil para: segurança (detecção de IPs suspeitos), performance (identificar usuários/recursos que degradam o sistema).

---

**P:** Como fazer o CloudTrail logs ser à prova de adulteração?  
**R:** Ativar **Log File Validation** no trail: CloudTrail gera um Digest file (hash SHA-256) por hora. Qualquer modificação no log file é detectável comparando com o digest. Armazenar logs em bucket S3 com **Object Lock (Compliance mode)** + MFA Delete + políticas restritivas. CloudTrail Lake para análise imutável.

---

**P:** O que é AWS Config Aggregator?  
**R:** Coleta dados de conformidade do AWS Config de múltiplas contas e regiões em uma **conta central**. Dois tipos de source: contas individuais (lista de account IDs) ou Organization (todas as contas da AWS Org automaticamente). Permite visão centralizada de compliance em multi-conta. A conta central precisa de permissão das fontes via delegated administrator.

---

**P:** O que é o SSM Patch Manager?  
**R:** Automatiza patching de instâncias EC2 (e on-premises). **Patch Baselines:** define quais patches instalar (severidade, classificação). **Maintenance Windows:** janelas de tempo para aplicar patches. **Compliance Reporting:** reporta no Config qual instância está em dia. Pode usar Run Command para aplicar patches on-demand.

---

**P:** Qual é a retenção padrão de logs do CloudWatch Logs?  
**R:** Por padrão: **indefinida** (logs nunca expiram). Configurável por Log Group: 1 dia a 10 anos, ou "Never Expire". Para reduzir custo: configurar retention policy. Para compliance: configurar retention adequada. Para longo prazo: exportar para S3 (CloudWatch Logs Subscription Filters → Kinesis Firehose → S3).

---

**P:** O que é AWS Trusted Advisor?  
**R:** Advisor que analisa a conta AWS em 5 categorias: **Cost Optimization** (instâncias ociosas, Reserved não utilizadas), **Performance** (EBS throughput, EC2 type), **Security** (SGs abertos, MFA root, S3 público), **Fault Tolerance** (RDS backups, ELB health checks), **Service Limits** (alertas ao aproximar limites). Verificações básicas gratuitas; completo com Business/Enterprise Support.

---

**P:** Como o CloudWatch Alarm monitora o estado de uma instância EC2 e aciona recover?  
**R:** Alarme na métrica `StatusCheckFailed_System` (falha no hardware subjacente). Ação de alarme: **EC2 Instance Recovery** — move a instância para hardware saudável mantendo o mesmo IP privado, IP Elástico, metadados e volumes EBS. (vs. Reboot que apenas reinicia no mesmo hardware).

---

**P:** O que é CloudWatch Synthetics Canary?  
**R:** Scripts (Node.js ou Python) que emulam comportamento de usuário em aplicações web/APIs em intervalos configuráveis. Monitora: disponibilidade, latência, fluxos de usuário (login → checkout). Detecta problemas antes de usuários reais serem afetados. Métricas enviadas para CloudWatch. Diferente de Real User Monitoring.

---

**P:** O que é EventBridge (ex-CloudWatch Events)?  
**R:** Barramento de eventos serverless. Conecta serviços AWS e SaaS. Sources: EC2 state changes, CodePipeline stages, Scheduled events (cron), S3 events, custom events. Targets: Lambda, SQS, SNS, Step Functions, Kinesis, ECS, API Gateway. Rules com event pattern matching (filtros JSON). EventBridge Pipes para transformação ponto-a-ponto.

---

**P:** O que é CloudWatch Logs Subscription Filter?  
**R:** Em tempo real, filtra e entrega logs de um Log Group para: **Kinesis Data Streams**, **Kinesis Firehose**, **Lambda**. Use cases: análise em tempo real de logs (KDS → Lambda), armazenamento longo prazo (KDF → S3), agregação multi-conta (cross-account subscription destinations).

---

**P:** Como o AWS Config detecta recursos não conformes automaticamente e remedeia?  
**R:** **Detecção:** Config Rule avaliada continuamente ou por mudança. Status NON_COMPLIANT alertado no console e via EventBridge. **Remediação automática:** SSM Automation Document executado automaticamente quando regra detecta não conformidade. Ex: bucket S3 público → remediation executa SSM que aplica Block Public Access.

---

**P:** O que é CloudTrail Insights?  
**R:** Detecta atividade anômala de API calls. Analisa padrão normal de write management events e alerta quando há desvio significativo (ex: pico incomum de `TerminateInstances`, `DeleteSecurityGroup`). Casos de uso: detectar credenciais comprometidas, erros de automação, ataques. Insights events são cobrados separadamente.

---

**P:** Qual é a diferença entre CloudWatch Metrics e CloudWatch Custom Metrics?  
**R:** **Built-in Metrics:** coletadas automaticamente por serviços AWS (CPU EC2, latência ELB, InvocationDuration Lambda). Granularidade padrão: 5 minutos (1 min com "detailed monitoring"). **Custom Metrics:** enviadas via PutMetricData API ou CloudWatch Agent de aplicações customizadas. Granularidade: 1 segundo (High Resolution). Exemplos: métricas de negócio, memória da aplicação.

---

**P:** O que é o AWS Security Hub?  
**R:** Centralizador de **security findings** de múltiplos serviços (GuardDuty, Inspector, Macie, IAM Analyzer, Config, Firewall Manager, 3rd party). Dashboard unificado. Scoring de postura de segurança contra padrões (CIS AWS Benchmarks, AWS Foundational Security Best Practices). Automatiza remediações via EventBridge. Multi-account via Organization.

---

**P:** O que é o SSM Run Command?  
**R:** Executa scripts (Shell, PowerShell, etc.) em instâncias EC2 e servidores on-premises gerenciados em larga escala. Sem SSH necessário. Targets: instâncias por tag, por instance ID, todos os managed instances. Auditável: output enviado para S3/CloudWatch. Rate control para aplicar gradualmente. Base do Patch Manager e outros automação SSM.

---

**P:** Qual a diferença entre CloudWatch Logs e CloudTrail para auditoria de segurança?  
**R:** **CloudTrail:** auditoria de ações na AWS (API calls, console logins, IAM changes). Respondem: "foi deletado/modificado? Por quem? Quando?". **CloudWatch Logs:** logs de aplicações, OS, serviços AWS (VPC Flow Logs, ALB access logs). Respondem: "o que aconteceu dentro da aplicação?". Para auditoria completa: CloudTrail (quem fez o quê) + CloudWatch Logs (o que aconteceu no sistema).
