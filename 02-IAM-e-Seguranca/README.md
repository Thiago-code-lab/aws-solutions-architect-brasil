# 02 IAM e Segurança

## 📋 Índice
- [Objetivos do Módulo](#-objetivos-do-módulo)
- [Conceitos Fundamentais](#-conceitos-fundamentais)
- [Arquitetura e Componentes](#-arquitetura-e-componentes)
- [Configurações Importantes para o Exame](#-configurações-importantes-para-o-exame)
- [Comparativo de Serviços](#-comparativo-de-serviços)
- [Dicas e Armadilhas do Exame](#-dicas-e-armadilhas-do-exame)
- [Para o Exame](#-para-o-exame)
- [Links Relacionados](#-links-relacionados)

## 🎯 Objetivos do Módulo

Ao terminar este módulo, você deve conseguir:

- diferenciar com precisão usuários, grupos, roles e policies IAM, sabendo quando usar cada um
- entender a lógica de avaliação de permissões da AWS e aplicá-la em cenários de cross-account
- usar permission boundaries e SCPs como ferramentas de governança sem bloquear operações legítimas
- distinguir os três tipos de CMK do KMS e descrever como o envelope encryption funciona
- decidir entre Secrets Manager e Parameter Store com base no requisito funcional
- configurar IAM Conditions para restringir acesso por região, MFA, VPC ou organização

## 📚 Conceitos Fundamentais

### IAM Users, Groups e Roles

Um IAM User é uma identidade permanente que representa uma pessoa ou aplicação. Ele pode ter uma senha para console e até dois access keys para acesso programático. Access keys são credenciais de longo prazo e nunca devem ser compartilhadas ou embutidas no código — a AWS recomenda substituí-las por roles sempre que possível.

IAM Groups são contêineres lógicos para usuários. Eles facilitam a gestão de permissões em escala, pois uma policy é atribuída ao grupo e todos os membros herdam automaticamente. Um usuário pode pertencer a múltiplos grupos. Grupos não podem ser aninhados e não podem assumir roles, o que é uma diferença crucial.

IAM Roles são identidades que qualquer principal elegível pode assumir temporariamente via AWS Security Token Service. Ao assumir um role, o principal recebe credenciais temporárias com prazo de validade. Um role tem dois componentes: a trust policy, que define quem pode assumi-lo, e a permissions policy, que define o que o role pode fazer. Roles são o mecanismo preferido para computação (EC2, Lambda, ECS tasks) e acesso cross-account.

### Tipos de Policies

Policies são documentos JSON com quatro campos centrais: Effect (Allow ou Deny), Action (operação da API), Resource (a quais recursos se aplica) e Condition (opcional, critérios adicionais).

**Identity-based policies** são attachadas a usuários, grupos ou roles. Podem ser AWS-managed (gerenciadas pela AWS, somente leitura para o cliente), customer-managed (criadas pelo cliente, reutilizáveis em múltiplas identidades, versionadas) ou inline (embutidas diretamente em uma identidade, relação 1:1, não pode ser reutilizada, deletada junto com a identidade).

**Resource-based policies** são attachadas ao recurso, como bucket policy do S3, key policy do KMS, queue policy do SQS, resource-based policy do Lambda ou SNS. O diferencial é que elas especificam explicitamente o principal (quem pode acessar), o que elimina a necessidade de assumir um role para acesso cross-account em vários casos.

**Permission Boundaries** são um teto de permissões. Mesmo que uma identity policy conceda acesso amplo, a permissão efetiva é a interseção entre o que a identity policy permite e o que o permission boundary permite. Isso é fundamental para ambientes onde administradores delegam a capacidade de criar roles, mas precisam garantir que as roles criadas não excedam um nível máximo.

**Service Control Policies (SCPs)** são aplicadas a nível de AWS Organizations em accounts ou OUs. Funcionam como guardrail, limitando o que qualquer principal (incluindo o account root) pode fazer dentro de uma conta — exceto o management account, que nunca é limitado por SCPs. SCPs não concedem permissões; elas apenas definem o teto do que é permitível.

### Lógica de Avaliação de Permissões

A AWS avalia se uma ação é permitida seguindo uma ordem determinada. Primeiro verifica se há um explicit deny em qualquer policy — se sim, a ação é negada independentemente de qualquer allow. Depois percorre SCPs (se Organizations habilitado), permission boundaries, identity policies e resource-based policies. Para acesso intra-conta, basta um allow em identity ou resource policy. Para cross-account, é necessário allow nas duas camadas: identidade DA conta chamante (ou trust + permission do role assumido) e resource policy na conta destino.

### IAM Conditions

Conditions permitem restringir quando e como uma permissão se aplica. As mais cobradas no exame:

- `aws:RequestedRegion` — restringe chamadas a regiões específicas. Útil para implementar data residency.
- `aws:MultiFactorAuthPresent` — exige que o chamante tenha autenticado com MFA.
- `aws:PrincipalOrgID` — restringe o principal a membros de uma organização. Muito útil em resource policies.
- `aws:SourceVpc` e `aws:SourceVpce` — restringe acesso a tráfego que origina de VPC ou VPC endpoint específico.
- `aws:CurrentTime` — control baseado em data e hora.
- `s3:prefix` — controle de acesso a prefixos específicos em buckets S3.

### AWS Organizations e IAM Identity Center

AWS Organizations agrupa múltiplas contas sob uma hierarquia gerenciada. A raiz contém a management account e OUs. SCPs são aplicados a OUs ou contas diretamente. Consolidated billing unifica faturas.

IAM Identity Center (antigo AWS SSO) centraliza o acesso a múltiplos accounts e aplicações. Em vez de criar usuários IAM em cada conta, você define permission sets (conjunto de policies que representam um nível de acesso) e os atribui a usuários e grupos por conta. É possível integrar com Active Directory, Okta e outros provedores externos. Para o exame, IAM Identity Center é a resposta preferida quando o cenário fala em gerenciar acesso de funcionários em múltiplas contas.

### KMS — Key Management Service

O KMS protege chaves criptográficas e as operações que as usam nunca expõem o material da chave. Há três tipos de Customer Master Keys (CMKs):

**AWS-owned CMK**: gerenciadas e mantidas pela AWS, completamente transparentes para o cliente. Você não pode vê-las nem gerenciá-las. Exemplos: criptografia padrão do S3 (SSE-S3), DynamoDB por padrão. Custo zero para o cliente.

**AWS-managed CMK**: criadas automaticamente por serviços AWS (como aws/s3, aws/ebs, aws/rds) na sua conta quando o serviço precisa. Você pode visualizá-las e ver o uso de auditoria no CloudTrail, mas não as gerencia diretamente. Rotação anual automática. Custo zero.

**Customer-managed CMK**: você cria, define a key policy, controla rotação (anual opcional), pode habilitar/desabilitar, pode importar material externo ou usar CloudHSM como key store. Cobradas por mês e por uso de API.

**Key Policy**: todo CMK tem obrigatoriamente uma key policy. Se nenhuma política de identidade for configurada, a key policy por si só controla o acesso. O documento padrão dá acesso ao root da conta, o que permite gerenciar o acesso via IAM policies. Sem essa cláusula, apenas principals explicitados na key policy têm acesso.

**Envelope Encryption**: é o padrão de criptografia em escala. A aplicação gera um data key via KMS (GenerateDataKey). A data key retorna em dois formatos: plaintext (para criptografar o dado) e encrypted (protegido pelo CMK). A aplicação criptografa o dado, descarta a data key plaintext e armazena o dado criptografado junto com a data key criptografada. Para descriptografar, envia a data key criptografada ao KMS (Decrypt) e obtém a plaintext data key de volta.

**Grants**: delegam acesso a uma operação específica do KMS de forma programática, sem alterar a key policy. Útil quando uma aplicação precisa de acesso temporário. Grants podem ser revogados.

**Multi-Region Keys**: par de chaves primária e réplicas em regiões diferentes que compartilham o mesmo material de chave e key ID. Permite criptografar em uma região e descriptografar em outra sem re-encrypt.

### Secrets Manager vs Parameter Store

Secrets Manager é projetado para segredos que precisam de rotação. Integra diretamente com RDS, Redshift, DocumentDB e outros serviços para rotação automática sem downtime. Cada segredo é cobrado mensalmente. Suporta resource-based policy para acesso cross-account.

Parameter Store é um armazenamento hierárquico de configuração. O tier standard é gratuito e suporta até 4KB. O tier advanced suporta até 8KB, policies (TTL, notificação) e maior throughput. SecureString usa KMS para criptografia. Não tem rotação nativa, mas pode ser integrado com Lambda para simular rotação. A hierarquia (como /app/prod/database/password) facilita o controle via IAM com s3:prefix equivalente de parâmetro.

### ACM — AWS Certificate Manager

ACM provisiona e renova certificados TLS/SSL. Certificados públicos são gratuitos e renovados automaticamente via DNS ou email validation. DNS validation é preferível por ser automático. Certificados ACM só podem ser usados com serviços que integram nativamente: ALB, API Gateway, CloudFront, Elastic Beanstalk. Não é possível exportar a chave privada de um certificado público ACM.

Para CloudFront, o certificado precisa estar em us-east-1 independentemente da distribuição. Para ALB e outros serviços regionais, o certificado deve estar na mesma região.

ACM Private CA provisiona uma autoridade certificadora privada para emitir certificados internos. É cobrada por CA provisionada e certificado emitido.

## 🏗️ Arquitetura e Componentes

```text
AWS Organizations
    ├── Management Account
    └── OU: Produção
         ├── Account A (conta de carga de trabalho)
         │    └── IAM Identity Center → Permission Set → Role assumida
         │
         └── Account B (conta compartilhada)
              └── S3 Bucket com resource policy:
                   Principal: arn:aws:iam::ACCOUNT_A:role/AppRole
                   Condition: aws:PrincipalOrgID = o-xxxx

Fluxo de avaliação de permissão:
  Chamada API
      │
      ▼ SCP permite?  ──NÃO──► DENY
      │SIM
      ▼ Permission Boundary permite? ──NÃO──► DENY
      │SIM
      ▼ Explicit Deny em qualquer policy? ──SIM──► DENY
      │NÃO
      ▼ Identity policy OU resource policy permite? ──NÃO──► DENY
      └──SIM──► ALLOW
```

## ⚙️ Configurações Importantes para o Exame

| Conceito | Detalhe importante |
|---|---|
| Limite de IAM Users por conta | 5.000 |
| Limite de Access Keys por usuário | 2 |
| Duração padrão de credencial temporária | 1 hora (ajustável até 12h com sts:DurationSeconds) |
| SCPs não afetam | management account da organização |
| Permission Boundary afeta | usuários e roles (não grupos) |
| CMK customer-managed rotação | opcional, anual |
| CMK AWS-managed rotação | obrigatória, anual |
| Secrets Manager cobrança | US$ 0,40/segredo/mês + US$ 0,05/10.000 chamadas |
| Parameter Store Standard | gratuito, até 10.000 parâmetros, 4KB |
| Parameter Store Advanced | cobrado, até 8KB, políticas de parâmetro |
| ACM CloudFront | certificado deve estar em us-east-1 |
| KMS GenerateDataKey | retorna chave em plaintext e ciphertext |

## 🔄 Comparativo de Serviços

| Característica | Secrets Manager | Parameter Store SecureString |
|---|---|---|
| Rotação automática nativa | ✅ Sim | ❌ Necessita Lambda externo |
| Integração com RDS | ✅ Nativa | ❌ Manual |
| Custo | Pago por segredo | KMS API + parâmetro Advanced |
| Tamanho máximo | 65.536 bytes | 8KB (Advanced) |
| Cross-account | Resource-based policy | Não nativamente |
| Caso de uso principal | Credenciais de banco de dados rotacionáveis | Configuração de aplicação e segredos simples |

| Característica | AWS-owned CMK | AWS-managed CMK | Customer-managed CMK |
|---|---|---|---|
| Visibilidade na conta | Não | Sim | Sim |
| Gerenciamento pelo cliente | Não | Não | Sim |
| Rotação | AWS gerencia | Anual automática | Anual (opcional) |
| Custo | Gratuito | Gratuito | US$ 1/mês + API |
| Key Policy customizável | Não | Não | Sim |

| Política | Quem sofre efeito | Onde é attachada | Objetivo |
|---|---|---|---|
| Identity policy | Usuario / Role / Grupo | Identidade | Concede permissões |
| Resource policy | Qualquer principal externo | Recurso (S3, KMS, SQS...) | Acesso cross-account direto |
| Permission Boundary | Usuario / Role | Identidade | Define teto de permissões |
| SCP | Contas e OUs | Organizations | Guardrail organizacional |
| Session Policy | Session principal | Passada ao AssumeRole | Limita a sessão temporária |

## 💡 Dicas e Armadilhas do Exame

- SCPs nunca afetam o management account. Se a questão menciona que "nem mesmo o admin pode fazer X", pense em permission boundary, não SCP.
- Para acesso cross-account sem assumir role, use resource-based policy com `aws:PrincipalOrgID` — é mais simples e escalável.
- KMS key policy não é o mesmo que IAM policy. A key policy é obrigatória e separada. Sem a linha que dá controle ao root da conta, ninguém além de quem está explicitamente na key policy consegue acesso — nem admin IAM.
- AWS-managed CMK não aparece como selecionável para criptografia personalizada. Se a questão exige controle do ciclo de vida da chave, a resposta é customer-managed CMK.
- Envelope encryption não armazena a data key em plaintext. Sempre armazena a versão criptografada junto com os dados.
- IAM Groups não assumem roles. Se o cenário exige que um grupo de EC2 acesse S3, a resposta é IAM Role na EC2 Instance Profile, não grupo IAM.
- Rotate credentials para applications: a resposta é Secrets Manager com rotação automática, especialmente quando a questão cita RDS, Redshift ou DocumentDB.
- IAM Identity Center é a resposta para gestão centralizada de acesso workforce em múltiplas contas. IAM Users em cada conta é anti-pattern para esse cenário.

## 💡 Para o Exame

- Decore a lógica de avaliação: explicit deny ganha, depois SCP, depois boundary, depois identity/resource policy.
- Cross-account = role assumption (trust policy + permission policy) OU resource-based policy com principal explícito.
- Rotação automática = Secrets Manager, não Parameter Store.
- Controle de chave de criptografia = customer-managed CMK.
- Acesso workforce multi-conta = IAM Identity Center.
- Restrict acesso por VPC = condition `aws:SourceVpc`, não NACL ou Security Group isoladamente.

## 📎 Links Relacionados

- [Questões do módulo](./questoes.md)
- [Flashcards do módulo](./flashcards.md)
- [Cheatsheet do módulo](./cheatsheet.md)
- [Casos de uso do módulo](./casos-de-uso.md)
- [Lab prático do módulo](./lab.md)
- [Links oficiais](./links.md)
- [Módulo 01: Introdução SAA-C03](../01-Introducao-SAA-C03/)
- [Módulo 07: VPC e Redes](../07-VPC-e-Redes/)
- [Módulo 14: Monitoramento](../14-Monitoramento-CloudWatch-CloudTrail/)
