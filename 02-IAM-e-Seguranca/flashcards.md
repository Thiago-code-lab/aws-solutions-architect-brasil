# Flashcards — IAM e Segurança

> Revise um cartão de cada vez. Tente responder antes de revelar a resposta.

---

**P:** Qual é a diferença entre IAM User e IAM Role?  
**R:** User tem credenciais permanentes (senha + access key). Role fornece credenciais temporárias via STS, assumida por serviços AWS, outros accounts ou identidades federadas. Nunca tem senha nem access key própria.

---

**P:** O que é Permission Boundary e para que serve?  
**R:** É uma política IAM gerenciada que define o máximo de permissões que um usuário ou role pode ter — mesmo que a identity policy conceda mais. Usado para delegar criação de roles sem escalada de privilégios.

---

**P:** Qual é a diferença entre AWS-managed policy, Customer-managed policy e Inline policy?  
**R:** AWS-managed: mantida pela AWS, reutilizável, não editável. Customer-managed: você cria e controla, reutilizável em múltiplas identidades. Inline: embarcada na identidade, 1:1, deletada junto com ela — evitar exceto para permissões exclusivas.

---

**P:** Em avaliação de políticas IAM, o que vem primeiro: SCP, Permission Boundary ou identity policy?  
**R:** Explicit Deny (qualquer fonte) → SCP bloqueia? → Permission Boundary bloqueia? → Identity policy ou resource policy permite? Para acesso cross-account: ambas identity E resource policy devem permitir.

---

**P:** O que faz a condição `aws:PrincipalOrgID`?  
**R:** Restringe acesso a um recurso apenas a principals que pertencem a uma AWS Organization específica. Útil em bucket policies para garantir que só contas da empresa acessem o bucket, sem listar cada account ID.

---

**P:** O que é uma SCP e quem ela afeta?  
**R:** Service Control Policy — guardrail de permissões máximas em AWS Organizations. Aplica-se a todas as contas-membro e OUs, mas NUNCA à management account. Um explicit allow na SCP não concede permissão por si só — a identity policy ainda precisa permitir.

---

**P:** Qual é a diferença entre `aws:SourceVpc` e `aws:SourceVpce`?  
**R:** `aws:SourceVpc` restringe ao VPC ID. `aws:SourceVpce` restringe ao VPC Endpoint ID específico. Use `aws:SourceVpce` quando quiser garantir que só acesso via um endpoint específico seja permitido.

---

**P:** Para que serve IAM Identity Center (antes AWS SSO)?  
**R:** Login único centralizado para múltiplas contas AWS e aplicações SaaS. Permite criar permission sets (conjunto de políticas) e atribuir a usuários/grupos por conta. Integra com AD, Okta e outros identity providers via SAML 2.0/SCIM.

---

**P:** Quais são os três tipos de CMK do KMS?  
**R:** 1) AWS-owned: gerenciada pela AWS, sem visibilidade para o cliente, sem custo adicional (ex: S3 SSE com aws/s3). 2) AWS-managed CMK: criada pelo serviço no seu account (aws/ebs), visível, rotação anual automática, não pode editar key policy. 3) Customer-managed CMK: você cria, paga por mês, edita key policy, define rotação, pode desabilitar.

---

**P:** O que é envelope encryption?  
**R:** Padrão em que: (1) KMS gera um Data Key (plaintext + encrypted com o CMK); (2) o cliente usa o plaintext data key para encriptar os dados; (3) descarta o plaintext key e armazena a encrypted data key junto aos dados; (4) para decriptar, envia a encrypted data key ao KMS e usa o plaintext key retornado.

---

**P:** Qual é o propósito da KMS key policy?  
**R:** Define quem pode gerenciar e usar a CMK. É obrigatória — toda CMK precisa de uma key policy. A policy inclui root account como administrador (garante recuperação), key administrators e key users. Resource-based policy, portanto permite cross-account sem role assumption.

---

**P:** O que são KMS Grants e quando usar?  
**R:** Grants concedem permissão temporária e programática para usar uma CMK sem alterar a key policy. São criados via `CreateGrant` e o token do grant é passado em chamadas de API (`kms:decrypt --grant-tokens`). Usados por serviços AWS internamente (ex: EBS quando cria snapshot de volume encriptado).

---

**P:** O que são KMS Multi-Region Keys?  
**R:** Chaves que possuem o mesmo material de chave replicado em múltiplas regiões. Permitem encriptar em us-east-1 e decriptar em eu-west-1 sem re-encriptar. Key ARNs são diferentes mas material é o mesmo. Útil para DR cross-region com dados encriptados.

---

**P:** Qual é a principal diferença entre Secrets Manager e Parameter Store?  
**R:** Secrets Manager: rotação automática nativa (integra com RDS, Redshift, DocumentDB), cobra por secret por mês. Parameter Store: sem rotação nativa, Standard gratuito (max 4KB), SecureString usa KMS. Secrets Manager é preferível para credentials com rotação; Parameter Store para configs de aplicação.

---

**P:** O que é ACM e quais são suas limitações?  
**R:** AWS Certificate Manager gerencia certificados SSL/TLS. Certificados públicos são gratuitos e renovados automaticamente. Só podem ser usados com serviços AWS integrados (ALB, CloudFront, API Gateway) — não podem ser exportados. Para CloudFront, o certificado deve estar em us-east-1.

---

**P:** O que é ACM Private CA?  
**R:** Certificate Authority privada gerenciada pela AWS. Emite certificados privados para serviços internos (mTLS, microservices). Tem custo mensal pela CA + por certificado. Certificados emitidos SÃO exportáveis, ao contrário dos públicos do ACM.

---

**P:** Como funciona cross-account role assumption?  
**R:** Conta A cria uma role com trust policy permitindo conta B assumir (sts:AssumeRole). Conta B cria uma policy permitindo sts:AssumeRole no ARN da role da conta A. O usuário na conta B chama AssumeRole, recebe credenciais temporárias da conta A.

---

**P:** Qual condição IAM requer MFA numa chamada de API?  
**R:** `"Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}` — garante que a operação só ocorre quando o principal autenticou com MFA. Útil para proteger ações sensíveis como deletar S3 objects ou desativar CMK.

---

**P:** Qual é a diferença entre identity-based policy e resource-based policy de acesso cross-account?  
**R:** Com resource-based policy (ex: S3 bucket policy, KMS key policy) que explicitly allows o principal de outra conta: acesso direto sem role assumption. Com identity-based policy apenas: precisa assumir role. Resource-based policies permitem cross-account sem hop de role — considerado mais direto para serviços que suportam.

---

**P:** O que é Parameter Store SecureString e quais são suas opções de chave KMS?  
**R:** SecureString encripta o valor usando KMS. Pode usar a aws/ssm CMK (sem custo de CMK, mas sem controle de key policy) ou uma customer-managed CMK (controle total, auditoria, pode revogar acesso). Para uso cross-account, customer-managed CMK é obrigatório.

---

**P:** O que significa `"Effect": "Deny"` em uma política IAM vs SCP?  
**R:** Em ambos os casos, explicit Deny sobrepõe qualquer Allow em qualquer outra política. Em IAM: se qualquer política aplicável tem Deny para a ação, a chamada é negada. Em SCP: Deny bloqueia toda a conta/OU, mesmo que identity policies permitam.
