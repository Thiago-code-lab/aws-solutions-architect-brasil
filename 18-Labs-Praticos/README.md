# Módulo 18 — Labs Práticos

## Sobre Este Módulo

Este módulo consolida os principais laboratórios AWS para estudo prático do SAA-C03. Cada lab inclui comandos AWS CLI e código Terraform equivalente para reforçar o aprendizado hands-on.

> **Custo estimado total dos labs**: < $5 (sempre execute cleanup após cada lab)

---

## Lab 1: Serverless API com Lambda + DynamoDB + API Gateway

**Objetivo**: Criar uma API serverless completa com CRUD em DynamoDB.

**Arquitetura:**
```
API Gateway HTTP API → Lambda → DynamoDB
```

**CLI — Passo a passo:**
```bash
# 1. Criar tabela DynamoDB
aws dynamodb create-table \
  --table-name Products \
  --attribute-definitions AttributeName=productId,AttributeType=S \
  --key-schema AttributeName=productId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1

# 2. Criar role IAM para Lambda
aws iam create-role \
  --role-name lambda-products-role \
  --assume-role-policy-document file://lambda-trust-policy.json

aws iam attach-role-policy \
  --role-name lambda-products-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess

# 3. Deploy da função Lambda
zip function.zip index.js
aws lambda create-function \
  --function-name products-api \
  --runtime nodejs20.x \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --role arn:aws:iam::ACCOUNT_ID:role/lambda-products-role

# 4. Criar API Gateway HTTP API
aws apigatewayv2 create-api \
  --name products-api \
  --protocol-type HTTP \
  --target arn:aws:lambda:us-east-1:ACCOUNT_ID:function:products-api

# Cleanup
aws lambda delete-function --function-name products-api
aws dynamodb delete-table --table-name Products
aws apigatewayv2 delete-api --api-id API_ID
```

**Terraform equivalente:**
```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" { region = "us-east-1" }

resource "aws_dynamodb_table" "products" {
  name         = "Products"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "productId"
  attribute { name = "productId"; type = "S" }
}

resource "aws_iam_role" "lambda" {
  name               = "lambda-products-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{ Effect = "Allow", Principal = { Service = "lambda.amazonaws.com" }, Action = "sts:AssumeRole" }]
  })
}

resource "aws_iam_role_policy_attachment" "dynamodb" {
  role       = aws_iam_role.lambda.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess"
}

data "archive_file" "lambda" {
  type        = "zip"
  source_file = "index.js"
  output_path = "function.zip"
}

resource "aws_lambda_function" "api" {
  function_name    = "products-api"
  filename         = "function.zip"
  handler          = "index.handler"
  runtime          = "nodejs20.x"
  role             = aws_iam_role.lambda.arn
  source_code_hash = data.archive_file.lambda.output_base64sha256
}
```

---

## Lab 2: VPC com Sub-redes Públicas e Privadas

**Objetivo**: Criar uma VPC completa com subnets públicas e privadas, NAT Gateway, bastion host.

```bash
# 1. Criar VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab-vpc}]' \
  --query Vpc.VpcId --output text)

# 2. Criar subnets (duas AZs)
PUB_SUB_1=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=pub-1a}]' \
  --query Subnet.SubnetId --output text)

PRIV_SUB_1=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.11.0/24 --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=priv-1a}]' \
  --query Subnet.SubnetId --output text)

# 3. Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID

# 4. Route table pública
PUB_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID --query RouteTable.RouteTableId --output text)
aws ec2 create-route --route-table-id $PUB_RT --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
aws ec2 associate-route-table --route-table-id $PUB_RT --subnet-id $PUB_SUB_1

# 5. NAT Gateway (para subnet privada)
EIP=$(aws ec2 allocate-address --domain vpc --query AllocationId --output text)
NAT_ID=$(aws ec2 create-nat-gateway --subnet-id $PUB_SUB_1 \
  --allocation-id $EIP --query NatGateway.NatGatewayId --output text)
# Aguardar NAT estar available
aws ec2 wait nat-gateway-available --filter Name=nat-gateway-id,Values=$NAT_ID

# 6. Route table privada (via NAT)
PRIV_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID --query RouteTable.RouteTableId --output text)
aws ec2 create-route --route-table-id $PRIV_RT --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_ID
aws ec2 associate-route-table --route-table-id $PRIV_RT --subnet-id $PRIV_SUB_1

echo "VPC criada: $VPC_ID"
```

---

## Lab 3: S3 com Versionamento, Ciclo de Vida e Replicação

```bash
# 1. Criar buckets
aws s3api create-bucket --bucket meu-bucket-principal-$(date +%s) --region us-east-1
aws s3api create-bucket --bucket meu-bucket-replica-$(date +%s) \
  --create-bucket-configuration LocationConstraint=us-west-2 --region us-west-2

# 2. Habilitar versionamento nos dois buckets
aws s3api put-bucket-versioning \
  --bucket meu-bucket-principal \
  --versioning-configuration Status=Enabled

# 3. Lifecycle policy (move para IA após 30d, Glacier após 90d, exclui após 365d)
aws s3api put-bucket-lifecycle-configuration \
  --bucket meu-bucket-principal \
  --lifecycle-configuration file://lifecycle.json

# lifecycle.json:
# {
#   "Rules": [{
#     "ID": "transition-rule",
#     "Status": "Enabled",
#     "Filter": {},
#     "Transitions": [
#       {"Days": 30, "StorageClass": "STANDARD_IA"},
#       {"Days": 90, "StorageClass": "GLACIER"}
#     ],
#     "Expiration": {"Days": 365}
#   }]
# }

# 4. Replicação entre regiões (CRR)
aws s3api put-bucket-replication \
  --bucket meu-bucket-principal \
  --replication-configuration file://replication.json
```

---

## Lab 4: CloudWatch Alarme + Auto Scaling

```bash
# 1. Criar Launch Template
aws ec2 create-launch-template \
  --launch-template-name web-template \
  --launch-template-data '{
    "ImageId": "ami-0c02fb55956c7d316",
    "InstanceType": "t3.micro",
    "SecurityGroupIds": ["sg-xxx"]
  }'

# 2. Criar Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name web-asg \
  --launch-template LaunchTemplateName=web-template,Version='$Latest' \
  --min-size 1 --max-size 5 --desired-capacity 2 \
  --vpc-zone-identifier "subnet-aaa,subnet-bbb"

# 3. Criar política de escalonamento
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name web-asg \
  --policy-name cpu-scale-out \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 70.0
  }'

# Cleanup
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name web-asg --force-delete
```

---

## Lab 5: RDS Multi-AZ com Snapshot e Restore

```bash
# 1. Criar grupo de subnets para RDS
aws rds create-db-subnet-group \
  --db-subnet-group-name lab-db-subnet \
  --db-subnet-group-description "Lab DB subnets" \
  --subnet-ids subnet-aaa subnet-bbb

# 2. Criar instância RDS MySQL Multi-AZ
aws rds create-db-instance \
  --db-instance-identifier lab-rds \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password "MyPassword123!" \
  --allocated-storage 20 \
  --multi-az \
  --db-subnet-group-name lab-db-subnet \
  --no-publicly-accessible

# 3. Aguardar disponibilidade
aws rds wait db-instance-available --db-instance-identifier lab-rds

# 4. Criar snapshot
aws rds create-db-snapshot \
  --db-instance-identifier lab-rds \
  --db-snapshot-identifier lab-rds-snap-$(date +%Y%m%d)

# 5. Restaurar de snapshot (para novo endpoint)
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier lab-rds-restored \
  --db-snapshot-identifier lab-rds-snap-YYYYMMDD

# Cleanup
aws rds delete-db-instance --db-instance-identifier lab-rds --skip-final-snapshot
```

---

## Referência de Comandos CLI Essenciais

```bash
# EC2
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 stop-instances --instance-ids i-xxx
aws ec2 start-instances --instance-ids i-xxx
aws ec2 terminate-instances --instance-ids i-xxx

# IAM
aws iam list-roles --query 'Roles[*].RoleName'
aws iam get-role --role-name ROLE_NAME
aws sts get-caller-identity  # mostra conta/user/role atual

# S3
aws s3 ls s3://meu-bucket --recursive
aws s3 cp arquivo.txt s3://meu-bucket/
aws s3 sync ./pasta s3://meu-bucket/
aws s3api get-bucket-policy --bucket meu-bucket

# Lambda
aws lambda list-functions
aws lambda invoke --function-name minha-funcao --payload '{"key":"value"}' output.json
aws lambda update-function-code --function-name minha-funcao --zip-file fileb://function.zip

# DynamoDB
aws dynamodb scan --table-name Products --max-items 10
aws dynamodb put-item --table-name Products \
  --item '{"productId": {"S": "p001"}, "name": {"S": "Produto Teste"}}'
aws dynamodb get-item --table-name Products \
  --key '{"productId": {"S": "p001"}}'
```

---

## Dicas para os Labs

1. **Sempre limpe os recursos** após o lab para evitar custos desnecessários
2. **Use tags** em todos os recursos criados (ex: `Project=lab`, `Environment=test`)
3. **Terraform destroy** é mais confiável que deletar recursos manualmente um a um
4. Use `aws configure` para configurar credenciais ou configure variáveis de ambiente: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`
5. Para verificar custos dos labs: AWS Cost Explorer → Last 7 days → filter by service
