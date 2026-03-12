# Flashcards — Módulo 11: Serverless, Lambda e API Gateway

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** Quais são os 3 tipos de invocação do Lambda?  
**R:** **Síncrona** (espera resposta — API GW, ALB, SDK); **Assíncrona** (não espera — S3 Events, SNS, EventBridge); **Event Source Mapping** (Lambda faz polling — SQS, Kinesis, DynamoDB Streams)

---

**P:** O que é Lambda concurrency e qual é o limite default por região?  
**R:** Número de instâncias Lambda executando simultaneamente. Default: **1.000 concurrent executions por região** (pode aumentar via suporte). Cada invocação síncrona consome 1 unidade de concurrency enquanto executa.

---

**P:** O que é Provisioned Concurrency no Lambda?  
**R:** Pré-aquece instâncias Lambda, eliminando cold starts. Instâncias ficam "quentes" e prontas para responder imediatamente. Tem custo adicional (pago por hora de concurrency provisionada). Útil para aplicações latency-sensitive.

---

**P:** O que acontece com mensagens SQS quando o Lambda retorna erro em Event Source Mapping?  
**R:** Lambda faz retry automático. Após esgotar as tentativas (configurable maxReceiveCount na DLQ do SQS), a mensagem vai para a **Dead Letter Queue (DLQ)**. O Lambda não lida diretamente com DLQ de SQS — a DLQ é configurada no próprio SQS.

---

**P:** O que é um Lambda Layer?  
**R:** Arquivo ZIP contendo dependências (bibliotecas, runtime personalizado) compartilhado entre múltiplas funções. Até 5 layers por função. Tamanho máximo total (código + layers): 250 MB descomprimido. Evita repetição de dependências em cada deployment package.

---

**P:** Qual é a diferença entre Lambda Proxy Integration e Custom Integration no API Gateway?  
**R:** **Proxy Integration:** API GW passa o request HTTP inteiro para Lambda (headers, body, query params) como objeto `event`. Lambda retorna o response completo (statusCode, headers, body). **Custom Integration:** API GW transforma o request/response via mapping templates (VTL). Proxy é mais simples; custom oferece mais controle.

---

**P:** O que é Lambda SnapStart e para qual runtime é aplicável?  
**R:** Feature para reduzir cold start em Java 11+. Ao publicar uma nova versão, a AWS inicializa a função e tira um snapshot do estado. Novas invocações restauram o snapshot em vez de inicializar do zero — reduz cold start de segundos para milissegundos.

---

**P:** Como o Lambda acessa recursos em uma VPC privada?  
**R:** Configurando o Lambda com **VPC Configuration** (VPC ID, subnets, security group). A AWS cria uma ENI (Elastic Network Interface) na subnet especificada. Lambda ganha IP privado na VPC e pode acessar RDS, ElastiCache etc. Requer: NAT Gateway para o Lambda ter acesso à internet.

---

**P:** Quais são os 3 tipos de API no API Gateway?  
**R:** **REST API** (funcionalidade completa, staging, caching, throttling por stage); **HTTP API** (mais simples, mais barato, menor latência, suporta JWT/OAuth nativo); **WebSocket API** (conexões bidirecionais persistentes, para chat, notificações em tempo real).

---

**P:** O que é throttling no API Gateway e como funciona?  
**R:** Limitar requests para proteger o backend. Limites: **10.000 requests/segundo** por conta (default) + **5.000 burst** concorrente. Por stage/método pode ser configurado separadamente. Quando excede: HTTP 429 (Too Many Requests). Pode aumentar conta via suporte.

---

**P:** Que tipo de autorizador Lambda verifica no API Gateway?  
**R:** **Token-based:** recebe token (JWT, OAuth) no header Authorization e retorna IAM policy (Allow/Deny). **Request-based:** recebe headers, query params e pode usar qualquer lógica de autenticação. Resultado é cacheado por TTL (até 3600s) para performance.

---

**P:** O que é um Lambda Alias e como difere de uma Version?  
**R:** **Version:** snapshot imutável de código + configuração com ARN único (ex: `function:1`). **Alias:** ponteiro nomeado (ex: `PROD`) que aponta para uma ou duas versions. Aliases suportam **canary deployment** (aliasName:weight para dividir tráfego — ex: 90% v1, 10% v2).

---

**P:** Qual é o tempo máximo de execução do Lambda?  
**R:** **15 minutos** (900 segundos). Para processos mais longos: usar Step Functions (orquestração de workflows), ECS/Fargate ou EC2.

---

**P:** Como o API Gateway suporta CORS?  
**R:** Configurando o CORS no API Gateway para responder às pre-flight OPTIONS requests com headers `Access-Control-Allow-Origin`, `Access-Control-Allow-Headers`, `Access-Control-Allow-Methods`. Para Proxy Integration, a Lambda também deve retornar os headers CORS nas respostas.

---

**P:** O que é o AWS SAM (Serverless Application Model)?  
**R:** Framework que simplifica deploy de aplicações serverless. Sintaxe simplificada de CloudFormation: `AWS::Serverless::Function`, `AWS::Serverless::Api`. Comandos: `sam build`, `sam deploy`, `sam local invoke` (teste local com Docker). Gera CloudFormation stack por baixo.

---

**P:** O que é a Lambda Execution Role?  
**R:** IAM Role assumida pela função Lambda durante a execução. Define quais APIs AWS a função pode chamar (ex: `dynamodb:PutItem`, `s3:GetObject`). A função assume a Role automaticamente — não precisa de credentials explícitas no código.

---

**P:** O que acontece durante um Lambda cold start?  
**R:** Primeira invocação (ou após período sem uso): AWS precisa alocar infraestrutura, baixar código, inicializar runtime, executar código fora do handler. Adiciona latência (milissegundos a segundos, maior para Java). Soluções: Provisioned Concurrency, SnapStart (Java), manter código de inicialização fora do handler.

---

**P:** Quais são os integradores nativos do API Gateway além do Lambda?  
**R:** HTTP Integration (redireciona para endpoint HTTP externo), AWS Service Integration (chama diretamente serviços AWS — ex: SQS, DynamoDB sem Lambda intermediário), Mock Integration (retorna resposta estática), VPC Link (integra com recursos em VPC privada via NLB).

---

**P:** O que é o API Gateway Deployment Stage e para que serve?  
**R:** Snapshot deployado da API com URL única (ex: `https://api-id.execute-api.us-east-1.amazonaws.com/prod`). Permite múltiplos ambientes: dev, staging, prod. Cada stage tem configurações independentes: throttling, cache, variáveis de stage (passadas para Lambda/integração), logging.

---

**P:** Como o Lambda processa múltiplas mensagens do SQS simultaneamente?  
**R:** Event Source Mapping processa mensagens em **batches** (até 10.000). Lambda recebe um batch de mensagens (batch size configurável). Um erro em qualquer mensagem do batch pode fazer todas serem retornadas para a fila. Solução: **Partial Batch Response** (reportar individualmente quais mensagens falharam).

---

**P:** O que é o Lambda Destinations e quando usar em vez de DLQ?  
**R:** Para invocações **assíncronas**: configura destinos de sucesso e falha (SQS, SNS, EventBridge, outra Lambda). Mais expressivo que DLQ (inclui contexto de invocação + input original + output/erro). DLQ apenas captura mensagens falhas sem resultado. Lambda Destinations é o padrão moderno.

---

**P:** Qual é o limite de tamanho do payload de invocação do Lambda?  
**R:** **Síncrono:** 6 MB (request payload) + 6 MB (response payload). **Assíncrono:** 256 KB (event payload). Para payloads maiores: salvar dados no S3 e passar o S3 key como evento.

---

**P:** O que é o API Gateway Usage Plan?  
**R:** Define limite de uso por API key: throttling (requests/segundo, burst) e quota (requests/dia ou semana ou mês). Permite monetizar APIs (diferentes planos para diferentes tier de clientes), controlar uso por cliente e prevenir abuse.

---

**P:** O que é Lambda@Edge?  
**R:** Executa código Lambda em CloudFront Points of Presence (edge locations) — próximo do usuário. Use cases: URL rewrite, A/B testing, autenticação no edge, customizar headers. Limitações vs Lambda regular: menor timeout (5s viewer, 30s origin), menor memória máx, sem VPC access, regiões limitadas.

---

**P:** Como o Lambda Function URL difere do API Gateway?  
**R:** Function URL: HTTPS endpoint dedicado diretamente para a função (sem API Gateway). Mais simples, menor latência, sem custo de API Gateway. Limitações: sem throttling avançado, sem cache, sem uso de múltiplos estágios, sem integração nativa com WAF (exceto via CloudFront). Use para webhooks simples ou funções com tráfego ocasional.
