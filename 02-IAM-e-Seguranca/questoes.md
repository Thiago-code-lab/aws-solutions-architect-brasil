# Questões — IAM e Segurança

## Questão 1
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma equipe de segurança quer garantir que desenvolvedores possam criar IAM roles para as suas aplicações, mas que essas roles nunca possam ter permissão para acessar o serviço de faturamento da AWS ou exportar chaves do KMS. Qual mecanismo IAM deve ser configurado?

- A) AWS Organizations SCP aplicada à OU dos desenvolvedores
- B) IAM Permission Boundary aplicada às roles criadas pelos desenvolvedores
- C) IAM Group com Explicit Deny para Billing e KMS
- D) S3 Bucket Policy com condição PrincipalOrgID

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Permission Boundaries definem o teto de permissões efetivas de uma role ou usuário. Mesmo que um desenvolvedor crie uma role com permissão full access, se o boundary não inclui Billing ou KMS:ExportKey, esses acessos são bloqueados. A alternativa A seria correta para restringir toda uma conta, mas não resolve o problema de roles criadas pelos próprios desenvolvedores dentro da conta. A alternativa C bloqueia o grupo de dev, mas não as roles que eles criam. A alternativa D é uma resource-based policy de S3, não resolve o caso.

**Conceito-chave:** Permission Boundary como teto para delegação controlada
</details>

## Questão 2
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Difícil

Uma empresa tem múltiplas contas AWS numa organização. A equipe de segurança quer garantir que nenhuma conta, independentemente de qualquer admin local, consiga desabilitar o CloudTrail. Qual solução resolve isso?

- A) Criar IAM Permission Boundary com Deny para cloudtrail:StopLogging em cada conta
- B) Criar uma SCP na organização com Deny para cloudtrail:StopLogging e aplicar à raiz
- C) Criar IAM Policy com Deny para CloudTrail no nível de conta
- D) Configurar uma bucket policy no bucket do CloudTrail

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

SCPs aplicadas à raiz da organização afetam todas as contas (exceto o management account) e nenhum administrador local pode sobrepor um explicit deny de SCP. A alternativa A exige configuração em cada conta individualmente e pode ser removida por um admin local. A alternativa C é uma IAM policy que também pode ser modificada por um admin da conta. A alternativa D protege o bucket, mas não impede o StopLogging ou DeleteTrail antes dos logs chegarem ao bucket.

**Conceito-chave:** SCP como guardrail organizacional imutável para admins locais
</details>

## Questão 3
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma aplicação Flask rodando em EC2 precisa ler segredos do banco de dados com rotação automática a cada 30 dias. O requisito de compliance exige que as credenciais nunca sejam armazenadas em código ou arquivos de configuração. Qual é a solução mais adequada?

- A) Armazenar as credenciais em variáveis de ambiente na instância
- B) Usar AWS Secrets Manager com role IAM na EC2 e rotação automática configurada
- C) Armazenar as credenciais em Parameter Store Standard e ler no cloud-init
- D) Embutir as credenciais no User Data da instância

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Secrets Manager é a solução preferida para credenciais rotacionáveis. A EC2 usa uma IAM role (Instance Profile) para chamar a API do Secrets Manager sem expor credenciais. A rotação automática é nativa para RDS, Redshift e outros banco. A alternativa A armazena em plaintext em variáveis de ambiente. A alternativa C não tem rotação nativa. A alternativa D expõe credenciais no User Data, visível via metadata.

**Conceito-chave:** Secrets Manager + Instance Profile para credenciais rotacionáveis
</details>

## Questão 4
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Difícil

Uma equipe quer que chamadas ao Amazon S3 de uma VPC específica sejam permitidas apenas para objetos com prefixo `/reports/`. Via bucket policy, qual combinação de elementos implementa isso corretamente?

- A) Condition: aws:SourceIp apontando para o CIDR da VPC + resource com prefixo /reports/*
- B) Condition: aws:SourceVpce apontando para o VPC Endpoint ID + resource com prefixo /reports/*
- C) Permission Boundary com s3:prefix = /reports/
- D) SCP com Condition: aws:SourceVpc e resource do bucket

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

`aws:SourceVpce` identifica o tráfego que passa por um VPC Endpoint específico, sendo mais preciso que o IP da VPC (que pode mudar). Combinado com o resource `arn:aws:s3:::bucket/reports/*`, restringe acesso ao prefixo correto. A alternativa A usa IP, que não é a forma recomendada para restrição via VPC. A alternativa C aplica Permission Boundary, que não resolve a restrição de prefixo em bucket policy. A alternativa D, SCPs não operam em resource policies de serviços específicos.

**Conceito-chave:** aws:SourceVpce em bucket policy para acesso via VPC Endpoint
</details>

## Questão 5
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma empresa criptografa dados no Amazon RDS usando KMS. Um auditor quer verificar qual tipo de CMK foi usada para garantir que a empresa controla todo o ciclo de vida da chave. Qual tipo de CMK atende a esse requisito?

- A) AWS-owned CMK
- B) AWS-managed CMK (aws/rds)
- C) Customer-managed CMK criada pelo cliente
- D) Data key gerada pelo RDS

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Somente o customer-managed CMK dá ao cliente controle total: definição da key policy, habilitação/desabilitação, rotação controlada e audit via CloudTrail com nível de detalhe. AWS-owned CMK não é visível na conta. AWS-managed CMK é visível, mas o cliente não gerencia. Data key é derivada de uma CMK e não é o que o auditor pergunta.

**Conceito-chave:** customer-managed CMK para controle do ciclo de vida criptográfico
</details>

## Questão 6
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma funcionalidade nova precisa requisitar objetos do S3 de uma conta B a partir de uma Lambda na conta A. A equipe prefere evitar criação de role na conta B por simplicidade. Qual abordagem funciona?

- A) Adicionar uma SCP na conta B que permite o ARN da Lambda
- B) Adicionar a conta A como principal na bucket policy do bucket na conta B
- C) Criar um IAM user na conta B e usar as credenciais na Lambda da conta A
- D) Configurar VPC Peering entre as contas

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Uma resource-based policy (bucket policy) pode especificar um principal de outra conta sem necessidade de criação de role. A Lambda na conta A precisa ter permissão para s3:GetObject na sua execution role, e o bucket na conta B precisa ter a bucket policy que permite aquele ARN. Essa é a forma mais simples de cross-account sem role assumption. A alternativa A, SCP não concede permissão, apenas restringe. A alternativa C compartilha credenciais long-term, má prática. A alternativa D não resolve IAM; VPC Peering é sobre rede.

**Conceito-chave:** cross-account via resource-based policy sem role assumption
</details>

## Questão 7
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Difícil

Um arquiteto precisa implementar envelope encryption para dados sensíveis armazenados em S3. Qual é a sequência correta de operações?

- A) Criptografar o dado com a CMK diretamente, armazenar no S3
- B) Gerar uma data key via KMS, criptografar o dado com ela em plaintext, criptografar a data key com a CMK, armazenar dado criptografado + data key criptografada
- C) Armazenar a data key em plaintext junto com o dado criptografado
- D) Usar somente a data key sem qualquer CMK

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Envelope encryption funciona: (1) chamada GenerateDataKey ao KMS retorna data key em plaintext e ciphertext; (2) criptografar o dado com a data key plaintext; (3) descartar a data key plaintext da memória; (4) armazenar o dado criptografado junto com a data key ciphertext. Para descriptografar, enviar a data key ciphertext ao KMS (Decrypt) para obter a plaintext de volta. A alternativa A criptografa diretamente com CMK, o que é ineficiente para grandes volumes e não escala. A alternativa C expõe a data key. A alternativa D não protege a data key.

**Conceito-chave:** sequência correta de envelope encryption com KMS
</details>

## Questão 8
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma empresa multinacional quer que todos os funcionários de diferentes subsidiárias acessem múltiplas contas AWS com login único (SSO) via Active Directory corporativo existente. Qual serviço é mais adequado?

- A) Criar IAM Users em cada conta AWS
- B) IAM Identity Center integrado com Active Directory via SAML ou SCIM
- C) AWS Cognito User Pools
- D) IAM Roles com ADFS como IdP por conta individualmente

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

IAM Identity Center oferece gerenciamento centralizado de acesso para múltiplas contas, integração nativa com AD via AWS Managed AD ou AD Connector (também suporta SAML/SCIM com outros IDPs como Okta), e permission sets que mapeiam nível de acesso por conta. Elimina a necessidade de gerenciar usuários IAM em cada conta. A alternativa A é a abordagem de mais esforço e menos segura. A alternativa C é para autenticação de clientes/usuários de aplicações, não funcionários internos. A alternativa D é operacionalmente complexo de manter em escala.

**Conceito-chave:** IAM Identity Center para SSO centralizado multi-conta
</details>

## Questão 9
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Um desenvolvedor inadvertidamente deletou uma CMK customer-managed do KMS que estava sendo usada para criptografar dados no EBS. Qual é o estado da CMK após a ação de delete?

- A) A CMK é deletada imediatamente e os dados são irrecuperáveis
- B) A CMK entra em período de espera de 7 a 30 dias antes da deleção efetiva, podendo ser cancelada
- C) A CMK é desabilitada mas permanece disponível indefinidamente
- D) A CMK é arquivada no S3 automaticamente

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

O KMS implementa um período de espera de 7 a 30 dias (configurável, padrão 30 dias) antes da deleção efetiva de uma CMK. Durante esse período, a deleção pode ser cancelada. Nenhuma operação criptográfica pode ser feita com a chave no período de espera. Isso protege contra exclusão acidental. Se a deleção ocorrer, dados criptografados exclusivamente por aquela chave se tornam irrecuperáveis.

**Conceito-chave:** período de espera de deleção de CMK no KMS
</details>

## Questão 10
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma aplicação de múltiplos microserviços usa uma VPC com subnets privadas. Cada microserviço precisa de acesso ao S3 sem expor tráfego à internet pública. Qual solução é mais eficiente e segura?

- A) Configurar NAT Gateway para tráfego S3
- B) Criar um VPC Gateway Endpoint para S3
- C) Criar um VPC Interface Endpoint para S3
- D) Usar Direct Connect para acessar S3

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

O Gateway Endpoint para S3 (e DynamoDB) é gratuito e acessado via route table. O tráfego permanece dentro da rede AWS sem precisar de NAT Gateway. É a solução mais econômica. A alternativa A envolve custo de processamento e transferência do NAT Gateway além de rotear para internet. A alternativa C, Interface Endpoint para S3 (PrivateLink) também funciona, mas tem custo por hora e por GB — preferível quando é necessário acessar S3 a partir de on-premises via Direct Connect. A alternativa D é para conectividade on-premises.

**Conceito-chave:** Gateway Endpoint para S3 sem custo de dados vs Interface Endpoint com custo
</details>

## Questão 11
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Fácil

Uma policy IAM tem as seguintes declarações: uma Allow para s3:GetObject em qualquer bucket, e uma Deny explícita para s3:GetObject no bucket `logs-confidenciais`. O que acontece quando o usuário tenta acessar um objeto em `logs-confidenciais`?

- A) O Allow da primeira declaração prevalece
- B) O Deny explícito prevalece e o acesso é negado
- C) Depende da ordem das declarações no JSON
- D) É preciso de outra policy Allow para resolver o conflito

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

A regra fundamental da avaliação IAM é: um explicit Deny sempre sobrepõe qualquer Allow, independentemente da ordem das declarações ou de quantas policies permitem. Não existe como um Allow superar um explicit Deny na mesma conta.

**Conceito-chave:** explicit Deny tem prioridade absoluta sobre Allow
</details>

## Questão 12
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Difícil

Uma empresa quer que certificados TLS de APIs expostas via ALB sejam gerenciados centralmente, com renovação automática sem intervenção manual. Qual combinação de serviços resolve isso?

- A) CloudFront com certificado auto-assinado criado pela equipe
- B) ACM com certificado público para o domínio, DNS validation, integrado ao ALB
- C) Gerar certificado com OpenSSL, importar no IAM e anexar ao ALB
- D) EC2 com Nginx gerenciando certificados manualmente

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

ACM provisiona e renova certificados públicos automaticamente. DNS validation é o método preferido por ser automatizado via CNAME record. O certificado é integrado diretamente ao ALB sem necessidade de exportar chave privada. A alternativa A não é gerenciamento centralizado e auto-assinado não é para produção. A alternativa C requer deleção e recriação manual na renovação. A alternativa D aumenta esforço operacional desnecessariamente.

**Conceito-chave:** ACM com DNS validation para renovação zero-touch
</details>

## Questão 13
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma equipe precisa diferenciar Parameter Store de Secrets Manager ao escolher onde armazenar uma feature flag booleana sem valor sensível e uma senha de banco de dados rotacionável. Qual alocação está correta?

- A) Ambos no Secrets Manager para padronizar
- B) Feature flag no Parameter Store Standard; senha de banco no Secrets Manager
- C) Ambos no Parameter Store Advanced
- D) Feature flag no Secrets Manager; senha de banco no EC2 Systems Manager agent

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Parameter Store Standard é gratuito e ideal para configuração não sensível ou segredos simples sem rotação. Secrets Manager é o mais adequado para credenciais que exigem rotação automática. Usar Secrets Manager para feature flags paga por algo que não precisava. A alternativa C usa Advanced sem necessidade. A alternativa D mistura conceitos incorretamente.

**Conceito-chave:** Parameter Store para config/segredos simples; Secrets Manager para rotação
</details>

## Questão 14
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Difícil

Uma organização precisa que uma chave KMS seja usada tanto em us-east-1 quanto em eu-west-1 para criptografar e descriptografar dados sem necessidade de re-encrypt ao mover dados entre regiões. Qual recurso KMS atende a esse requisito?

- A) Cross-region replication de CMK via S3
- B) KMS Multi-Region Keys com réplica em eu-west-1
- C) Usar a mesma data key exportada em ambas as regiões
- D) Criar CMKs independentes em cada região e usar AWS DataSync para sincronizar

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Multi-Region Keys compartilham o mesmo material de chave entre regiões distintas, usando o mesmo key ID (com prefixo mrk-). Um dado criptografado em us-east-1 pode ser descriptografado em eu-west-1 sem re-encrypt, pois as chaves são criptograficamente equivalentes. A alternativa A não existe. A alternativa C data key exportada expõe o material fora do KMS — rompe o modelo de segurança. A alternativa D cria chaves com materiais diferentes, impedindo cross-region decrypt direto.

**Conceito-chave:** KMS Multi-Region Keys para encrypt/decrypt cross-region
</details>

## Questão 15
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma EC2 num ambiente de produção precisa apenas de permissões para colocar métricas no CloudWatch e nada mais. Qual é a melhor abordagem de menor privilégio?

- A) Criar um IAM User com a policy AmazonCloudWatchFullAccess
- B) Criar uma IAM Role com uma customer-managed policy concedendo apenas cloudwatch:PutMetricData e associar à instância via Instance Profile
- C) Criar uma IAM Group com permissão de CloudWatch e adicionar a instância ao grupo
- D) Usar a access key do admin na variável de ambiente AWS_ACCESS_KEY_ID

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

IAM Roles com Instance Profile fornecem credenciais temporárias automaticamente rotacionadas. A policy deve ser mínima: apenas `cloudwatch:PutMetricData` no resource `*`. Evita uso de credenciais long-term. A alternativa A usa usuário (long-term credentials) e permissões Full Access — viola least privilege. A alternativa C, instâncias não podem ser membros de grupos IAM. A alternativa D é a pior prática: credentials hardcodadas no ambiente.

**Conceito-chave:** Instance Profile + least privilege policy para EC2
</details>
