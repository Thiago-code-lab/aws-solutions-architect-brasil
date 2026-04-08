# Lab Prático — RDS: Multi-AZ, Read Replica e Backup (Módulo 06)

> **Região:** us-east-1 | **Custo estimado:** ~$0.30 (RDS db.t3.micro, < 1h de uso)  
> **Pré-requisitos:** AWS CLI configurado, Terraform >= 1.5, cliente MySQL

---

## Objetivo

Criar RDS MySQL com Multi-AZ, promover Read Replica, realizar backup manual e monitorar métricas via CloudWatch.

---

## Parte 1 — Terraform: RDS Multi-AZ + Read Replica

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

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# Subnet Group usando VPC default
resource "aws_db_subnet_group" "lab" {
  name       = "lab-rds-subnet-group"
  subnet_ids = data.aws_subnets.default.ids
}

# Security Group para RDS
resource "aws_security_group" "rds" {
  name   = "lab-rds-sg"
  vpc_id = data.aws_vpc.default.id

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Em prod: IP específico ou SG de app
    description = "MySQL lab"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Parameter Group customizado
resource "aws_db_parameter_group" "lab" {
  name   = "lab-mysql8-pg"
  family = "mysql8.0"

  parameter {
    name  = "max_connections"
    value = "100"
  }

  parameter {
    name  = "slow_query_log"
    value = "1"
  }

  parameter {
    name  = "long_query_time"
    value = "2"
  }
}

# RDS MySQL Multi-AZ — instância principal
resource "aws_db_instance" "primary" {
  identifier        = "lab-mysql-primary"
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  storage_type      = "gp2"

  db_name  = "loja"
  username = "admin"
  password = "Lab@12345Secure!"

  db_subnet_group_name   = aws_db_subnet_group.lab.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  parameter_group_name   = aws_db_parameter_group.lab.name

  multi_az              = true  # HA: standby em outra AZ
  publicly_accessible   = true  # para lab; false em produção
  skip_final_snapshot   = true

  backup_retention_period = 7  # 7 dias de backups automáticos
  backup_window           = "03:00-04:00"
  maintenance_window      = "Mon:04:00-Mon:05:00"

  deletion_protection = false  # true em produção

  tags = { Environment = "lab", Module = "06-rds" }
}

# Read Replica (mesma região)
resource "aws_db_instance" "read_replica" {
  identifier_prefix   = "lab-mysql-replica-"
  replicate_source_db = aws_db_instance.primary.identifier
  instance_class      = "db.t3.micro"
  publicly_accessible = true
  skip_final_snapshot = true
  auto_minor_version_upgrade = false

  tags = { Role = "read-replica" }
}

output "primary_endpoint"  { value = aws_db_instance.primary.endpoint }
output "replica_endpoint"  { value = aws_db_instance.read_replica.endpoint }
output "primary_arn"       { value = aws_db_instance.primary.arn }
```

---

## Parte 2 — Popular e Testar

```bash
terraform init && terraform apply -auto-approve

PRIMARY=$(terraform output -raw primary_endpoint)
REPLICA=$(terraform output -raw replica_endpoint)
RDS_ARN=$(terraform output -raw primary_arn)

echo "Primary: $PRIMARY"
echo "Replica: $REPLICA"

# Popular banco primário
mysql -h "${PRIMARY%:*}" -P 3306 -u admin -p"Lab@12345Secure!" loja << 'SQL'
CREATE TABLE produtos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(200) NOT NULL,
  preco DECIMAL(10,2),
  categoria VARCHAR(50),
  estoque INT DEFAULT 0,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_categoria (categoria)
);

INSERT INTO produtos (nome, preco, categoria, estoque) VALUES
  ('Notebook Pro', 3500.00, 'Eletrônicos', 10),
  ('Mouse Wireless', 89.90, 'Periféricos', 50),
  ('Teclado Mecânico', 350.00, 'Periféricos', 25),
  ('Monitor 27 pol.', 1200.00, 'Eletrônicos', 8),
  ('Headset USB', 180.00, 'Periféricos', 30);

SELECT COUNT(*) AS total, categoria FROM produtos GROUP BY categoria;
SQL

# Aguardar replicação (< 1 min em mesma região)
sleep 15

# Verificar na Read Replica
echo "Verificando dados na Read Replica..."
mysql -h "${REPLICA%:*}" -P 3306 -u admin -p"Lab@12345Secure!" loja \
  -e "SELECT id, nome, preco FROM produtos ORDER BY id;"

# Tentar ESCREVER na replica (deve falhar)
echo "Testando write na replica (deve dar erro)..."
mysql -h "${REPLICA%:*}" -P 3306 -u admin -p"Lab@12345Secure!" loja \
  -e "INSERT INTO produtos VALUES (NULL, 'Teste', 1, 'X', 1, NOW());" 2>&1 | head -3
```

---

## Parte 3 — Backup Manual e Monitoramento

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
RDS_ARN=$(terraform output -raw primary_arn)

# Criar snapshot manual
SNAPSHOT_ID="lab-mysql-snapshot-$(date +%Y%m%d%H%M)"
aws rds create-db-snapshot \
  --db-instance-identifier "lab-mysql-primary" \
  --db-snapshot-identifier "$SNAPSHOT_ID"

# Aguardar snapshot ficar disponível
aws rds wait db-snapshot-available \
  --db-snapshot-identifier "$SNAPSHOT_ID"

echo "Snapshot pronto: $SNAPSHOT_ID"

# Listar todos os snapshots (auto + manual)
aws rds describe-db-snapshots \
  --db-instance-identifier "lab-mysql-primary" \
  --query 'DBSnapshots[*].[DBSnapshotIdentifier, SnapshotType, Status, AllocatedStorage]' \
  --output table

# Monitorar métricas CloudWatch
echo ""
echo "=== Métricas CloudWatch RDS ==="
START=$(date -u -d "1 hour ago" +%FT%TZ 2>/dev/null || date -u -v-1H +%FT%TZ)
END=$(date -u +%FT%TZ)

for METRIC in "CPUUtilization" "FreeStorageSpace" "DatabaseConnections" "ReadLatency" "WriteLatency"; do
  VALUE=$(aws cloudwatch get-metric-statistics \
    --namespace AWS/RDS \
    --metric-name "$METRIC" \
    --dimensions "Name=DBInstanceIdentifier,Value=lab-mysql-primary" \
    --start-time "$START" --end-time "$END" \
    --period 3600 --statistics Average \
    --query 'Datapoints[0].Average' --output text 2>/dev/null)
  echo "  $METRIC: ${VALUE:-sem dados}"
done

# Criar alarme para CPU alta
aws cloudwatch put-metric-alarm \
  --alarm-name "lab-rds-cpu-alta" \
  --alarm-description "CPU RDS acima de 80%" \
  --namespace AWS/RDS \
  --metric-name CPUUtilization \
  --dimensions "Name=DBInstanceIdentifier,Value=lab-mysql-primary" \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

echo "Alarme de CPU criado."
```

---

## Limpeza

```bash
# Deletar snapshot manual
aws rds delete-db-snapshot --db-snapshot-identifier "$SNAPSHOT_ID"

# Deletar alarme
aws cloudwatch delete-alarms --alarm-names "lab-rds-cpu-alta"

# Destruir infraestrutura
terraform destroy -auto-approve
```

---

## O Que Você Aprendeu

- **Multi-AZ:** standby sincrônico na mesma região; failover automático em ~1-2 min; NÃO serve para leitura
- **Read Replica:** assíncrono; pode ser cross-region; promovível a instância independente; serve para leitura
- **Snapshot manual vs automático:** snapshots manuais não expiram; automáticos respeitam `backup_retention_period`
- **Restaurar snapshot:** cria nova instância (não substitui a atual) — RDS sem `UPDATE` de instância in-place
- **Parameter Group:** configurações do engine (slow query log, max connections) — aplicado com reinicialização (static) ou imediato (dynamic)
