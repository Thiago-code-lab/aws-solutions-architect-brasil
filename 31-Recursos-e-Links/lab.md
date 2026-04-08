# Lab Prático — Recursos e Links: Setup Completo do Ambiente AWS (Módulo 21)

> **Custo estimado:** ~$0 (configurações locais e verificações de conta)  
> **Pré-requisitos:** AWS CLI já instalado; Python 3.8+

---

## Objetivo

Configurar um ambiente de trabalho profissional para AWS: múltiplos profiles CLI, aliases produtivos, ferramenta de gerenciamento de versões do Terraform, e boas práticas de segurança de credenciais.

---

## Parte 1 — AWS CLI: Múltiplos Perfis

```bash
# Verificar versão e configuração atual
aws --version
aws configure list-profiles

# Configurar perfil dev (menor privilégio para desenvolvimento)
aws configure --profile dev
# Preencher: Access Key ID, Secret, região us-east-1, output json

# Configurar perfil prod (acesso mais restrito, MFA obrigatório)
aws configure --profile prod

# Configurar profile que usa SSO (recomendado para organizações)
aws configure sso --profile sso-dev
# Preencher: SSO start URL, SSO region, account ID, role name

# Usar perfil específico
export AWS_PROFILE=dev
aws sts get-caller-identity

# Aliases úteis no ~/.bashrc ou ~/.zshrc
cat >> ~/.bashrc << 'ALIASES'
# AWS Profile shortcuts
alias aws-dev='export AWS_PROFILE=dev && echo "Usando perfil: dev"'
alias aws-prod='export AWS_PROFILE=prod && echo "Usando perfil: prod"'
alias aws-who='aws sts get-caller-identity'
alias aws-region='aws ec2 describe-availability-zones --query "AvailabilityZones[0].RegionName" --output text'

# Aliases de investigação rápida
alias s3ls='aws s3 ls'
alias ec2ls='aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,PublicIpAddress,Tags[?Key=='\''Name'\''].Value|[0]]" --output table'
alias lambdals='aws lambda list-functions --query "Functions[*].[FunctionName, Runtime, LastModified]" --output table'
ALIASES

source ~/.bashrc
```

---

## Parte 2 — Gerenciamento Seguro de Credenciais

```bash
# NUNCA armazenar chaves em código. Verificar se há keys no repositório:
# (Instalar git-secrets para prevenir commits acidentais)
# https://github.com/awslabs/git-secrets

# Rotação de Access Keys — boas práticas
echo "=== Verificar se há keys antigas (>90 dias) ==="
aws iam list-access-keys \
  --query 'AccessKeyMetadata[*].[AccessKeyId, Status, CreateDate]' \
  --output table

# Criar nova key e deletar a antiga
NEW_KEY=$(aws iam create-access-key \
  --query 'AccessKey.[AccessKeyId, SecretAccessKey]' \
  --output text)
echo "Nova key criada (configure no ~/.aws/credentials antes de deletar a antiga)"

# Verificar políticas de senha da conta
aws iam get-account-password-policy 2>/dev/null || echo "Sem password policy configurada"

# Configurar política de senha segura
aws iam update-account-password-policy \
  --minimum-password-length 14 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --allow-users-to-change-password \
  --max-password-age 90 \
  --password-reuse-prevention 5

echo "Password policy configurada!"

# Verificar usuários sem MFA
aws iam list-users --query 'Users[*].UserName' --output text | \
  tr '\t' '\n' | \
  while read USER; do
    MFA=$(aws iam list-mfa-devices --user-name "$USER" --query 'MFADevices' --output text)
    if [ -z "$MFA" ]; then
      echo "ALERTA: Usuário sem MFA: $USER"
    fi
  done
```

---

## Parte 3 — tfenv: Gerenciar Versões do Terraform

```bash
# Instalar tfenv (Linux/macOS)
git clone --depth=1 https://github.com/tfutils/tfenv.git ~/.tfenv
export PATH="$HOME/.tfenv/bin:$PATH"
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc

# Listar versões disponíveis
tfenv list-remote | head -10

# Instalar versão específica (usada no curso)
tfenv install 1.7.5
tfenv use 1.7.5
terraform --version

# Fixar versão por projeto (arquivo .terraform-version)
echo "1.7.5" > .terraform-version
echo "Terraform fixado na versão 1.7.5 para este diretório"

# Para Windows: usar winget ou chocolatey
# winget install Hashicorp.Terraform
# choco install terraform --version=1.7.5
```

---

## Parte 4 — AWS CLI Output Formats e Queries JMESPath

```bash
# Dominar JMESPath é essencial para o exame (e para trabalhar com AWS CLI)

# Filtrar instâncias rodando
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].[InstanceId, InstanceType, PrivateIpAddress]' \
  --output table

# Funções JMESPath úteis
# sort_by — ordenar por campo
aws s3api list-buckets \
  --query 'sort_by(Buckets, &CreationDate)[*].[Name, CreationDate]' \
  --output table

# contains — filtrar texto
aws ec2 describe-security-groups \
  --query 'SecurityGroups[?contains(GroupName, `lab`)].[ GroupId, GroupName]' \
  --output table

# length — contar resultados
aws s3api list-buckets --query 'length(Buckets)' --output text

# max_by/min_by — extremos
aws ec2 describe-instances \
  --query 'max_by(Reservations[*].Instances[], &LaunchTime).[InstanceId, LaunchTime]' \
  --output table

# Saída em diferentes formatos
aws iam list-users --output json   # JSON completo
aws iam list-users --output table  # tabela ASCII (fácil de ler)
aws iam list-users --output text   # texto plano (bom para scripts)
aws iam list-users --output yaml   # YAML (human-friendly)
```

---

## Parte 5 — Script de Auditoria de Conta

```python
# auditoria_conta.py — Verificações rápidas de segurança e custos
import boto3
import json
from datetime import datetime, timedelta, timezone

def verificar_conta():
    iam = boto3.client('iam')
    sts = boto3.client('sts')
    s3  = boto3.client('s3')
    ce  = boto3.client('ce', region_name='us-east-1')

    identity = sts.get_caller_identity()
    print(f"\n=== Auditoria da Conta: {identity['Account']} ===")
    print(f"Data: {datetime.now().strftime('%Y-%m-%d %H:%M')}")

    # 1. Verificar root sem MFA
    acct_summary = iam.get_account_summary()['SummaryMap']
    root_mfa = acct_summary.get('AccountMFAEnabled', 0)
    status_mfa = "OK" if root_mfa else "ALERTA: MFA não habilitado na conta root!"
    print(f"\n1. MFA conta root: {status_mfa}")

    # 2. Access Keys antigas
    print("\n2. Access Keys (usuários IAM):")
    users = iam.list_users()['Users']
    for user in users:
        keys = iam.list_access_keys(UserName=user['UserName'])['AccessKeyMetadata']
        for key in keys:
            created = key['CreateDate']
            age = (datetime.now(timezone.utc) - created).days
            flag = "ALERTA: >90 dias" if age > 90 else "ok"
            print(f"   {user['UserName']}: key {key['AccessKeyId'][:8]}... ({age} dias) [{flag}]")

    # 3. Buckets S3 públicos
    print("\n3. Buckets S3 com acesso público potencial:")
    buckets = s3.list_buckets()['Buckets']
    for bucket in buckets:
        try:
            pab = s3.get_public_access_block(Bucket=bucket['Name'])['PublicAccessBlockConfiguration']
            if not all(pab.values()):
                print(f"   VERIFICAR: {bucket['Name']} — algum Block Public Access desabilitado")
        except Exception:
            print(f"   VERIFICAR: {bucket['Name']} — sem configuração Block Public Access")

    # 4. Custo últimos 30 dias
    print("\n4. Custo últimos 30 dias (top 3 serviços):")
    start = (datetime.now() - timedelta(days=30)).strftime('%Y-%m-%d')
    end = datetime.now().strftime('%Y-%m-%d')
    try:
        cost = ce.get_cost_and_usage(
            TimePeriod={'Start': start, 'End': end},
            Granularity='MONTHLY',
            Metrics=['BlendedCost'],
            GroupBy=[{'Type': 'DIMENSION', 'Key': 'SERVICE'}]
        )
        servicos = cost['ResultsByTime'][0]['Groups']
        top3 = sorted(servicos, key=lambda x: float(x['Metrics']['BlendedCost']['Amount']), reverse=True)[:3]
        for s in top3:
            print(f"   {s['Keys'][0]}: ${float(s['Metrics']['BlendedCost']['Amount']):.2f}")
    except Exception as e:
        print(f"   Sem acesso ao Cost Explorer: {e}")

    print("\n=== Auditoria concluída ===")

if __name__ == '__main__':
    verificar_conta()
```

```bash
pip3 install boto3
python3 auditoria_conta.py
```

---

## Parte 6 — Preparação Final para o Exame

```bash
# Verificar se todos os serviços-chave estão configurados na sua conta
echo "=== Checklist pré-exame ==="

check() {
  echo -n "Verificando $1... "
  if eval "$2" > /dev/null 2>&1; then
    echo "OK"
  else
    echo "não disponível (não crítico para lab)"
  fi
}

check "EC2" "aws ec2 describe-availability-zones"
check "S3" "aws s3 ls"
check "IAM" "aws iam list-users"
check "Lambda" "aws lambda list-functions"
check "DynamoDB" "aws dynamodb list-tables"
check "RDS" "aws rds describe-db-instances"
check "VPC" "aws ec2 describe-vpcs"
check "CloudWatch" "aws cloudwatch list-metrics --namespace AWS/EC2"
check "Cost Explorer" "aws ce list-cost-category-definitions"

echo ""
echo "=== Recursos de Estudo (últimas 48h antes do exame) ==="
echo "1. Revisar cheatsheet.md de cada módulo"
echo "2. Focar em módulos com <70% no simulado Tutorials Dojo"
echo "3. Não estudar conteúdo novo — consolidar o que já sabe"
echo "4. Dormir bem, alimentar-se, chegar 30min antes"
echo "5. Leia TODAS as opções antes de marcar — elimine por absurdo"
```

---

## Limpeza

```bash
rm -f auditoria_conta.py
echo "Nada a destruir — sem recursos cloud criados neste lab."
```

---

## O Que Você Aprendeu

- AWS CLI `--profile` + `AWS_PROFILE`: trabalhar com múltiplas contas de forma segura
- JMESPath queries: `sort_by`, `contains`, `length`, `max_by` — essenciais para automação
- **Nunca** hardcodar credenciais: usar roles, profiles, SSO ou AWS Vault
- Password policy + MFA + key rotation = conta mais segura
- `simulate-principal-policy` + `auditoria_conta.py` = debugging e governança proativa
- Antes do exame: consolidar > estudar novo; sono e alimentação impactam desempenho cognitivo
