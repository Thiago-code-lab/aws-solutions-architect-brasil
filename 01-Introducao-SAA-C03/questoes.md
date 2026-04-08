# Questões — Introdução SAA-C03

## Questão 1
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Fácil

Uma analista está começando a estudar para o SAA-C03 e quer priorizar o domínio com maior peso para montar as primeiras revisões. Qual domínio deve receber a maior atenção inicial?

- A) Design Secure Applications and Architectures
- B) Design Cost-Optimized Architectures
- C) Design Resilient Architectures
- D) Design High-Performing Architectures

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Design Resilient Architectures representa 30% do exame, o maior peso entre os quatro domínios. Por isso, costuma ser um bom ponto de partida para revisão estratégica. A alternativa D é próxima, mas High-Performing tem 28%. A alternativa A tem 24% e a B 18%.

**Conceito-chave:** pesos oficiais dos domínios do SAA-C03
</details>

## Questão 2
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Uma equipe acostumada com o CLF-C02 está migrando seu plano de estudos para o SAA-C03. O gerente técnico quer entender a principal diferença de profundidade entre os exames. Qual afirmação descreve melhor essa diferença?

- A) O SAA-C03 cobra apenas nomes de serviços, enquanto o CLF-C02 cobra cenários
- B) O SAA-C03 exige decisões arquiteturais com trade-offs, enquanto o CLF-C02 é mais introdutório
- C) O SAA-C03 não cobre segurança, apenas computação e redes
- D) O CLF-C02 e o SAA-C03 têm exatamente o mesmo foco, mudando apenas o idioma

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

O SAA-C03 exige escolha contextual entre serviços e padrões arquiteturais, considerando disponibilidade, segurança, desempenho e custo. O CLF-C02 trabalha mais no reconhecimento conceitual dos serviços. A alternativa A inverte os papéis. A alternativa C é falsa porque segurança tem peso relevante. A alternativa D ignora a diferença real de profundidade.

**Conceito-chave:** diferença entre prova introdutória e prova associate
</details>

## Questão 3
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Uma startup tem equipe enxuta e precisa lançar uma API com o menor esforço operacional possível. Durante uma revisão para o SAA-C03, um arquiteto usa esse cenário para explicar como a prova pensa. Qual abordagem é mais alinhada com a lógica da AWS em questões do exame?

- A) Priorizar sempre EC2 para ter controle total do ambiente
- B) Priorizar serviços gerenciados quando eles atendem ao requisito com menos administração
- C) Evitar qualquer serviço serverless por causa de limites de timeout
- D) Escolher a opção com maior performance teórica, independentemente do custo

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Em cenários que pedem menor esforço operacional, a AWS normalmente favorece serviços gerenciados como Lambda, API Gateway, RDS, DynamoDB, S3 e ECS Fargate, desde que atendam aos requisitos técnicos. A alternativa A aumenta a carga operacional. A alternativa C generaliza um limite real, mas fora de contexto. A alternativa D ignora o equilíbrio entre custo e necessidade.

**Conceito-chave:** least operational overhead
</details>

## Questão 4
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Fácil

Qual conjunto representa melhor os serviços com maior recorrência no SAA-C03 e que, por isso, deveriam receber estudo aprofundado nas primeiras semanas?

- A) Snowmobile, Braket, Ground Station e GameLift
- B) EC2, S3, VPC, IAM, RDS, DynamoDB, Lambda e Route 53
- C) Device Farm, CodeArtifact, Honeycode e WorkSpaces
- D) AWS Wickr, IQ, Managed Blockchain e Panorama

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Esses serviços aparecem com alta frequência porque formam o núcleo das decisões arquiteturais cobradas. As demais alternativas listam serviços que podem aparecer pontualmente, mas não representam o centro do exame.

**Conceito-chave:** priorização de estudo por incidência
</details>

## Questão 5
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Durante um grupo de estudos, um aluno afirma que uma arquitetura Multi-AZ já resolve qualquer requisito de continuidade de negócio. Qual resposta corrige melhor essa afirmação?

- A) Multi-AZ cobre falha de zona, mas não necessariamente um desastre regional
- B) Multi-AZ é equivalente a Multi-Region
- C) Multi-AZ só se aplica a aplicações serverless
- D) Multi-AZ elimina a necessidade de backup

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: A**

Multi-AZ eleva disponibilidade dentro de uma região, protegendo contra falhas de zona de disponibilidade. No entanto, um evento regional ainda pode exigir estratégia multi-region. A alternativa B confunde conceitos. A alternativa C é falsa. A alternativa D ignora que backup e DR continuam necessários.

**Conceito-chave:** diferença entre HA regional e DR multi-region
</details>

## Questão 6
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Uma questão do exame descreve uma aplicação global HTTP que precisa reduzir latência para usuários finais e também cachear conteúdo estático em edge locations. Qual serviço tende a ser a melhor resposta?

- A) AWS Global Accelerator
- B) Amazon CloudFront
- C) AWS Direct Connect
- D) Amazon EventBridge

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

CloudFront é a CDN da AWS e oferece cache em edge locations, sendo a escolha mais adequada para conteúdo HTTP com distribuição global. Global Accelerator melhora roteamento global para aplicações TCP/UDP e também HTTP, mas não é um cache edge. Direct Connect é conectividade privada. EventBridge não se aplica.

**Conceito-chave:** CloudFront vs Global Accelerator
</details>

## Questão 7
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Um cenário pede rotação automática de credenciais de banco de dados e integração nativa com KMS. Qual serviço costuma ser mais alinhado ao enunciado?

- A) AWS Secrets Manager
- B) AWS Systems Manager Parameter Store Standard
- C) Amazon S3 Versioning
- D) Amazon CloudWatch Logs

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: A**

Secrets Manager é a escolha mais forte quando o requisito inclui segredos sensíveis e rotação automática. Parameter Store pode armazenar segredos, mas a rotação não é o seu diferencial nativo principal. S3 Versioning e CloudWatch Logs não resolvem esse problema.

**Conceito-chave:** Secrets Manager vs Parameter Store
</details>

## Questão 8
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Médio

Ao ler uma questão que destaca orçamento apertado e carga imprevisível, qual sinal do enunciado mais sugere avaliar tecnologias serverless ou elásticas por demanda?

- A) Requisito de uso de hardware dedicado
- B) Requisito de baixa mudança organizacional
- C) Requisito de cobrança compatível com consumo e baixa ociosidade
- D) Requisito de licenciamento perpétuo on-premises

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Quando o problema enfatiza consumo variável e redução de ociosidade, serviços com cobrança sob demanda e elasticidade nativa passam a ser mais aderentes. Hardware dedicado vai na direção oposta. Baixa mudança organizacional pode favorecer lift-and-shift, mas não responde à otimização de consumo. Licenciamento on-premises não se aplica.

**Conceito-chave:** custo orientado a consumo
</details>

## Questão 9
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Difícil

Uma empresa precisa armazenar dados altamente relacionais com transações ACID e leitura em escala para usuários distribuídos. Em uma questão introdutória de arquitetura, qual combinação mostra melhor o tipo de raciocínio esperado pelo exame?

- A) Substituir tudo por DynamoDB porque sempre escala melhor
- B) Usar Amazon RDS ou Aurora para consistência transacional e avaliar read replicas para leitura
- C) Usar Amazon S3 como banco transacional principal
- D) Usar Amazon CloudFront para replicação de banco

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

O exame recompensa a escolha de um banco relacional gerenciado quando o workload pede transação e modelo relacional, complementando com estratégias de escala de leitura quando apropriado. A alternativa A simplifica demais e ignora requisitos. A alternativa C não oferece semântica transacional relacional. A alternativa D não é solução de banco de dados.

**Conceito-chave:** alinhar requisito de dados ao serviço correto
</details>

## Questão 10
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Médio

Em uma questão de arquitetura, a empresa quer reduzir acoplamento entre produtores e consumidores, absorver picos e reprocessar mensagens em caso de falha. Qual serviço costuma aparecer como resposta central?

- A) Amazon SQS
- B) Amazon Route 53
- C) Amazon Inspector
- D) AWS Artifact

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: A**

SQS é um mecanismo clássico de desacoplamento com buffer, absorção de burst e reprocessamento por meio de visibilidade, retenção e DLQ. Route 53 é DNS. Inspector faz análise de vulnerabilidade. Artifact é acesso a relatórios de compliance.

**Conceito-chave:** padrão de desacoplamento com fila
</details>

## Questão 11
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Difícil

Uma questão afirma que uma empresa quer restringir permissões máximas que equipes de desenvolvimento podem delegar, mesmo quando administradores locais criam políticas adicionais. Qual conceito deve vir à mente?

- A) Amazon CloudFront signed cookies
- B) Permission Boundaries
- C) S3 Multipart Upload
- D) RDS Read Replicas

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: B**

Permission Boundaries definem o teto efetivo de permissões que uma identidade IAM pode receber. Isso é um conceito clássico de governança e costuma surgir em cenários com delegação controlada. As demais opções tratam de problemas totalmente diferentes.

**Conceito-chave:** governança de permissões em IAM
</details>

## Questão 12
**Domínio:** Design Cost-Optimized Architectures  
**Dificuldade:** Fácil

Uma equipe está entre duas alternativas válidas. Ambas atendem ao requisito técnico, mas uma delas elimina manutenção de sistema operacional, patching e capacidade ociosa. Na lógica do exame, qual tende a ser preferida?

- A) A alternativa com maior esforço operacional
- B) A alternativa com mais componentes manuais
- C) A alternativa gerenciada, se o requisito funcional for atendido
- D) A alternativa com hardware dedicado por padrão

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Esse é um padrão recorrente no SAA-C03. Se a solução gerenciada atende ao cenário sem comprometer requisitos críticos, ela tende a ser preferida por reduzir operação e risco. As demais aumentam custo ou complexidade sem necessidade clara.

**Conceito-chave:** preferência por managed services no exame
</details>

## Questão 13
**Domínio:** Design High-Performing Architectures  
**Dificuldade:** Médio

Qual interpretação é mais correta quando uma questão menciona “near real-time analytics” sobre eventos contínuos produzidos por aplicações?

- A) Pode haver necessidade de streaming, como Kinesis, em vez de processamento batch isolado
- B) Sempre deve ser usado um data warehouse tradicional com carga semanal
- C) O enunciado aponta diretamente para S3 Glacier Deep Archive
- D) O requisito exclui qualquer serviço gerenciado

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: A**

“Near real-time” é um indicador de que o dado precisa fluir e ser processado com latência baixa, o que muitas vezes aponta para soluções de streaming ou event-driven. Batch semanal não atende. Glacier é arquivamento. Serviço gerenciado não é excluído.

**Conceito-chave:** palavras-chave do enunciado
</details>

## Questão 14
**Domínio:** Design Secure Applications and Architectures  
**Dificuldade:** Médio

Uma empresa quer registrar chamadas de API feitas na conta AWS para fins de auditoria e investigação. Qual serviço deve ser lembrado primeiro?

- A) Amazon CloudTrail
- B) Amazon CloudWatch Synthetics
- C) AWS Budgets
- D) Amazon Macie

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: A**

CloudTrail registra eventos de API e é central para auditoria. CloudWatch Synthetics testa endpoints. Budgets monitora custo. Macie analisa dados sensíveis em S3. Todos podem aparecer no exame, mas apenas CloudTrail atende diretamente ao requisito descrito.

**Conceito-chave:** auditoria de ações na conta AWS
</details>

## Questão 15
**Domínio:** Design Resilient Architectures  
**Dificuldade:** Difícil

Um candidato está revisando uma questão em que todas as opções funcionam, mas apenas uma atende ao requisito com a melhor combinação de resiliência, simplicidade operacional e custo. Qual abordagem mental é a mais adequada para resolver esse tipo de problema?

- A) Escolher sempre a arquitetura mais complexa, pois tende a ser mais robusta
- B) Escolher a opção mais barata isoladamente, sem considerar operação
- C) Avaliar requisito principal, restrições e trade-offs antes de decidir
- D) Ignorar palavras como “securely”, “cost-effective” e “minimal changes” porque confundem

<details>
<summary>✅ Resposta e Explicação</summary>

**Resposta correta: C**

Esse é o núcleo do SAA-C03. A prova mede a capacidade de avaliar trade-offs diante de requisitos explícitos e implícitos. Complexidade excessiva não é prêmio automático. Menor custo isolado pode gerar risco ou operação inviável. As palavras do enunciado são decisivas e nunca devem ser ignoradas.

**Conceito-chave:** raciocínio arquitetural orientado a trade-offs
</details>
