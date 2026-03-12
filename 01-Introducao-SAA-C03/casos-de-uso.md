# Casos de Uso — Introdução SAA-C03

## Caso 1 — Plataforma SaaS iniciando revisão arquitetural

### Cenário
Uma SaaS B2B já roda na AWS, mas o time percebeu que sabe operar serviços sem conseguir justificar decisões de arquitetura. A liderança quer preparar dois arquitetos para o SAA-C03 e, ao mesmo tempo, revisar o desenho atual da plataforma.

### Arquitetura de raciocínio

```text
Negócio -> Requisitos -> Restrições -> Escolha de serviço -> Trade-off -> Operação
   |           |             |               |                  |          |
   +-----------+-------------+---------------+------------------+----------+
```

### Decisões técnicas justificadas
- O estudo deve começar por serviços centrais como IAM, VPC, EC2, S3, RDS e Route 53, porque eles aparecem em grande parte dos cenários.
- O grupo precisa revisar não só serviços, mas padrões: HA, DR, desacoplamento e custo total.
- A melhor forma de internalizar isso é combinar teoria, questões e casos reais.

### Trade-offs considerados
- Estudar por serviço isolado é rápido, mas gera pouco contexto.
- Estudar por cenário é mais lento, porém aproxima o candidato do formato real da prova.

## Caso 2 — E-commerce com dúvida entre robustez e simplicidade

### Cenário
Uma loja online quer crescer antes da Black Friday. O time técnico propõe várias soluções altamente sofisticadas, mas a diretoria quer saber qual abordagem é adequada sem inflar custo e operação. O objetivo do estudo é aprender a distinguir o necessário do excessivo.

### Arquitetura de raciocínio

```text
Usuários -> ALB -> ASG EC2 -> Aurora Multi-AZ
              |         |
              |         +-> CloudWatch
              |
              +-> CloudFront -> S3 estático
```

### Decisões técnicas justificadas
- ALB e ASG endereçam elasticidade com menor esforço do que gestão manual de múltiplas instâncias.
- Aurora Multi-AZ atende continuidade regional sem exigir multi-region logo de início.
- CloudFront reduz latência e descarrega conteúdo estático.

### Trade-offs considerados
- Multi-region ativo-ativo seria mais resiliente, mas pode ser exagero se o requisito real for apenas HA regional.
- Instâncias dedicadas podem oferecer isolamento, mas aumentam custo sem necessidade explícita.

## Caso 3 — Startup de dados com orçamento apertado

### Cenário
Uma startup precisa processar eventos de aplicação com custo controlado. O exame SAA-C03 usaria esse contexto para avaliar se o candidato entende quando priorizar serviços gerenciados e cobrança sob demanda.

### Arquitetura de raciocínio

```text
Aplicação -> EventBridge -> Lambda -> DynamoDB
                          |
                          +-> CloudWatch Logs
```

### Decisões técnicas justificadas
- Lambda reduz ociosidade e patching para workloads variáveis.
- EventBridge facilita integração orientada a eventos com baixo acoplamento.
- DynamoDB encaixa bem quando o acesso é previsível por chave e a escala é horizontal.

### Trade-offs considerados
- EC2 pode oferecer mais controle, mas exige administração constante.
- Banco relacional pode ser melhor em cenários transacionais complexos, mas aqui o padrão de acesso favorece NoSQL.

## Caso 4 — Empresa regulada com foco em governança

### Cenário
Uma organização financeira precisa de rastreabilidade, menor privilégio e chaves de criptografia controladas. Ela quer que a preparação para o exame reforce boas práticas reais de arquitetura segura.

### Arquitetura de raciocínio

```text
Usuários -> IAM Identity Center -> Contas AWS
                       |
                       +-> IAM Roles / SCPs
                       +-> KMS
                       +-> CloudTrail
```

### Decisões técnicas justificadas
- IAM Identity Center simplifica acesso centralizado.
- SCPs ajudam a limitar o que contas podem fazer no nível organizacional.
- KMS e CloudTrail aparecem como pilares de criptografia e auditoria.

### Trade-offs considerados
- Mais governança centralizada reduz risco, mas exige desenho de permissões cuidadoso.
- Políticas excessivamente amplas aceleram o curto prazo e prejudicam compliance.
