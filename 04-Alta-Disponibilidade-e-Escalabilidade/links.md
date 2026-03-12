# Links de Referência — Alta Disponibilidade e Escalabilidade

## Documentação Oficial AWS

### Elastic Load Balancing
- [ELB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/) — Visão geral e comparativo de tipos
- [ALB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/) — Application Load Balancer
- [NLB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/) — Network Load Balancer
- [GWLB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/) — Gateway Load Balancer
- [CLB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/) — Classic (legado)

### ALB — Roteamento e Recursos
- [ALB Listener Rules](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html) — Condições e ações
- [Weighted Target Groups (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) — Canary/Blue-Green
- [ALB Sticky Sessions](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/sticky-sessions.html) — Duration e Application-based
- [ALB Lambda Targets](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/lambda-functions.html) — Usar Lambda como target

### NLB
- [NLB Static IP Addresses](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html) — IP estático e Elastic IP
- [NLB with PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/create-endpoint-service.html) — VPC Endpoint Service

### GWLB
- [GWLB and Appliances](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html) — GENEVE e appliances
- [VPC Ingress Routing with GWLB](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html#ingress-routing) — Roteamento de entrada com GWLB

### Auto Scaling
- [EC2 Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/ec2/userguide/) — Guia completo
- [Launch Templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html) — Substituição dos Launch Configurations
- [Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html) — Target Tracking, Step, Simple
- [Predictive Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-predictive-scaling.html) — ML-based scaling
- [Scheduled Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/schedule_time.html)
- [Lifecycle Hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html) — Pending:Wait e Terminating:Wait
- [Termination Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-termination-policies.html)
- [Mixed Instance Policy](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-mixed-instances-groups.html) — Spot + On-Demand mix
- [Health Check Types](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-health-checks.html) — EC2 vs ELB vs Custom

---

## Whitepapers

- [Reliability Pillar — Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/) — Pilar de confiabilidade
- [Performance Efficiency Pillar](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/) — Scaling e elasticidade
- [AWS Architecture Center — High Availability](https://aws.amazon.com/architecture/high-availability/) — Patterns de HA

---

## AWS Blog Posts Técnicos

- [Building a Scalable and Highly Available Application](https://aws.amazon.com/blogs/architecture/) — Blog AWS Architecture
- [Introducing GWLB](https://aws.amazon.com/blogs/networking-and-content-delivery/introducing-aws-gateway-load-balancer/) — Gateway Load Balancer launch blog
- [Blue/Green Deployments with ALB Weighted Target Groups](https://aws.amazon.com/blogs/devops/use-aws-codedeploy-to-implement-blue-green-deployments-for-aws-lambda-functions/) — Deploy sem downtime

---

## FAQs para o Exame

- [Elastic Load Balancing FAQ](https://aws.amazon.com/elasticloadbalancing/faqs/)
- [EC2 Auto Scaling FAQ](https://aws.amazon.com/ec2/autoscaling/faqs/)

---

## Ferramentas Úteis

- [ELB Access Logs](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html) — Para debug de roteamento
- [ASG Activity History](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-verify-scaling-activity.html) — Ver histórico de scaling

---

## Conceitos SAA-C03 Chave Neste Módulo

| Conceito | Frequência no Exame |
|---|---|
| ALB vs NLB: quando usar cada (IP estático → NLB) | Alta |
| GWLB para inserção de appliances de segurança | Alta |
| ALB weighted target groups para canary/blue-green | Alta |
| ELB health check no ASG (não apenas EC2 health) | Alta |
| Target Tracking cria alarmes automaticamente | Alta |
| Lifecycle Hooks para inicialização/drenagem custom | Alta |
| Cross-zone LB: padrão/custo por tipo de ELB | Média |
| Launch Template > Launch Configuration | Média |
| PrivateLink exige NLB como backend | Alta |
| Predictive Scaling requer 14 dias de histórico | Média |
