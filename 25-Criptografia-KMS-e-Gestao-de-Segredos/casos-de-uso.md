# Casos de Uso - Criptografia, KMS e Gestao de Segredos

## Caso 1: Banco relacional com rotacao de senha

Cenario:
- aplicacao em ECS acessa Aurora
- auditoria exige rotacao automatica

Arquitetura:
- Secrets Manager com rotacao
- task role do ECS para ler segredo
- KMS customer-managed key para criptografia do segredo

## Caso 2: API publica com TLS gerenciado

Cenario:
- API Gateway + CloudFront
- sem gerenciamento manual de certificados

Arquitetura:
- ACM publico com renovacao automatica
- certificado em us-east-1 para CloudFront

## Caso 3: Empresa com exigencia de chave sob controle estrito

Cenario:
- compliance exige segregacao de acesso a chaves

Arquitetura:
- customer-managed keys no KMS
- key policy minima + grants para aplicacoes
- trilha de auditoria via CloudTrail

## Caso 4: Aplicacao legado com segredo em arquivo

Cenario:
- senha hardcoded no codigo

Arquitetura:
- migrar para Parameter Store/Secrets Manager
- remover credencial do repositorio
- uso de IAM role no runtime