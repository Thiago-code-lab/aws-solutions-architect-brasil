# Lab Prático — Well-Architected Framework: WAT + Trusted Advisor + Cost Explorer (Módulo 16)

> **Região:** us-east-1 | **Custo estimado:** ~$0 (Well-Architected Tool, Trusted Advisor e Cost Explorer são gratuitos)  
> **Pré-requisitos:** AWS CLI configurado, conta com acesso à console

---

## Objetivo

Usar as ferramentas nativas AWS para avaliar a arquitetura segundo o Well-Architected Framework, checar recomendações do Trusted Advisor e analisar custos com Cost Explorer.

---

## Parte 1 — AWS Well-Architected Tool

```bash
# Listar lenses disponíveis
aws wellarchitected list-lenses \
  --query 'LenseSummaries[*].[LensAlias, LensName]' \
  --output table

# Criar um workload para avaliação
WORKLOAD_ID=$(aws wellarchitected create-workload \
  --workload-name "Lab-Plataforma-Ecommerce" \
  --description "Avaliação da plataforma de e-commerce para fim de lab" \
  --environment PRODUCTION \
  --aws-regions "us-east-1" \
  --lenses "wellarchitected" \
  --review-owner "time-devops@empresa.com" \
  --query 'WorkloadId' --output text)

echo "Workload criado: $WORKLOAD_ID"

# Ver detalhes do workload
aws wellarchitected get-workload \
  --workload-id "$WORKLOAD_ID" \
  --query 'Workload.[WorkloadName, Environment, RiskCounts]' \
  --output table

# Listar milestone (estado inicial)
aws wellarchitected list-milestones \
  --workload-id "$WORKLOAD_ID"

# Explorar perguntas por pilar
PILLARS=("operationalExcellence" "security" "reliability" "performance" "costOptimization" "sustainability")

for PILLAR in "${PILLARS[@]}"; do
  echo ""
  echo "=== Pilar: $PILLAR ==="
  aws wellarchitected list-answers \
    --workload-id "$WORKLOAD_ID" \
    --lens-alias "wellarchitected" \
    --pillar-id "$PILLAR" \
    --query 'AnswerSummaries[*].[QuestionId, QuestionTitle, Risk]' \
    --output table 2>/dev/null | head -30
done

# Responder uma pergunta (pilar de segurança — SEC 1)
aws wellarchitected update-answer \
  --workload-id "$WORKLOAD_ID" \
  --lens-alias "wellarchitected" \
  --question-id "sec_securely_operate" \
  --selected-choices "sec_securely_operate_aws_account" "sec_securely_operate_multi_accounts" \
  --notes "Usamos AWS Organizations com SCPs para governança centralizada"

# Gerar milestone após responder perguntas
MILESTONE_NUMBER=$(aws wellarchitected create-milestone \
  --workload-id "$WORKLOAD_ID" \
  --milestone-name "Sprint-Review-2025-Q1" \
  --query 'MilestoneNumber' --output text)

echo "Milestone criado: Sprint-Review-2025-Q1 (Número: $MILESTONE_NUMBER)"

# Ver review do lens com riscos
aws wellarchitected get-lens-review \
  --workload-id "$WORKLOAD_ID" \
  --lens-alias "wellarchitected" \
  --query 'LensReview.[LensName, RiskCounts]' \
  --output json
```

---

## Parte 2 — Trusted Advisor

```bash
# Listar todas as verificações disponíveis
aws trustedadvisor list-checks \
  --query 'checks[*].[id, name, category, status]' \
  --output table 2>/dev/null | head -50

# Verificações por categoria
CATEGORIES=("cost_optimizing" "security" "fault_tolerance" "performance" "service_limits")

for CAT in "${CATEGORIES[@]}"; do
  echo ""
  echo "=== Categoria: $CAT ==="
  aws trustedadvisor list-checks \
    --query "checks[?category=='${CAT}'].[id, name]" \
    --output table 2>/dev/null | head -20
done

# Verificação de segurança — Security Groups portas abertas
OPEN_PORTS_CHECK="HCP4007jHTy"  # ID do check de SGs com portas abertas

aws trustedadvisor get-check-result \
  --check-id "$OPEN_PORTS_CHECK" \
  --query 'result.[status, timestamp, flaggedResources[*].[region, status]]' \
  --output json 2>/dev/null

# Verificação de MFA na conta root
MFA_CHECK="7DAFEmoDos"

aws trustedadvisor get-check-result \
  --check-id "$MFA_CHECK" \
  --query 'result.[status, checkId]' \
  --output json 2>/dev/null

# Verificação de instâncias subutilizadas (Cost)
UNDERUTIL_CHECK="Qch7DwouX1"

aws trustedadvisor get-check-result \
  --check-id "$UNDERUTIL_CHECK" \
  --query 'result.[status, flaggedResources]' \
  --output json 2>/dev/null

# Refresh de verificações (forçar atualização)
aws trustedadvisor refresh-check --check-id "$OPEN_PORTS_CHECK"
echo "Check atualizado. Aguardar ~2-3 min para resultado."

# Resumo de todos os resultados
aws trustedadvisor describe-checks-result-summary \
  --query 'result.[status, categorySpecificSummary]' \
  --output json 2>/dev/null
```

---

## Parte 3 — AWS Compute Optimizer

```bash
# Verificar se o Compute Optimizer está habilitado
aws compute-optimizer get-enrollment-status \
  --query 'status' --output text

# Habilitar se necessário
aws compute-optimizer update-enrollment-status --status Active

# Aguardar 24h para gerar recomendações — listar o que está disponível
aws compute-optimizer get-ec2-instance-recommendations \
  --query 'instanceRecommendations[*].[instanceArn, finding, findingReasonCodes]' \
  --output table 2>/dev/null

# Recomendações para Auto Scaling Groups
aws compute-optimizer get-auto-scaling-group-recommendations \
  --query 'autoScalingGroupRecommendations[*].[autoScalingGroupArn, finding]' \
  --output table 2>/dev/null

# Recomendações para volumes EBS
aws compute-optimizer get-ebs-volume-recommendations \
  --query 'volumeRecommendations[*].[volumeArn, finding]' \
  --output table 2>/dev/null

# Exportar recomendações para S3 (para análise massiva)
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REPORT_BUCKET="lab-compute-optimizer-${ACCOUNT_ID}"

aws s3api create-bucket --bucket "$REPORT_BUCKET"

aws compute-optimizer export-ec2-instance-recommendations \
  --s3-destination-config bucket="$REPORT_BUCKET",keyPrefix="recommendations/" \
  --query 'jobId' --output text
```

---

## Parte 4 — Cost Explorer

```bash
# Custo total últimos 30 dias por serviço
START=$(date -d "-30 days" +%Y-%m-%d 2>/dev/null || date -v-30d +%Y-%m-%d)
END=$(date +%Y-%m-%d)

aws ce get-cost-and-usage \
  --time-period "Start=${START},End=${END}" \
  --granularity MONTHLY \
  --metrics BlendedCost UnblendedCost UsageQuantity \
  --group-by "Type=DIMENSION,Key=SERVICE" \
  --query 'ResultsByTime[0].Groups[*].[Keys[0], Metrics.BlendedCost.Amount]' \
  --output table

# Top 5 serviços mais caros
aws ce get-cost-and-usage \
  --time-period "Start=${START},End=${END}" \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by "Type=DIMENSION,Key=SERVICE" \
  --query 'sort_by(ResultsByTime[0].Groups, &Metrics.BlendedCost.Amount)[-5:][*].[Keys[0], Metrics.BlendedCost.Amount]' \
  --output table

# Custo por tag (exemplo: Environment)
aws ce get-cost-and-usage \
  --time-period "Start=${START},End=${END}" \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by "Type=TAG,Key=Environment" \
  --query 'ResultsByTime[0].Groups[*].[Keys[0], Metrics.BlendedCost.Amount]' \
  --output table

# Forecast para os próximos 30 dias
FORECAST_START=$(date +%Y-%m-%d)
FORECAST_END=$(date -d "+30 days" +%Y-%m-%d 2>/dev/null || date -v+30d +%Y-%m-%d)

aws ce get-cost-forecast \
  --time-period "Start=${FORECAST_START},End=${FORECAST_END}" \
  --granularity MONTHLY \
  --metric BLENDED_COST \
  --query '[Total.Amount, Total.Unit]' \
  --output text

# Custo diário (útil para detectar anomalias)
aws ce get-cost-and-usage \
  --time-period "Start=${START},End=${END}" \
  --granularity DAILY \
  --metrics BlendedCost \
  --query 'ResultsByTime[*].[TimePeriod.Start, Total.BlendedCost.Amount]' \
  --output table | tail -10
```

---

## Parte 5 — Savings Plans Analysis

```bash
# Ver Savings Plans já adquiridos
aws savingsplans describe-savings-plans \
  --query 'savingsPlans[*].[savingsPlanId, offeringId, state, commitment, end]' \
  --output table

# Recomendações de Savings Plans
aws ce get-savings-plans-purchase-recommendation \
  --savings-plans-type COMPUTE_SP \
  --term-in-years ONE_YEAR \
  --payment-option NO_UPFRONT \
  --lookback-period-in-days SIXTY_DAYS \
  --query 'SavingsPlansPurchaseRecommendation.[SavingsPlansType, LookbackPeriodInDays, SavingsPlansPurchaseRecommendationDetails[0].EstimatedSavingsSavingsPlansEligibleCosts]' \
  --output json

# Comparar On-Demand vs Savings Plans vs Reserved Instances
echo ""
echo "=== Comparativo de Modelos de Preço ==="
echo "On-Demand: sem compromisso, preço cheio"
echo "Compute Savings Plan 1yr No Upfront: ~17% desconto"
echo "EC2 Savings Plan 1yr No Upfront: ~up to 40% desconto"
echo "RI 1yr No Upfront: até 40% para tipo específico"
echo "RI 1yr All Upfront: até 56% para tipo específico"
echo "RI 3yr All Upfront: até 72% para tipo específico"
echo "Spot Instances: até 90% desconto (interruptíveis)"

# Cost Anomaly Detection — criar monitor
aws ce create-anomaly-monitor \
  --anomaly-monitor '{
    "MonitorName": "lab-monitor-servico",
    "MonitorType": "DIMENSIONAL",
    "MonitorDimension": "SERVICE"
  }' \
  --query 'MonitorArn' --output text
```

---

## Parte 6 — IAM Access Analyzer (Pilar Segurança)

```bash
# Criar analisador no nível da conta
ANALYZER_ARN=$(aws accessanalyzer create-analyzer \
  --analyzer-name "lab-account-analyzer" \
  --type ACCOUNT \
  --query 'arn' --output text)

echo "Analisador criado: $ANALYZER_ARN"

# Aguardar análise (~1-2 min) e listar findings
sleep 60

aws accessanalyzer list-findings \
  --analyzer-arn "$ANALYZER_ARN" \
  --query 'findings[*].[id, resourceType, resource, status]' \
  --output table

# Ver detalhes de um finding (se existir)
FIRST_FINDING=$(aws accessanalyzer list-findings \
  --analyzer-arn "$ANALYZER_ARN" \
  --query 'findings[0].id' --output text 2>/dev/null)

if [ "$FIRST_FINDING" != "None" ] && [ -n "$FIRST_FINDING" ]; then
  aws accessanalyzer get-finding \
    --analyzer-arn "$ANALYZER_ARN" \
    --id "$FIRST_FINDING" \
    --output json
fi
```

---

## Limpeza

```bash
# Deletar workload do Well-Architected Tool
aws wellarchitected delete-workload --workload-id "$WORKLOAD_ID"

# Deletar analisador IAM Access Analyzer
aws accessanalyzer delete-analyzer --analyzer-name "lab-account-analyzer"

# Deletar anomaly monitor
MONITOR_ARN=$(aws ce list-cost-category-definitions --query 'CostCategoryReferences[0]' 2>/dev/null)
# Listagem manual dos monitors:
aws ce get-anomaly-monitors --query 'AnomalyMonitors[*].[MonitorArn, MonitorName]' --output table

# Limpar bucket Compute Optimizer
aws s3 rm "s3://${REPORT_BUCKET}" --recursive
aws s3api delete-bucket --bucket "$REPORT_BUCKET" 2>/dev/null

echo "Limpeza concluída. Well-Architected Tool, Access Analyzer deletados."
```

---

## O Que Você Aprendeu

- **WAT (Well-Architected Tool):** criar workloads, responder perguntas por pilar, gerar milestones, rastrear riscos HIGH/MEDIUM/NONE
- **Trusted Advisor:** cheques automáticos em 5 categorias; refresh manual disponível via CLI
- **Compute Optimizer:** recomendações de right-sizing baseadas em CloudWatch metrics (requer 14+ dias de dados)
- **Cost Explorer:** análise por serviço, tag, granularidade diária/mensal, forecasting
- **Savings Plans:** escolha entre COMPUTE_SP (mais flexível) e EC2_SP (mais barato para tipo fixo)
- **IAM Access Analyzer:** detecta recursos com acesso externo não intencional (S3, SQS, Lambda, etc.)
