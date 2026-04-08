# Flashcards — Módulo 08: DNS, Route 53 e CloudFront

---

**P:** Por que CNAME não pode ser usado no domínio raiz (apex)?
**R:** RFC 1034 proíbe CNAME no apex da zona (não pode coexistir com outros records como SOA/NS). AWS criou **Alias record** como extensão proprietária que funciona no apex e aponta para recursos AWS sem custos adicionais de query

---

**P:** Qual o TTL padrão de registros Route 53 e o que acontece durante failover?
**R:** TTL configurável (ex: 300s = 5 min). Durante failover, o TTL determina quanto tempo os resolvers DNS ao redor do mundo cachearão o antigo endereço. TTL baixo = failover mais rápido, mais queries (custo). TTL alto = failover mais lento, menos queries

---

**P:** Quais são os 8 tipos de roteamento Route 53?
**R:** Simple, Weighted, Latency, Failover, Geolocation, Geoproximity (Traffic Flow), Multi-Value Answer, IP-based routing

---

**P:** Qual política de roteamento usa traffic flow com bias configurável?
**R:** **Geoproximity routing** (Traffic Flow). Roteia baseado em distância geográfica. O "bias" aumenta ou diminui a região de influência de um recurso (bias positivo → atrai mais tráfego; negativo → repele). Requer Route 53 Traffic Flow

---

**P:** Qual a diferença entre Geolocation e Latency routing?
**R:** Geolocation: roteia baseado em país/continente do cliente (ex: usuários do Brasil → servidor BR). Latency: roteia baseado em latência MEDIDA entre cliente e AWS regions (ex: usuário no Brasil → servidor no us-east-1 se for mais rápido que sa-east-1)

---

**P:** O que é um Calculated Health Check no Route 53?
**R:** Combina resultado de múltiplos Health Checks individuais usando AND, OR ou NOT. Permite criar lógica: "considerar saudável se pelo menos 2 de 3 endpoints estão UP". Útil para ser mais resiliente a falsos positivos

---

**P:** Como monitorar recursos privados (sem IP público) com Route 53 Health Checks?
**R:** Route 53 Health Checkers operam da internet. Para recursos privados: crie CloudWatch Metric Alarm baseado em métricas do recurso → Health Check do tipo "CloudWatch Alarm". Quando Alarm → ALARM: Health Check falha

---

**P:** Qual é a diferença entre OAI e OAC no CloudFront?
**R:** OAI (Origin Access Identity): legado, funciona com S3 padrão. OAC (Origin Access Control): novo padrão, suporta todas as regiões S3, SSE-KMS, S3 Object Lambda, mais métodos HTTP. Para novos projetos: sempre use OAC

---

**P:** O que é Cache Policy vs Origin Request Policy no CloudFront?
**R:** Cache Policy: define o que entra na cache key (headers, cookies, query strings) e TTL. Origin Request Policy: define o que é enviado à origin (headers, cookies, query strings) ALÉM do que está na cache key. Origin pode receber mais informações do que a cache considera

---

**P:** Quanto custa uma CloudFront Cache Invalidation?
**R:** Primeiros **1.000 paths** de invalidação por mês: gratuito. Acima: $0,005 por path. Wildcard `/*` conta como 1 path. Melhor prática: use versionamento de assets (hashing no nome do arquivo) para evitar invalidações

---

**P:** O que é CloudFront Functions vs Lambda@Edge? Qual é mais rápido?
**R:** CloudFront Functions: sub-milissegundo, JavaScript, apenas Viewer Request/Response, mais barato. Lambda@Edge: até 30s (Origin), Node.js/Python, 4 eventos incluindo Origin. CF Functions é mais rápido e barato; Lambda@Edge é mais poderoso

---

**P:** Para que serve o AWS Global Accelerator?
**R:** Melhora performance e disponibilidade para aplicações globais usando 2 IPs Anycast estáticos. Tráfego entra pelo edge AWS mais próximo e trafega pela rede backbone privada AWS. Ideal para: IPs fixos (firewall), tráfego não-HTTP (UDP/TCP genérico), failover em segundos (sem TTL DNS)

---

**P:** Qual é a diferença entre Route 53 e Global Accelerator para failover?
**R:** Route 53: failover baseado em DNS com TTL → pode demorar minutos para propagar. Global Accelerator: failover em segundos (sem TTL, roteamento por Anycast), mas cobre apenas endpoints AWS. Para failover ultra-rápido e IPs estáticos: GA. Para simplicidade e flexibilidade: Route 53

---

**P:** O que é Route 53 Resolver Inbound vs Outbound Endpoint?
**R:** Inbound: permite que on-premises resolva nomes DNS de recursos AWS (queries chegam de fora para a VPC). Outbound: permite que recursos na VPC resolvam nomes DNS on-premises (forwarding rules para DNS servers on-prem via DX/VPN)

---

**P:** Quais recursos AWS são alvos válidos de Alias record no Route 53?
**R:** ALB/NLB/CLB, CloudFront, API Gateway, Elastic Beanstalk, VPC Interface Endpoints, Global Accelerator, S3 website endpoint, outras zonas no Route 53. Alias resolve no servidor DNS — nenhum custo extra de query

---

**P:** O que é WAF Rate-based rule?
**R:** Limita o número de requests de um mesmo IP dentro de uma janela de tempo (5 minutos). Ex: bloquear IP que faz mais de 1.000 requests em 5 minutos. Protege contra brute force e DDoS de camada 7 (volumétrico)

---

**P:** O que é AWS Shield Standard vs Advanced?
**R:** Standard: gratuito, automático para todos, protege contra DDoS de camada 3/4 (SYN floods, UDP reflection). Advanced: pago (~$3.000/mês), proteção para camada 7 também, DRT (DDoS Response Team) 24/7, créditos de custo por DDoS, relatórios detalhados

---

**P:** Como configurar HTTPS obrigatório no CloudFront?
**R:** No Behavior: "Viewer Protocol Policy" = "Redirect HTTP to HTTPS" ou "HTTPS Only". Use certificado ACM (região us-east-1 para CloudFront). Para origem S3: OAC não exige HTTPS mas recomendado. Para origens custom: configurar Origin Protocol Policy

---

**P:** O que é Multi-Value Answer routing no Route 53?
**R:** Retorna múltiplos IPs (até 8) para o mesmo nome DNS, cada um com Health Check. Diferente de Simple (retorna todos sem health check). Clientes geralmente usam o primeiro IP funcional. NÃO é load balancer (não substitui ALB) — apenas retorna múltiplos registros saudáveis

---

**P:** Como o CloudFront Geo Restriction funciona e qual é a alternativa mais granular?
**R:** Geo Restriction: bloqueia/permite países via MaxMind GeoIP (retorna 403). Toda a distribution ou por behavior. Alternativa mais granular: integrar Lambda@Edge que checa o header `CloudFront-Viewer-Country` e aplica lógica customizada (ex: redirect para versão localizada, não apenas block)
