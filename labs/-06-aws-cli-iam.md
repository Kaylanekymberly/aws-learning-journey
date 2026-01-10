# Lab 05 - AWS CLI e Gerenciamento IAM

**Autor:** Kaylanekymberly  
**Data:** 10 de janeiro de 2026  
**Status:** ✅ Concluído  
**Repositório:** [aws-learning-journey/labs](https://github.com/kaylanekymberly/aws-learning-journey)

---

##  Índice

1. [Visão Geral](#visão-geral)
2. [Objetivos do Lab](#objetivos-do-lab)
3. [Pré-requisitos](#pré-requisitos)
4. [Arquitetura e Conceitos](#arquitetura-e-conceitos)
5. [Passo a Passo Detalhado](#passo-a-passo-detalhado)
6. [Comandos AWS CLI Utilizados](#comandos-aws-cli-utilizados)
7. [Minhas Experiências](#minhas-experiências)
8. [Desafios Enfrentados](#desafios-enfrentados)
9. [Lições Aprendidas](#lições-aprendidas)
10. [Recursos Adicionais](#recursos-adicionais)

---

##  Visão Geral

Este laboratório foca em operações práticas com AWS CLI e gerenciamento de identidade e acesso (IAM). Durante o lab, foram executadas tarefas fundamentais de administração em ambientes AWS, incluindo verificação de permissões, execução de comandos em múltiplas instâncias e atualização de configurações.

### Tarefas Concluídas

- ✅ Verificar configurações e permissões IAM
- ✅ Executar tarefas em vários servidores EC2
- ✅ Atualizar configurações de aplicação via AWS CLI
- ✅ Acessar linha de comando em instâncias EC2

---

##  Objetivos do Lab

### Objetivos Principais

1. **Dominar AWS CLI**: Aprender comandos essenciais para gerenciamento de recursos AWS
2. **Gerenciar IAM**: Configurar e validar políticas de permissão
3. **Automatizar Tarefas**: Executar operações em múltiplas instâncias simultaneamente
4. **Troubleshooting**: Resolver problemas de acesso e configuração

### Competências Desenvolvidas

- AWS CLI configuration e autenticação
- IAM roles, policies e permissions
- EC2 instance management
- Systems Manager (SSM) para acesso remoto
- Scripting e automação de tarefas AWS

---

##  Pré-requisitos

### Conhecimentos Necessários

- Conceitos básicos de AWS (EC2, IAM, S3)
- Linha de comando Linux/Bash
- Noções de JSON para políticas IAM
- Conceitos de redes (VPC, Security Groups)

### Ferramentas Necessárias

```bash
# AWS CLI v2
aws --version

# Credenciais configuradas
aws configure list

# Session Manager plugin (opcional)
session-manager-plugin --version
```

### Permissões IAM Necessárias

- `iam:GetUser`
- `iam:ListAttachedUserPolicies`
- `ec2:DescribeInstances`
- `ssm:StartSession`
- `ssm:SendCommand`

---

## Arquitetura e Conceitos

### Diagrama do Lab

```
┌─────────────────────────────────────────────────────┐
│                    AWS Account                       │
│                                                      │
│  ┌──────────────┐         ┌──────────────┐         │
│  │   IAM User   │────────▶│  IAM Role    │         │
│  │  (Você)      │         │ (EC2 Role)   │         │
│  └──────────────┘         └──────────────┘         │
│         │                        │                  │
│         │ AWS CLI                │                  │
│         ▼                        ▼                  │
│  ┌──────────────────────────────────────┐          │
│  │         EC2 Instances                │          │
│  │  ┌──────┐  ┌──────┐  ┌──────┐       │          │
│  │  │ Web1 │  │ Web2 │  │ Web3 │       │          │
│  │  └──────┘  └──────┘  └──────┘       │          │
│  └──────────────────────────────────────┘          │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Conceitos-Chave

**IAM (Identity and Access Management)**
- Serviço que controla acesso aos recursos AWS
- Usa princípio de menor privilégio
- Composto por Users, Groups, Roles e Policies

**AWS CLI**
- Interface de linha de comando para gerenciar serviços AWS
- Permite automação e scripting
- Usa credenciais IAM para autenticação

**Systems Manager (SSM)**
- Gerenciamento centralizado de instâncias
- Permite acesso sem SSH direto
- Execução de comandos em múltiplas instâncias

---

##  Passo a Passo Detalhado

### Etapa 1: Verificar Configurações e Permissões

#### 1.1. Configurar AWS CLI

```bash
# Configurar credenciais AWS
aws configure

# Entradas necessárias:
# AWS Access Key ID: [Sua Access Key]
# AWS Secret Access Key: [Sua Secret Key]
# Default region name: us-east-1
# Default output format: json
```

#### 1.2. Verificar Identidade

```bash
# Verificar usuário atual
aws sts get-caller-identity

# Saída esperada:
# {
#     "UserId": "AIDAXXXXXXXXXXXXXXXX",
#     "Account": "123456789012",
#     "Arn": "arn:aws:iam::123456789012:user/seu-usuario"
# }
```

#### 1.3. Listar Políticas Anexadas

```bash
# Listar políticas do usuário
aws iam list-attached-user-policies --user-name seu-usuario

# Verificar detalhes de uma política específica
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

#### 1.4. Testar Permissões

```bash
# Testar permissão de listar instâncias EC2
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table

# Se retornar erro, verificar políticas IAM
```

---

### Etapa 2: Executar Tarefas em Vários Servidores

#### 2.1. Listar Instâncias EC2

```bash
# Listar todas as instâncias em execução
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].[InstanceId,PrivateIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# Salvar IDs das instâncias em arquivo
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text > instance-ids.txt
```

#### 2.2. Executar Comando Único via SSM

```bash
# Executar comando simples em uma instância
aws ssm send-command \
  --instance-ids "i-1234567890abcdef0" \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["echo Hello from SSM", "whoami", "pwd"]' \
  --output text

# Verificar status do comando
aws ssm list-command-invocations \
  --command-id "comando-id-aqui" \
  --details
```

#### 2.3. Script para Execução em Múltiplas Instâncias

```bash
#!/bin/bash
# Arquivo: run-on-multiple-instances.sh

# Ler IDs das instâncias
INSTANCE_IDS=$(aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

# Comando a ser executado
COMMAND="df -h && free -m && uptime"

# Executar em todas as instâncias
COMMAND_ID=$(aws ssm send-command \
  --instance-ids $INSTANCE_IDS \
  --document-name "AWS-RunShellScript" \
  --parameters "commands=['$COMMAND']" \
  --output text \
  --query 'Command.CommandId')

echo "Comando enviado. ID: $COMMAND_ID"

# Aguardar conclusão
sleep 5

# Verificar resultados
for instance in $INSTANCE_IDS; do
  echo "=== Resultados para $instance ==="
  aws ssm get-command-invocation \
    --command-id $COMMAND_ID \
    --instance-id $instance \
    --query 'StandardOutputContent' \
    --output text
done
```

#### 2.4. Executar o Script

```bash
chmod +x run-on-multiple-instances.sh
./run-on-multiple-instances.sh
```

---

### Etapa 3: Atualizar Configurações da Aplicação

#### 3.1. Backup de Configurações via S3

```bash
# Criar bucket S3 para backups (se não existir)
aws s3 mb s3://meu-bucket-configs-backup-$(date +%Y%m%d)

# Fazer backup da configuração atual de todas as instâncias
INSTANCES=$(aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

for instance in $INSTANCES; do
  aws ssm send-command \
    --instance-ids $instance \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["sudo tar -czf /tmp/config-backup.tar.gz /etc/app/config.yml", "aws s3 cp /tmp/config-backup.tar.gz s3://meu-bucket-configs-backup-'$(date +%Y%m%d)'/config-$INSTANCE_ID-'$(date +%Y%m%d_%H%M%S)'.tar.gz"]'
done
```

#### 3.2. Atualizar Configuração Remotamente

```bash
# Script para atualizar configuração
cat > update-config.sh << 'EOF'
#!/bin/bash
# Fazer backup
sudo cp /etc/app/config.yml /etc/app/config.yml.backup

# Atualizar configuração
sudo sed -i 's/port: 8080/port: 8081/g' /etc/app/config.yml
sudo sed -i 's/debug: false/debug: true/g' /etc/app/config.yml

# Validar sintaxe
python3 -c "import yaml; yaml.safe_load(open('/etc/app/config.yml'))"

# Reiniciar aplicação
sudo systemctl restart myapp.service

# Verificar status
sudo systemctl status myapp.service
EOF

# Fazer upload do script para S3
aws s3 cp update-config.sh s3://meu-bucket-scripts/update-config.sh

# Executar em todas as instâncias
aws ssm send-command \
  --instance-ids $(cat instance-ids.txt) \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["aws s3 cp s3://meu-bucket-scripts/update-config.sh /tmp/", "chmod +x /tmp/update-config.sh", "/tmp/update-config.sh"]'
```

#### 3.3. Validar Mudanças

```bash
# Verificar se a aplicação está rodando em todas as instâncias
for instance in $(cat instance-ids.txt); do
  echo "=== Verificando $instance ==="
  aws ssm send-command \
    --instance-ids $instance \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["sudo systemctl status myapp.service", "curl -s http://localhost:8081/health"]' \
    --query 'Command.CommandId' \
    --output text
done
```

---

### Etapa 4: Acessar Linha de Comando em uma Instância

#### 4.1. Sessão Interativa com Session Manager

```bash
# Iniciar sessão interativa
aws ssm start-session --target i-1234567890abcdef0

# Você estará dentro da instância!
# Comandos disponíveis:
sh-4.2$ whoami
sh-4.2$ sudo -i
root@ip-10-0-1-100:~#
```

#### 4.2. Port Forwarding para Aplicação Local

```bash
# Redirecionar porta da instância para sua máquina local
aws ssm start-session \
  --target i-1234567890abcdef0 \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["8080"],"localPortNumber":["8080"]}'

# Agora você pode acessar: http://localhost:8080
```

#### 4.3. Transferir Arquivos via S3

```bash
# Da sua máquina para a instância
aws s3 cp arquivo-local.txt s3://temp-bucket/arquivo.txt

# Na instância (via SSM)
aws ssm send-command \
  --instance-ids i-1234567890abcdef0 \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["aws s3 cp s3://temp-bucket/arquivo.txt /tmp/arquivo.txt"]'

# Da instância para sua máquina
aws ssm send-command \
  --instance-ids i-1234567890abcdef0 \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["aws s3 cp /var/log/app.log s3://temp-bucket/app.log"]'

aws s3 cp s3://temp-bucket/app.log ./app.log
```

#### 4.4. Comandos Úteis Durante a Sessão

```bash
# Monitorar logs em tempo real
sudo tail -f /var/log/messages

# Verificar recursos
top
htop
free -m
df -h

# Verificar conexões de rede
sudo netstat -tlnp
sudo ss -tlnp

# Verificar processos da aplicação
ps aux | grep myapp
systemctl status myapp

# Verificar logs da aplicação
sudo journalctl -u myapp -f
```

---

##  Comandos AWS CLI Utilizados

### Comandos IAM

```bash
# Identidade
aws sts get-caller-identity
aws sts get-session-token

# Usuários e políticas
aws iam get-user --user-name nome-usuario
aws iam list-attached-user-policies --user-name nome-usuario
aws iam get-policy --policy-arn arn-da-policy
aws iam get-policy-version --policy-arn arn-da-policy --version-id v1

# Roles
aws iam list-roles
aws iam get-role --role-name nome-role
```

### Comandos EC2

```bash
# Listar instâncias
aws ec2 describe-instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 describe-instances --instance-ids i-xxx i-yyy

# Informações de rede
aws ec2 describe-security-groups
aws ec2 describe-vpcs
aws ec2 describe-subnets

# Tags
aws ec2 create-tags --resources i-xxx --tags Key=Name,Value=WebServer
```

### Comandos SSM

```bash
# Session Manager
aws ssm start-session --target i-xxx
aws ssm terminate-session --session-id session-xxx

# Run Command
aws ssm send-command --document-name documento --instance-ids i-xxx
aws ssm list-commands
aws ssm get-command-invocation --command-id cmd-xxx --instance-id i-xxx

# Parameter Store
aws ssm get-parameter --name /app/config/database
aws ssm put-parameter --name /app/config/port --value "8081"
```

### Comandos S3

```bash
# Operações básicas
aws s3 ls
aws s3 mb s3://meu-bucket
aws s3 cp arquivo.txt s3://meu-bucket/
aws s3 sync ./diretorio/ s3://meu-bucket/pasta/
```

---

##  Minhas Experiências

### Momento "Aha!"

Durante o lab, tive um momento de clareza quando entendi que **o AWS CLI é simplesmente uma interface para as APIs AWS**. Cada comando que executamos está fazendo uma chamada HTTP para os serviços da AWS. Isso mudou completamente minha perspectiva sobre automação!

### O que Funcionou Muito Bem

1. **Session Manager vs SSH**: Usar o SSM Session Manager foi revolucionário! Não precisei gerenciar chaves SSH nem abrir portas no Security Group. A segurança melhorou significativamente.

2. **Automação com Scripts**: Criar scripts bash combinados com AWS CLI tornou tarefas repetitivas muito mais eficientes. O que levaria 30 minutos manualmente, agora leva 2 minutos.

3. **JSON Queries**: Dominar o parâmetro `--query` do AWS CLI foi essencial. Poder filtrar e formatar a saída diretamente no comando economizou muito tempo.

### Descobertas Interessantes

- **IAM é o coração da AWS**: Tudo começa com permissões corretas
- **Tags são essenciais**: Sem tags adequadas, gerenciar recursos vira um pesadelo
- **Dry-run é seu amigo**: Sempre teste comandos destrutivos com `--dry-run`

---

##  Desafios Enfrentados

### Desafio 1: Credenciais AWS CLI Não Funcionavam

**Situação:** Após configurar `aws configure`, recebia erro `UnauthorizedOperation`

**Causa Raiz:** 
- Credenciais estavam corretas, mas a política IAM não tinha as permissões necessárias
- Estava usando `PowerUserAccess` que não inclui permissões IAM

**Solução:**
```bash
# Verificar políticas atuais
aws iam list-attached-user-policies --user-name meu-usuario

# Solicitar ao administrador para adicionar política
# Ou criar política customizada com permissões específicas
```

**Lição:** Sempre verificar permissões IAM antes de começar. Use `aws sts get-caller-identity` para confirmar identidade.

---

### Desafio 2: Session Manager Não Conectava

**Situação:** Comando `aws ssm start-session` retornava erro de timeout

**Causa Raiz:**
- Instância EC2 não tinha o SSM Agent instalado
- IAM Role da instância não tinha permissões SSM
- Security Group bloqueava tráfego HTTPS (porta 443)

**Solução:**
```bash
# 1. Verificar se SSM Agent está rodando
aws ssm send-command \
  --instance-ids i-xxx \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["sudo systemctl status amazon-ssm-agent"]'

# 2. Instalar SSM Agent (se necessário)
# Ubuntu/Debian:
sudo snap install amazon-ssm-agent --classic
sudo systemctl start amazon-ssm-agent

# 3. Verificar IAM Role
aws ec2 describe-instances \
  --instance-ids i-xxx \
  --query 'Reservations[*].Instances[*].IamInstanceProfile'

# 4. Anexar política necessária ao role
aws iam attach-role-policy \
  --role-name EC2-SSM-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

**Lição:** Session Manager requer configuração em 3 camadas: Agent, IAM Role e Network (HTTPS saída).

---

### Desafio 3: Comandos em Múltiplas Instâncias Falhavam Parcialmente

**Situação:** Script executava em algumas instâncias mas falhava em outras

**Causa Raiz:**
- Instâncias tinham configurações diferentes (umas Ubuntu, outras Amazon Linux)
- Caminhos de arquivos não eram consistentes
- Algumas instâncias não tinham Python instalado

**Solução:**
```bash
# Script adaptativo que detecta o OS
#!/bin/bash

# Detectar sistema operacional
if [ -f /etc/os-release ]; then
    . /etc/os-release
    OS=$ID
fi

# Comandos específicos por OS
case $OS in
    ubuntu|debian)
        CONFIG_PATH="/etc/myapp/config.yml"
        SERVICE_CMD="systemctl"
        ;;
    amzn|amazonlinux)
        CONFIG_PATH="/etc/myapp/app.conf"
        SERVICE_CMD="systemctl"
        ;;
    *)
        echo "OS não suportado: $OS"
        exit 1
        ;;
esac

# Verificar pré-requisitos
command -v python3 >/dev/null 2>&1 || { echo "Python3 não instalado"; exit 1; }

# Executar operações
echo "Processando configuração em $CONFIG_PATH..."
```

**Lição:** Sempre considere heterogeneidade do ambiente. Crie scripts robustos que verificam pré-requisitos.

---

### Desafio 4: Rate Limiting e Throttling

**Situação:** Ao executar scripts em muitas instâncias, recebia erro `ThrottlingException`

**Causa Raiz:**
- AWS CLI tem limites de taxa para APIs
- Executar comandos paralelos em 50+ instâncias excedeu o limite

**Solução:**
```bash
#!/bin/bash
# Script com controle de rate limiting

INSTANCES=$(aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

# Processar em lotes
BATCH_SIZE=10
DELAY=5

count=0
batch=""

for instance in $INSTANCES; do
    batch="$batch $instance"
    count=$((count + 1))
    
    # Quando atingir o tamanho do lote
    if [ $count -eq $BATCH_SIZE ]; then
        echo "Processando lote: $batch"
        aws ssm send-command \
          --instance-ids $batch \
          --document-name "AWS-RunShellScript" \
          --parameters 'commands=["echo Processing..."]'
        
        echo "Aguardando $DELAY segundos..."
        sleep $DELAY
        
        # Reset
        count=0
        batch=""
    fi
done

# Processar instâncias restantes
if [ ! -z "$batch" ]; then
    echo "Processando lote final: $batch"
    aws ssm send-command \
      --instance-ids $batch \
      --document-name "AWS-RunShellScript" \
      --parameters 'commands=["echo Processing..."]'
fi
```

**Lição:** Sempre implemente batch processing e delays quando trabalhar em escala.

---

### Desafio 5: Logs e Troubleshooting

**Situação:** Difícil rastrear falhas quando comandos SSM não executavam corretamente

**Causa Raiz:**
- Saída de comandos não era salva automaticamente
- Erros aconteciam mas não eram visíveis

**Solução:**
```bash
# Criar função de logging melhorada
run_command_with_logging() {
    local instances=$1
    local command=$2
    local log_dir="./logs/$(date +%Y%m%d_%H%M%S)"
    
    mkdir -p $log_dir
    
    # Executar comando
    COMMAND_ID=$(aws ssm send-command \
        --instance-ids $instances \
        --document-name "AWS-RunShellScript" \
        --parameters "commands=['$command']" \
        --output-s3-bucket-name meu-bucket-logs \
        --output-s3-key-prefix ssm-logs/ \
        --query 'Command.CommandId' \
        --output text)
    
    echo "Command ID: $COMMAND_ID" | tee $log_dir/command-info.txt
    
    # Aguardar conclusão
    sleep 10
    
    # Coletar resultados
    for instance in $instances; do
        echo "=== Logs para $instance ===" | tee -a $log_dir/${instance}.log
        aws ssm get-command-invocation \
            --command-id $COMMAND_ID \
            --instance-id $instance \
            --output json | tee -a $log_dir/${instance}.log
    done
    
    echo "Logs salvos em: $log_dir"
}

# Usar a função
run_command_with_logging "i-xxx i-yyy" "df -h && uptime"
```

**Lição:** Sempre implemente logging robusto. Salve outputs em S3 e localmente para análise posterior.

---

##  Lições Aprendidas

### Boas Práticas AWS CLI

1. **Sempre use --dry-run quando disponível**
   ```bash
   aws ec2 terminate-instances --instance-ids i-xxx --dry-run
   ```

2. **Use --query para filtrar saídas**
   ```bash
   aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name]'
   ```

3. **Defina --output para facilitar parsing**
   ```bash
   # Para scripts: use json ou text
   aws s3 ls --output text | awk '{print $3}'
   
   # Para visualização: use table
   aws ec2 describe-instances --output table
   ```

4. **Configure perfis para múltiplas contas**
   ```bash
   # ~/.aws/config
   [profile producao]
   region = us-east-1
   output = json
   
   [profile desenvolvimento]
   region = us-west-2
   output = json
   
   # Usar perfil
   aws s3 ls --profile producao
   ```

### Segurança IAM

- **Princípio do Menor Privilégio**: Sempre conceda apenas as permissões necessárias
- **Use Roles em vez de Keys**: Para EC2, Lambda, ECS, sempre prefira IAM Roles
- **Rotacione Credenciais**: Configure rotação automática de access keys
- **Habilite MFA**: Para usuários com privilégios elevados
- **Audite Permissões**: Revise regularmente quem tem acesso a quê

### Automação

```bash
# Template de script robusto
#!/bin/bash
set -euo pipefail  # Fail fast

# Cores para output
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Logging
log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')]${NC} $1"
}

error() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
}

# Verificar pré-requisitos
command -v aws >/dev/null 2>&1 || { error "AWS CLI não instalado"; exit 1; }
aws sts get-caller-identity >/dev/null 2>&1 || { error "Credenciais AWS inválidas"; exit 1; }

# Seu código aqui
log "Iniciando script..."
```

---

##  Recursos Adicionais

### Documentação Oficial

- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Systems Manager User Guide](https://docs.aws.amazon.com/systems-manager/)


##  Notas Finais

Este foi um lab extremamente valioso que estabeleceu as fundações para automação avançada na AWS. As habilidades adquiridas aqui - especialmente no uso de AWS CLI e IAM - são aplicáveis a praticamente todos os outros serviços AWS.

**Principais Takeaways:**
- AWS CLI é essencial para automação
- IAM é crítico para segurança
- Sempre teste em ambiente de desenvolvimento primeiro
- Documente tudo - seu eu futuro agradece!

---

**© 2026 Kaylanekymberly - Todos os direitos reservados**

*Esta documentação faz parte do repositório [aws-learning-journey](https://github.com/kaylanekymberly/aws-learning-journey) e não pode ser reproduzida sem autorização.*
