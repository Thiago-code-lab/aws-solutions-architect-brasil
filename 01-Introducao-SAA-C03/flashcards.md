# Flashcards — Introdução SAA-C03

---
**P:** Qual domínio tem maior peso no SAA-C03?  
**R:** Design Resilient Architectures, com 30%.

---
**P:** Qual domínio tem o menor peso formal?  
**R:** Design Cost-Optimized Architectures, com 18%.

---
**P:** O SAA-C03 cobra mais catálogo de serviços ou decisão arquitetural?  
**R:** Decisão arquitetural com contexto e trade-offs.

---
**P:** O que diferencia mais o SAA-C03 do CLF-C02?  
**R:** A profundidade nas escolhas técnicas e justificativa entre alternativas plausíveis.

---
**P:** “Least operational overhead” costuma apontar para quê?  
**R:** Serviços gerenciados, se atenderem ao requisito.

---
**P:** Multi-AZ substitui Multi-Region?  
**R:** Não. Multi-AZ trata falhas de AZ; Multi-Region trata desastre regional.

---
**P:** CloudFront vs Global Accelerator em uma frase?  
**R:** CloudFront é CDN com cache; Global Accelerator acelera tráfego global sem ser cache edge.

---
**P:** RDS vs DynamoDB no raciocínio inicial?  
**R:** RDS para relacional/transacional; DynamoDB para chave-valor/documento com escala horizontal.

---
**P:** SQS vs SNS em uma frase?  
**R:** SQS desacopla com fila; SNS distribui mensagens para múltiplos assinantes.

---
**P:** Secrets Manager vs Parameter Store?  
**R:** Secrets Manager é mais forte para segredos com rotação nativa.

---
**P:** CloudTrail registra o quê?  
**R:** Chamadas de API e eventos de auditoria na conta AWS.

---
**P:** Qual bloco de serviços aparece em grande parte do exame?  
**R:** EC2, ELB, ASG, S3, VPC, IAM, RDS, DynamoDB, Lambda, Route 53 e KMS.

---
**P:** “Minimal changes to the application” indica o quê?  
**R:** Evitar alternativas que exigem refatoração grande.

---
**P:** Near real-time analytics tende a apontar para quê?  
**R:** Streaming ou processamento orientado a eventos, não batch tardio.

---
**P:** O exame prefere mais controle ou menos operação?  
**R:** Menos operação, desde que o requisito funcional e não funcional seja atendido.

---
**P:** O que a prova mede além de conhecimento técnico?  
**R:** Capacidade de ler contexto e priorizar trade-offs corretos.

---
**P:** Qual é a região padrão recomendada para labs deste repositório?  
**R:** us-east-1.

---
**P:** O que significa “cost-effective” em questão da AWS?  
**R:** Melhor equilíbrio entre custo total e requisito, não apenas menor preço bruto.

---
**P:** Qual erro comum de preparação para o SAA-C03?  
**R:** Estudar serviços raros demais e negligenciar os blocos centrais.

---
**P:** Como revisar melhor ao longo de 8 semanas?  
**R:** Alternar teoria densa, questões, revisão curta e simulados na reta final.

---
**P:** O que fazer quando duas alternativas parecem corretas?  
**R:** Releia requisito principal, restrições e palavras qualificadoras do enunciado.
