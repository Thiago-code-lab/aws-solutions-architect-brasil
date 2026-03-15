# Questões — Módulo 11: Serverless Lambda e API Gateway

> **Domínio SAA-C03**: Design de Arquiteturas de Alta Performance | Arquiteturas Seguras  
> **Dificuldade**: Média-Alta

---

**1.** Uma função Lambda é invocada por uma API Gateway e precisa acessar um banco de dados RDS em uma subnet privada. A função começa a ter timeouts. Qual é a causa mais provável?

- A) A Lambda não tem permissão IAM para acessar o RDS
- B) A Lambda em VPC não tem rota de saída para a internet, mas o endpoint do RDS requer resolução DNS externa
- C) A Lambda em VPC não pode se conectar ao RDS porque não está na mesma subnet
- D) A Lambda precisa de um Elastic IP para acessar recursos dentro da VPC

<details><summary>Resposta</summary>

**A** — Lambda em VPC precisa de permissão no Security Group do RDS para a porta do banco (ex: MySQL 3306). A causa mais provável de timeout é o SG do RDS não permitir tráfego do SG da Lambda. Note: Lambda em VPC pode estar em subnet diferente da RDS desde que o SG permita e exista route entre subnets da mesma VPC.

</details>

---

**2.** Uma empresa tem uma função Lambda que consome mensagens de uma fila SQS. A função tem timeout de 30 segundos. As mensagens começam a ser processadas mais de uma vez. Qual configuração da fila deve ser ajustada?

- A) Aumentar o Message Retention Period
- B) Configurar o Visibility Timeout para pelo menos 6x o timeout da Lambda (180 segundos)
- C) Habilitar Long Polling com wait time de 20 segundos
- D) Reduzir o Batch Size para 1

<details><summary>Resposta</summary>

**B** — O Visibility Timeout da fila SQS deve ser ≥ 6× o timeout da Lambda. Se a Lambda demora 30s para processar, o VT precisa ser ≥ 180s. Se o VT expirar antes da Lambda terminar, a mensagem reaparece na fila e outro worker vai processá-la, causando duplicação.

</details>

---

**3.** Uma API Gateway REST tem um backend Lambda que às vezes levanta exceções. A empresa quer retornar HTTP 500 para erros internos e HTTP 400 para erros de validação. Qual configuração permite isso?

- A) Lambda Proxy Integration com retorno customizado no handler
- B) Lambda Custom Integration com Integration Response mapeando regex de erro para status codes
- C) API Gateway Usage Plans com quota de errors
- D) CloudWatch Alarms para detectar erros e invocar Lambda de fallback

<details><summary>Resposta</summary>

**A** — Com Lambda Proxy Integration, a Lambda tem controle total sobre o status code de resposta. O handler retorna `{ statusCode: 400/500, body: ... }` e a API Gateway passa exatamente esse status code. Com Lambda Custom Integration (B), você pode usar regex de error message, mas é mais complexo.

</details>

---

**4.** Uma função Lambda precisa processar 1.000 mensagens por segundo de uma fila SQS Standard. Qual configuração maximiza o throughput de processamento?

- A) Aumentar o timeout da Lambda para 15 minutos
- B) Configurar múltiplos event source mappings para a mesma fila
- C) Aumentar o Batch Size (até 10.000) e aumentar a concorrência reservada da Lambda
- D) Migrar para SQS FIFO para maior throughput

<details><summary>Resposta</summary>

**C** — Maior Batch Size = menos invocações Lambda para o mesmo volume, reduzindo overhead. Aumentar concorrência permite processar múltiplos batches em paralelo. SQS FIFO tem throughput menor (300 TPS sem batching), não mais. Um Event Source Mapping por fila.

</details>

---

**5.** A empresa precisa que uma função Lambda mantenha uma conexão a um banco de dados RDS entre invocações para reduzir o overhead de reconexão. Como implementar isso corretamente?

- A) Usar `/tmp` para armazenar o objeto de conexão
- B) Inicializar a conexão no código de inicialização fora do handler; a conexão persiste no execution environment entre invocações quentes
- C) Usar um Layer para compartilhar a conexão entre funções Lambda
- D) Usar ElastiCache para armazenar a string de conexão

<details><summary>Resposta</summary>

**B** — O código fora do handler é executado apenas na inicialização do execution environment (cold start) e persiste entre invocações "quentes" dentro do mesmo environment. Inicializar `db_connection = get_db_connection()` fora do handler = reconexão apenas em cold starts. `/tmp` é armazenamento de arquivo, não objetos Python.

</details>

---

**6.** Uma empresa usa API Gateway + Lambda para sua API. O tráfego aumenta 10x em eventos promocionais e throttling começa a ocorrer. O que fazer para proteger o backend enquanto permite burst razoável?

- A) Habilitar API Gateway Cache com TTL de 5 minutos
- B) Configurar Usage Plans com throttling por API Key e aumentar o limit de concorrência da Lambda
- C) Migrar de REST API para HTTP API (menor custo)
- D) Adicionar SQS entre API Gateway e Lambda para bufferizar requests

<details><summary>Resposta</summary>

**D** — Para picos extremos: API Gateway → SQS → Lambda processa no ritmo que consegue. Isso elimina throttling pois a API retorna 200 imediatamente e a Lambda consome a fila. Para APIs síncronas (resposta imediata necessária), aumentar concurrência da Lambda e configurar throttling no API GW são as alternativas.

</details>

---

**7.** Uma função Lambda processa imagens do S3, mas frequentemente excede o tempo de processamento. A imagem processada deve ser salva de volta no S3. Qual configuração de Lambda é mais adequada?

- A) Aumentar memória para 10 GB e timeout para 15 minutos; usar /tmp para processamento intermediário
- B) Usar Lambda container image (até 10 GB) com biblioteca de processamento
- C) Dividir em duas Lambdas: uma para download, outra para processamento, coordenadas por Step Functions
- D) Usar EC2 Spot para processamento de imagens pesadas

<details><summary>Resposta</summary>

**A** — Lambda suporta até 10 GB de memória e 15 minutos de timeout com até 10 GB de `/tmp`. Para processamento de imagens grandes, aumentar memória (que também aumenta CPU) e usar `/tmp` para trabalho temporário. Container image (B) resolve o problema de package size, não de timeout.

</details>

---

**8.** Um arquiteto precisa criar um endpoint HTTPS que acione uma Lambda toda vez que um arquivo CSV é depositado no S3. Qual é o caminho mais simples?

- A) API Gateway + Lambda + S3 Event Notification
- B) S3 Event Notification diretamente triga a Lambda (sem API Gateway)
- C) EventBridge Rule para evento S3 → Lambda
- D) SQS + S3 Event Notification → Lambda ESM

<details><summary>Resposta</summary>

**B** — S3 pode invcar Lambda diretamente via S3 Event Notification configurado no bucket (sem custo adicional, sem API GW necessário). É o caminho mais simples. EventBridge (C) funciona mas é mais complexo para este caso simples.

</details>

---

**9.** Uma API Gateway REST tem 50 endpoints e processa 100.000 requests por dia. O time quer reduzir o custo. O que analisar?

- A) Migrar todos os endpoints para HTTP API (redução ~71% no custo por request vs REST API)
- B) Habilitar caching em endpoints GET com dados pouco mutáveis
- C) Mover a API para dentro de uma VPC (VPC-only endpoint)
- D) A e B são estratégias complementares válidas

<details><summary>Resposta</summary>

**D** — HTTP API custa menos por request (e suporta a maioria dos casos); caching reduz backend calls (e custo total). A migração deve avaliar features de REST API necessárias (WAF nativo, request validation, usage plans — não disponíveis no HTTP API).

</details>

---

**10.** Uma Lambda Java tem cold starts de 8 segundos causando má experiência para usuários da API. Qual é a solução mais eficiente para eliminar cold starts sem mudar o runtime?

- A) Aumentar a memória da Lambda de 512 MB para 10 GB
- B) Lambda SnapStart — disponível para Java 11+ (snapshots do runtime inicializado)
- C) Adicionar a Lambda em um placement group
- D) Usar Scheduled EventBridge para "pré-aquecer" a Lambda chamando-a a cada minuto

<details><summary>Resposta</summary>

**B** — SnapStart para Java 11+: AWS tira snapshot do execution environment após a fase de init e o restaura em lugar de inicializar do zero, reduzindo cold start para sub-segundo. "Pré-aquecer" (D) funciona mas é uma gambiarra; Provisioned Concurrency é a alternativa oficial se SnapStart não estiver disponível.

</details>

---

**11.** Uma empresa usa Lambda para processar transações financeiras assíncronas. Eles precisam garantir que nenhuma transação seja perdida mesmo se a Lambda falhar. O que configurar?

- A) Aumentar o timeout da Lambda para 15 minutos
- B) Configurar Lambda Destinations com On Failure destination para SQS DLQ
- C) Habilitar Enhanced Monitoring no Lambda
- D) Lambda está configurada com SQS como ESM — o próprio SQS retém mensagens até o maxReceiveCount

<details><summary>Resposta</summary>

**D** — Com SQS como ESM, mensagens ficam na fila até serem processadas com sucesso ou atingirem o maxReceiveCount. Após maxReceiveCount tentativas, vão para a DLQ. Isso garante que nenhuma mensagem seja perdida. Lambda Destinations (B) é para invocações assíncronas diretas (não ESM).

</details>

---

**12.** Uma API Gateway expõe dados sensíveis e a empresa exige que todos os requests sejam autenticados com JWT tokens do seu provedor de identidade existente (não Cognito). Qual configuração usar?

- A) Cognito User Pool Authorizer (precisaria migrar o Identity Provider)
- B) Lambda Authorizer com bearer token — Lambda valida o JWT e retorna política IAM
- C) IAM Authorization com SigV4
- D) API Key com Usage Plan

<details><summary>Resposta</summary>

**B** — Lambda Authorizer com TOKEN type: recebe o JWT no header Authorization, a Lambda valida o token contra o Identity Provider existente (sem migração) e retorna uma IAM policy (Allow/Deny). Cache de até 3600s evita verificação em todo request.

</details>

---

**13.** Uma Lambda precisa acessar um segredo do Secrets Manager. Qual é a forma mais segura de conceder essa permissão?

- A) Criar um usuário IAM com acesso ao Secrets Manager e embutir as credenciais no código da Lambda
- B) Adicionar a permissão `secretsmanager:GetSecretValue` à execution role da Lambda
- C) Armazenar o segredo em uma variável de ambiente da Lambda (criptografada com KMS)
- D) Usar a VPC to access Secrets Manager sem IAM

<details><summary>Resposta</summary>

**B** — IAM Role (Execution Role) da Lambda: a Lambda assume automaticamente a role durante execução. Adicionar `secretsmanager:GetSecretValue` na role é o padrão seguro. Nunca usar usuário IAM com credenciais hardcoded (A). Variável de ambiente (C) não é tão dinâmica quanto buscar direto do Secrets Manager.

</details>

---

**14.** Uma empresa tem um job batch que deve rodar a cada hora e processar dados do S3. O job demora 5-10 minutos. Qual solução é mais custo-efetiva?

- A) EC2 t3.medium rodando 24/7 com cron
- B) EventBridge Rule (rate: 1 hour) → Lambda com timeout de 15 minutos
- C) ECS Fargate Task acionada por EventBridge
- D) B ou C dependendo da necessidade de CPU/memória acima dos limites da Lambda

<details><summary>Resposta</summary>

**D** — Se o job cabe em Lambda (≤ 15 min, ≤ 10 GB memória): EventBridge → Lambda é mais simples e barato. Se precisar de mais recursos ou > 15 min: EventBridge → ECS Fargate Task (paga apenas pela duração da task, sem custo idle). Ambos são serverless e cobram apenas pelo tempo de execução.

</details>

---

**15.** Uma arquitetura tem: API Gateway → Lambda A → Lambda B (chamada síncrona). Lambda B ocasionalmente falha. Como tornar a arquitetura mais resiliente?

- A) Configurar retry automático no código da Lambda A
- B) Substituir a chamada síncrona Lambda A → B por Lambda A → SQS → Lambda B (async with retry)
- C) Adicionar um CloudWatch Alarm para detectar erros na Lambda B
- D) Configurar Provisioned Concurrency na Lambda B

<details><summary>Resposta</summary>

**B** — Chamadas síncronas Lambda-to-Lambda criam acoplamento: se B falha, A falha. Introduzir SQS desacopla as duas: A publica na fila e retorna; SQS + Lambda B é assíncrono com retry automático (até maxReceiveCount tentativas) e DLQ para falhas permanentes. Isso melhora resilência e escalabilidade.

</details>

---

## Questões de Múltipla Seleção

**Bônus 1.** Quais das seguintes afirmações sobre Lambda são VERDADEIRAS? (Selecione 2)

- A) Lambda suporta até 15 minutos de timeout e 10 GB de memória
- B) Lambda pode ser configurada com Provisioned Concurrency para eliminar cold starts
- C) Lambda sempre tem cold start nas primeiras 10 invocações
- D) Lambda timeout máximo é 5 minutos para funções em VPC

<details><summary>Resposta</summary>

**A e B** — Timeout máximo = 15 min e memória máxima = 10 GB. Provisioned Concurrency elimina cold starts mantendo environments pré-aquecidos. Não existe limite de 10 invocações para cold start (C); Lambda em VPC tem o mesmo timeout de 15 min que fora de VPC (D).

</details>
