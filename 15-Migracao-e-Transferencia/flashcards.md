# Flashcards — Módulo 15: Migração e Transferência

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** Quais são os 7 Rs da estratégia de migração cloud?  
**R:** **Retire** (descomissionar), **Retain** (manter on-premises), **Rehost** (lift-and-shift, mínima mudança), **Relocate** (mover VMware para VMware Cloud on AWS), **Replatform** (otimização pontual sem refactoring — ex: RDS em vez de MySQL self-managed), **Repurchase** (mudar para SaaS), **Refactor/Rearchitect** (redesign para cloud-native).

---

**P:** O que é AWS MGN (Application Migration Service) e como funciona?  
**R:** Servilho de **rehosting** (lift-and-shift) para EC2. Agente instalado no servidor de origem faz **replicação contínua** (bloco a bloco) para a AWS. No cutover: servidor origin para, AWS lança instância EC2 do snapshot mais recente. Minimiza downtime (~minutos). Substitui SMS (Server Migration Service).

---

**P:** Qual é a capacidade do Snowmobile?  
**R:** **100 PB** por viagem (caminhão físico com container de storage). Para migração de datacenters inteiros (100 PB+). Mais rápido que transferência pela internet quando volume é > 10 PB (regra geral: Snowball quando > 10 TB, Snowmobile quando > 10 PB). 

---

**P:** Quais são as diferenças entre as 3 variantes Snow Family?  
**R:** **Snowcone Small (8 TB SSD) / Large (14 TB SSD):** portátil, locais remotos, edge computing, 2 vCPUs. **Snowball Edge Storage Optimized (80 TB HDD):** migração em escala, 40 vCPUs, transferência para S3. **Snowball Edge Compute Optimized (42 TB HDD):** edge computing pesado, 52 vCPUs, GPU opcional. **Snowmobile:** 100 PB, datacenters inteiros.

---

**P:** O que é AWS DataSync e quais fontes suporta?  
**R:** Serviço de transferência de dados **online** (pela rede). Fontes: NFS, SMB (on-premises ou outros clouds), HDFS, S3 on-premises, self-managed object storage. Destinos: S3 (qualquer classe), EFS, FSx for Windows/Lustre/ONTAP. Features: **preserva metadata** (permissões, timestamps), verificação de integridade, transferência agendada ou on-demand.

---

**P:** Qual é a diferença entre AWS DataSync e AWS Storage Gateway?  
**R:** **DataSync:** migração/transferência de dados em lote (one-time ou agendado). Move arquivos de A para B. **Storage Gateway:** integração **híbrida contínua** — extensão de armazenamento on-premises para AWS. File GW: NFS/SMB → S3. Volume GW: iSCSI → EBS/S3. Tape GW: emula tape library → S3/Glacier. DataSync = mover dados; Storage Gateway = extensão permanente.

---

**P:** O que é AWS DMS e quando é necessário também usar SCT?  
**R:** **DMS (Database Migration Service):** migra dados com CDC (Change Data Capture) para mínimo downtime. **SCT (Schema Conversion Tool):** converte schema e código PL/SQL entre engines incompatíveis. **Regra:** migração **homogênea** (Oracle → Oracle RDS): apenas DMS. Migração **heterogênea** (Oracle → Aurora PostgreSQL): SCT primeiro (converte schema) + DMS (migra dados).

---

**P:** O que é replicação contínua no AWS DMS (CDC)?  
**R:** **CDC (Change Data Capture):** após a carga inicial completa, DMS captura e aplica mudanças do banco de origem em tempo real (inserts, updates, deletes). Mantém os bancos sincronizados durante a migração. Permite cutover com mínima janela de manutenção — fonte pode continuar recebendo transações até o cutover.

---

**P:** O que é o AWS Migration Hub?  
**R:** Dashboard centralizado para rastrear o progresso de migração de múltiplas aplicações e servidores. Integra com MGN, DMS, CloudEndure. Categoriza servidores discovered e rastreia status (Discovered → Not Started → In Progress → Migrated). Não executa migrações — é apenas tracking e visibilidade multi-ferramenta.

---

**P:** O que é AWS Application Discovery Service?  
**R:** Descobre e avalia workloads on-premises para planejamento de migração. **Agentless Discovery (VMware only):** via VMware vCenter, coleta VM specs (CPU, memória, storage) sem instalar agentes. **Agent-based Discovery:** instala agente em cada servidor, coleta: CPU, memória, processos, conexões de rede (mapeamento de dependências). Dados enviados para Migration Hub.

---

**P:** O que é AWS Transfer Family?  
**R:** Serviço gerenciado para transferência segura de arquivos usando protocolos legados: **SFTP, FTPS, FTP e AS2**. Armazenamento backend: S3 ou EFS. Casos de uso: B2B file exchange, parceiros que precisam de SFTP, migração de servidores FTP legados para AWS. Mantém os clientes existentes sem mudar de protocolo.

---

**P:** Como calcular se devo usar Snowball ou transferência de rede?  
**R:** Regra de thumb: **Snowball** quando a transferência pela internet levaria > 1 semana. Fórmula: `tempo = tamanho_dados / largura_banda_disponível`. Ex: 100 TB / 100 Mbps = ~100 dias. Use Snowball. 1 TB / 1 Gbps = ~2,2 horas. Use internet. Considerar também: custo de banda, segurança dos dados in-transit.

---

**P:** Qual é o benefício do Direct Connect para migração em vez de VPN?  
**R:** **Direct Connect:** conexão privada dedicada entre data center e AWS via parceiro (10 Gbps ou 100 Gbps). Latência consistente, banda dedicada, mais caro, leva semanas para provisionar. **VPN:** criptografado sobre internet pública, variação de latência, mais barato, provisionado em horas. Para migração de grandes volumes: Direct Connect é mais eficiente (bandwidth constante).

---

**P:** O que é o AWS Snowball Edge com computação e quando usar?  
**R:** **Snowball Edge Compute Optimized:** 52 vCPUs, GPU opcional (NVIDIA Tesla), 42 TB HDD. Use quando: precisa processar dados **no edge** (fábricas, navios, locais sem conectividade estável) antes de enviar — pre-processing, compressão, ML inference no campo. Roda EC2 instances (sbe-c, sbe-m família) e Lambda functions.

---

**P:** O que é AWS DataSync Location?  
**R:** Endpoint configurado no DataSync representando ou a origem ou o destino de uma transferência. Tipos: NFS Server, SMB Share, Hadoop HDFS, Amazon S3, Amazon EFS, Amazon FSx, object storage. Uma **Task** conecta dois Locations (source → destination) e define as configurações de transferência (schedule, filter, verification).

---

**P:** O que é a fase de "Discovery" na metodologia de migração AWS?  
**R:** Fase inicial: inventariar e avaliar workloads on-premises. Ferramentas: Application Discovery Service, Migration Evaluator (TCO analysis — calcula quanto custa manter on-premises vs migrar para AWS). Output: lista de servidores/apps com specs, dependências mapeadas, priorização por complexidade e impacto.

---

**P:** Quantas horas leva o provisionamento de um Snowball após solicitação?  
**R:** Tipicamente **5-7 dias úteis** para entrega após pedido. Processo: solicitar no console → AWS ships Snowball → empresa carrega dados → devolve para AWS → AWS importa para S3 (1-3 dias após recebimento). Total: típico de 2-3 semanas. Para urgências: múltiplos Snowballs em paralelo.

---

**P:** O que é o VMware Cloud on AWS e quando usar Relocate?  
**R:** **VMware Cloud on AWS:** run VMware SDDC (Software-Defined Data Center) em hardware dedicado AWS. Para migrações de VMs VMware sem precisar refatorar. Você gerencia via vCenter como on-premises. **Relocate:** migrar VMs VMware para VMware Cloud on AWS sem conversão — migration quase zero-touch, mantendo o mesmo stack VMware.

---

**P:** Como o DMS suporta Oracle para Amazon Aurora PostgreSQL?  
**R:** Processo em 2 passos: **(1) SCT:** converte schema Oracle (tabelas, constraints, views, procedures PL/SQL → PL/pgSQL). Relatório de conversão indica complexidade manual. **(2) DMS:** carga inicial full-load dos dados + CDC ongoing. DMS suporta Oracle como source com LogMiner (para CDC), capturando redo logs.

---

**P:** O que é AWS DataSync preservation de metadata e por que importa?  
**R:** DataSync preserva: **POSIX permissions** (owner, group, permissions), **timestamps** (creation, modification, access), **ACLs** (para SMB/NFS). Por que importa: arquivos migrados mantêm permissões originais — aplicações que dependem de permissões de arquivo continuam funcionando sem reconfiguração manual. S3 Object Metadata também é preservado.

---

**P:** O que são as Snowball Edge "job types"?  
**R:** **Import to S3:** transferir dados de on-premises para S3 (uso mais comum). **Export from S3:** extrair dados do S3 para on-premises (dado sai da AWS). **Local Compute and Storage:** usar Snowball em edge para processamento sem enviar dados — runs EC2/Lambda localmente. Cada job type tem processo de shipping diferente.

---

**P:** O que é AWS Migration Evaluator?  
**R:** Ferramenta gratuita de análise de TCO (Total Cost of Ownership) on-premises vs AWS. Instala agentes ou usa importação de dados de utilização. Gera relatório: "business case" mostrando custo estimado na AWS vs custo atual on-premises. Base para aprovação executiva de migração.

---

**P:** O que é Server Migration Service (SMS) e qual é seu substituto?  
**R:** SMS: serviço legado de replicação de VMs on-premises para AMIs na AWS. **Substituído pelo AWS MGN (Application Migration Service)** — mais moderno, replicação contínua incremental em vez de snapshot periódico, menor downtime. SMS foi descontinuado para novos usuários. MGN é o serviço recomendado.

---

**P:** Como o Snowball protege os dados físicos?  
**R:** Proteção física e lógica: criptografia AES-256 automática, **Trusted Platform Module (TPM)** para verificação de integridade do hardware, case tamper-evident, não é possível acessar os dados sem as chaves do KMS associadas ao job. Após importar para S3, os dados são apagados do Snowball conforme NIST 800-88.

---

**P:** O que é o AWS DataSync Task Schedule vs Task Execution?  
**R:** **Task:** configuração persistente da transferência (source, destination, options). **Task Execution:** uma rodada de execução da task. **Schedule:** execução periódica automática (cron/rate). Uma Task pode ter múltiplas Executions. Cada execution pode ser: LAUNCHING → PREPARING → TRANSFERRING → VERIFYING → SUCCESS/ERROR.
