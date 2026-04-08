# Cheatsheet - Criptografia, KMS e Gestao de Segredos

## Mapa rapido

| Necessidade | Melhor escolha | Observacao |
|---|---|---|
| Chave com controle de policy e auditoria | KMS customer-managed key | Mais controle para compliance |
| Rotacao automatica de segredos | Secrets Manager | Integracao forte com RDS |
| Parametros de app com baixo custo | Parameter Store | SecureString usa KMS |
| Certificado TLS para ALB/CloudFront/API Gateway | ACM | Public cert gratuito |
| Controle criptografico em HSM dedicado | CloudHSM | Maior complexidade operacional |

## Regras de prova

- key policy e essencial para permissao no KMS
- explicit deny sempre prevalece
- use IAM role para apps em vez de chaves estaticas
- CloudFront + ACM publico => certificado em us-east-1

## Armadilhas

- confundir Secrets Manager com Parameter Store em cenario de rotacao automatica
- esquecer permissao kms:Decrypt no principal consumidor
- assumir que criptografar no servico dispensa gestao de acesso
- ignorar custo de chamadas KMS em alto throughput