# Cheatsheet — Módulo 08: DNS, Route 53 e CloudFront

## Route 53 — Tipos de Records

| Record | Uso | Apex? | Custo de Query |
|---|---|---|---|
| A | IPv4 address | Sim | Padrão |
| AAAA | IPv6 address | Sim | Padrão |
| CNAME | Alias para outro hostname | **NÃO** | Padrão |
| **Alias** | Recursos AWS (ALB, CF, S3) | **SIM** | Gratuito |
| MX | Email servers | Sim | Padrão |
| TXT | Verificação, SPF, DKIM | Sim | Padrão |
| NS | Nameservers da zona | Sim | Padrão |

## Políticas de Roteamento Route 53

| Política | Caso de Uso | Health Check? |
|---|---|---|
| Simple | Single resource, sem HA | Não |
| Weighted | A/B testing, migração gradual | Opcional |
| Latency | Menor latência para usuário | Opcional |
| Failover | Active-Passive DR | **Obrigatório** para primário |
| Geolocation | Conteúdo por país/região | Opcional |
| Geoproximity | Por distância geográfica com bias | Opcional |
| Multi-Value | Múltiplos IPs saudáveis (pseudo LB) | Recomendado |
| IP-based | Roteamento por CIDR de origem | Não |

## CloudFront — Componentes e Configuração

| Componente | Descrição |
|---|---|
| Distribution | Ponto de entrada — define origems, behaviors, certificado |
| Origin | Origem do conteúdo: S3, ALB, EC2, API GW, HTTP custom |
| Behavior | Regra de cache associada a um path pattern (ex: `/api/*`) |
| Cache Policy | Define cache key (query strings, headers, cookies) e TTLs |
| Origin Request Policy | O que enviar à origin além da cache key |
| OAC | Substitui OAI — autentica CF com origin S3 via SigV4 |

## CloudFront TTLs
- `Cache-Control: max-age=<segundos>` no response do origin → define TTL no edge
- Default TTL: **86.400s** (24h) se origin não definir
- Minimum TTL e Maximum TTL no behavior sobrescrevem o header do origin se for menor/maior
- Invalidation: força cache miss imediato (1.000 paths/mês = gratuito)

## Lambda@Edge vs CloudFront Functions

| | CloudFront Functions | Lambda@Edge |
|---|---|---|
| Runtime | JavaScript (ES5.1) | Node.js, Python |
| Gatilhos | Viewer Request, Viewer Response | Viewer Req/Res + Origin Req/Res |
| Tempo máx. | < 1ms | 5s (Viewer), 30s (Origin) |
| Memória | 2 MB | 128 MB (Viewer), 10 GB (Origin) |
| Body request | NÃO (em Viewer Req) | SIM (com size limits) |
| Preço | $0,0000001 por invocação | $0,00000625 por 100ms |
| Casos de uso | Headers, redirects, URL rewrites simples | Auth, A/B test, JWT validation, body inspection |

## Global Accelerator vs Route 53 vs CloudFront

| | Route 53 | Global Accelerator | CloudFront |
|---|---|---|---|
| Tipo | DNS | Anycast network proxy | CDN |
| IPs | DNS (muda) | 2 Anycast IPs estáticos | IPs CF (mudam) |
| Protocolo | HTTP/S, TCP, UDP | TCP, UDP, HTTP/S | HTTP/S |
| Failover speed | Minutos (TTL) | < 1 minuto | Varia |
| Caching | Não | Não | **Sim** |
| Caso principal | DNS routing | Global TCP apps, IPs fixos, non-HTTP | Conteúdo estático/dinâmico global |

## Proteção e Segurança CloudFront + WAF

| Serviço | Protege contra |
|---|---|
| AWS WAF | OWASP Top 10 (SQL injection, XSS), rate limiting, IP reputation, custom rules |
| Shield Standard | DDoS L3/L4 (gratuito, automático) |
| Shield Advanced | DDoS L7, DRT 24/7, crédito de custo (~$3K/mês) |
| Geo Restriction | Bloquear/permitir países (MaxMind GeoIP) |
| Signed URLs/Cookies | Conteúdo pago/privado com expiração |

## Dicas Rápidas de Prova
- **CNAME não funciona no apex** (empresa.com); use Alias
- Alias records são **gratuitos** para Route 53 → recursos AWS
- CloudFront caching: `Cache-Control: no-cache` força revalidação; TTL=0 nunca armazena em cache
- OAC > OAI para novos projetos (SSE-KMS, todas regiões S3, mais métodos HTTP)
- Route 53 Health Check de recurso privado: via **CloudWatch Alarm** (HCs não alcançam recursos sem IP público)
- Calculated Health Check: combine múltiplos HCs com AND/OR/NOT
- WAF pode ser associado a: **CloudFront, ALB, API Gateway, AppSync**
- Global Accelerator: ideal quando precisa de **IP fixo** para whitelist de firewall
- Lambda@Edge executa nas **edge locations** (regions específicas); CloudFront Functions em **todos os PoPs**
- Route 53 TTL baixo antes de migração DNS → acelera propagação de mudanças
