# Casos de Uso Reais — Simulados e Questões (Módulo 19)

## Caso 1 — Analisando uma Questão Difícil sobre Alta Disponibilidade

**Contexto:** Questão de exame pede a arquitetura de maior disponibilidade para um banco de dados relacional com requisito de failover automático < 30 segundos, sem perda de dados.

**Questão Simulada:**
> Uma empresa precisa de um banco de dados relacional MySQL com RTO < 30 segundos e RPO = 0 (zero perda de dados). Qual configuração atende esses requisitos?
> A) RDS MySQL com duas Read Replicas em AZs diferentes
> B) RDS MySQL Multi-AZ com replica síncrona em AZ separada
> C) RDS MySQL com backup automático diário habilitado
> D) EC2 MySQL com script de failover Lambda

**Análise de Por Que Cada Opção é Certa ou Errada:**
```
A) ❌ Read Replicas são ASSÍNCRONAS → pode ter lag (RPO ≠ 0)
      Read Replicas não fazem failover automático para writes
      Read Replicas = escalar reads, não HA

B) ✅ CORRETO:
   Multi-AZ = réplica SÍNCRONA → RPO = 0 (sem perda de dados)
   Failover automático: 15-30 segundos
   DNS endpoint muda automaticamente para standby
   
C) ❌ Backup diário = RPO de até 24h (muito longe de 0)
      Restore de backup leva horas (RTO muito alto)

D) ❌ Gerenciamento manual = complexidade, possível erro humano
      EC2 MySQL = você gerencia SO, patches, HA
      Lambda failover = latência adicional e falha possível

PALAVRA-CHAVE IDENTIFICADA: "failover automático" + "sem perda de dados"
→ Multi-AZ RDS (síncrono = RPO 0, automático = RTO < 30s)
```

---

## Caso 2 — Questão-Armadilha: Multi-AZ vs Read Replica

**Contexto:** Questão de exame mistura os dois conceitos para confundir candidatos.

**Questão Simulada:**
> Uma aplicação de e-commerce tem lentidão em consultas de relatório que impactam as transações dos clientes. O DBA identifica que queries analíticas pesadas são executadas no mesmo banco que processa pedidos. Qual é a solução mais simples?
> A) Habilitar RDS Multi-AZ no banco existente
> B) Criar uma Read Replica e direcionar queries analíticas para ela
> C) Migrar para Aurora Serverless
> D) Adicionar ElastiCache na frente do RDS

**Análise:**
```
A) ❌ Multi-AZ = HA/failover, não melhora performance
      A replica Multi-AZ é STANDBY (não aceita reads)
      Habilitar Multi-AZ não resolve o problema de carga

B) ✅ CORRETO:
   Read Replica = cópia assíncrona para leitura
   Direcionar queries analíticas para Read Replica endpoint
   Banco principal fica dedicado apenas para transações (writes)
   
C) ⚠️ Funciona mas é mudança maior (migração de dados, custo)
      "Mais simples" aponta para Read Replica
      
D) ⚠️ ElastiCache ajuda para queries repetidas/idênticas
      Queries analíticas são geralmente únicas (ad hoc)
      Não resolve queries complexas que mudam sempre

LIÇÃO: "problema de performance" + "queries de leitura pesadas" → Read Replica
       "failover/disponibilidade" → Multi-AZ
```

---

## Caso 3 — Analisando Questão sobre Custo-Eficiência

**Contexto:** Questão pede escolha de storage S3 baseada em padrão de acesso específico.

**Questão Simulada:**
> Uma empresa armazena logs de aplicação no S3. Logs do mês atual são acessados diariamente para debug. Logs com mais de 30 dias são raramente acessados mas precisam estar disponíveis em minutos quando necessário. Logs com mais de 1 ano podem ter retrieval de horas. Qual combinação de classes S3 é mais econômica?
> A) Tudo em S3 Standard
> B) S3 Standard por 30 dias, depois Glacier Deep Archive
> C) S3 Standard por 30 dias, Standard-IA por 30-365 dias, depois Glacier Flexible Retrieval
> D) S3 Intelligent-Tiering para todos os objetos

**Análise:**
```
Mapeamento dos requisitos:
- 0-30 dias: acesso frequente, debug diário → S3 Standard
- 30-365 dias: raramente acessados, disponível "em minutos" → Standard-IA
  (NOT Glacier — Glacier leva horas para Flexible ou 12h para Deep Archive)
- > 1 ano: retrieval pode demorar horas → Glacier Flexible (1-5h) ✓

A) ❌ Custo muito alto, tudo em Standard mesmo arquivos antigos
B) ❌ Depois de 30 dias vai para Deep Archive (12h retrieval)
      Mas o requisito 30-365 dias pede "disponível em minutos"!
C) ✅ CORRETO — mapeamento perfeito por período
D) ⚠️ Funciona automaticamente mas:
      Taxa de monitoramento por objeto ($0.0025/1000 objetos)
      Para muitos objetos pequenos: pode ser mais caro que C
      "Mais econômica" → C é mais otimizado quando padrão é conhecido

LIÇÃO: Glacier = horas (não minutos). Standard-IA = milissegundos (custo menor)
```

---

## Caso 4 — Questão sobre Segurança: Qual Serviço para Qual Ameaça

**Contexto:** Questões de segurança frequentemente confundem GuardDuty, WAF, Shield, Macie e Inspector.

**Questão Simulada:**
> Uma empresa detectou que seu bucket S3 com dados de clientes contém CPFs e cartões de crédito expostos sem criptografia. Qual serviço da AWS identifica e alerta sobre esses dados sensíveis?
> A) AWS GuardDuty
> B) Amazon Macie
> C) AWS WAF
> D) AWS Config

**Análise:**
```
A) ❌ GuardDuty = detecta AMEAÇAS (comportamento anômalo, mineração, exfiltração)
      Analisa: VPC Flow Logs, CloudTrail, DNS
      NÃO analisa CONTEÚDO de dados no S3
      
B) ✅ CORRETO — Amazon Macie:
   Usa ML para descobrir dados sensíveis em S3
   Identifica: PII (CPF, nome, email), PCI (cartões), credenciais
   Gera findings: "S3BucketEncryptionDisabled" + "SensitiveData:S3Object/PII"
   
C) ❌ WAF = filtro de requisições web HTTP (SQLi, XSS)
      NÃO analisa conteúdo de objetos armazenados
      
D) ❌ AWS Config = conformidade de CONFIGURAÇÃO de recursos
      Verifica se bucket tem criptografia habilitada (configuração)
      NÃO verifica CONTEÚDO dos objetos

MAPEAMENTO SERVIÇOS DE SEGURANÇA:
├── Dados sensíveis em S3 → Macie
├── Ameaças e comportamento anômalo → GuardDuty
├── Ataques web HTTP → WAF
├── DDoS → Shield
├── Configuração incorreta de recursos → Config
├── Vulnerabilidades em EC2/containers → Inspector
└── Centralização de findings → Security Hub
```

---

## Caso 5 — Simulando Estratégia de Revisão no Dia do Exame

**Contexto:** Candidato tem 65 questões e 130 minutos. Como gerenciar o tempo e as questões marcadas.

**Simulação de Execução:**
```
QUESTÃO 1-20 (40 min — ~2 min/questão):
Q1: Sei a resposta → respondo imediatamente (45s)
Q5: Dúvida entre A e C → marco, escolho C (o que parece melhor), arquivo "revisar"
Q12: Muito longa, muitos detalhes → marco, pulo para próxima (economiza tempo)
Q15: Sei com certeza → 30s

QUESTÃO 21-65 (continuando...):
Q33: Palavra "multi-AZ" + "performance" → Red flag (armadilha Multi-AZ vs Read Replica)
     Leio com atenção → é sobre leitura → Read Replica → respondo
Q47: Questão sobre custo de NAT Gateway → sei que NAT GW é cobrado por hora
     Respondo com confiança
Q55: 3 serviços que nunca vi combinados → marco, escolho eliminando os que parecem errados

REVISÃO (questões marcadas — 30 min):
Q5 (dúvida A vs C): Releio a questão, identifico "síncrone" → Multi-AZ → mudei para B
Q12 (longa): Agora com calma, leio → questão de Kinesis vs SQS, identifico "replay"
             → Kinesis. Respondo.
Q55 (serviços desconhecidos): 2 opções restantes, escolho a com serviços gerenciados
                               → Princípio: "managed > self-managed" na AWS

LIÇÕES DA SIMULAÇÃO:
1. Questões longas: ler o ÚLTIMO parágrafo primeiro (geralmente tem a constraint chave)
2. Questões com listas: eliminar as claramente erradas
3. "Mínimo custo" → Spot/Reserved/Serverless/S3-IA/Glacier
4. "Mínimo operacional" → Managed services (RDS > EC2+MySQL)
5. "Máxima disponibilidade" → Multi-AZ + Multi-Region + Auto Scaling
6. Nunca gastar > 3 minutos em uma questão — marcar e seguir
```
