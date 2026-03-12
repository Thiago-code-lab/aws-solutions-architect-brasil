# Lab Prático — VPC: Redes Privadas, NAT Gateway e Security (Módulo 05)

> **Região:** us-east-1 | **Custo estimado:** ~$0.05 (NAT Gateway $0.045/h — destruir em < 1h)  
> **Pré-requisitos:** AWS CLI configurado, Terraform >= 1.5

---

## Objetivo

Criar uma VPC completa com subnets públicas e privadas, NAT Gateway, bastion host, Security Groups e NACLs — replicando arquitetura típica de produção.

---

## Parte 1 — Terraform: VPC Completa

```hcl
# main.tf
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" { region = "us-east-1" }

locals {
  prefix = "lab-vpc"
}

# VPC principal
resource "aws_vpc" "lab" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = { Name = "${local.prefix}" }
}

# Subnets públicas (2 AZs para HA)
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.lab.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = { Name = "${local.prefix}-public-a", Tier = "public" }
}

resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.lab.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = { Name = "${local.prefix}-public-b", Tier = "public" }
}

# Subnets privadas (2 AZs)
resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.lab.id
  cidr_block        = "10.0.10.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "${local.prefix}-private-a", Tier = "private" }
}

resource "aws_subnet" "private_b" {
  vpc_id            = aws_vpc.lab.id
  cidr_block        = "10.0.11.0/24"
  availability_zone = "us-east-1b"
  tags = { Name = "${local.prefix}-private-b", Tier = "private" }
}

# Internet Gateway
resource "aws_internet_gateway" "lab" {
  vpc_id = aws_vpc.lab.id
  tags   = { Name = "${local.prefix}-igw" }
}

# Elastic IP para NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
  tags   = { Name = "${local.prefix}-nat-eip" }
}

# NAT Gateway na subnet pública a
resource "aws_nat_gateway" "lab" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public_a.id
  tags          = { Name = "${local.prefix}-natgw" }
  depends_on    = [aws_internet_gateway.lab]
}

# Route table pública
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.lab.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.lab.id
  }
  tags = { Name = "${local.prefix}-rt-public" }
}

resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_b" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public.id
}

# Route table privada → NAT
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.lab.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.lab.id
  }
  tags = { Name = "${local.prefix}-rt-private" }
}

resource "aws_route_table_association" "private_a" {
  subnet_id      = aws_subnet.private_a.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private_b" {
  subnet_id      = aws_subnet.private_b.id
  route_table_id = aws_route_table.private.id
}

# Security Group: Bastion Host
resource "aws_security_group" "bastion" {
  name   = "${local.prefix}-bastion-sg"
  vpc_id = aws_vpc.lab.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Em produção: IP específico
    description = "SSH acesso"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Security Group: Instâncias privadas — só aceita SSH do Bastion
resource "aws_security_group" "private_app" {
  name   = "${local.prefix}-app-sg"
  vpc_id = aws_vpc.lab.id

  ingress {
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion.id]
    description     = "SSH apenas pelo bastion"
  }

  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion.id]
    description     = "App port"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# NACL para subnet privada
resource "aws_network_acl" "private" {
  vpc_id     = aws_vpc.lab.id
  subnet_ids = [aws_subnet.private_a.id, aws_subnet.private_b.id]

  # Permitir SSH vindo da VPC (10.0.0.0/16)
  ingress {
    rule_no    = 100
    action     = "allow"
    protocol   = "tcp"
    cidr_block = "10.0.0.0/16"
    from_port  = 22
    to_port    = 22
  }

  # Permitir portas efêmeras (respostas HTTP/HTTPS)
  ingress {
    rule_no    = 200
    action     = "allow"
    protocol   = "tcp"
    cidr_block = "0.0.0.0/0"
    from_port  = 1024
    to_port    = 65535
  }

  # Bloquear acesso direto HTTP de fora da VPC
  ingress {
    rule_no    = 300
    action     = "deny"
    protocol   = "tcp"
    cidr_block = "0.0.0.0/0"
    from_port  = 80
    to_port    = 80
  }

  egress {
    rule_no    = 100
    action     = "allow"
    protocol   = "-1"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  tags = { Name = "${local.prefix}-private-nacl" }
}

output "vpc_id"         { value = aws_vpc.lab.id }
output "public_a_id"    { value = aws_subnet.public_a.id }
output "private_a_id"   { value = aws_subnet.private_a.id }
output "nat_eip"        { value = aws_eip.nat.public_ip }
```

---

## Parte 2 — Verificar e Testar a Rede

```bash
terraform init && terraform apply -auto-approve

VPC_ID=$(terraform output -raw vpc_id)
PUB_SUBNET=$(terraform output -raw public_a_id)
PRIV_SUBNET=$(terraform output -raw private_a_id)
NAT_IP=$(terraform output -raw nat_eip)

echo "VPC: $VPC_ID"
echo "NAT EIP: $NAT_IP"

# Verificar route tables
aws ec2 describe-route-tables \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'RouteTables[*].[RouteTableId, Routes[*].[DestinationCidrBlock, GatewayId, NatGatewayId]]' \
  --output json | python3 -m json.tool | head -60

# Verificar NACLs
aws ec2 describe-network-acls \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'NetworkAcls[*].[NetworkAclId, Entries[*].[RuleNumber, RuleAction, CidrBlock]]' \
  --output table

# Verificar Security Groups
aws ec2 describe-security-groups \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'SecurityGroups[*].[GroupName, IpPermissions[*].[IpProtocol, FromPort, IpRanges[0].CidrIp]]' \
  --output json | python3 -m json.tool

# VPC Flow Logs (recomendado em prod)
FLOW_LOG_BUCKET="lab-vpc-logs-$(aws sts get-caller-identity --query Account --output text)"
aws s3api create-bucket --bucket "$FLOW_LOG_BUCKET"

aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids "$VPC_ID" \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination "arn:aws:s3:::${FLOW_LOG_BUCKET}"

echo "Flow Logs habilitados para VPC $VPC_ID"
```

---

## Limpeza

```bash
# Desabilitar e deletar Flow Logs
FLOW_LOG_ID=$(aws ec2 describe-flow-logs \
  --filter "Name=resource-id,Values=$VPC_ID" \
  --query 'FlowLogs[0].FlowLogId' --output text)
aws ec2 delete-flow-logs --flow-log-ids "$FLOW_LOG_ID"
aws s3 rm "s3://${FLOW_LOG_BUCKET}" --recursive
aws s3api delete-bucket --bucket "$FLOW_LOG_BUCKET"

# Destruir infraestrutura Terraform
terraform destroy -auto-approve
```

---

## O Que Você Aprendeu

- VPC padrão: `10.0.0.0/16` → subnets `/24` = 251 hosts úteis por subnet
- NAT Gateway: PAGO por hora e por GB. Alternativa barata: NAT Instance (EC2 t3.micro)
- Security Group vs NACL: SG stateful (só ALLOW); NACL stateless (ALLOW e DENY, requer regra de retorno)
- Bastion Host: ponto único de entrada SSH na subnet pública; Session Manager é alternativa sem bastion
- VPC Flow Logs: auditoria de tráfego de rede; essencial para troubleshooting e compliance
