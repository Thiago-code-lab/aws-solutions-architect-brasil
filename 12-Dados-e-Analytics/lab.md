# Lab Prático — Dados e Analytics: Athena + Glue + S3 (Módulo 12)

> **Região:** us-east-1 | **Custo estimado:** < $1.00 (Athena cobra $5/TB escaneado; lab usa ~100 MB)  
> **Pré-requisitos:** AWS CLI configurado, Terraform >= 1.5

---

## Objetivo

Construir um pipeline de analytics simples: dados CSV no S3 → Glue Crawler cataloga → Athena consulta — sem precisar provisionar nenhum servidor.

---

## Parte 1 — Infraestrutura com Terraform

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

data "aws_caller_identity" "current" {}

locals {
  account_id = data.aws_caller_identity.current.account_id
  prefix     = "lab-analytics"
}

# Bucket S3 — dados brutos
resource "aws_s3_bucket" "raw" {
  bucket        = "${local.prefix}-raw-${local.account_id}"
  force_destroy = true

  tags = { Environment = "lab", Module = "12-analytics" }
}

resource "aws_s3_bucket_versioning" "raw" {
  bucket = aws_s3_bucket.raw.id
  versioning_configuration { status = "Enabled" }
}

# Bucket S3 — resultados Athena
resource "aws_s3_bucket" "athena_results" {
  bucket        = "${local.prefix}-athena-results-${local.account_id}"
  force_destroy = true
}

# Glue Database
resource "aws_glue_catalog_database" "analytics" {
  name = "lab_analytics_db"
}

# IAM Role para Glue Crawler
resource "aws_iam_role" "glue_role" {
  name = "${local.prefix}-glue-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "glue.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "glue_service" {
  role       = aws_iam_role.glue_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole"
}

resource "aws_iam_role_policy" "glue_s3" {
  name = "${local.prefix}-glue-s3-policy"
  role = aws_iam_role.glue_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["s3:GetObject", "s3:PutObject", "s3:ListBucket"]
      Resource = [
        aws_s3_bucket.raw.arn,
        "${aws_s3_bucket.raw.arn}/*"
      ]
    }]
  })
}

# Glue Crawler
resource "aws_glue_crawler" "vendas" {
  database_name = aws_glue_catalog_database.analytics.name
  name          = "${local.prefix}-vendas-crawler"
  role          = aws_iam_role.glue_role.arn

  s3_target {
    path = "s3://${aws_s3_bucket.raw.bucket}/vendas/"
  }

  schema_change_policy {
    update_behavior = "UPDATE_IN_DATABASE"
    delete_behavior = "DEPRECATE_IN_DATABASE"
  }
}

# Athena Workgroup
resource "aws_athena_workgroup" "lab" {
  name          = "${local.prefix}-workgroup"
  force_destroy = true

  configuration {
    result_configuration {
      output_location = "s3://${aws_s3_bucket.athena_results.bucket}/results/"
    }
    bytes_scanned_cutoff_per_query = 1073741824  # 1 GB limite por segurança
  }
}

output "raw_bucket" {
  value = aws_s3_bucket.raw.bucket
}

output "glue_database" {
  value = aws_glue_catalog_database.analytics.name
}
```

---

## Parte 2 — Dados de Exemplo (CSV)

```bash
# Criar dados de vendas de exemplo
cat > vendas_2024_01.csv << 'EOF'
pedido_id,data,produto,categoria,quantidade,preco_unitario,regiao
1001,2024-01-15,Notebook,Eletrônicos,1,2500.00,Sudeste
1002,2024-01-15,Mouse,Periféricos,3,89.90,Sul
1003,2024-01-16,Monitor,Eletrônicos,2,1200.00,Norte
1004,2024-01-16,Teclado,Periféricos,5,149.90,Sudeste
1005,2024-01-17,Notebook,Eletrônicos,2,2500.00,Centro-Oeste
1006,2024-01-17,Headset,Periféricos,4,299.90,Nordeste
1007,2024-01-18,Monitor,Eletrônicos,1,1200.00,Sul
1008,2024-01-18,SSD 1TB,Armazenamento,6,399.90,Sudeste
1009,2024-01-19,Mouse,Periféricos,2,89.90,Norte
1010,2024-01-20,Notebook,Eletrônicos,3,2500.00,Sudeste
EOF

cat > vendas_2024_02.csv << 'EOF'
pedido_id,data,produto,categoria,quantidade,preco_unitario,regiao
2001,2024-02-01,Monitor,Eletrônicos,4,1200.00,Sudeste
2002,2024-02-05,Notebook,Eletrônicos,2,2699.00,Sul
2003,2024-02-10,Headset,Periféricos,8,299.90,Nordeste
2004,2024-02-14,SSD 1TB,Armazenamento,10,399.90,Sudeste
2005,2024-02-20,Teclado,Periféricos,6,149.90,Centro-Oeste
EOF
```

---

## Parte 3 — Deploy e Carga de Dados

```bash
# 1. Deploy infraestrutura
terraform init
terraform apply -auto-approve

# 2. Capturar nome do bucket
BUCKET=$(terraform output -raw raw_bucket)
echo "Bucket: $BUCKET"

# 3. Upload dos CSVs particionados por data
aws s3 cp vendas_2024_01.csv "s3://${BUCKET}/vendas/year=2024/month=01/vendas_jan.csv"
aws s3 cp vendas_2024_02.csv "s3://${BUCKET}/vendas/year=2024/month=02/vendas_fev.csv"

# 4. Executar o Glue Crawler (cataloga o schema automaticamente)
aws glue start-crawler --name lab-analytics-vendas-crawler

# 5. Aguardar crawler terminar
watch -n 10 'aws glue get-crawler --name lab-analytics-vendas-crawler \
  --query "Crawler.State" --output text'
# Aguarde aparecer "READY"
```

---

## Parte 4 — Consultas no Athena via CLI

```bash
DB="lab_analytics_db"
WORKGROUP="lab-analytics-workgroup"
BUCKET=$(terraform output -raw raw_bucket)
RESULT_BUCKET="lab-analytics-athena-results-$(aws sts get-caller-identity --query Account --output text)"

# Função auxiliar para executar queries
run_query() {
  local sql="$1"
  local query_id=$(aws athena start-query-execution \
    --query-string "$sql" \
    --query-execution-context "Database=${DB}" \
    --work-group "${WORKGROUP}" \
    --query 'QueryExecutionId' --output text)
  
  echo "Query ID: $query_id"
  
  # Aguardar conclusão
  while true; do
    status=$(aws athena get-query-execution \
      --query-execution-id "$query_id" \
      --query 'QueryExecution.Status.State' --output text)
    [ "$status" = "SUCCEEDED" ] && break
    [ "$status" = "FAILED" ] && echo "FALHOU!" && break
    sleep 2
  done
  
  # Resultados
  aws athena get-query-results \
    --query-execution-id "$query_id" \
    --query 'ResultSet.Rows[*].Data[*].VarCharValue' \
    --output table
}

# QUERY 1 — Verificar tabela criada pelo Crawler
run_query "SHOW TABLES IN ${DB}"

# QUERY 2 — Total de vendas por categoria
run_query "
SELECT categoria,
       COUNT(*) AS num_pedidos,
       SUM(quantidade) AS total_itens,
       ROUND(SUM(quantidade * preco_unitario), 2) AS receita_total
FROM vendas
GROUP BY categoria
ORDER BY receita_total DESC"

# QUERY 3 — Top 3 regiões por receita
run_query "
SELECT regiao,
       ROUND(SUM(quantidade * preco_unitario), 2) AS receita
FROM vendas
GROUP BY regiao
ORDER BY receita DESC
LIMIT 3"

# QUERY 4 — Usando partições (mais eficiente — menos dados escaneados)
run_query "
SELECT data, produto, quantidade
FROM vendas
WHERE year = '2024' AND month = '01'
ORDER BY data"

# QUERY 5 — Criar tabela particionada em formato Parquet (melhor performance)
run_query "
CREATE TABLE IF NOT EXISTS vendas_parquet
WITH (
  format = 'PARQUET',
  parquet_compression = 'SNAPPY',
  external_location = 's3://${BUCKET}/vendas_parquet/',
  partitioned_by = ARRAY['year', 'month']
) AS
SELECT pedido_id, data, produto, categoria, quantidade, preco_unitario, regiao,
       year, month
FROM vendas"
```

---

## Parte 5 — Compara Custo CSV vs Parquet

```bash
# Ver bytes escaneados nas últimas queries
aws athena list-query-executions \
  --work-group "${WORKGROUP}" \
  --query 'QueryExecutionIds[0:5]' \
  --output text | tr '\t' '\n' | while read qid; do
    aws athena get-query-execution \
      --query-execution-id "$qid" \
      --query 'QueryExecution.Statistics.DataScannedInBytes'
done

# Custo por query: DataScannedInBytes / 1TB * $5.00
# CSV (raw): scan completo do arquivo
# Parquet: scan apenas das colunas necessárias (column pruning)
# Resultado típico: Parquet 10-20x menor scan = 10-20x mais barato
```

---

## Limpeza

```bash
# Remover arquivos S3 primeiro (force_destroy está habilitado)
aws s3 rm "s3://${BUCKET}" --recursive

# Destruir infraestrutura
terraform destroy -auto-approve
```

---

## O Que Você Aprendeu

- Glue Crawler detecta schema automaticamente e cria tabela no Data Catalog
- Athena cobra por bytes escaneados — particionar e usar Parquet reduz custo drasticamente
- Particionamento (year/month) permite Athena filtrar apenas arquivos relevantes
- Athena é serverless — zero servidores, paga apenas por queries executadas
- `bytes_scanned_cutoff_per_query` no Workgroup protege contra queries acidentalmente caras
