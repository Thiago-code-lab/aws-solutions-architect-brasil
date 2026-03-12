# 01 Introdução SAA-C03

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

- entender exatamente o que o exame SAA-C03 mede e como os quatro domínios aparecem nas questões
- diferenciar o nível de profundidade exigido no SAA-C03 em relação ao CLF-C02
- montar um plano de estudos de 8 semanas sem perder tempo com assuntos de baixo retorno
- reconhecer os serviços com maior probabilidade de aparecer nas provas e em quais contextos eles são cobrados
- pensar como arquiteto de soluções: identificar requisito, restrição, risco, trade-off e decisão mais adequada ao cenário

## 📚 Conceitos Fundamentais

O exame AWS Certified Solutions Architect – Associate, código SAA-C03, não é uma prova de definição decorada. Ele avalia a sua capacidade de tomar decisões arquiteturais em cenários realistas. Em vez de perguntar apenas o que um serviço faz, a AWS normalmente apresenta uma necessidade de negócio, uma restrição operacional e uma exigência técnica. A partir disso, você precisa apontar a alternativa que melhor equilibra disponibilidade, desempenho, segurança e custo. Essa mudança de foco é o que separa o SAA-C03 de exames mais introdutórios.

Os quatro domínios da prova ajudam a organizar esse raciocínio. O primeiro é Design Resilient Architectures, com 30% do exame. É aqui que entram multi-AZ, multi-region, desacoplamento, filas, replicação, backup, disaster recovery e tolerância a falhas. Em muitas questões, a resposta correta não é a tecnologia mais avançada, mas sim a que reduz ponto único de falha com menor complexidade operacional. Por isso, entender bem Route 53, Load Balancers, Auto Scaling, S3 Replication, RDS Multi-AZ, Aurora e padrões com SQS ou EventBridge é fundamental.

O segundo domínio é Design High-Performing Architectures, com 28%. Esse bloco mede a sua habilidade de selecionar a opção de compute, banco, rede ou armazenamento mais adequada para throughput, latência, escalabilidade e padrão de acesso. É comum a prova comparar EC2 com Lambda, RDS com DynamoDB, CloudFront com Global Accelerator, gp3 com io2, ElastiCache Redis com Memcached, além de pedir ajustes de performance para S3, Kinesis e Athena. O ponto central aqui é entender perfil de workload. A AWS sempre recompensa respostas que alinham o tipo de serviço com a característica do tráfego ou da aplicação.

O terceiro domínio é Design Secure Applications and Architectures, com 24%. Não basta conhecer IAM em nível superficial. Você precisa dominar diferença entre role, policy, resource-based policy, SCP, permission boundary, trust policy e KMS key policy. Também precisa saber quando usar Secrets Manager em vez de Parameter Store, quando preferir VPC Endpoint a tráfego público, como CloudTrail e Config se complementam e qual serviço reduz superfície de ataque sem aumentar demais a operação. Muitas questões de segurança são, na verdade, questões de desenho: como dar acesso mínimo necessário, como isolar tráfego, como criptografar dados em repouso e em trânsito e como auditar mudanças.

O quarto domínio é Design Cost-Optimized Architectures, com 18%. Embora tenha o menor peso, ele aparece cruzado com todos os outros. A AWS gosta de perguntar qual solução mantém o requisito com menor custo operacional, menor esforço de administração ou melhor aderência ao consumo real. Savings Plans, instâncias Spot, S3 Intelligent-Tiering, serverless, classes adequadas de armazenamento, desligamento automatizado e serviços gerenciados surgem o tempo todo. A armadilha comum é escolher a arquitetura mais poderosa, porém excessiva para a necessidade apresentada.

Comparando com o CLF-C02, a diferença mais importante não é a quantidade de serviços, mas a profundidade da decisão. No CLF-C02, bastava saber que o Amazon S3 é armazenamento de objetos. No SAA-C03, você precisa saber qual classe usar, como configurar lifecycle, quando habilitar versioning, quando usar CRR, quando aplicar pre-signed URL, quando preferir um VPC endpoint e como isso impacta segurança e custo. O mesmo vale para EC2, VPC, IAM, RDS, DynamoDB e CloudFront. O exame assume que você já conhece o catálogo em alto nível; agora ele quer que você desenhe soluções coerentes.

Uma forma prática de estudar é pensar em camadas. A primeira camada é de base: IAM, VPC, EC2, S3, RDS, Route 53 e CloudWatch. A segunda é de padrões: alta disponibilidade, escalabilidade, cache, desacoplamento, streaming e observabilidade. A terceira é de decisão: escolher entre serviços que resolvem problemas parecidos, avaliando gerenciamento, elasticidade, latência, compliance e custo. Quando a prova menciona “mínimo esforço operacional”, por exemplo, ela está apontando para serviços gerenciados. Quando menciona “baixa latência global para HTTP”, CloudFront tende a entrar em cena. Quando fala em “TCP/UDP global com IP fixo”, o Global Accelerator passa a fazer mais sentido que CloudFront.

O exame também exige raciocínio sobre trade-offs. Não existe resposta perfeita. Uma arquitetura multi-region é mais resiliente, mas mais cara e mais complexa. Um banco relacional resolve consistência transacional, mas pode não escalar horizontalmente como uma base NoSQL. Uma solução serverless reduz administração, porém pode exigir cuidado com cold starts, limites de timeout e integração com VPC. O arquiteto não pergunta apenas “funciona?”. Ele pergunta “funciona dentro do requisito, com risco aceitável, custo coerente e operação sustentável?”. Esse é o modelo mental que você deve treinar desde o primeiro módulo.

Outro ponto importante é a leitura atenta do enunciado. Muitas alternativas da AWS são tecnicamente válidas, mas uma delas atende melhor ao detalhe escondido no cenário: requisito de compliance, equipe pequena, latência entre regiões, tráfego previsível, necessidade de auditoria ou orçamento restrito. Em geral, as palavras que definem a resposta correta são: “most cost-effective”, “least operational overhead”, “highly available”, “fault tolerant”, “securely”, “near real-time”, “serverless” e “with minimal changes”. Quem ignora esses qualificadores cai nas pegadinhas.

Para um cronograma realista de 8 semanas, a melhor estratégia é alternar profundidade e revisão. Estude um módulo técnico por vez, feche com questões e volte aos tópicos fracos no dia seguinte. No fim de cada semana, revise usando flashcards e cheatsheets. Na reta final, use os estudos de caso para conectar serviços e simular a forma como a prova mistura temas. O SAA-C03 raramente cobra um serviço de forma isolada; ele cobra interação entre serviços em um contexto de arquitetura.

Os serviços mais recorrentes merecem atenção desproporcional, porque aparecem em boa parte da prova: EC2, Auto Scaling, ELB, S3, VPC, IAM, RDS, Aurora, DynamoDB, Lambda, API Gateway, SQS, SNS, EventBridge, Route 53, CloudFront, CloudWatch, CloudTrail e KMS. Se você domina bem esse grupo, já cobre a maior parte das decisões cobradas. Serviços menos frequentes como Snow Family, Transfer Family, Neptune, QLDB, App Runner ou Bedrock entram mais como escolha contextual. O erro comum é gastar energia demais em serviços raros e de menos nos blocos centrais.

Em resumo, o SAA-C03 mede maturidade arquitetural em nível associate. Você não precisa ser especialista em todos os serviços, mas precisa reconhecer rapidamente qual padrão a AWS está tentando avaliar e escolher a solução mais equilibrada. Este repositório foi estruturado exatamente para isso: consolidar fundamentos, treinar leitura de cenários e transformar catálogo de serviços em decisão de arquitetura.

## 🏗️ Arquitetura e Componentes

O diagrama abaixo representa a forma como o exame costuma conectar os principais blocos de decisão arquitetural:

```text
                 +-----------------------+
                 |  Requisitos de Negócio|
                 |  HA | Segurança | Custo|
                 +-----------+-----------+
                             |
                             v
                 +-----------------------+
                 |   Decisão Arquitetural |
                 | Trade-offs e restrições|
                 +-----------+-----------+
                             |
      +----------------------+-----------------------+
      |                      |                       |
      v                      v                       v
+-------------+      +---------------+      +----------------+
| Compute     |      | Data / Storage|      | Network / Edge |
| EC2 Lambda  |      | S3 RDS Dynamo |      | VPC R53 CF ELB |
+------+------+      +-------+-------+      +--------+-------+
       |                     |                       |
       +---------------------+-----------------------+
                             |
                             v
                 +-----------------------+
                 | Segurança e Operação  |
                 | IAM KMS CW CT Config  |
                 +-----------------------+
```

## ⚙️ Configurações Importantes para o Exame

| Item | Valor / Peso | Por que importa |
|---|---:|---|
| Questões | 65 | Define o ritmo de prova e priorização de tempo |
| Tempo total | 130 minutos | Média próxima de 2 minutos por questão |
| Domínio Resilient | 30% | Maior peso, aparece em HA, DR e desacoplamento |
| Domínio High-Performing | 28% | Forte presença em EC2, armazenamento e bancos |
| Domínio Secure | 24% | IAM, KMS, VPC e auditoria aparecem bastante |
| Domínio Cost-Optimized | 18% | Menor peso formal, mas presente em quase todas as comparações |
| Região padrão para labs | us-east-1 | Maior compatibilidade e menor atrito com exemplos |
| Palavra-chave “least operational overhead” | Alta prioridade | Normalmente favorece serviços gerenciados |
| Palavra-chave “multi-Region” | Alta prioridade | Costuma eliminar respostas apenas multi-AZ |
| Palavra-chave “near real-time” | Alta prioridade | Pode indicar streaming, filas ou replicação contínua |

## 🔄 Comparativo de Serviços

| Situação de prova | Opção mais forte | Alternativa comum | Regra prática |
|---|---|---|---|
| Aplicação HTTP global com cache | CloudFront | Global Accelerator | CloudFront é edge cache e CDN; GA acelera TCP/UDP ou HTTP sem cache |
| API orientada a eventos | EventBridge | SNS | EventBridge é melhor para roteamento por regra e múltiplos produtores |
| Banco relacional com HA gerenciada | RDS Multi-AZ | EC2 self-managed DB | Para exame, a AWS quase sempre prefere o serviço gerenciado |
| Latência ultrabaixa com acesso key-value previsível | DynamoDB | RDS | Use DynamoDB quando escalabilidade horizontal é prioridade |
| Compute com administração mínima e tráfego variável | Lambda | EC2 | Lambda tende a vencer se o cenário tolera limites serverless |
| Criptografia de segredo com rotação nativa | Secrets Manager | Parameter Store | Secrets Manager é a resposta mais forte para segredos sensíveis rotacionáveis |

## 💡 Dicas e Armadilhas do Exame

- Multi-AZ não significa multi-region. Se o requisito fala em desastre regional, pense além de uma única região.
- “Mais seguro” nem sempre significa “mais complexo”. Em muitos cenários, a AWS privilegia serviço gerenciado com integração nativa a IAM e KMS.
- “Menor custo” não é equivalente a “mais barato no curto prazo”. A prova considera custo total de operação, manutenção e risco.
- Se a alternativa exige administração manual de servidores e existe opção gerenciada que atende ao requisito, a opção gerenciada costuma ganhar.
- Leia com atenção o tipo de tráfego. HTTP, gRPC, TCP, UDP, batch, stream e fila não são a mesma coisa.
- Guarde as diferenças entre alta disponibilidade, escalabilidade, fault tolerance e disaster recovery. A AWS usa esses termos com precisão.
- Se o enunciado enfatiza poucas mudanças na aplicação existente, descarte respostas que exigem refatoração grande.

## 💡 Para o Exame

- Priorize o estudo profundo de EC2, ELB, ASG, S3, IAM, VPC, RDS, DynamoDB, Lambda, SQS, Route 53, CloudFront, CloudWatch e KMS.
- Treine sempre com a pergunta mental: qual requisito está dirigindo a decisão?
- Em caso de dúvida entre duas alternativas plausíveis, escolha a que melhor equilibra segurança, disponibilidade e menor esforço operacional.
- Faça simulados só depois de consolidar os blocos centrais; simulados sem base viram memorização superficial.

## 📎 Links Relacionados

- [Questões do módulo](./questoes.md)
- [Flashcards do módulo](./flashcards.md)
- [Cheatsheet do módulo](./cheatsheet.md)
- [Casos de uso do módulo](./casos-de-uso.md)
- [Lab prático do módulo](./lab.md)
- [Links oficiais](./links.md)
- [Módulo 02: IAM e Segurança](../02-IAM-e-Seguranca/)
- [Módulo 16: Well-Architected Framework](../16-Well-Architected-Framework/)
- [Módulo 19: Simulados e Questões](../19-Simulados-e-Questoes/)
