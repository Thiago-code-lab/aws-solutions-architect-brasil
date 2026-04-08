# Links e Recursos — Serverless: Lambda e API Gateway (Módulo 11)

## Documentação Oficial AWS

- [AWS Lambda — Documentação Oficial](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [AWS Lambda — Guia do Desenvolvedor (quotas e limites)](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [Amazon API Gateway — Documentação Oficial](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [API Gateway — HTTP API vs REST API vs WebSocket](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html)
- [Lambda — Configuração de VPC](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html)
- [Lambda — Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)
- [Lambda — Destinations (êxito e falha)](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html#invocation-async-destinations)

## FAQs

- [AWS Lambda FAQ](https://aws.amazon.com/lambda/faqs/)
- [Amazon API Gateway FAQ](https://aws.amazon.com/api-gateway/faqs/)

## Whitepapers e Guias

- [Serverless Application Lens — Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)
- [Serverless Architectures with AWS Lambda (Whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/serverless-architectures-lambda/serverless-architectures-lambda.html)
- [Best Practices for Serverless Security](https://aws.amazon.com/blogs/compute/best-practices-for-security-of-lambda-based-serverless-applications/)

## Artigos de Blog AWS

- [Choosing the right API Gateway endpoint type](https://aws.amazon.com/blogs/compute/choosing-the-right-api-gateway-endpoint-type/)
- [Lambda power tuning com AWS Lambda Power Tuning](https://aws.amazon.com/blogs/aws/new-app-for-aws-lambda-power-tuning/)
- [Implementing the serverless 3-musketeers pattern](https://aws.amazon.com/blogs/compute/implementing-the-serverless-3-musketeers-pattern/)

## Vídeos re:Invent

- [Best Practices for AWS Lambda (SVS401)](https://www.youtube.com/results?search_query=aws+reinvent+lambda+best+practices+SVS401)
- [Optimizing Lambda Performance (SVS340)](https://www.youtube.com/results?search_query=aws+reinvent+optimizing+lambda+performance)
- [Building APIs with Amazon API Gateway (SVS215)](https://www.youtube.com/results?search_query=aws+reinvent+api+gateway+building+apis)

## Ferramentas de Estudo

- [Tutorials Dojo — Lambda & API GW Cheat Sheet](https://tutorialsdojo.com/aws-lambda/)
- [AWS Skill Builder — Developing Serverless Solutions on AWS](https://explore.skillbuilder.aws/learn/course/external/view/elearning/37/developing-serverless-solutions-on-aws)

## Números-Chave para o Exame

| Serviço | Limite |
|---------|--------|
| Lambda timeout máximo | 15 minutos |
| Lambda memória máxima | 10.240 MB |
| Lambda payload síncrono | 6 MB |
| Lambda payload assíncrono | 256 KB |
| API GW payload máximo | 10 MB |
| Lambda /tmp storage | 512 MB (padrão) / 10 GB (máx) |
| Lambda conc. por conta | 1.000 (padrão, ajustável) |
