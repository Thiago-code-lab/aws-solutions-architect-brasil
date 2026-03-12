# Casos de Uso — Módulo 08: DNS, Route 53 e CloudFront

## Caso 1: Failover Automático com Route 53 Health Checks

**Cenário:** Aplicação crítica em us-east-1 (primária) com failover automático para eu-west-1 em caso de falha regional.

**Arquitetura:**
```
Route 53 (api.empresa.com)
  ├── Record PRIMARY: ALB us-east-1 [FAILOVER + Health Check ativo]
  │     └── Health Check: HTTP /health, a cada 30s, threshold 3 falhas
  └── Record SECONDARY: ALB eu-west-1 [FAILOVER, sem HC obrigatório]

Fluxo de Failover:
1. Health Checker detecta falha (3 checks consecutivos failing)
2. Route 53 marca record PRIMARY como UNHEALTHY
3. DNS passa a responder com o SECONDARY
4. TTL baixo (60s) = propagação rápida
```

**Configuração de Health Check para recurso privado:**
- CloudWatch Metric Alarm (baseado em ALB HealthyHostCount = 0)
- Route 53 HC tipo "CloudWatch Alarm" referenciando o alarme
- Sem necessidade de expor endpoint público para o health check

---

## Caso 2: CloudFront com Lambda@Edge para Autenticação JWT

**Cenário:** API RESTful servida via CloudFront + API Gateway. Cada request deve ter JWT válido verificado na edge, sem sobrecarregar o API Gateway.

**Arquitetura:**
```
Client → CloudFront → Lambda@Edge (Viewer Request) → API Gateway → Lambda
                            │
                  [Valida JWT assinado com RS256]
                  [Se inválido → 401 Unauthorized]
                  [Se válido → adiciona user-id header → origin]

Lambda@Edge (us-east-1, replicado globalmente):
- Node.js: verifica assinatura JWT com public key armazenada em CloudFront KVS
- < 5ms overhead
- Sem chamada ao backend para validar token → escala indefinidamente
```

**Cache Policy:**
- `Authorization` header na cache key → cada token único = cache miss (não armazenar dados do usuário)
- Para conteúdo público: sem Authorization na cache key → máximo cache hit

---

## Caso 3: Performance Global com Route 53 + CloudFront

**Cenário:** Aplicação SaaS global serve conteúdo estático, API dinâmica e assets de vídeo para usuários em 6 continentes.

**Arquitetura em Camadas:**
```
DNS: Route 53
  └── app.empresa.com → Alias → CloudFront Distribution

CloudFront Distribution:
  ├── Behavior: /static/* → S3 Bucket (TTL: 1 ano)
  │     └── Cache-Control: max-age=31536000, immutable
  ├── Behavior: /api/* → ALB us-east-1 (TTL: 0, no-cache)
  │     └── Origin Request Policy: forward all headers + cookies
  └── Behavior: /videos/* → S3 Bucket (Signed URLs, TTL: 1h)

Route 53 Latency Routing para o ALB:
  ├── ALB us-east-1 (latência América)
  ├── ALB eu-west-1 (latência Europa)
  └── ALB ap-southeast-1 (latência Ásia)
```

**Resultado:** Conteúdo estático servido da edge CloudFront mais próxima (< 20ms). API dinâmica vai para API ALB regional com menor latência. Vídeos: acesso controlado por tempo com Signed URLs.

---

## Caso 4: Migração DNS com Weighted Routing

**Cenário:** Empresa migra aplicação de on-premises para AWS sem downtime Zero. Precisa validar a nova infraestrutura com tráfego real gradualmente.

**Estratégia de Migração (Canary Deployment):**
```
app.empresa.com (hosted zone pública)
  ├── A record: 203.0.113.10 (on-prem) — Peso: 100
  └── A record: ALB AWS        — Peso: 0 (começa em 0 = sem tráfego)

Semana 1: on-prem 95 / AWS 5  → validar logs AWS
Semana 2: on-prem 80 / AWS 20 → monitorar performance
Semana 3: on-prem 50 / AWS 50 → conferir DB migration
Semana 4: on-prem 0  / AWS 100 → migração completa
```

**Monitoramento durante migração:**
- CloudWatch Dashboards para ambos os ambientes em paralelo
- Route 53 Health Check em ambos os endpoints
- TTL baixo (60s) para agilidade em ajustes de peso

---

## Caso 5: Proteção contra DDoS com WAF + Shield Advanced + CloudFront

**Cenário:** Plataforma de e-commerce sofre ataques volumétricos e tentativas de credential stuffing (brute force de login).

**Camadas de Proteção:**
```
Internet
  └── CloudFront + Shield Advanced (L3/L4 DDoS automático)
        └── AWS WAF (L7):
              ├── AWSManagedRulesCommonRuleSet → OWASP Top 10
              ├── AWSManagedRulesBotControlRuleSet → Bots
              ├── Rate-based rule: > 1000 req/5min por IP → BLOCK
              │   (previne credencial stuffing em /login)
              └── IP Set rule: blacklist de IPs conhecidos maliciosos

              Após WAF → ALB → App Servers
```

**Resposta a ataques:**
- DRT (DDoS Response Team) da Shield Advanced acessa sua conta para mitigação manual
- WAF Logs para análise + EventBridge rule para alertas automáticos via SNS
- Créditos AWS para custos extras de scaling causados pelo DDoS (Shield Advanced benefit)
