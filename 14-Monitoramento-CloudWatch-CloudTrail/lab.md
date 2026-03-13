# Lab Prático — Monitoramento: CloudWatch + CloudTrail + Config (Módulo 14)

> **Região:** us-east-1 | **Custo estimado:** < $0.50 (CloudWatch primeiros 10 alarmes gratuitos; CloudTrail primeiros 90 dias Event History gratuito)  
> **Pré-requisitos:** AWS CLI configurado, Terraform >= 1.5

---

## Objetivo

Implementar um stack completo de observabilidade: métricas customizadas, alarmes, CloudTrail com alertas para ações críticas, e uma regra de Config.

---

## Parte 1 — CloudWatch: Métricas Customizadas e Alarmes

```bash
# Publicar métrica customizada (simular app que conta pedidos)
aws cloudwatch put-metric-data \
  --namespace "MinhaApp/Pedidos" \
  --metric-data '[
    {
      "MetricName": "PedidosCriados",
      "Dimensions": [{"Name": "Ambiente", "Value": "producao"}],
      "Value": 42,
      "Unit": "Count"
    },
    {
      "MetricName": "TaxaErro",
      "Dimensions": [{"Name": "Ambiente", "Value": "producao"}],
      "Value": 2.5,
      "Unit": "Percent"
    }
  ]'

# Verificar que a métrica foi registrada
aws cloudwatch list-metrics \
  --namespace "MinhaApp/Pedidos" \
  --query 'Metrics[*].[MetricName, Dimensions[0].Value]' \
  --output table

# Criar alarme para taxa de erro
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Criar tópico SNS para alertas
SNS_ARN=$(aws sns create-topic \
  --name lab-alertas-monitoramento \
  --query 'TopicArn' --output text)

# Inscrever email no tópico (substituir com seu email)
aws sns subscribe \
  --topic-arn "$SNS_ARN" \
  --protocol email \
  --notification-endpoint "seu-email@exemplo.com"

echo "Confirme a inscrição no email antes de continuar."
echo "SNS Topic ARN: $SNS_ARN"

# Criar alarme de taxa de erro > 5%
aws cloudwatch put-metric-alarm \
  --alarm-name "lab-taxa-erro-alta" \
  --alarm-description "Taxa de erro da aplicação acima de 5%" \
  --metric-name "TaxaErro" \
  --namespace "MinhaApp/Pedidos" \
  --dimensions Name=Ambiente,Value=producao \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 5.0 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions "$SNS_ARN" \
  --ok-actions "$SNS_ARN" \
  --treat-missing-data notBreaching

# Verificar estado do alarme
aws cloudwatch describe-alarms \
  --alarm-names "lab-taxa-erro-alta" \
  --query 'MetricAlarms[*].[AlarmName, StateValue, Threshold]' \
  --output table

# Simular ALARME (publicar valor acima do threshold)
aws cloudwatch put-metric-data \
  --namespace "MinhaApp/Pedidos" \
  --metric-data '[
    {
      "MetricName": "TaxaErro",
      "Dimensions": [{"Name": "Ambiente", "Value": "producao"}],
      "Value": 15.0,
      "Unit": "Percent"
    }
  ]'

echo "Aguarde ~5 minutos e verifique o estado do alarme..."
```

---

## Parte 2 — CloudWatch Logs: Filtros e Queries

```bash
# Criar um Log Group de teste
aws logs create-log-group --log-group-name "/lab/aplicacao"
aws logs create-log-stream \
  --log-group-name "/lab/aplicacao" \
  --log-stream-name "stream-teste"

# Injetar logs simulados
TIMESTAMP=$(date +%s%3N)

aws logs put-log-events \
  --log-group-name "/lab/aplicacao" \
  --log-stream-name "stream-teste" \
  --log-events "[
    {\"timestamp\": ${TIMESTAMP}, \"message\": \"INFO: Pedido 1001 criado com sucesso\"},
    {\"timestamp\": $((TIMESTAMP + 1000)), \"message\": \"ERROR: Falha ao processar pedido 1002 - timeout\"},
    {\"timestamp\": $((TIMESTAMP + 2000)), \"message\": \"INFO: Pedido 1003 criado com sucesso\"},
    {\"timestamp\": $((TIMESTAMP + 3000)), \"message\": \"ERROR: Conexão recusada ao banco de dados\"},
    {\"timestamp\": $((TIMESTAMP + 4000)), \"message\": \"WARN: Tempo de resposta anormal: 2500ms\"}
  ]"

# CloudWatch Logs Insights Query
aws logs start-query \
  --log-group-name "/lab/aplicacao" \
  --start-time $(($(date +%s) - 3600)) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message
    | filter @message like /ERROR/
    | sort @timestamp desc
    | limit 20'

# Obter resultado (use o queryId retornado acima)
# aws logs get-query-results --query-id "QUERY_ID_AQUI"

# Criar Metric Filter (conta ERRORs por minuto)
aws logs put-metric-filter \
  --log-group-name "/lab/aplicacao" \
  --filter-name "ContarErros" \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=ErroCount,metricNamespace=MinhaApp/Logs,metricValue=1,defaultValue=0
```

---

## Parte 3 — CloudTrail: Auditoria com Alertas

```bash
# Criar bucket para CloudTrail
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
CT_BUCKET="lab-cloudtrail-${ACCOUNT_ID}"

aws s3api create-bucket --bucket "$CT_BUCKET" --region us-east-1

# Criar bucket policy (CloudTrail precisa de permissão para gravar)
cat > cloudtrail_bucket_policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::${CT_BUCKET}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::${CT_BUCKET}/AWSLogs/${ACCOUNT_ID}/*",
      "Condition": {
        "StringEquals": {"s3:x-amz-acl": "bucket-owner-full-control"}
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket "$CT_BUCKET" \
  --policy file://cloudtrail_bucket_policy.json

# Criar trail
aws cloudtrail create-trail \
  --name "lab-trail" \
  --s3-bucket-name "$CT_BUCKET" \
  --include-global-service-events \
  --is-multi-region-trail \
  --enable-log-file-validation

# Iniciar logging
aws cloudtrail start-logging --name "lab-trail"

# Verificar status
aws cloudtrail get-trail-status --name "lab-trail" \
  --query '[IsLogging, LatestDeliveryTime]'

# Criar Metric Filter no CloudWatch para detectar DeleteBucket
aws logs create-log-group --log-group-name "CloudTrailLogs"

aws cloudtrail update-trail \
  --name "lab-trail" \
  --cloud-watch-logs-log-group-arn "arn:aws:logs:us-east-1:${ACCOUNT_ID}:log-group:CloudTrailLogs:*" \
  --cloud-watch-logs-role-arn "arn:aws:iam::${ACCOUNT_ID}:role/CloudTrail_CloudWatchLogs_Role"

# Criar filtro para ações de deleção de bucket
aws logs put-metric-filter \
  --log-group-name "CloudTrailLogs" \
  --filter-name "DeleteBucketFilter" \
  --filter-pattern '{ ($.eventName = DeleteBucket) }' \
  --metric-transformations \
    metricName=DeleteBucketCount,metricNamespace=CloudTrailMetrics,metricValue=1

# Alarme para deleção de bucket
aws cloudwatch put-metric-alarm \
  --alarm-name "lab-s3-bucket-deletado" \
  --metric-name "DeleteBucketCount" \
  --namespace "CloudTrailMetrics" \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions "$SNS_ARN" \
  --treat-missing-data notBreaching
```

---

## Parte 4 — AWS Config: Regra de Conformidade

```bash
# Ver regras Config já disponíveis
aws configservice describe-config-rules \
  --query 'ConfigRules[*].ConfigRuleName' \
  --output table

# Criar regra: instâncias EC2 sem IP público
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "lab-ec2-no-public-ip",
    "Description": "Verifica se instâncias EC2 têm IP público associado",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "EC2_INSTANCE_NO_PUBLIC_IP"
    },
    "Scope": {
      "ComplianceResourceTypes": ["AWS::EC2::Instance"]
    }
  }'

# Ver status de conformidade (pode levar alguns minutos)
aws configservice describe-compliance-by-config-rule \
  --config-rule-names "lab-ec2-no-public-ip" \
  --query 'ComplianceByConfigRules[*].[ConfigRuleName,Compliance.ComplianceType]' \
  --output table

# Ver recursos não conformes
aws configservice get-compliance-details-by-config-rule \
  --config-rule-name "lab-ec2-no-public-ip" \
  --compliance-types "NON_COMPLIANT" \
  --query 'EvaluationResults[*].EvaluationResultIdentifier.EvaluationResultQualifier.[ResourceType,ResourceId]' \
  --output table
```

---

## Limpeza

```bash
# Parar CloudTrail
aws cloudtrail stop-logging --name "lab-trail"
aws cloudtrail delete-trail --name "lab-trail"

# Remover bucket CloudTrail
aws s3 rm "s3://${CT_BUCKET}" --recursive
aws s3api delete-bucket --bucket "$CT_BUCKET"

# Remover alarmes
aws cloudwatch delete-alarms \
  --alarm-names "lab-taxa-erro-alta" "lab-s3-bucket-deletado"

# Remover SNS topic
aws sns delete-topic --topic-arn "$SNS_ARN"

# Remover log groups
aws logs delete-log-group --log-group-name "/lab/aplicacao"
aws logs delete-log-group --log-group-name "CloudTrailLogs"

# Remover regra Config
aws configservice delete-config-rule --config-rule-name "lab-ec2-no-public-ip"

# Remover arquivo temporário
rm -f cloudtrail_bucket_policy.json
```

---

## O Que Você Aprendeu

- Métricas customizadas no CloudWatch permitem monitorar KPIs de negócio (não só infra)
- Alarmes podem ser compostos (Composite Alarms) para evitar falsos positivos
- CloudWatch Logs Insights oferece SQL-like queries em logs sem ETL
- CloudTrail registra TODAS as chamadas de API — fundamental para auditoria
- Metric Filters transformam eventos de log em métricas (bridge entre logs e alarmes)
- Config avalia conformidade continuamente — não apenas quando configurado
