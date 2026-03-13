# Questões — Módulo 15: Migração e Transferência de Dados

> **Domínio SAA-C03**: Design de Arquiteturas Resilientes  
> **Dificuldade**: Média

---

**1.** Uma empresa precisa migrar 500 TB de dados de um datacenter para AWS. A conexão internet é de 1 Gbps compartilhado com a empresa. A transferência via internet levaria mais de 45 dias. Qual serviço usar?

- A) AWS DataSync com scheduler de transferência
- B) AWS Direct Connect para transferência direta
- C) AWS Snowball Edge Storage Optimized (múltiplos dispositivos)
- D) S3 Transfer Acceleration

<details><summary>Resposta</summary>

**C** — Snowball Edge Storage Optimized: 80 TB/dispositivo. Para 500 TB seriam necessários ~7 dispositivos. DataSync via internet (A) demoraria 45+ dias com 1 Gbps compartilhado. Direct Connect (B) requer meses de provisionamento e setup. Snow Family é ideal quando a transferência via rede levaria semanas.

</details>

---

**2.** Uma empresa está migrando um banco de dados Oracle 12c para Amazon Aurora PostgreSQL. A equipe precisa converter os stored procedures PL/SQL que são incompatíveis com PostgreSQL. Qual ferramenta usar?

- A) AWS DMS (Database Migration Service)
- B) AWS Schema Conversion Tool (SCT)
- C) AWS Glue para transformação de dados
- D) Amazon RDS para Oracle em lugar de Aurora

<details><summary>Resposta</summary>

**B** — SCT (Schema Conversion Tool): converte o schema e código procedural (stored procedures, functions, triggers, packages PL/SQL) de Oracle para o dialeto PostgreSQL. Identifica objetos que não podem ser convertidos automaticamente e gera um relatório. Depois, use DMS para a migração dos dados.

</details>

---

**3.** Uma empresa quer migrar 50 servidores Windows on-prem para Amazon EC2 com o mínimo de alterações. A janela de manutenção é de apenas 2 horas. Qual serviço usar?

- A) AWS Application Migration Service (MGN) com replicação contínua antes do cutover
- B) AWS SMS (Server Migration Service) com cutover agendado
- C) Criar AMIs manualmente e lançar instâncias EC2
- D) AWS DMS para migrar servidor completo

<details><summary>Resposta</summary>

**A** — MGN instala um replication agent nos servidores fonte e replica continuamente os discos para AWS. Quando pronto para cutover, a janela de manutenção é apenas para o delta final (minutos, não horas). RTO de ~30-60 minutos vs janela de 2 horas é alcançável. SMS é o serviço legado substituído pelo MGN.

</details>

---

**4.** Uma empresa de varejo tem parceiros B2B que só conseguem transferir arquivos via SFTP. A empresa quer receber esses arquivos diretamente no S3 sem manter um servidor SFTP. Qual serviço usar?

- A) AWS DataSync com endpoint SFTP
- B) AWS Transfer Family com protocolo SFTP e backend S3
- C) AWS Storage Gateway File Gateway
- D) S3 com pre-signed URLs para upload

<details><summary>Resposta</summary>

**B** — Transfer Family: cria um endpoint SFTP gerenciado (sem servidor para gerenciar). Parceiros continuam usando seus clientes SFTP existentes sem alterações. Arquivos são armazenados diretamente em S3. Autenticação pode ser via IAM, Cognito ou Lambda customizado (para LDAP/AD).

</details>

---

**5.** Uma empresa quer migrar um banco de dados MySQL de 2 TB on-prem para Amazon RDS MySQL com tempo de inatividade menor que 15 minutos. Qual é a estratégia?

- A) Exportar dump MySQL, subir para S3, restaurar no RDS (downtime de horas)
- B) DMS full load + CDC: migrar dados iniciais, depois replicar changes, cutover com janela mínima
- C) Criar Read Replica no RDS apontando para MySQL on-prem e promover
- D) AWS Database Migration Service com snapshot migration

<details><summary>Resposta</summary>

**B** — DMS com CDC (Change Data Capture): primeiro faz full load do banco para RDS (sem impacto na produção). Depois habilita CDC para replicar mudanças em tempo real. Quando os dois bancos estiverem em sincronia, a janela de cutover é mínima (apenas o lag acumulado, que pode ser segundos). Downtime total < 15 minutos.

</details>

---

**6.** Um time quer migrar dados de arquivos NAS on-prem para Amazon EFS, garantindo que timestamps e permissões Unix sejam preservados. Qual serviço usar?

- A) AWS Snowball Edge + rsync
- B) AWS DataSync com preservação de metadados
- C) FTP para S3 + script de cópia
- D) AWS Storage Gateway File Gateway

<details><summary>Resposta</summary>

**B** — DataSync: preserva metadados de arquivo (timestamps de modificação, acesso, criação; permissões Unix uid/gid/chmod). Suporta NFS → EFS. Realiza verificação de integridade (checksums) durante e após a transferência. Mais confiável e rápido que rsync manual.

</details>

---

**7.** Uma empresa quer avaliar sua frota de servidores on-prem antes da migração para identificar dependências entre servidores e padrões de utilização. O ambiente é VMware. Qual serviço e modo usar?

- A) Application Discovery Service com Agent-based mode em cada servidor
- B) Application Discovery Service Agentless com VMware vCenter Connector
- C) AWS Migration Hub com análise manual
- D) AWS MGN com análise de pré-migração

<details><summary>Resposta</summary>

**B** — ADS Agentless: appliance de VM no VMware vCenter coletada CPU, memória, storage e rede de VMs sem instalar agent em cada servidor. Mais rápido de implementar para inventário inicial. Agent-based (A) fornece dados mais detalhados (processos, conexões de rede) mas requer instalação em cada servidor.

</details>

---

**8.** Uma empresa descobriu que 30% das suas aplicações on-prem têm baixo uso e serão descontinuadas nos próximos 6 meses. Qual é a estratégia de migração correta para essas aplicações?

- A) Rehost (lift-and-shift) para EC2
- B) Replatform para serviços gerenciados
- C) Retire (não migrar — eliminar)
- D) Retain (manter on-prem até descontinuação)

<details><summary>Resposta</summary>

**C** — Retire: se a aplicação será descontinuada em 6 meses, não faz sentido investir em migração. Eliminar agora reduz custos operacionais imediatamente. Retain (D) seria correto se houvesse uma razão técnica para manter on-prem temporariamente, mas sabendo que serão descontinuadas, retire imediatamente.

</details>

---

**9.** Uma empresa quer mover seu CRM legado on-prem (sem suporte do fornecedor) para um CRM SaaS moderno na nuvem. Qual estratégia de migração os 7Rs classificaria isso?

- A) Rehost
- B) Replatform
- C) Repurchase
- D) Refactor

<details><summary>Resposta</summary>

**C** — Repurchase: substituir por um produto SaaS (ex: Salesforce, HubSpot, SAP). O código não é migrado; os dados são migrados para o novo sistema SaaS. A empresa abandona o sistema legado e adota uma solução moderna pronta.

</details>

---

**10.** Uma empresa precisa migrar 10 PB de dados de fitas magnéticas para S3 para retenção de longo prazo. Qual serviço AWS usar?

- A) 100 dispositivos Snowball Edge
- B) AWS Snowmobile (caminhão com 100 PB de capacidade)
- C) Direct Connect dedicado + DataSync
- D) 125 dispositivos Snowball Edge Storage Optimized

<details><summary>Resposta</summary>

**D** — Snowball Edge Storage Optimized: 80 TB por dispositivo. Para 10 PB: 10.000 TB / 80 TB = 125 dispositivos. Snowmobile (B) é para migrações > 100 PB (exabyte-scale). Para 10 PB, múltiplos Snowball Edges em paralelo é a solução prática.

</details>

---

**11.** Uma empresa quer migrar seu banco Oracle RAC on-prem para AWS. O Oracle RAC usa features proprietárias não disponíveis no RDS. Qual é a melhor opção?

- A) RDS Oracle Multi-AZ
- B) Amazon RDS Custom for Oracle (acesso ao sistema operacional e engine)
- C) Oracle em EC2 com shared storage EFS
- D) Aurora PostgreSQL com compatibilidade Oracle

<details><summary>Resposta</summary>

**B** — RDS Custom for Oracle: opção gerenciada que ainda permite acesso ao SO e customização da engine (para clientes que precisam de patches específicos, features como RAC via workarounds, ou dependências externas). Mais gerenciado que Oracle em EC2 puro, mas mais flexível que RDS Oracle padrão.

</details>

---

**12.** Durante uma migração DMS de SQL Server para Aurora MySQL, o time percebe que triggers no SQL Server não são compatíveis. Qual é o papel do SCT nesse cenário?

- A) O SCT migra automaticamente os dados como o DMS
- B) O SCT converte automaticamente os triggers SQL Server T-SQL para MySQL SQL, identificando incompatibilidades para ajuste manual
- C) O SCT verifica apenas a compatibilidade de tipos de dados
- D) O SCT não é necessário para SQL Server → MySQL

<details><summary>Resposta</summary>

**B** — SCT analisa o código procedural SQL Server (T-SQL: stored procedures, triggers, functions) e converte para MySQL SQL onde possível. Para código que não pode ser convertido automaticamente (features proprietárias T-SQL), gera relatório com ações manuais necessárias e indica complexidade de conversão por objeto.

</details>

---

**13.** Uma empresa opera um data center com 200 servidores on-prem e quer saber quanto vai custar operá-los na AWS antes de comprometer a migração. Qual ferramenta usar?

- A) AWS Cost Explorer com projeções
- B) AWS Migration Evaluator (Agentless Collector + TCO Analysis)
- C) AWS Compute Optimizer
- D) Trusted Advisor com análise de custo

<details><summary>Resposta</summary>

**B** — AWS Migration Evaluator (antigo TSO Logic): coleta dados de utilização do ambiente on-prem (via agentless collector ou importação de dados) e fornece análise detalhada de TCO (Total Cost of Ownership) comparando custos atuais vs AWS. Base para business case de migração.

</details>

---

**14.** Uma empresa precisa mover dados diariamente de um servidor NFS on-prem para um bucket S3 de forma agendada e verificada. Qual serviço usar?

- A) Storage Gateway File Gateway
- B) AWS DataSync com task agendada
- C) Direct Connect + S3 Transfer Acceleration
- D) AWS Snowcone com DataSync agent embutido

<details><summary>Resposta</summary>

**B** — DataSync: instala o agente no servidor on-prem (ou usa DataSync VM agent), conecta ao NFS source e S3 destination, cria uma task com schedule (diário). Verifica integridade com checksums e realiza apenas transferência incremental após a primeira full sync. Mais simples que gerenciar Storage Gateway para migração.

</details>

---

**15.** Uma empresa de manufatura tem equipamentos IoT antigos em locais remotos com conectividade de internet limitada. Precisam coletar e processar dados localmente antes de enviar para AWS. Qual dispositivo Snow usar?

- A) AWS Snowmobile
- B) AWS Snowball Edge Compute Optimized (edge computing + storage)
- C) AWS Snowcone com edge computing básico
- D) AWS Outposts

<details><summary>Resposta</summary>

**B** — Snowball Edge Compute Optimized: 52 vCPUs, 208 GB RAM, armazenamento de 42 TB + 28 TB NVMe SSD, GPU opcional. Executa EC2 instances (AMIs localmente), Lambda functions e SageMaker inference no edge. Ideal para processar dados de IoT em locais remotos e sincronizar com AWS quando conectividade está disponível.

</details>
