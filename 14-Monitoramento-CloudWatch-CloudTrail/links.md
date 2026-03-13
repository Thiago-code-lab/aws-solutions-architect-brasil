# Links e Recursos — Monitoramento: CloudWatch e CloudTrail (Módulo 14)

## Documentação Oficial AWS

- [Amazon CloudWatch — Documentação Oficial](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [CloudWatch Logs — Documentação](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
- [CloudWatch Logs Insights — Sintaxe de Queries](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [CloudWatch Alarms — Configuração](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [CloudWatch Metrics — Namespaces e Métricas disponíveis](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html)
- [AWS CloudTrail — Documentação](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [CloudTrail — O que é logado](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html)
- [AWS Config — Documentação](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
- [AWS Config — Managed Rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
- [Amazon EventBridge — Documentação](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [AWS X-Ray — Documentação](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html)

## FAQs

- [Amazon CloudWatch FAQ](https://aws.amazon.com/cloudwatch/faqs/)
- [AWS CloudTrail FAQ](https://aws.amazon.com/cloudtrail/faqs/)
- [AWS Config FAQ](https://aws.amazon.com/config/faqs/)

## Whitepapers

- [AWS Security Logging and Monitoring Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/detection.html)
- [Security Pillar — Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [AWS Compliance Framework e LGPD/GDPR — Logging requirements](https://aws.amazon.com/compliance/gdpr-center/)

## Artigos de Blog AWS

- [Operational visibility with Amazon CloudWatch](https://aws.amazon.com/blogs/mt/operational-visibility-with-amazon-cloudwatch/)
- [Using CloudWatch Logs Insights for advanced log analysis](https://aws.amazon.com/blogs/mt/using-amazon-cloudwatch-logs-insights-for-advanced-log-analysis/)
- [Automate security response with AWS Config and Lambda](https://aws.amazon.com/blogs/security/how-to-auto-remediate-internet-accessible-ports-with-aws-config-and-aws-security-hub/)
- [Best practices for AWS CloudTrail](https://aws.amazon.com/blogs/mt/aws-cloudtrail-best-practices/)

## Vídeos re:Invent

- [Centralized observability with CloudWatch (COP323)](https://www.youtube.com/results?search_query=aws+reinvent+cloudwatch+centralized+observability)
- [Threat detection with GuardDuty and CloudTrail](https://www.youtube.com/results?search_query=aws+reinvent+guardduty+cloudtrail+threat+detection)
- [AWS X-Ray distributed tracing in practice](https://www.youtube.com/results?search_query=aws+reinvent+x-ray+distributed+tracing)

## Ferramentas de Estudo

- [Tutorials Dojo — CloudWatch Cheat Sheet](https://tutorialsdojo.com/amazon-cloudwatch/)
- [Tutorials Dojo — CloudTrail Cheat Sheet](https://tutorialsdojo.com/aws-cloudtrail/)
- [Tutorials Dojo — AWS Config Cheat Sheet](https://tutorialsdojo.com/aws-config/)
- [AWS Skill Builder — Monitoring, Logging, and Remediation](https://explore.skillbuilder.aws/learn/course/external/view/elearning/1499/aws-cloud-operations)

## Comparativo de Serviços de Monitoramento

| Serviço | O que monitora | Retenção padrão |
|---------|---------------|-----------------|
| CloudWatch Metrics | Métricas de recursos AWS | 15 meses |
| CloudWatch Logs | Logs de aplicação/serviços | Configurável (padrão: never expire) |
| CloudTrail | Chamadas de API (who did what) | 90 dias (management events grátis) |
| Config | Estado de configuração dos recursos | 7 anos (configurável) |
| GuardDuty | Ameaças de segurança (VPC Flow, DNS, CloudTrail) | 90 dias |
| X-Ray | Rastreamento de requests distribuídos | 30 dias |
