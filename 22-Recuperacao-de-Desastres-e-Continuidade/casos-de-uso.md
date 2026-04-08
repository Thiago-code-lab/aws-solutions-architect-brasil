# Casos de Uso - Recuperacao de Desastres e Continuidade

## Caso 1: E-commerce com pico sazonal

Cenario:
- 99.95% de disponibilidade
- RTO menor que 15 minutos
- RPO menor que 5 minutos

Arquitetura recomendada:
- Warm Standby em segunda regiao
- Aurora Global Database
- S3 com CRR
- Route 53 failover com health checks

Por que funciona:
- atende RTO/RPO baixos sem custo extremo de active/active

## Caso 2: ERP interno com baixo orcamento

Cenario:
- pode ficar ate 4 horas indisponivel
- aceita perda de ate 1 hora de dados

Arquitetura recomendada:
- Backup and Restore
- AWS Backup com politicas diarias e retencao
- snapshots EBS e RDS cross-region

Por que funciona:
- menor custo para requisito de negocio menos restritivo

## Caso 3: Plataforma de pagamentos

Cenario:
- indisponibilidade quase zero
- compliance e trilha de auditoria

Arquitetura recomendada:
- Active/Active multi-region
- DynamoDB Global Tables
- API Gateway + Lambda em duas regioes
- Route 53 latency/failover

Por que funciona:
- reduz impacto de falha regional e melhora latencia global

## Caso 4: Migracao de datacenter para AWS

Cenario:
- recuperar rapidamente VMs legadas
- pouca mudanca na aplicacao

Arquitetura recomendada:
- AWS Elastic Disaster Recovery
- replicacao continua on-premises -> AWS
- runbook de cutover e failback

Por que funciona:
- acelera estrategia de DR sem refatoracao imediata