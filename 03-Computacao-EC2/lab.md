# Lab — Computação EC2

> **Região:** us-east-1  
> **Custo estimado:** ~$0,05–$0,20/hora durante o lab (EC2 t3.micro + EBS)  
> **Pré-requisito:** AWS CLI configurado, Terraform >= 1.5, VPC e subnet default disponíveis

## Objetivos do Lab
1. Criar um Launch Template com IMDSv2 obrigatório e User Data
2. Criar um Spot Fleet com diversificação de tipos
3. Comparar gp2 vs gp3 na prática com benchmark de IOPS
4. Criar e usar um Partition Placement Group

---

## Parte 1: Launch Template com IMDSv2 via AWS CLI

```bash
# Variáveis
REGION="us-east-1"
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" --output text --region $REGION)
SUBNET_ID=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" \
  --query "Subnets[0].SubnetId" --output text --region $REGION)
AMI_ID=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64 \
  --region $REGION --query "Parameters[0].Value" --output text)

echo "VPC: $VPC_ID | Subnet: $SUBNET_ID | AMI: $AMI_ID"

# Criar Security Group
SG_ID=$(aws ec2 create-security-group \
  --group-name "lab-ec2-sg" \
  --description "Lab EC2 SAA-C03" \
  --vpc-id $VPC_ID \
  --region $REGION \
  --query GroupId --output text)

# Permitir SSH (somente para teste — restrinja ao seu IP em produção)
MY_IP=$(curl -s https://checkip.amazonaws.com)
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp --port 22 \
  --cidr "${MY_IP}/32" \
  --region $REGION

# Key pair (se não tiver)
aws ec2 create-key-pair \
  --key-name lab-ec2-keypair \
  --query KeyMaterial --output text \
  --region $REGION > ~/.ssh/lab-ec2-keypair.pem
chmod 400 ~/.ssh/lab-ec2-keypair.pem

# USER DATA script codificado em base64
USER_DATA=$(cat <<'EOF' | base64
#!/bin/bash
yum update -y
# Instalar SSM Agent (já vem no AL2023 mas garantir)
yum install -y amazon-ssm-agent
systemctl enable --now amazon-ssm-agent

# Benchmark simples de disco
yum install -y fio
echo "Lab EC2 ready" > /var/www/html/index.html
yum install -y httpd
systemctl enable --now httpd
EOF
)

# Criar Launch Template com IMDSv2 obrigatório
LT_ID=$(aws ec2 create-launch-template \
  --launch-template-name "lab-saa-c03-lt" \
  --version-description "v1 - IMDSv2 required" \
  --launch-template-data "{
    \"ImageId\": \"$AMI_ID\",
    \"InstanceType\": \"t3.micro\",
    \"KeyName\": \"lab-ec2-keypair\",
    \"SecurityGroupIds\": [\"$SG_ID\"],
    \"MetadataOptions\": {
      \"HttpTokens\": \"required\",
      \"HttpEndpoint\": \"enabled\",
      \"HttpPutResponseHopLimit\": 1
    },
    \"UserData\": \"$USER_DATA\",
    \"TagSpecifications\": [{
      \"ResourceType\": \"instance\",
      \"Tags\": [{\"Key\": \"Name\", \"Value\": \"lab-ec2-imdsv2\"}]
    }]
  }" \
  --region $REGION \
  --query LaunchTemplate.LaunchTemplateId --output text)

echo "Launch Template criado: $LT_ID"
```

---

## Parte 2: Testar IMDSv2

```bash
# Lançar instância a partir do Launch Template
INSTANCE_ID=$(aws ec2 run-instances \
  --launch-template LaunchTemplateId=$LT_ID,Version='$Latest' \
  --subnet-id $SUBNET_ID \
  --associate-public-ip-address \
  --region $REGION \
  --query "Instances[0].InstanceId" --output text)

echo "Instância: $INSTANCE_ID"
aws ec2 wait instance-running --instance-ids $INSTANCE_ID --region $REGION

PUBLIC_IP=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].PublicIpAddress" --output text --region $REGION)

# SSH na instância e testar IMDSv2
ssh -i ~/.ssh/lab-ec2-keypair.pem ec2-user@$PUBLIC_IP

# Dentro da instância:
# Tentar IMDSv1 (deve falhar com 401)
curl -s http://169.254.169.254/latest/meta-data/instance-id
# Esperado: 401 Unauthorized

# IMDSv2 (correto)
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 300")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
# Esperado: i-0xxxxxxxxxxxx
```

---

## Parte 3: Terraform — Placement Group + gp3 EBS

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

provider "aws" {
  region = "us-east-1"
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "available" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

data "aws_ssm_parameter" "al2023" {
  name = "/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64"
}

# ──────────────────────────────────────────────────────
# 1. Partition Placement Group (para Kafka/Cassandra)
# ──────────────────────────────────────────────────────

resource "aws_placement_group" "partition_pg" {
  name            = "lab-partition-pg"
  strategy        = "partition"
  partition_count = 3 # 3 partições = 3 racks separados
}

# ──────────────────────────────────────────────────────
# 2. Security Group
# ──────────────────────────────────────────────────────

resource "aws_security_group" "lab_sg" {
  name        = "lab-ec2-terraform-sg"
  description = "Lab EC2 Terraform"
  vpc_id      = data.aws_vpc.default.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ──────────────────────────────────────────────────────
# 3. Launch Template com gp3 e IMDSv2
# ──────────────────────────────────────────────────────

resource "aws_launch_template" "lab_lt" {
  name_prefix            = "lab-ec2-"
  image_id               = data.aws_ssm_parameter.al2023.value
  instance_type          = "t3.micro"
  vpc_security_group_ids = [aws_security_group.lab_sg.id]

  # IMDSv2 obrigatório
  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"
    http_put_response_hop_limit = 1
  }

  # EBS gp3 como root volume
  block_device_mappings {
    device_name = "/dev/xvda"
    ebs {
      volume_size           = 20
      volume_type           = "gp3"
      iops                  = 3000 # baseline gp3
      throughput            = 125  # MB/s baseline gp3
      delete_on_termination = true
      encrypted             = true
    }
  }

  # EBS gp3 adicional para dados (IOPS customizados)
  block_device_mappings {
    device_name = "/dev/xvdb"
    ebs {
      volume_size           = 50
      volume_type           = "gp3"
      iops                  = 6000  # acima do baseline, sem custo extra por volume
      throughput            = 250   # MB/s
      delete_on_termination = true
      encrypted             = true
    }
  }

  user_data = base64encode(<<-EOF
    #!/bin/bash
    yum update -y
    yum install -y fio lsblk

    # Formatar e montar /dev/nvme1n1 (segundo volume EBS)
    mkfs.xfs /dev/nvme1n1
    mkdir -p /data
    mount /dev/nvme1n1 /data
    echo "/dev/nvme1n1 /data xfs defaults 0 2" >> /etc/fstab

    # Benchmark IOPS no gp3 (reportar no CloudWatch Logs)
    fio --name=iops_test --ioengine=libaio --rw=randread \
      --bs=4k --numjobs=4 --size=1G --runtime=30 \
      --directory=/data --output=/tmp/fio_results.txt
    cat /tmp/fio_results.txt
  EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name   = "lab-ec2-gp3-pg"
      Module = "03-Computacao-EC2"
    }
  }

  lifecycle {
    create_before_destroy = true
  }
}

# ──────────────────────────────────────────────────────
# 4. Instâncias no Placement Group (simulando Kafka brokers)
# ──────────────────────────────────────────────────────

resource "aws_instance" "kafka_broker" {
  count = 3

  launch_template {
    id      = aws_launch_template.lab_lt.id
    version = "$Latest"
  }

  placement_group         = aws_placement_group.partition_pg.id
  placement_partition_number = count.index + 1
  subnet_id               = data.aws_subnets.available.ids[0]

  tags = {
    Name      = "lab-kafka-broker-${count.index + 1}"
    Partition = count.index + 1
  }
}

# ──────────────────────────────────────────────────────
# Outputs
# ──────────────────────────────────────────────────────

output "placement_group_name" {
  value = aws_placement_group.partition_pg.name
}

output "broker_instance_ids" {
  value = aws_instance.kafka_broker[*].id
}

output "launch_template_id" {
  value = aws_launch_template.lab_lt.id
}
```

---

## Parte 4: Comparar gp2 vs gp3 via CLI

```bash
# Criar volume gp2 (legado)
VOL_GP2=$(aws ec2 create-volume \
  --size 100 \
  --volume-type gp2 \
  --availability-zone us-east-1a \
  --region us-east-1 \
  --query VolumeId --output text)

# Criar volume gp3 com mesmos IOPS que gp2 (3 IOPS/GB = 300 IOPS para 100GB)
# mas com throughput configurável
VOL_GP3=$(aws ec2 create-volume \
  --size 100 \
  --volume-type gp3 \
  --iops 3000 \
  --throughput 125 \
  --availability-zone us-east-1a \
  --region us-east-1 \
  --query VolumeId --output text)

echo "gp2: $VOL_GP2 | gp3: $VOL_GP3"

# Verificar preços via describe
aws ec2 describe-volumes \
  --volume-ids $VOL_GP2 $VOL_GP3 \
  --query "Volumes[*].{ID:VolumeId,Type:VolumeType,IOPS:Iops,Throughput:Throughput}" \
  --output table --region us-east-1
```

---

## Cleanup

```bash
# Terraform
terraform destroy -auto-approve

# CLI — terminar instâncias
aws ec2 terminate-instances --instance-ids $INSTANCE_ID --region us-east-1

# Deletar volumes
aws ec2 delete-volume --volume-id $VOL_GP2 --region us-east-1
aws ec2 delete-volume --volume-id $VOL_GP3 --region us-east-1

# Deletar launch template
aws ec2 delete-launch-template \
  --launch-template-id $LT_ID --region us-east-1

# Deletar key pair
aws ec2 delete-key-pair \
  --key-name lab-ec2-keypair --region us-east-1
rm ~/.ssh/lab-ec2-keypair.pem

# Deletar security group (após instâncias terminadas)
aws ec2 delete-security-group --group-id $SG_ID --region us-east-1

echo "Cleanup concluído!"
```

---

## Pontos de Revisão

- [ ] Qual a diferença entre `HttpPutResponseHopLimit: 1` vs `2` no IMDSv2?
- [ ] Por que o volume gp3 pode ter mais IOPS que o gp2 para o mesmo tamanho?
- [ ] No Partition Placement Group, instâncias de partições diferentes compartilham hardware?
- [ ] Por que usar `launch_template` em vez de `launch_configuration` no Terraform?
- [ ] O que acontece com os dados do volume gp3 quando `delete_on_termination = true`?
