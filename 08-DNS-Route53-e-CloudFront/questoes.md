# Questões de Prova — Módulo 08: DNS, Route 53 e CloudFront

<!-- Domínio SAA-C03: Design High-Performing Architectures / Design Secure Architectures -->

---

**1.** Uma empresa tem um ALB e quer registrar `api.empresa.com` apontando para ele. O domínio raiz (`empresa.com`) precisa apontar para o mesmo ALB. Qual tipo de record usar para o domínio raiz?

- A) CNAME record apontando para o DNS do ALB
- B) A record apontando para o IP do ALB
- C) Alias record apontando para o ALB
- D) NS record delegando para o ALB

<details>
<summary>Resposta</summary>
**C — Alias record**
CNAME **não pode** ser usado no domínio raiz (zona apex) — é limitação do DNS. Alias é extensão Route 53 que funciona no apex e aponta para recursos AWS (ALB, CloudFront, S3, API Gateway, etc.) sem custo de query adicional. IP do ALB é dinâmico — nunca use A record direto para ALB.
</details>

---

**2.** Uma aplicação global precisa rotear usuários para a região AWS com menor latência. Qual política de roteamento Route 53 usar?

- A) Geolocation routing
- B) Latency-based routing
- C) Weighted routing
- D) Geoproximity routing

<details>
<summary>Resposta</summary>
**B — Latency-based routing**
Latency routing mede latência entre o cliente e cada região AWS e redireciona para a região com menor latência. Geolocation roteia baseado na localização geográfica (país/continente) — não mede latência real. Geoproximity roteia baseado em distância geográfica (pode ter bias configurável) via Traffic Flow.
</details>

---

**3.** Qual política de roteamento Route 53 é usada para realizar testes A/B, enviando 10% do tráfego para uma nova versão da aplicação?

- A) Failover routing
- B) Latency routing
- C) Weighted routing
- D) Multivalue routing

<details>
<summary>Resposta</summary>
**C — Weighted routing**
Weighted permite atribuir pesos (0-255) a múltiplos records. Para 10%/90%: record A com peso 10, record B com peso 90. Útil para canary deployments, blue/green testing, migrações graduais. Soma dos pesos define a proporção; peso 0 retira o record da rotação sem apagá-lo.
</details>

---

**4.** Uma empresa tem aplicação em us-east-1 (primária) e ap-southeast-1 (backup). Quer failover automático. Qual configuração Route 53?

- A) Weighted routing com peso 90/10
- B) Failover routing com Health Check na região primária
- C) Latency routing (automático para menor latência)
- D) Multivalue routing com 2 registros

<details>
<summary>Resposta</summary>
**B — Failover routing com Health Check**
Failover routing define um record PRIMARY e um SECONDARY. Route 53 usa Health Checks para monitorar o primário; se falhar, automaticamente responde com o SECONDARY. É o padrão para Active-Passive DR. Health Check pode monitorar endpoint, outra rota ou CloudWatch Alarm.
</details>

---

**5.** Uma empresa quer servir conteúdo estático (imagens, CSS, JS) do S3 via CloudFront com máxima segurança, sem tornar o bucket S3 público. Qual é o mecanismo atual recomendado pela AWS?

- A) Origin Access Identity (OAI) com bucket policy
- B) Origin Access Control (OAC) com bucket policy
- C) Signed URLs no CloudFront
- D) S3 presigned URLs geradas pelo Lambda

<details>
<summary>Resposta</summary>
**B — Origin Access Control (OAC)**
OAC é o sucessor do OAI (lançado em 2022). Suporta S3 em todas as regiões, SSE-KMS, HTTP methods adicionais e S3 Object Lambda. O bucket precisa de bucket policy permitindo o CloudFront distribution ID via OAC. OAI ainda funciona mas é considerado legacy. O bucket permanece privado.
</details>

---

**6.** Um site de streaming precisa servir conteúdo premium pago apenas para usuários autenticados via CloudFront. Qual mecanismo usar?

- A) WAF com regra de autenticação
- B) CloudFront Signed URLs ou Signed Cookies
- C) OAC com bucket policy restritiva
- D) CloudFront Geo Restriction para países não-pagantes

<details>
<summary>Resposta</summary>
**B — Signed URLs ou Signed Cookies**
Signed URLs: para um arquivo específico por usuário (video-id específico). Signed Cookies: para múltiplos arquivos ou todo o conteúdo premium (netflix-style). Assinados com chave privada pelo backend; CloudFront valida com key pair. OAC protege o S3, não os usuários finais. WAF não autentica usuários.
</details>

---

**7.** Qual é a diferença entre CloudFront Functions e Lambda@Edge?

- A) CloudFront Functions suportam apenas Viewer Request/Response; Lambda@Edge suporta também Origin Request/Response
- B) CloudFront Functions têm latência < 1ms e menor custo; Lambda@Edge é mais poderoso (Node.js/Python, até 10s)
- C) Lambda@Edge pode acessar body da requisição; CloudFront Functions não (em Viewer Request)
- D) Todas as acima são verdadeiras

<details>
<summary>Resposta</summary>
**D — Todas são verdadeiras**
CloudFront Functions: apenas Viewer Request/Response, JavaScript, sub-milissegundo, < $0,0000001, sem acesso a body. Lambda@Edge: suporte a 4 eventos incluindo Origin Request/Response, Node.js/Python, até 30s (Origin), até 5s (Viewer), pode acessar body. Lambda@Edge = casos complexos (auth, rewrite, A/B); CF Functions = transformações simples (headers, redirect).
</details>

---

**8.** Uma empresa quer bloquear acesso ao CloudFront para países em lista denegrida. Qual recurso usar?

- A) WAF com Geographic Match rule
- B) CloudFront Geo Restriction (Allowlist ou Denylist)
- C) Route 53 Geolocation routing com NXDOMAIN
- D) Lambda@Edge verificando X-Forwarded-For

<details>
<summary>Resposta</summary>
**B — CloudFront Geo Restriction**
CloudFront Geo Restriction usa MaxMind GeoIP database para bloquear (Denylist) ou permitir apenas (Allowlist) países específicos — retorna HTTP 403. WAF Geographic Match também funciona mas tem custo adicional e é mais complexo. Route 53 Geolocation seria contornável. Geo Restriction é a solução nativa e mais simples.
</details>

---

**9.** Qual é a principal diferença entre Route 53 e AWS Global Accelerator para melhorar performance global?

- A) Route 53 usa anycast IPs; Global Accelerator usa DNS
- B) Global Accelerator usa 2 IPs Anycast estáticos e roteia pelo backbone AWS; Route 53 usa DNS-based routing
- C) Route 53 tem menor latência; Global Accelerator é mais barato
- D) Global Accelerator só funciona com ALB; Route 53 funciona com qualquer endpoint

<details>
<summary>Resposta</summary>
**B — Correto**
Global Accelerator: 2 IPs Anycast estáticos, tráfego entra no edge AWS mais próximo e trafega pelo backbone privado da AWS até o endpoint. Benefício: não depende de DNS TTL, failover em segundos, consistência de IP (whitelisting de firewall). Route 53: DNS-based, depende de TTL propagation para failover, IP muda. Para: non-HTTP ou requisito de IP fixo → GA. Para simples roteamento HTTP/S → Route 53.
</details>

---

**10.** Uma empresa usa CloudFront e percebe que uma atualização de conteúdo no S3 não aparece para usuários. O que fazer imediatamente (sem esperar o TTL expirar)?

- A) Apagar e recriar a CloudFront Distribution
- B) Criar Invalidation no CloudFront para o arquivo ou path afetado
- C) Habilitar S3 versioning e criar nova versão
- D) Alterar o TTL para 0 no Origin

<details>
<summary>Resposta</summary>
**B — CloudFront Invalidation**
Invalidation força os edge caches a descartarem objetos específicos (ex: `/images/logo.png` ou `/*`). O novo request ao edge faz cache miss e busca a versão atual da origin. Primeiros 1.000 paths de invalidation por mês são gratuitos; depois há custo. Melhor prática a longo prazo: usar versionamento nos arquivos (asset hashing) para evitar invalidações.
</details>

---

**11.** Um serviço de aplicação precisa de Health Checks para um recurso interno numa subnet privada (sem IP público). Como configurar Health Check no Route 53?

- A) Não é possível — Route 53 só monitora endpoints públicos
- B) Criar um Health Check do tipo "CloudWatch Alarm" baseado em métricas do recurso privado
- C) Usar um Calculated Health Check combinando outros HCs
- D) Instalar o Route 53 Health Check Agent na instância privada

<details>
<summary>Resposta</summary>
**B — Health Check de CloudWatch Alarm**
Route 53 Health Checkers operam da internet pública e não alcançam recursos privados. Para monitorar recursos privados: crie uma métrica CloudWatch customizada, configure um Alarm, e crie um Route 53 Health Check baseado nesse Alarm. Quando o Alarm dispara → Health Check falha → failover DNS ativado.
</details>

---

**12.** Uma empresa quer usar WAF para proteger sua API no API Gateway contra ataques SQL Injection e XSS. Qual configuração usar?

- A) WAF Classic com IP-based rules
- B) AWS WAF v2 com AWS Managed Rules (AWSManagedRulesCommonRuleSet)
- C) Shield Standard integrado ao API Gateway
- D) Security Groups no API Gateway

<details>
<summary>Resposta</summary>
**B — AWS WAF v2 com Managed Rules**
AWS WAF v2 pode ser associado ao API Gateway, ALB, CloudFront e AppSync. AWSManagedRulesCommonRuleSet inclui proteções contra SQL injection, XSS e outras vulnerabilidades OWASP. AWS mantém as regras atualizadas. Shield Standard protege contra DDoS de camada de rede, não SQL injection. API Gateway não tem Security Groups.
</details>

---

**13.** O que é "Resolver" no Route 53 e qual sua função?

- A) Serviço que resolve nomes DNS para IPs fora da AWS
- B) Route 53 Resolver DNS para VPC + Endpoints de Inbound/Outbound para DNS híbrido
- C) Componente que converte nomes Route 53 em aliases para recursos AWS
- D) Ferramenta de debug de propagação DNS global

<details>
<summary>Resposta</summary>
**B — Route 53 Resolver com Endpoints**
Route 53 Resolver responde queries DNS dentro da VPC (169.254.169.253). Para DNS híbrido: **Inbound Endpoint** permite que on-premises resolva nomes AWS (DNS queries chegam de on-prem para a VPC); **Outbound Endpoint** permite que a VPC resolva nomes on-premises (por regras de forwarding para servidores DNS on-prem via DX/VPN).
</details>

---

**14.** Uma PLataforma de conteúdo usa CloudFront. Como forçar que todo tráfego use HTTPS (redirecionar HTTP para HTTPS)?

- A) Configurar bucket S3 para redirecionar HTTP→HTTPS
- B) No CloudFront Behavior: Viewer Protocol Policy = "Redirect HTTP to HTTPS"
- C) No CloudFront Distribution: Force SSL/TLS via ACM sempre
- D) Usar Lambda@Edge para inspecionar e redirecionar

<details>
<summary>Resposta</summary>
**B — Viewer Protocol Policy no Behavior**
Cada CloudFront Behavior tem "Viewer Protocol Policy" com opções: HTTP and HTTPS, Redirect HTTP to HTTPS, ou HTTPS Only. Definir "Redirect HTTP to HTTPS" faz CloudFront retornar 301 para requests HTTP → HTTPS automaticamente. É a configuração nativa, sem Lambda@Edge necessário.
</details>

---

**15.** Qual record type DNS é necessário para verificação de domínio de certificados ACM e para receber email?

- A) CNAME (para ACM) e MX (para email)
- B) TXT (para ACM) e NS (para email)
- C) A record para ambos
- D) Alias (para ACM) e SPF record (para email)

<details>
<summary>Resposta</summary>
**A — CNAME (ACM) e MX (email)**
ACM (AWS Certificate Manager) oferece validação de domínio via CNAME: você adiciona o CNAME gerado pelo ACM ao seu DNS e a AWS verifica periodicamente. MX records apontam para servidores de email (ex: SES, Office 365, Google Workspace) e são obrigatórios para receber email no domínio. TXT records são usados para SPF/DKIM/verificação de domínio em outros serviços.
</details>
