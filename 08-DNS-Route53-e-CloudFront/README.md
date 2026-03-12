# Módulo 08 — DNS, Route 53 e CloudFront

## Amazon Route 53

Serviço DNS gerenciado e registrador de domínios da AWS. Nome vem da porta DNS (53) e da Rota 66 americana.

### Tipos de Hosted Zones
- **Public Hosted Zone**: resolve nomes na internet pública
- **Private Hosted Zone**: resolve nomes dentro de uma ou mais VPCs (não acessível publicamente)

### Record Types
| Tipo | Descrição | AWS-specific |
|---|---|---|
| A | IPv4 address | — |
| AAAA | IPv6 address | — |
| CNAME | Aponta para outro hostname | Não pode ser criado no apex (root domain) |
| **Alias** | Aponta para recurso AWS (ELB, CloudFront, S3, etc.) | ✅ Sim — responde como A/AAAA, gratuito para recursos AWS |
| NS | Name Servers da hosted zone | — |
| SOA | Start of Authority | — |
| MX | Mail servers | — |
| TXT | Texto (SPF, DKIM, validação ACM) | — |
| PTR | Reverse DNS | — |
| SRV, CAA | Serviços, autoridade de certificados | — |

> **Alias vs CNAME**: Alias pode ser criado no apex (exemplo.com), CNAME não pode. Alias não cobra por query. Alias resolve para o IP real (sem CNAME chain). **Não pode criar Alias para EC2 DNS name.**

### Routing Policies

| Política | Comportamento | Quando usar |
|---|---|---|
| **Simple** | Retorna 1 ou múltiplos valores aleatórios | Single resource, sem health check obrigatório |
| **Weighted** | Distribui por peso (0–255) | A/B testing, migração gradual |
| **Latency** | Roteia para região de menor latência ao usuário | Multi-region, baixa latência |
| **Failover** | Primary → Secondary se primary falhar | Active-passive DR |
| **Geolocation** | Por país/continente/estado (US) | Conteúdo localizado, compliance |
| **Geoproximity** | Por localização + bias (ampliar/reduzir região) | Redistribuição de tráfego por coordenadas |
| **Multivalue Answer** | Retorna até 8 valores, com health check | Distribuição de carga sem ELB (client-side) |
| **IP-based** | Por CIDR do cliente | ISP-specific routing, otimização de rede |

> Geoproximity requer **Traffic Flow** e **Traffic Policy** — não é configurável via console simples.

### Health Checks
- Verificam endpoint HTTP/HTTPS/TCP a cada 10s (Fast HCs) ou 30s
- Calculados por pelo menos 18 verificadores globais (threshold configurável de 1–10)
- **Calculated Health Checks**: combina múltiplos health checks (AND/OR)
- **CloudWatch Alarm Health Check**: para recursos privados (sem IP público) — HC verifica o alarm, não o recurso direto
- Podem ser associados a records de Weighted, Failover, Latency, Geolocation

---

## Amazon CloudFront

CDN (Content Delivery Network) global com 450+ PoPs (Edge Locations) e 13 Regional Edge Caches.

### Componentes
- **Distribution**: unidade de configuração do CloudFront (domain: `xxxx.cloudfront.net`)
- **Origin**: backend (S3, ALB, EC2, API Gateway, qualquer endereço HTTP)
- **Behavior (Cache Behavior)**: regra por path pattern (ex: `/api/*` → sem cache; `/*.jpg` → cache 1 dia)
- **Cache Policy**: TTL (min/max/default), headers/cookies/query strings para incluir na cache key
- **Origin Request Policy**: o que enviar ao origin além do que está na cache key
- **Edge Location**: onde os arquivos são cacheados

### CloudFront + S3

| Método | Descrição | Para novo conteúdo? |
|---|---|---|
| **OAC (Origin Access Control)** | Política no S3 bucket que só aceita requisições assinadas pelo CloudFront | ✅ Recomendado |
| **OAI (Origin Access Identity)** | Identidade virtual IAM para o CF acessar S3 privado | ⚠️ Legado (ainda funciona) |

Com OAC, o bucket S3 deve ter:
```json
{
  "Principal": {"Service": "cloudfront.amazonaws.com"},
  "Condition": {"StringEquals": {"AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT:distribution/DIST_ID"}}
}
```

### CloudFront Cache Invalidation
- Remove objetos do cache antes do TTL expirar: `/*` (tudo) ou path específico
- Cobrado por invalidation path após as primeiras 1.000/mês
- Alternativa: usar versionamento de arquivos (ex: `app.v2.js`) — sem custo de invalidação

### Geo Restriction
- **Allowlist**: apenas países listados podem acessar
- **Blocklist**: países listados são bloqueados
- Baseado em banco de dados GeoIP (não infalível — VPNs podem contornar)

### Lambda@Edge vs CloudFront Functions

| Característica | Lambda@Edge | CloudFront Functions |
|---|---|---|
| Runtime | Node.js, Python | JavaScript (ECMAScript 5.1) |
| Latência | Moderada (~1–10ms) | Muito baixa (<1ms) |
| Memória | Até 10 GB (L@E viewer), 128MB request | Máx 2 MB de memória |
| Requests/s | Sem limite de taxa | 10 milhões+ req/s |
| Duração | Até 30s (origin), 5s (viewer) | Máximo 1ms |
| Acesso à rede | Sim (pode chamar APIs externas) | Não |
| Eventos | Viewer Request/Response, Origin Request/Response | Viewer Request/Response apenas |
| Uso típico | Reescrita de URL complexa, A/B com cookies, autenticação | Manipulação de headers, URL rewrite simples, redirect |

---

## AWS Global Accelerator

Usa a rede global da AWS para rotear tráfego de usuários ao endpoint mais próximo com menor latência:

- Fornece **2 Anycast IPs estáticos** globais (não mudam — excelente para whitelisting)
- Roteia para o endpoint saudável mais próximo via rede backbone AWS
- Suporta endpoints: ALB, NLB, EC2, Endereços IP elásticos
- **Health checks** detectam failover e reroteiam automaticamente
- **Diferença do CloudFront**: CloudFront entrega conteúdo cacheado (CDN); Global Accelerator é proxy TCP/UDP, melhor para aplicações não-HTTP (jogos, IoT, VoIP)

### Route 53 vs Global Accelerator

| Aspecto | Route 53 | Global Accelerator |
|---|---|---|
| IP estático | Não (DNS) | Sim (2 Anycast IPs) |
| Failover | DNS propagation (~60s) | Quase instantâneo |
| TCP/UDP genérico | Não (apenas DNS) | Sim |
| Conteúdo cacheado | Não | Não |
| Custo | Por query | Taxa horária + por GB |

---

## AWS WAF (Web Application Firewall)

Protege aplicações web contra exploits OWASP:

- Deploy: **CloudFront, ALB, API Gateway, AppSync, Cognito User Pool**
- **Web ACL**: contém rules e rule groups
- Rules inspecionam: IP, headers, body (até 8 KB), URI string, query string, método HTTP
- **Managed Rule Groups**: conjuntos pré-configurados (OWASP Top 10, bots, AWS Managed Intelligence)
- **Rate-based Rules**: limita por IP (ex: máx 100 req/5min)
- Ação: Allow, Block, Count (monitoring sem bloquear), CAPTCHA, Challenge

---

## Dicas de Prova

- **Alias record** = gratuito + pode ser apex; CNAME = cobra por query + não pode ser apex
- **Geoproximity** precisa de Traffic Policy (Traffic Flow); Geolocation não precisa
- **Failover**: primary health check falha → roteia para secondary (passive DR)
- CloudFront + S3: OAC é o novo padrão (substituiu OAI)
- **Lambda@Edge** = processamento complexo com chamadas externas; **CF Functions** = transformações em < 1ms
- **Global Accelerator** = IPs estáticos + rede backbone AWS; bom para jogos, IoT, VoIP e apps TCP/UDP
- **Route 53 Multivalue** ≠ ELB: apenas distribui, não balanceia carga real; cliente escolhe arbitrariamente
- Resolver DNS de Private Hosted Zone de outra VPC: habilitar **VPC Association** ou usar **Route 53 Resolver** (Inbound/Outbound Endpoints) para on-prem
- CloudFront pode ter múltiplos origins em um mesmo distribution (path-based routing entre origins)
- WAF pode ir no ALB **ou** no CloudFront — para proteção closer to edge, use CloudFront + WAF
