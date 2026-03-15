# Casos de Uso Reais — Labs Práticos (Módulo 18)

## Caso 1 — Debug: Lambda não consegue acessar RDS

**Contexto:** Time de desenvolvimento implantou Lambda que deve gravar dados em RDS MySQL dentro de uma VPC privada. Lambda funciona localmente mas retorna timeout em produção.

**Diagnóstico Passo a Passo:**
```
SINTOMA: Lambda invocation timeout (sem erro de conexão explícito)

PASSO 1 — Verificar VpcConfig da Lambda:
aws lambda get-function-configuration \
  --function-name MinhaFuncao \
  --query 'VpcConfig'

Resultado esperado:
{
  "SubnetIds": ["subnet-aaa", "subnet-bbb"],
  "SecurityGroupIds": ["sg-lambda"],
  "VpcId": "vpc-xxx"
}
Se SubnetIds vazio → Lambda não está na VPC → Adicionar VpcConfig

PASSO 2 — Verificar tipo de subnet (deve ser PRIVADA):
aws ec2 describe-subnets \
  --subnet-ids subnet-aaa \
  --query 'Subnets[*].[SubnetId,MapPublicIpOnLaunch,AvailabilityZone]'

Se MapPublicIpOnLaunch=True → subnet pública → usar subnet privada

PASSO 3 — Verificar Security Groups:
SG Lambda: permite saída para porta 3306 ao SG RDS?
aws ec2 describe-security-groups --group-ids sg-lambda
→ Verificar: Outbound rule: TCP 3306 → sg-rds (ou CIDR)

SG RDS: permite entrada do SG Lambda?
aws ec2 describe-security-groups --group-ids sg-rds  
→ Verificar: Inbound rule: TCP 3306 → sg-lambda

PASSO 4 — Verificar IAM Role da Lambda:
aws lambda get-function-configuration \
  --function-name MinhaFuncao --query 'Role'
→ Role deve ter: AWSLambdaVPCAccessExecutionRole (para criar ENI)

PASSO 5 — Verificar timeout da Lambda (deve ser > tempo de conexão):
aws lambda get-function-configuration \
  --function-name MinhaFuncao --query 'Timeout'
→ Padrão é 3s. RDS cold connection pode levar 1-2s. Aumentar para 30s.

SOLUÇÃO APLICADA:
1. Lambda estava em subnet PÚBLICA (sem acesso ao RDS privado)
2. Corrigir: mover Lambda para subnets privadas com NAT GW
3. Aumentar timeout de 3s para 30s
```

---

## Caso 2 — Debug: S3 retornando 403 Forbidden

**Contexto:** Aplicação ECS recebe 403 ao tentar fazer PutObject em bucket S3. Funciona na máquina do desenvolvedor mas não na AWS.

**Diagnóstico:**
```
PASSO 1 — Identificar qual identity está fazendo o request:
Na máquina local: usa credenciais pessoais do dev (com permissão total)
Na ECS: usa IAM Role da task (pode não ter permissão)

Ver qual role o container está usando:
curl http://169.254.170.2/v2/credentials  (apenas de dentro do container ECS)
→ Retorna temporaryCredentials com RoleArn

PASSO 2 — Verificar a Task Role:
aws ecs describe-task-definition --task-definition MinhaTask \
  --query 'taskDefinition.taskRoleArn'
→ Se null → sem Task Role → nenhuma permissão S3

Corrigir: criar IAM Role para task com:
{
  "Effect": "Allow",
  "Action": ["s3:PutObject", "s3:GetObject"],
  "Resource": "arn:aws:s3:::meu-bucket/*"
}
Atribuir como taskRoleArn na task definition

PASSO 3 — Verificar Bucket Policy:
aws s3api get-bucket-policy --bucket meu-bucket
→ Se Deny explícito para a role → corrigir bucket policy
→ Se Condition: aws:SourceVpc → Lambda/ECS deve estar na VPC correta

PASSO 4 — Verificar Block Public Access:
aws s3api get-public-access-block --bucket meu-bucket
→ Não é o problema se tentando com IAM Role (não é acesso público)

PASSO 5 — Verificar KMS (se bucket usa SSE-KMS):
A role tem permissão de kms:GenerateDataKey e kms:Decrypt na key?
aws kms list-grants --key-id arn:aws:kms:...:key/xxx

SOLUÇÃO:
Task Role não existed → criar e associar com permissões corretas
```

---

## Caso 3 — Deploy com Terraform: State Lock e Conflito de Equipe

**Contexto:** Time A e Time B ambos executam `terraform apply` ao mesmo tempo. Time B recebe erro de state lock. Ao destravar, ambos tiveram mudanças conflitantes no state.

**Conceito e Resolução:**
```
CONFIGURAÇÃO RECOMENDADA (S3 Backend + DynamoDB Lock):
# versions.tf
terraform {
  backend "s3" {
    bucket         = "terraform-states-prod"
    key            = "api-service/terraform.tfstate"  
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:..."  # SSE-KMS
    dynamodb_table = "terraform-locks"  # para state locking
  }
}

# DynamoDB table (criada manualmente ou com script bootstrap):
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

QUANDO STATE FICA TRAVADO (apply foi interrompido):
terraform force-unlock LOCK_ID
# LOCK_ID é mostrado na mensagem de erro

MELHOR PRÁTICA PARA TIMES:
1. Cada equipe tem seu próprio state file (keys diferentes no S3)
2. Ou: usar workspaces do Terraform
3. Ou: usar Terraform Cloud/Enterprise (locking nativo)
4. CI/CD pipeline (GitHub Actions) garante que apenas 1 apply roda por vez

DEBUG CONFLITO:
terraform state list  (ver quais recursos estão no state)
terraform state show aws_instance.web  (ver estado atual do recurso)
terraform import aws_instance.web i-1234567890  (importar recurso existente)
```

---

## Caso 4 — SAM: Lambda com Camada (Layer) de Dependências

**Contexto:** Lambda Python precisa de biblioteca pandas (100 MB) que excede o limite de upload zip de 50 MB. Deploy está falhando.

**Solução com SAM Layers:**
```
ESTRUTURA DO PROJETO:
projeto-lambda/
  template.yaml
  src/
    app.py  (código principal)
  layers/
    pandas-layer/
      python/
        requirements.txt  (pandas==2.1.0, numpy==1.26.0)

template.yaml:
Resources:
  PandasLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      LayerName: pandas-layer
      ContentUri: layers/pandas-layer/
      CompatibleRuntimes:
        - python3.12
    Metadata:
      BuildMethod: python3.12  # SAM build compila dependências

  MinhaFuncao:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Runtime: python3.12
      Layers:
        - !Ref PandasLayer
      MemorySize: 512  # pandas precisa de memória suficiente
      Timeout: 60

COMANDOS:
sam build  (compila layer + código)
sam deploy --guided  (primeira vez)

VERIFICAR:
aws lambda list-layers
aws lambda get-layer-version --layer-name pandas-layer --version-number 1

LIMITE IMPORTANTE:
- Código + layers descomprimidos: máximo 250 MB
- Se pandas ainda > 250 MB: usar container image Lambda (até 10 GB)
```

---

## Caso 5 — Debugging ECS Fargate Task que para Imediatamente

**Contexto:** ECS Fargate task é iniciada mas para em segundos com status STOPPED. Console mostra razão genérica. Time não sabe o que está errado.

**Diagnóstico Completo:**
```
PASSO 1 — Ver stoppedReason:
aws ecs describe-tasks \
  --cluster meu-cluster \
  --tasks arn:aws:ecs:...:task/TASK_ID \
  --query 'tasks[0].{stoppedReason:stoppedReason,status:lastStatus,containers:containers[*].{name:name,reason:reason,exitCode:exitCode}}'

MOTIVOS COMUNS + SOLUÇÕES:

A) "CannotPullContainerError: ... no such host"
   → ECR está em VPC privada sem endpoint
   Solução: Criar VPC Endpoint para ECR (ecr.api, ecr.dkr, s3)
   OU: Usar subnet pública com NAT GW

B) "CannotPullContainerError: 403 Forbidden"  
   → Task Execution Role não tem permissão ECR
   Solução: Adicionar AmazonECRReadOnly à Task Execution Role

C) exitCode: 1, reason: "Essential container exited"
   → App crashou. Ver logs no CloudWatch:
   aws logs tail /ecs/minha-task --follow
   Verificar: variáveis de ambiente, secrets do Secrets Manager

D) "OutOfMemoryError: Container killed"
   → memoryReservation muito baixo
   Solução: Aumentar memória na task definition

E) "ResourceInitializationError: unable to pull secrets"
   → Secrets Manager ou SSM Parameter Store inacessível
   Solução: VPC Endpoint para secretsmanager ou rota para internet

PASSO 2 — Ver logs detalhados:
aws logs describe-log-groups | grep ecs
aws logs get-log-events \
  --log-group-name /ecs/minha-task \
  --log-stream-name ecs/container-name/TASK_ID

PASSO 3 — Testar localmente:
docker run -e DATABASE_URL=... -e SECRET=... minha-imagem:latest
→ Reproduzir erro localmente é muito mais rápido
```
