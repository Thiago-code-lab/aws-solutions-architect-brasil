# Flashcards — Módulo 18: Labs Práticos

> **Formato:** P = Pergunta | R = Resposta  
> **Total:** 25 flashcards

---

**P:** Como criar uma Lambda Function com permissão para DynamoDB via CLI?  
**R:**
```bash
# 1. Criar IAM Role
aws iam create-role --role-name LambdaDynamoRole \
  --assume-role-policy-document file://trust-lambda.json

# 2. Attach policy
aws iam attach-role-policy --role-name LambdaDynamoRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess

# 3. Deploy Lambda
aws lambda create-function --function-name MyFunc \
  --runtime python3.12 --role arn:aws:iam::ACCOUNT:role/LambdaDynamoRole \
  --handler lambda_function.lambda_handler --zip-file fileb://function.zip
```

---

**P:** Como criar uma VPC com subnet pública e privada via Terraform?  
**R:**
```hcl
resource "aws_vpc" "main" { cidr_block = "10.0.0.0/16" }
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id; cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true; availability_zone = "us-east-1a"
}
resource "aws_subnet" "private" {
  vpc_id = aws_vpc.main.id; cidr_block = "10.0.2.0/24"
  availability_zone = "us-east-1b"
}
resource "aws_internet_gateway" "igw" { vpc_id = aws_vpc.main.id }
```

---

**P:** Como verificar se uma EC2 em subnet privada tem conectividade com a internet?  
**R:** A instância privada precisa de um **NAT Gateway** na subnet pública com rota na subnet privada: `0.0.0.0/0 → NAT GW`. Para testar: `curl -I https://aws.amazon.com` (dentro da instância via SSM Session Manager). Sem NAT GW: timeout. Com NAT GW + rota correta: HTTP 200.

---

**P:** Qual é o comando AWS CLI para fazer upload de arquivo para S3 com criptografia SSE-KMS?  
**R:**
```bash
aws s3 cp arquivo.txt s3://meu-bucket/arquivo.txt \
  --sse aws:kms \
  --sse-kms-key-id arn:aws:kms:us-east-1:123456:key/key-id
```
Verificar:
```bash
aws s3api head-object --bucket meu-bucket --key arquivo.txt \
  --query 'ServerSideEncryption'
```

---

**P:** Como testar uma API Gateway + Lambda localmente antes do deploy?  
**R:**
```bash
# Instalar SAM CLI
pip install aws-sam-cli

# Invocar Lambda localmente
sam local invoke "NomeFuncao" -e events/event.json

# Subir API local (porta 3000)
sam local start-api

# Testar
curl -X POST http://localhost:3000/endpoint \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

---

**P:** Qual comando Terraform visualiza as mudanças antes de aplicar?  
**R:**
```bash
terraform plan -out=tfplan    # mostra mudanças + salva plano
terraform show tfplan         # visualiza o plano salvo
terraform apply tfplan        # aplica exatamente o plano salvo
```
O `-out=tfplan` garante que `apply` aplica exatamente o que foi revisado no `plan`, sem surpresas de mudanças no estado entre plan e apply.

---

**P:** Como configurar o Auto Scaling Group para escalar baseado em CPU via CLI?  
**R:**
```bash
# Criar scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name MyASG \
  --policy-name cpu-scale-out \
  --scaling-adjustment 1 \
  --adjustment-type ChangeInCapacity

# Criar CloudWatch Alarm que aciona a policy
aws cloudwatch put-metric-alarm \
  --alarm-name High-CPU \
  --metric-name CPUUtilization --namespace AWS/EC2 \
  --threshold 70 --comparison-operator GreaterThanThreshold \
  --statistic Average --period 300 --evaluation-periods 2 \
  --alarm-actions arn:aws:autoscaling:...policy/cpu-scale-out
```

---

**P:** Como habilitar versionamento no S3 e listar versões de um objeto?  
**R:**
```bash
# Habilitar versionamento
aws s3api put-bucket-versioning \
  --bucket meu-bucket \
  --versioning-configuration Status=Enabled

# Listar versões de um objeto
aws s3api list-object-versions \
  --bucket meu-bucket --prefix arquivo.txt \
  --query 'Versions[*].[VersionId,LastModified,IsLatest]'

# Restaurar versão específica
aws s3api copy-object --bucket meu-bucket \
  --copy-source "meu-bucket/arquivo.txt?versionId=abc123" \
  --key arquivo.txt
```

---

**P:** Como criar uma RDS instance Multi-AZ com Terraform?  
**R:**
```hcl
resource "aws_db_instance" "main" {
  identifier        = "prod-db"
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.medium"
  allocated_storage = 20
  username          = "admin"
  password          = var.db_password
  multi_az          = true
  skip_final_snapshot = false
  final_snapshot_identifier = "prod-db-final"
  backup_retention_period = 7
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
}
```

---

**P:** Como verificar os logs de uma função Lambda no CloudWatch via CLI?  
**R:**
```bash
# Listar log streams (invocações)
aws logs describe-log-streams \
  --log-group-name /aws/lambda/MinhaFuncao \
  --order-by LastEventTime --descending --max-items 5

# Ver logs da última invocação
aws logs get-log-events \
  --log-group-name /aws/lambda/MinhaFuncao \
  --log-stream-name "2024/01/01/[$LATEST]abc123" \
  --query 'events[*].message'
```

---

**P:** Como criar um DynamoDB table com GSI via Terraform?  
**R:**
```hcl
resource "aws_dynamodb_table" "orders" {
  name         = "Orders"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "orderId"
  range_key    = "createdAt"
  attribute { name = "orderId"   type = "S" }
  attribute { name = "createdAt" type = "S" }
  attribute { name = "userId"    type = "S" }

  global_secondary_index {
    name            = "UserIdIndex"
    hash_key        = "userId"
    range_key       = "createdAt"
    projection_type = "ALL"
  }
}
```

---

**P:** Como configurar CORS no API Gateway via Terraform?  
**R:**
```hcl
resource "aws_api_gateway_method" "options" {
  rest_api_id   = aws_api_gateway_rest_api.api.id
  resource_id   = aws_api_gateway_resource.resource.id
  http_method   = "OPTIONS"
  authorization = "NONE"
}
resource "aws_api_gateway_integration" "options" {
  http_method = aws_api_gateway_method.options.http_method
  type        = "MOCK"
  request_templates = { "application/json" = "{\"statusCode\": 200}" }
}
# + method_response e integration_response com headers CORS
```

---

**P:** Como criar um CloudWatch Alarm para monitorar erros da Lambda e enviar para SNS?  
**R:**
```bash
# Criar tópico SNS
aws sns create-topic --name lambda-errors-alert

# Criar alarme
aws cloudwatch put-metric-alarm \
  --alarm-name "LambdaErrors-MinhaFuncao" \
  --metric-name Errors --namespace AWS/Lambda \
  --dimensions Name=FunctionName,Value=MinhaFuncao \
  --statistic Sum --period 60 \
  --threshold 5 --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:ACCOUNT:lambda-errors-alert
```

---

**P:** Como fazer limpeza (cleanup) de recursos Terraform para evitar custos?  
**R:**
```bash
# Destruir todos os recursos do state
terraform destroy

# Verificar o que será destruído antes
terraform plan -destroy

# Forçar destruição de bucket S3 com objetos (force_destroy=true no .tf)
# ou via CLI:
aws s3 rm s3://meu-bucket --recursive
aws s3api delete-bucket --bucket meu-bucket

# Verificar recursos remanescentes na conta
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Environment,Values=lab
```

---

**P:** Como diagnosticar por que uma Lambda não consegue acessar uma instância RDS em VPC?  
**R:** Checklist: (1) Lambda tem VPC config com subnet privada? (2) Security Group do Lambda tem outbound na porta 3306? (3) Security Group RDS tem inbound da porta 3306 **do SG da Lambda** (não do CIDR)? (4) Subnets da Lambda roteiam para NAT GW (para chamar APIs AWS)? (5) RDS está na mesma VPC que a Lambda? (6) RDS endpoint correto (não o IP)?

---

**P:** Como criar um ECS Fargate Service com Terraform?  
**R:**
```hcl
resource "aws_ecs_service" "app" {
  name            = "my-app"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2
  launch_type     = "FARGATE"
  network_configuration {
    subnets          = [aws_subnet.private.id]
    security_groups  = [aws_security_group.app.id]
    assign_public_ip = false
  }
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 8080
  }
}
```

---

**P:** Como usar o AWS CLI para testar uma IAM Policy sem aplicar?  
**R:**
```bash
# Simula se uma ação é permitida para o principal
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT:role/MinhaRole \
  --action-names "s3:GetObject" \
  --resource-arns "arn:aws:s3:::meu-bucket/*"

# Resultado: allowed/denied + qual policy bloqueou
# Também útil: iam simulate-custom-policy para testar policy JSON diretamente
```

---

**P:** Como fazer o deploy de uma aplicação Lambda com SAM e criar API Gateway automaticamente?  
**R:**
```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.12
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```
```bash
sam build && sam deploy --guided
```

---

**P:** Qual é o comando CLI para listar instâncias EC2 em execução por região?  
**R:**
```bash
# Instâncias em running na região default
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# Todas as regiões (script bash)
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "Region: $region"
  aws ec2 describe-instances --region $region --filters "Name=instance-state-name,Values=running" --query 'Reservations[*].Instances[*].InstanceId' --output text
done
```

---

**P:** Como configurar S3 Lifecycle Policy para transição para Glacier via CLI?  
**R:**
```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket meu-bucket \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "move-to-glacier",
      "Status": "Enabled",
      "Filter": {"Prefix": "logs/"},
      "Transitions": [
        {"Days": 30, "StorageClass": "STANDARD_IA"},
        {"Days": 90, "StorageClass": "GLACIER"}
      ],
      "Expiration": {"Days": 365}
    }]
  }'
```

---

**P:** Como debugar um erro 403 Forbidden no S3?  
**R:** Checklist: (1) Bucket Policy bloqueia? `aws s3api get-bucket-policy --bucket bucket`. (2) Block Public Access habilitado? `aws s3api get-public-access-block --bucket bucket`. (3) IAM Policy da role/user tem `s3:GetObject`? `aws iam simulate-principal-policy`. (4) ACL do objeto? `aws s3api get-object-acl --bucket bucket --key key`. (5) KMS key: a role tem `kms:Decrypt`?

---

**P:** Qual é o comando para escalar manualmente um ECS Service?  
**R:**
```bash
# Aumentar desired count para 5
aws ecs update-service \
  --cluster meu-cluster \
  --service meu-servico \
  --desired-count 5

# Verificar deploy
aws ecs describe-services \
  --cluster meu-cluster \
  --services meu-servico \
  --query 'services[0].[runningCount,desiredCount,deployments]'
```

---

**P:** Como configurar CloudFront com S3 origin usando OAC (Origin Access Control)?  
**R:**
```hcl
resource "aws_cloudfront_origin_access_control" "oac" {
  name = "my-oac"
  origin_access_control_origin_type = "s3"
  signing_behavior = "always"
  signing_protocol = "sigv4"
}
# Na distribuição CloudFront: origin com origin_access_control_id = oac.id
# Bucket Policy: permitir apenas cloudfront.amazonaws.com com condition
# "aws:SourceArn": "arn:aws:cloudfront::ACCOUNT:distribution/DISTID"
```

---

**P:** Como verificar o custo estimado de recursos provisionados pelo Terraform?  
**R:** Opção 1: **Infracost** (tool open-source): `infracost breakdown --path .` — mostra custo mensal estimado por recurso Terraform. Opção 2: **AWS Pricing Calculator** (manual). Opção 3: **Terraform Cloud** tem cost estimation integrado. Opção 4: após `terraform apply`, verificar AWS Cost Explorer com tags do projeto.

---

**P:** Como criar um Secret no Secrets Manager e recuperar com Lambda?  
**R:**
```bash
# Criar secret
aws secretsmanager create-secret \
  --name prod/db/credentials \
  --secret-string '{"username":"admin","password":"supersecret"}'
```
```python
# Lambda Python
import boto3, json
def lambda_handler(event, context):
    sm = boto3.client('secretsmanager')
    secret = sm.get_secret_value(SecretId='prod/db/credentials')
    creds = json.loads(secret['SecretString'])
    # usar creds['username'] e creds['password']
```
Lambda Role precisa de `secretsmanager:GetSecretValue`.
