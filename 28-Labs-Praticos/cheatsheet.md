# Cheatsheet — Labs Práticos (Módulo 18)

## Comandos AWS CLI — Referência Rápida

### Lambda
```bash
# Criar função
aws lambda create-function \
  --function-name minha-funcao \
  --runtime python3.12 \
  --role arn:aws:iam::ACCOUNT:role/lambda-role \
  --handler app.lambda_handler \
  --zip-file fileb://function.zip

# Invocar (síncrono)
aws lambda invoke --function-name minha-funcao output.json

# Invocar (assíncrono)
aws lambda invoke --function-name minha-funcao \
  --invocation-type Event output.json

# Ver logs recentes
aws logs tail /aws/lambda/minha-funcao --follow
```

### S3
```bash
# Criar bucket
aws s3api create-bucket --bucket meu-bucket --region us-east-1

# Upload
aws s3 cp arquivo.txt s3://meu-bucket/

# Sync diretório
aws s3 sync ./local/ s3://meu-bucket/prefixo/

# Ativar versionamento
aws s3api put-bucket-versioning \
  --bucket meu-bucket \
  --versioning-configuration Status=Enabled

# Block public access
aws s3api put-public-access-block \
  --bucket meu-bucket \
  --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

### EC2 / Auto Scaling
```bash
# Listar instâncias em execução
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]" \
  --output table

# Criar Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name meu-asg \
  --min-size 1 --max-size 5 --desired-capacity 2 \
  --launch-template LaunchTemplateId=lt-xxx,Version='$Latest' \
  --vpc-zone-identifier "subnet-aaa,subnet-bbb"
```

### CloudWatch
```bash
# Criar alarme CPU > 80%
aws cloudwatch put-metric-alarm \
  --alarm-name "CPU-Alto" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=AutoScalingGroupName,Value=meu-asg \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:ACCOUNT:alertas

# Publicar métrica customizada
aws cloudwatch put-metric-data \
  --namespace "MinhaApp/Performance" \
  --metric-name "PedidosPorSegundo" \
  --value 42 --unit Count
```

### DynamoDB
```bash
# Criar tabela
aws dynamodb create-table \
  --table-name Pedidos \
  --attribute-definitions AttributeName=pedidoId,AttributeType=S \
  --key-schema AttributeName=pedidoId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# Inserir item
aws dynamodb put-item \
  --table-name Pedidos \
  --item '{"pedidoId": {"S": "001"}, "status": {"S": "pendente"}}'
```

---

## Terraform — Recursos Comuns

### Estrutura de Projeto
```
projeto/
  main.tf          # recursos principais
  variables.tf     # declaração de variáveis
  outputs.tf       # outputs
  terraform.tfvars # valores das variáveis
  versions.tf      # required_providers
```

### Bloco de Versões (padrão)
```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

### Recursos Mais Cobrados no Exame

| Recurso Terraform | Serviço AWS |
|------------------|-------------|
| `aws_vpc` + `aws_subnet` | VPC / Subnets |
| `aws_security_group` | Security Groups |
| `aws_instance` | EC2 |
| `aws_lambda_function` | Lambda |
| `aws_s3_bucket` + `aws_s3_bucket_versioning` | S3 |
| `aws_dynamodb_table` | DynamoDB |
| `aws_db_instance` | RDS |
| `aws_elasticache_cluster` | ElastiCache |
| `aws_ecs_cluster` + `aws_ecs_service` | ECS |
| `aws_iam_role` + `aws_iam_role_policy_attachment` | IAM |
| `aws_cloudwatch_metric_alarm` | CloudWatch |

---

## Terraform — Comandos Essenciais

| Comando | O que faz |
|---------|-----------|
| `terraform init` | Inicializa providers, baixa plugins |
| `terraform validate` | Valida sintaxe HCL |
| `terraform plan` | Preview de mudanças sem aplicar |
| `terraform plan -out=tfplan` | Salva plano para aplicar depois |
| `terraform apply` | Aplica mudanças (pede confirmação) |
| `terraform apply -auto-approve` | Aplica sem confirmação |
| `terraform apply tfplan` | Aplica plano salvo |
| `terraform destroy` | Destrói toda infraestrutura |
| `terraform state list` | Lista recursos no state |
| `terraform state show aws_instance.web` | Detalhes de recurso específico |
| `terraform output` | Exibe outputs definidos |
| `terraform fmt` | Formata arquivos HCL |
| `terraform graph` | Gera grafo de dependências |

---

## SAM — Comandos Essenciais

```bash
# Inicializar projeto SAM
sam init

# Build local
sam build

# Testar localmente
sam local invoke FunctionName
sam local start-api  # API GW local na porta 3000

# Deploy (guiado)
sam deploy --guided

# Deploy com parâmetros salvos
sam deploy

# Ver logs
sam logs -n FunctionName --stack-name minha-stack --tail

# Deletar stack
sam delete --stack-name minha-stack
```

---

## Debug — Checklist por Problema

### Lambda não consegue acessar RDS
```
1. Lambda e RDS na mesma VPC? (Lambda precisa de VpcConfig)
2. Security Group da Lambda permite saída para porta 5432/3306?
3. Security Group do RDS permite entrada do SG da Lambda?
4. Lambda tem subnets privadas (não públicas)?
5. Subnets privadas têm NAT Gateway para internet (se precisar)?
6. IAM Role da Lambda tem permissão básica (AWSLambdaVPCAccessExecutionRole)?
7. Timeout da Lambda é suficiente (conexão DB pode demorar)?
```

### S3 retornando 403 Forbidden
```
1. Bucket Policy está bloqueando? (verificar aws s3api get-bucket-policy)
2. Block Public Access ativo e ACL pública tentando ser usada?
3. IAM Policy do usuário/role tem s3:GetObject?
4. Objeto existe? (aws s3 ls s3://bucket/chave)
5. Bucket em outra região que o cliente espera?
6. KMS key: usuário tem permissão de decrypt na chave?
7. VPC Endpoint policy bloqueando? (se acesso via VPC Gateway Endpoint)
```

### ECS Task não inicia (STOPPED)
```
1. Ver stoppedReason: aws ecs describe-tasks --cluster X --tasks Y
2. Motivos comuns:
   - CannotPullContainerError: ECR privado sem IAM ou VPC endpoint
   - OutOfMemory: memória insuficiente definida na task definition
   - ResourceInitializationError: exec role faltando
3. CloudWatch Logs: /ecs/<task-family> — ver erros da aplicação
4. Task Role tem permissões necessárias (ex: s3:GetObject)?
5. Security Group do ECS permite saída para RDS/serviços externos?
```

---

## Ordem de Limpeza no Terraform (`terraform destroy`)

Dependências que podem causar erro se apagadas na ordem errada:
```
1. ECS Services (antes do Cluster)
2. ALB / Target Groups (antes das subnets/SGs)
3. RDS / ElastiCache (antes da subnet group e SGs)
4. Lambda (pode ter ENIs em VPC)
5. NAT Gateway (antes do EIP pode ser liberado)
6. Subnets (antes da VPC)
7. VPC (por último — tudo dentro dela deve ser removido antes)
```

> Geralmente `terraform destroy` resolve em ordem por dependências. Problemas ocorrem com recursos criados fora do Terraform.

---

## IAM — Simulate Policy (debug de permissões)

```bash
# Simular se ação é permitida
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT:role/minha-role \
  --action-names s3:PutObject \
  --resource-arns arn:aws:s3:::meu-bucket/*
  
# Resultado esperado: "EvalDecision": "allowed"
```

---

## Estimativa de Custo — Lab Típico

| Recurso | Custo Estimado (lab 2h) |
|---------|------------------------|
| EC2 t3.micro | ~$0.01 |
| NAT Gateway | ~$0.09/h + $0.045/GB |
| RDS db.t3.micro Multi-AZ | ~$0.06/h |
| Lambda (1M req) | Free Tier |
| S3 (< 5GB) | Free Tier |
| ECS Fargate (0.25vCPU/0.5GB) | ~$0.01/h |

> **Dica:** Sempre destruir recursos ao final. NAT Gateway é cobrado por hora mesmo sem tráfego.

---

## Dicas de Prova

| Pista na Questão | Resposta Esperada |
|-----------------|------------------|
| "testar Lambda localmente antes de deploy" | SAM local invoke / sam local start-api |
| "infraestrutura como código, reproductível" | Terraform ou CloudFormation |
| "Lambda precisa de RDS em VPC privada" | Lambda VpcConfig + SG do RDS liberado |
| "visualizar mudanças antes de aplicar" | terraform plan |
| "Lambda não acessa internet em VPC" | Falta NAT Gateway na subnet privada |
| "armazenar segredos para Lambda acessar" | Secrets Manager (não variáveis de ambiente) |
| "deploy serverless com YAML simplificado" | SAM (template.yaml) |
| "estado de infra compartilhado entre equipes" | Terraform state em S3 + DynamoDB lock |
| "debug erro 403 S3" | Checar bucket policy + Block Public Access + IAM |
| "ECS task para logo após iniciar" | Ver stoppedReason + CloudWatch Logs /ecs/ |
