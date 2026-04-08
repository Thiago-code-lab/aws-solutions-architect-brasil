# 25 Criptografia, KMS e Gestao de Segredos

## Objetivos do modulo

- dominar criptografia em repouso e em transito no contexto SAA-C03
- diferenciar AWS KMS, CloudHSM, Secrets Manager, Parameter Store e ACM
- decidir entre chaves AWS-managed e customer-managed com base em compliance
- aplicar padroes de seguranca com menor sobrecarga operacional

## Conceitos fundamentais

A prova SAA-C03 cobra seguranca aplicada: nao basta saber o nome do servico, e preciso escolher a melhor combinacao de criptografia, controle de acesso e rotacao de segredos.

## KMS na pratica

- KMS gerencia chaves e operacoes criptograficas auditaveis.
- Customer managed keys permitem controle de policy, rotacao e grants.
- Key policy e obrigatoria para acesso correto.
- Envelope encryption e padrao para dados em escala.

Fluxo resumido de envelope encryption:
1. aplicacao pede data key ao KMS (GenerateDataKey)
2. KMS retorna data key plaintext e cifrada
3. aplicacao cifra os dados com a key plaintext
4. armazena dados cifrados + data key cifrada

## Segredos e certificados

- Secrets Manager: segredos com rotacao automatica (forte para credenciais de banco).
- Parameter Store SecureString: configuracoes e segredos simples, custo menor.
- ACM: certificados TLS para ALB, CloudFront e API Gateway.
- ACM Private CA: emissao de certificados privados internos.

## Dicas de exame

- se houver requisito de rotacao automatica de credencial, resposta tende a Secrets Manager.
- se pede "full control over encryption keys", use customer managed KMS key.
- se precisa HSM dedicado e controle criptografico especializado, considere CloudHSM.
- para CloudFront com ACM, certificado publico deve estar em us-east-1.
- criptografia sem controle de permissao nao resolve risco; IAM + key policy importam.

## Links relacionados

- [Cheatsheet](./cheatsheet.md)
- [Casos de uso](./casos-de-uso.md)
- [Questoes](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Links oficiais](./links.md)