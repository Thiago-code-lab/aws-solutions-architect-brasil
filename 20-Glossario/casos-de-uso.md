# Casos de Uso Reais — Glossário na Prática (Módulo 20)

## Caso 1 — Quando "RTO" e "RPO" Determinam a Arquitetura

**Contexto:** Questão de exame define RTO e RPO como restrições. Candidato precisa saber qual estratégia de DR escolher baseado nesses valores.

**Cenários de Aplicação:**
```
CENÁRIO A: RPO = 0, RTO < 30s
(nenhuma perda de dados, recuperação quase instantânea)
→ Arquitetura: Active-Active Multi-Região
→ Aurora Global Database (sync lag < 1s ≈ RPO≈0)  
→ Route 53 Latency Routing (failover automático ≈ 60s) 
→ Custo: $$$$

CENÁRIO B: RPO ≤ 15min, RTO ≤ 1h
→ Arquitetura: Warm Standby
→ Aurora Global Database (read replica standby pronta)
→ Auto Scaling: min = 1, max = 20 (scale when needed)
→ Custo: $$$

CENÁRIO C: RPO ≤ 1h, RTO ≤ 4h  
→ Arquitetura: Pilot Light
→ RDS backup automático (snapshots) + restore rápido
→ EC2 AMIs prontas, inicia quando necessário
→ Custo: $$

CENÁRIO D: RPO ≤ 24h, RTO pode ser dias
→ Arquitetura: Backup & Restore
→ S3 + Glacier para backups
→ Scripts de restore manuais
→ Custo: $

MNEMÔNICO:
RPO = "quanto de dados posso PERDER?" (olha para o passado)
RTO = "quanto tempo posso ficar PARADO?" (olha para o futuro)
```

---

## Caso 2 — Distinguindo "Durabilidade" de "Disponibilidade"

**Contexto:** Questão confunde os dois termos intencionalmente.

**Diferença na Prática:**
```
CENÁRIO: S3 Standard
Durabilidade: 99.999999999% (11 noves)
  → Probabilidade de PERDER um objeto: 0.000000001% por ano
  → Em 10 milhões de objetos, espera-se perder < 1 objeto em 10.000 anos
  → "Meus dados sempre existirão?"

Disponibilidade: 99.99% 
  → Downtime permitido: ~52 min/ano
  → "Consigo acessar meus dados agora?"

ANALOGIA:
Cofre do banco: DURABILIDADE alta (dinheiro não some)
                DISPONIBILIDADE: pode ficar fechado no feriado

QUESTÃO ARMADILHA:
"Qual classe S3 oferece menor durabilidade?"
→ S3 One Zone-IA: 99.999999999% (mesma durabilidade!)
  Diferença: apenas 1 AZ → DISPONIBILIDADE menor se AZ falha
  Durabilidade não muda (dados replicados em 1 AZ)

GABARITO: "One Zone-IA tem menor DISPONIBILIDADE (não durabilidade)"
```

---

## Caso 3 — Aplicando "Princípio do Menor Privilégio" em IAM

**Contexto:** Questão pede política IAM mínima para um caso específico.

**Cenário:** Lambda precisa ler de 1 tabela DynamoDB específica e gravar logs no CloudWatch.

**Análise de Políticas:**
```
ERRADO — Permissão excessiva:
{
  "Effect": "Allow",
  "Action": "dynamodb:*",
  "Resource": "*"
}
→ DynamoDB:* = full access a QUALQUER tabela
→ Violação do Princípio do Menor Privilégio

ERRADO — Mais específico mas ainda amplo:
{
  "Effect": "Allow",
  "Action": ["dynamodb:GetItem", "dynamodb:Scan", "dynamodb:Query"],
  "Resource": "*"  // ← ainda "*"
}
→ Permite leitura de TODAS as tabelas

CORRETO — Mínimo necessário:
{
  "Effect": "Allow",
  "Action": ["dynamodb:GetItem", "dynamodb:Query"],
  "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/MinhaTabela"
}

Para CloudWatch Logs (criado automaticamente pelo Lambda se role tiver):
{
  "Effect": "Allow",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Resource": "arn:aws:logs:us-east-1:123456789:log-group:/aws/lambda/MinhaFuncao:*"
}

NOTA: AWSLambdaBasicExecutionRole já inclui logs básicos — pode usar managed policy
      Para DynamoDB: sempre especificar ARN da tabela específica
```

---

## Caso 4 — CIDR e Subnets VPC na Prática

**Contexto:** Questões de VPC frequentemente pedem calcular IPs disponíveis ou identificar conflitos de CIDR.

**Exercícios:**
```
EXERCÍCIO 1: Quantos IPs utilizáveis em uma /24?
IPv4 /24 = 256 endereços totais
AWS reserva 5 (rede, GW, DNS+2, broadcast)
256 - 5 = 251 IPs utilizáveis

EXERCÍCIO 2: Subnets para HA em 2 AZs com VPC 10.0.0.0/16:
VPC: 10.0.0.0/16 (65.536 endereços)
Plano de subnets:
  Public AZ-a:  10.0.1.0/24 (251 usáveis)
  Public AZ-b:  10.0.2.0/24 (251 usáveis)
  Private AZ-a: 10.0.10.0/24 (251 usáveis)
  Private AZ-b: 10.0.11.0/24 (251 usáveis)
  DB AZ-a:      10.0.20.0/24 (251 usáveis)
  DB AZ-b:      10.0.21.0/24 (251 usáveis)

EXERCÍCIO 3: VPC Peering — conflito de CIDR:
VPC A: 10.0.0.0/16
VPC B: 10.0.0.0/24  ← CONFLITO! /24 está dentro do /16
→ VPC Peering NÃO funciona com CIDRs sobrepostos
Solução: usar CIDRs distintos (ex: VPC B: 172.16.0.0/16)

ARMADILHA DE EXAME:
/28 = menor subnet permitida na AWS
/28 = 16 endereços totais, 16 - 5 = 11 utilizáveis
```

---

## Caso 5 — Shared Responsibility Model em Cenários Reais

**Contexto:** Questão identifica quem é responsável por problemas de segurança.

**Cenários e Responsabilidades:**
```
CENÁRIO 1: EC2 com Windows Server foi infectado por ransomware
  Responsabilidade do CLIENTE:
  ✓ Patches do Windows Server (cliente)
  ✓ Antivírus instalado no SO (cliente)
  ✓ Security Group configurado corretamente (cliente)
  Responsabilidade da AWS:
  ✓ Hypervisor isolado (AWS)
  ✓ Hardware físico (AWS)
  → CLIENTE é responsável pela infecção do SO

CENÁRIO 2: Falha física de disco no servidor que hospeda EC2
  → AWS é responsável (hardware físico)
  → AWS substitui hardware, EC2 migra automaticamente (Live Migration)
  → Dados em EBS são preservados (replicados dentro da AZ)

CENÁRIO 3: Bucket S3 com dados públicos acidentalmente
  → CLIENTE configurou permissão incorreta
  → AWS ofereceu ferramentas (Block Public Access, Macie, Config)
  → Cliente optou por não usar → responsabilidade do CLIENTE

CENÁRIO 4: RDS MySQL com SO não atualizado
  → AWS gerencia SO do RDS → AWS é responsável por patches do SO
  → Cliente gerencia: dados, usuários do banco, configurações

CENÁRIO 5: Lambda com código vulnerável a SQL Injection
  → CLIENTE escreveu o código → CLIENTE é responsável
  → AWS fornece a plataforma de execução (segura)
  → Segurança do CÓDIGO = responsabilidade do desenvolvedor

REGRA MNEMÔNICA:
AWS = "of the cloud" (hardware, rede física, hypervisor, SO managed services)
Cliente = "in the cloud" (SO EC2, código, dados, config IAM, rede virtual)
```
