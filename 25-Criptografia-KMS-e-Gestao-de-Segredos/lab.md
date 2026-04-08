# Lab - KMS + Secrets Manager + IAM Role

## Objetivo
Proteger credenciais de aplicacao usando segredo rotacionavel e acesso por role.

## Passos

1. Crie uma KMS customer-managed key para segredos.
2. Crie um segredo no Secrets Manager (usuario/senha de teste) usando a chave.
3. Crie role IAM para workload (EC2/Lambda) com permissao `secretsmanager:GetSecretValue` e `kms:Decrypt`.
4. Associe a role ao recurso de compute.
5. Leia o segredo via SDK/CLI sem credenciais estaticas.
6. Ative rotacao automatica (quando aplicavel).

## Validacao

- workload le segredo com role
- acesso negado para principal sem permissao
- eventos de acesso aparecem no CloudTrail

## Limpeza

- remover segredo de laboratorio
- remover role/policies e chave criada