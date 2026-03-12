# Questões — Módulo 18: Labs Práticos

> **Domínio SAA-C03**: Verificação Hands-on  
> **Dificuldade**: Prática

---

**1.** Após criar uma VPC com o comando `aws ec2 create-vpc --cidr-block 10.0.0.0/16`, você tenta lançar uma instância EC2 na nova VPC mas não consegue acessar a internet. Qual é o passo que provavelmente está faltando?

- A) Configurar o DNS da VPC
- B) Criar um Internet Gateway e associá-lo à VPC, e adicionar rota 0.0.0.0/0 na route table apontando para o IGW
- C) Habilitar IPv6 na VPC
- D) Criar um NAT Gateway antes de criar instâncias

<details><summary>Resposta</summary>

**B** — Uma VPC recém-criada não tem IGW. Para acesso à internet: (1) criar IGW, (2) attach ao VPC, (3) criar ou modificar route table adicionando rota 0.0.0.0/0 → IGW, (4) associar route table com a subnet pública, (5) a instância precisa de IP público (auto-assign ou EIP). Todos esses passos são necessários.

</details>

---

**2.** Você criou uma função Lambda que tenta conectar a um RDS em uma subnet privada, mas recebe timeout. O Lambda está na mesma VPC. Qual é o primeiro item a verificar?

- A) O timeout configurado na Lambda (muito baixo)
- B) Se o Security Group do RDS permite tráfego da porta 3306 proveniente do Security Group da Lambda
- C) Se a Lambda tem memória suficiente
- D) Se o RDS está em Multi-AZ

<details><summary>Resposta</summary>

**B** — Para Lambda em VPC acessar RDS: o SG do RDS precisa ter uma inbound rule permitindo a porta do banco (MySQL: 3306, PostgreSQL: 5432) proveniente do SG da Lambda (referenciado pelo Security Group ID, não por IP). Verificar SGs é sempre o primeiro passo em problemas de conectividade.

</details>

---

**3.** Você executou `terraform apply` e criou recursos AWS. Ao tentar destruí-los com `terraform destroy`, você recebe um erro informando que o S3 bucket não pode ser deletado porque não está vazio. Qual é a solução correta no Terraform?

- A) Deletar os objetos manualmente no console S3 antes de rodar `terraform destroy`
- B) Adicionar `force_destroy = true` no recurso `aws_s3_bucket` e re-aplicar antes de destruir
- C) Usar `terraform state rm aws_s3_bucket.nome` para remover do state
- D) Criar uma lifecycle rule no bucket para expirar os objetos

<details><summary>Resposta</summary>

**B** — `force_destroy = true` no resource `aws_s3_bucket` permite que o Terraform delete o bucket mesmo com objetos. Faça `terraform apply` para atualizar o recurso com essa configuração, depois `terraform destroy`. Remover do state (C) deixa o recurso órfão na AWS (não é deletado).

</details>

---

**4.** Você quer testar se uma política IAM está funcionando corretamente antes de associá-la a um usuário. Qual ferramenta AWS CLI usar?

- A) `aws iam simulate-principal-policy`
- B) `aws iam get-policy`
- C) `aws iam list-attached-user-policies`
- D) `aws sts assume-role`

<details><summary>Resposta</summary>

**A** — `aws iam simulate-principal-policy`: simula o resultado de autorização IAM para um principal específico realizando ações específicas em recursos específicos. Retorna Allow ou Deny sem fazer a ação. Perfeito para testar políticas antes de aplicar. `simulate-custom-policy` testa uma policy em JSON sem usuário específico.

</details>

---

**5.** Você está criando um módulo Terraform para um bucket S3. O módulo deve aceitar o nome do bucket como variável obrigatória e criar o bucket com versioning habilitado. Qual é a sintaxe correta?

- A) 
```hcl
resource "aws_s3_bucket" "main" {
  bucket = var.bucket_name
}
resource "aws_s3_bucket_versioning" "main" {
  bucket = aws_s3_bucket.main.id
  versioning_configuration { status = "Enabled" }
}
```
- B) `aws s3api create-bucket --bucket ${var.bucket_name} --versioning Enabled`
- C) `resource "s3" "bucket" { name = var.bucket_name; versioning = true }`
- D) Nenhuma das anteriores

<details><summary>Resposta</summary>

**A** — Na sintaxe Terraform AWS Provider 4.x+, o `aws_s3_bucket_versioning` é um recurso separado (não mais um bloco dentro do `aws_s3_bucket`). A configuração em (A) é a correta e moderna. A opção (C) tem sintaxe inválida.

</details>

---

**6.** Após deployar uma aplicação ECS Fargate, você percebe que os containers ficam reiniciando constantemente. Qual é o primeiro lugar para investigar?

- A) VPC Flow Logs
- B) CloudWatch Logs do container (task definition deve ter logConfiguration com awslogs driver)
- C) AWS X-Ray traces
- D) CloudTrail para o serviço ECS

<details><summary>Resposta</summary>

**B** — Containers em loop de restart geralmente indicam erro na aplicação ou misconfiguration. O log do container (stdout/stderr) no CloudWatch Logs é o primeiro lugar. A task definition deve ter `logConfiguration: awslogs` apontando para um Log Group. Sem isso, os logs são perdidos e debugging é impossível.

</details>

---

**7.** Você está usando AWS CLI e recebe o erro `An error occurred (AccessDenied) when calling the DescribeInstances operation`. Como diagnosticar qual permissão está faltando?

- A) Verificar `aws iam list-attached-user-policies` para ver as policies do usuário
- B) Usar `aws iam simulate-principal-policy --action-names ec2:DescribeInstances` para simular a autorização
- C) Verificar `aws sts get-caller-identity` para confirmar qual identidade está sendo usada, depois verificar as policies dessa identidade
- D) C é o primeiro passo; depois B para confirmar a política exata que está negando

<details><summary>Resposta</summary>

**D** — Fluxo correto: primeiro `aws sts get-caller-identity` para confirmar que você está autenticado como o identity correto (não como outra role). Depois `simulate-principal-policy` para entender qual statement está negando a ação. `list-attached-user-policies` não mostra policies de grupos ou inline policies.

</details>

---

**8.** Você criou um RDS MySQL com `--no-publicly-accessible` em uma subnet privada. A aplicação em EC2 na mesma VPC não consegue conectar. O que verificar? (Selecione 2)

- A) O Security Group do RDS permite inbound na porta 3306 do Security Group do EC2
- B) O EC2 está na mesma VPC que o RDS (ou VPCs conectadas por peering/TGW)
- C) O RDS tem Elastic IP associado
- D) A subnet do RDS tem uma rota para o Internet Gateway

<details><summary>Resposta</summary>

**A e B** — Para conectividade EC2 → RDS na mesma VPC: (1) ambos na mesma VPC; (2) SG do RDS permite porta 3306 do SG do EC2. RDS privado não precisa de EIP (C) nem IGW route (D) — esses são para acesso público.

</details>

---

**9.** Você quer fazer uma query no DynamoDB retornando apenas itens onde `status = "active"` e `age > 25`. A tabela tem `userId` como partition key e `createdAt` como sort key. Qual operação DynamoDB usar?

- A) GetItem com FilterExpression
- B) Query com FilterExpression
- C) Scan com FilterExpression (lê a tabela inteira e filtra)
- D) Batch GetItem

<details><summary>Resposta</summary>

**C** — `status` e `age` não são chaves da tabela nem de nenhum GSI/LSI descrito. Query requer especificar a partition key. Scan lê a tabela inteira e aplica FilterExpression (funciona mas caro para tabelas grandes). A solução ideal seria criar um GSI com `status` como partition key e então fazer Query, mas com os índices descritos, Scan é a única opção.

</details>

---

**10.** Você criou uma Lambda function com o código funcionando localmente, mas ao invocar na AWS recebe `Runtime.ImportModuleError: No module named 'boto3'`. Qual é a causa provável?

- A) Boto3 precisa ser instalado separadamente em Lambdas Python
- B) O zip de deploy não inclui as dependências corretamente
- C) O runtime Python selecionado não suporta boto3
- D) Boto3 já está incluído no runtime Lambda Python; o problema é outro módulo que está sendo importado

<details><summary>Resposta</summary>

**D** — Boto3 JÁ está incluído no runtime Lambda Python (pré-instalado pela AWS). Se o erro diz `No module named 'boto3'`, na verdade é possível que o erro seja de outro módulo listado antes, ou que o zip foi criado incorretamente substituindo o boto3 do runtime por uma versão local incompleta. Verifique se o deployment package está correto.

</details>

---

**11.** Após criar um CloudFront distribution com um S3 bucket como origem, você ainda consegue acessar os objetos S3 diretamente via URL do bucket, bypassando o CloudFront. Como resolver?

- A) Configurar bucket policy para negar acesso de todos exceto do CloudFront OAC (Origin Access Control)
- B) Desabilitar o static website hosting no S3
- C) Configurar CORS no bucket S3
- D) Usar S3 pre-signed URLs ao invés do CloudFront

<details><summary>Resposta</summary>

**A** — OAC (Origin Access Control): cria uma identidade no CloudFront. O bucket policy é atualizado para permitir apenas GetObject do Principal `cloudfront.amazonaws.com` com condição `AWS:SourceArn` do distribution. Qualquer acesso direto ao S3 URL retorna 403.

</details>

---

**12.** Você quer monitorar quantas requisições com status 5xx sua API Lambda está retornando e criar um alarme quando ultrapassar 10 em 5 minutos. Qual sequência de comandos CLI implementa isso?

- A) Criar Metric Filter no Log Group + criar Alarme na métrica customizada
- B) Usar métricas padrão do API Gateway (`5XXError`) diretamente para criar o Alarme
- C) Criar CloudTrail Event Rule para status 500
- D) Usar X-Ray com threshold de erros

<details><summary>Resposta</summary>

**B** — API Gateway publica automaticamente a métrica `5XXError` no CloudWatch (namespace `AWS/ApiGateway`). Crie o Alarm diretamente nessa métrica: `aws cloudwatch put-metric-alarm --alarm-name api-5xx --namespace AWS/ApiGateway --metric-name 5XXError --dimensions Name=ApiName,Value=minha-api --period 300 --evaluation-periods 1 --threshold 10 --comparison-operator GreaterThanThreshold --statistic Sum`.

</details>

---

**13.** Você fez `terraform apply` mas o recurso `aws_iam_role_policy_attachment` não está no seu código. O Terraform está mostrando esse recurso no state (`terraform state list`). O que acontece se você faz `terraform apply` novamente?

- A) O Terraform vai deletar o role policy attachment porque não está no código
- B) O Terraform não faz nada com recursos que não estão no código (apenas gerencia o que está no code)
- C) Precisaria de `terraform state rm` para remover o recurso do state
- D) O Terraform vai mostrar erro sobre recurso órfão

<details><summary>Resposta</summary>

**A** — Terraform gerencia recursos declarativos: o estado desejado = o que está no código. Se um recurso está no state mas não no código, o próximo `terraform apply` vai deletar esse recurso para alinhar o state com o código. Para remover do state sem deletar o recurso real: `terraform state rm resource.name`.

</details>

---

**14.** Um S3 bucket event notification está configurado para invocar uma Lambda quando objetos são criados. Mas ao testar, a Lambda não é invocada. Quais dois itens verificar?

- A) Resource Policy da Lambda permitindo que S3 invoque (Principal: s3.amazonaws.com) + Notification configurada no bucket correto
- B) IAM Role da Lambda com s3:GetObject + SG da Lambda
- C) Bucket versioning habilitado + Lambda em mesma região do bucket
- D) CloudWatch Logs habilitado na Lambda + S3 Access Logging

<details><summary>Resposta</summary>

**A** — Para S3 invocar Lambda: (1) Resource-based policy na Lambda (`aws lambda add-permission`) permitindo `lambda:InvokeFunction` com Principal `s3.amazonaws.com` e condition `ArnEquals` no bucket ARN; (2) Notification configuration no bucket apontando para o ARN da Lambda correta. Sem a resource policy, S3 recebe access denied ao tentar invocar.

</details>

---

**15.** Você está usando `aws s3 sync` para backup de arquivos de um servidor para S3. O comando inclui arquivos desnecessários (node_modules, .git). Como excluir esses diretórios?

- A) `aws s3 sync . s3://meu-bucket --exclude "node_modules/*" --exclude ".git/*"`
- B) `aws s3 sync . s3://meu-bucket --ignore node_modules .git`
- C) Criar um arquivo .s3ignore
- D) `aws s3 sync . s3://meu-bucket --filter "node_modules"`

<details><summary>Resposta</summary>

**A** — `--exclude` com glob pattern: `aws s3 sync . s3://meu-bucket --exclude "node_modules/*" --exclude ".git/*"`. Cada exclusão é um `--exclude` separado. O padrão usa glob (* e **). Nota: se quiser incluir apenas certos tipos, use `--exclude "*" --include "*.js"`. S3 não tem arquivo .s3ignore.

</details>
