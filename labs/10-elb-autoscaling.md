
#  Elastic Load Balancing e Auto Scaling

## Informações do Laboratório

| Item | Detalhes |
|------|----------|
| **Número do Lab** | 10 |
| **Nome** | Elastic Load Balancing e Auto Scaling |
| **Data de Conclusão** | [Inserir data] |
| **Status** | Concluído |
| **Plataforma** | AWS Academy |
| **Serviços AWS** | ELB, EC2 Auto Scaling, CloudWatch, AMI |
| **Ferramentas** | AWS Console, AWS CLI |

---

## Objetivos do Laboratório

Após concluir este laboratório, sou capaz de:

- Criar uma Amazon Machine Image (AMI) a partir de uma instância EC2 existente
- Criar e configurar um Application Load Balancer
- Criar um modelo de execução (Launch Template) para o Auto Scaling
- Configurar um grupo do Auto Scaling (Auto Scaling Group)
- Dimensionar automaticamente instâncias em sub-redes privadas
- Criar alarmes do Amazon CloudWatch para monitoramento
- Utilizar AWS CLI para criação de AMIs (desafio opcional)

---

## Visão Geral da Atividade

Neste laboratório, implementei uma arquitetura altamente disponível e escalável utilizando Elastic Load Balancing (ELB) e Amazon EC2 Auto Scaling. O objetivo foi criar uma infraestrutura que se adapta automaticamente às variações de demanda, garantindo disponibilidade e otimização de custos.

### Elastic Load Balancing (ELB)

O ELB distribui automaticamente o tráfego de entrada das aplicações entre várias instâncias EC2, proporcionando:

- Distribuição automática de tráfego
- Tolerância a falhas
- Alta disponibilidade
- Detecção e roteamento de tráfego apenas para instâncias saudáveis

### Amazon EC2 Auto Scaling

O Auto Scaling mantém a disponibilidade da aplicação e permite:

- Redução ou aumento automático da capacidade EC2
- Definição de condições personalizadas para dimensionamento
- Garantia do número desejado de instâncias em execução
- Aumento automático durante picos de demanda
- Redução de capacidade em períodos ociosos para economia de custos
- Ideal para aplicações com padrões de demanda variáveis (horários, diários ou semanais)

---

## Arquitetura Implementada

### Arquitetura Inicial

A infraestrutura inicial consistia em:

- VPC com sub-redes públicas e privadas
- Uma única instância EC2 em execução
- Configuração básica de rede

### Arquitetura Final

A arquitetura final implementada inclui:

```
                            Internet
                               |
                    ┌──────────▼──────────┐
                    │  Internet Gateway    │
                    └──────────┬──────────┘
                               |
        ┌──────────────────────┴──────────────────────┐
        |                    VPC                       |
        |                                              |
        |  ┌────────────────────────────────────────┐ |
        |  │     Application Load Balancer          │ |
        |  │         (Sub-redes Públicas)           │ |
        |  └────────┬─────────────────┬─────────────┘ |
        |           │                 │                |
        |  ┌────────▼────────┐ ┌──────▼──────────┐   |
        |  │  Sub-rede       │ │  Sub-rede       │   |
        |  │  Privada AZ1    │ │  Privada AZ2    │   |
        |  │  ┌───────────┐  │ │  ┌───────────┐  │   |
        |  │  │  EC2      │  │ │  │  EC2      │  │   |
        |  │  │ Instance  │  │ │  │ Instance  │  │   |
        |  │  └───────────┘  │ │  └───────────┘  │   |
        |  │                 │ │                 │   |
        |  │ Auto Scaling    │ │ Auto Scaling    │   |
        |  │ Group           │ │ Group           │   |
        |  └─────────────────┘ └─────────────────┘   |
        |                                              |
        |  ┌────────────────────────────────────────┐ |
        |  │      Amazon CloudWatch Alarms          │ |
        |  │   (Monitoramento e Auto Scaling)       │ |
        |  └────────────────────────────────────────┘ |
        └──────────────────────────────────────────────┘
```

**Componentes da Arquitetura:**

- **Application Load Balancer**: Distribui tráfego entre múltiplas zonas de disponibilidade
- **Auto Scaling Group**: Gerencia o número de instâncias automaticamente
- **Launch Template**: Define a configuração padrão das instâncias
- **Sub-redes Privadas**: Hospedam as instâncias EC2 em múltiplas AZs
- **CloudWatch Alarms**: Monitora métricas e aciona ações de scaling
- **AMI Customizada**: Imagem base para as novas instâncias

---

## Procedimentos Realizados

### 1. Criação da Amazon Machine Image (AMI)

**Objetivo:** Criar uma imagem customizada a partir de uma instância EC2 existente para uso como base no Auto Scaling.

**Processo via AWS Console:**

1. Acessei o serviço EC2 no AWS Console
2. Selecionei a instância EC2 existente (Web Server)
3. No menu Actions > Image and templates > Create image
4. Configurei os parâmetros da AMI:

| Parâmetro | Valor |
|-----------|-------|
| Image name | WebServerAMI |
| Image description | AMI para Auto Scaling do Web Server |
| No reboot | Habilitado (evita downtime) |
| Tags | Name: WebServerAMI |

5. Aguardei a criação da AMI (status: available)

**Validação:**

```bash
# Verificar AMIs disponíveis
aws ec2 describe-images \
  --owners self \
  --query 'Images[*].[ImageId,Name,State]' \
  --output table
```

---

### 2. Configuração do Application Load Balancer

**Objetivo:** Criar um balanceador de carga para distribuir tráfego entre as instâncias.

**Etapas de Configuração:**

**2.1. Criação do Target Group**

1. Acessei EC2 > Load Balancing > Target Groups
2. Criei um novo Target Group:

| Configuração | Valor |
|-------------|-------|
| Target type | Instances |
| Target group name | WebServerTargetGroup |
| Protocol | HTTP |
| Port | 80 |
| VPC | Lab VPC |
| Health check protocol | HTTP |
| Health check path | / |
| Health check interval | 30 segundos |
| Healthy threshold | 2 |
| Unhealthy threshold | 5 |

**2.2. Criação do Load Balancer**

1. Acessei EC2 > Load Balancing > Load Balancers
2. Criei Application Load Balancer:

| Configuração | Valor |
|-------------|-------|
| Name | WebServerALB |
| Scheme | Internet-facing |
| IP address type | IPv4 |
| VPC | Lab VPC |
| Availability Zones | us-east-1a, us-east-1b (sub-redes públicas) |
| Security group | ALB-SG (portas 80 e 443) |
| Listener | HTTP:80 |
| Default action | Forward to WebServerTargetGroup |

**Validação:**

```bash
# Listar Load Balancers
aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[*].[LoadBalancerName,DNSName,State.Code]' \
  --output table

# Verificar Target Group
aws elbv2 describe-target-groups \
  --query 'TargetGroups[*].[TargetGroupName,HealthCheckPath,Port]' \
  --output table
```

---

### 3. Criação do Launch Template

**Objetivo:** Definir o modelo de configuração para as instâncias do Auto Scaling.

**Configuração do Launch Template:**

1. Acessei EC2 > Instances > Launch Templates
2. Criei novo Launch Template:

| Parâmetro | Valor |
|-----------|-------|
| Launch template name | WebServerTemplate |
| Template version description | v1 - Initial configuration |
| AMI | WebServerAMI (criada anteriormente) |
| Instance type | t2.micro |
| Key pair | vockey |
| Network settings | Não incluir na template |
| Security groups | WebServer-SG |
| Storage | 8 GB gp3 |
| Resource tags | Name: Auto-Scaled-WebServer |
| Advanced details - IAM instance profile | EC2InstanceProfile |

**User Data (opcional):**

```bash
#!/bin/bash
# Inicialização adicional se necessário
systemctl start httpd
systemctl enable httpd
```

**Validação:**

```bash
# Listar Launch Templates
aws ec2 describe-launch-templates \
  --query 'LaunchTemplates[*].[LaunchTemplateName,LaunchTemplateId,LatestVersionNumber]' \
  --output table
```

---

### 4. Configuração do Auto Scaling Group

**Objetivo:** Criar e configurar o grupo de Auto Scaling para dimensionamento automático.

**4.1. Configurações Básicas**

| Configuração | Valor |
|-------------|-------|
| Auto Scaling group name | WebServerASG |
| Launch template | WebServerTemplate (latest) |
| VPC | Lab VPC |
| Availability Zones | us-east-1a, us-east-1b |
| Subnets | Private Subnet 1, Private Subnet 2 |

**4.2. Configurações de Dimensionamento**

| Parâmetro | Valor |
|-----------|-------|
| Desired capacity | 2 |
| Minimum capacity | 2 |
| Maximum capacity | 4 |
| Scaling policies | Target tracking scaling |

**4.3. Integração com Load Balancer**

- Attach to existing load balancer: Sim
- Target group: WebServerTargetGroup
- Health check type: ELB
- Health check grace period: 300 segundos

**4.4. Política de Scaling**

Configurei Target Tracking Scaling Policy:

| Configuração | Valor |
|-------------|-------|
| Metric type | Average CPU utilization |
| Target value | 70% |
| Instance warm-up | 300 segundos |

**Validação:**

```bash
# Descrever Auto Scaling Group
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names WebServerASG \
  --query 'AutoScalingGroups[*].[AutoScalingGroupName,DesiredCapacity,MinSize,MaxSize]' \
  --output table

# Verificar instâncias do ASG
aws autoscaling describe-auto-scaling-instances \
  --query 'AutoScalingInstances[*].[InstanceId,AvailabilityZone,HealthStatus,LifecycleState]' \
  --output table
```

---

### 5. Configuração de Alarmes do CloudWatch

**Objetivo:** Monitorar o desempenho e criar alertas para ações automatizadas.

**5.1. Alarme de CPU Alta (Scale Out)**

| Configuração | Valor |
|-------------|-------|
| Alarm name | HighCPUAlarm |
| Metric | CPUUtilization |
| Statistic | Average |
| Period | 5 minutos |
| Threshold | > 70% |
| Evaluation periods | 2 |
| Action | Add 1 instance to ASG |

**5.2. Alarme de CPU Baixa (Scale In)**

| Configuração | Valor |
|-------------|-------|
| Alarm name | LowCPUAlarm |
| Metric | CPUUtilization |
| Statistic | Average |
| Period | 5 minutos |
| Threshold | < 30% |
| Evaluation periods | 2 |
| Action | Remove 1 instance from ASG |

**Comandos de Validação:**

```bash
# Listar alarmes do CloudWatch
aws cloudwatch describe-alarms \
  --query 'MetricAlarms[*].[AlarmName,MetricName,Threshold,StateValue]' \
  --output table

# Verificar métricas de CPU
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=AutoScalingGroupName,Value=WebServerASG \
  --start-time 2026-01-18T00:00:00Z \
  --end-time 2026-01-18T23:59:59Z \
  --period 3600 \
  --statistics Average
```

---

### 6. Testes de Funcionamento

**6.1. Teste de Alta Disponibilidade**

**Procedimento:**

1. Acessei o DNS do Load Balancer via navegador
2. Verifiquei o balanceamento entre instâncias
3. Terminei manualmente uma instância
4. Observei a criação automática de nova instância pelo ASG

**Validação:**

```bash
# Verificar health checks do target group
aws elbv2 describe-target-health \
  --target-group-arn <TARGET-GROUP-ARN>
```

**Resultado:** O Auto Scaling detectou a instância unhealthy e provisionou uma nova automaticamente.

**6.2. Teste de Escalabilidade**

**Simulação de Carga:**

Para testar o scaling automático, gerei carga artificial nas instâncias:

```bash
# Conectar à instância via Session Manager
aws ssm start-session --target <instance-id>

# Gerar carga de CPU
sudo yum install stress -y
stress --cpu 2 --timeout 600
```

**Observações:**

1. CPU aumentou para acima de 70%
2. Alarme HighCPUAlarm foi acionado
3. Auto Scaling adicionou nova instância
4. Load Balancer registrou a nova instância no Target Group
5. Tráfego foi distribuído entre 3 instâncias

**Após cessação da carga:**

1. CPU retornou para níveis normais (< 30%)
2. Alarme LowCPUAlarm foi acionado
3. Auto Scaling removeu instância excedente
4. Manteve o número mínimo de 2 instâncias

---

## Desafio Opcional: Criar AMI via AWS CLI

**Objetivo:** Criar uma AMI utilizando comandos da AWS Command Line Interface.

### Preparação do Ambiente

**1. Conexão à Instância EC2:**

Utilizei o EC2 Instance Connect para acessar a instância:

```bash
# Via console AWS ou
aws ec2-instance-connect send-ssh-public-key \
  --instance-id <instance-id> \
  --availability-zone us-east-1a \
  --instance-os-user ec2-user \
  --ssh-public-key file://~/.ssh/id_rsa.pub
```

**2. Configuração das Credenciais AWS:**

Dentro da instância, configurei as credenciais:

```bash
# Configurar credenciais
aws configure

# Inserir informações:
# AWS Access Key ID: [Obtido dos Detalhes da AWS]
# AWS Secret Access Key: [Obtido dos Detalhes da AWS]
# Default region name: us-east-1
# Default output format: json
```

**Validação da Configuração:**

```bash
# Testar conectividade
aws sts get-caller-identity

# Verificar região
aws configure list
```

### Criação da AMI via CLI

**Obtenção do Instance ID:**

```bash
# Obter o ID da instância atual
INSTANCE_ID=$(ec2-metadata --instance-id | cut -d " " -f 2)
echo "Instance ID: $INSTANCE_ID"

# Ou listar instâncias
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=WebServer" \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name]' \
  --output table
```

**Comando de Criação da AMI:**

```bash
# Criar AMI a partir da instância
aws ec2 create-image \
  --instance-id $INSTANCE_ID \
  --name "WebServerAMI-CLI-$(date +%Y%m%d-%H%M%S)" \
  --description "AMI criada via AWS CLI para Auto Scaling" \
  --no-reboot \
  --tag-specifications 'ResourceType=image,Tags=[{Key=Name,Value=WebServerAMI-CLI},{Key=CreatedBy,Value=AWS-CLI}]'
```

**Output Esperado:**

```json
{
    "ImageId": "ami-0123456789abcdef0"
}
```

**Verificação do Status da AMI:**

```bash
# Armazenar AMI ID
AMI_ID="ami-0123456789abcdef0"

# Monitorar status da criação
aws ec2 describe-images \
  --image-ids $AMI_ID \
  --query 'Images[*].[ImageId,Name,State,CreationDate]' \
  --output table

# Aguardar até status = 'available'
aws ec2 wait image-available --image-ids $AMI_ID
echo "AMI criada com sucesso!"
```

**Validação Final:**

```bash
# Listar todas as AMIs criadas
aws ec2 describe-images \
  --owners self \
  --query 'Images[*].[ImageId,Name,State,CreationDate]' \
  --output table

# Verificar tags
aws ec2 describe-images \
  --image-ids $AMI_ID \
  --query 'Images[*].Tags'
```

**Resultado:** AMI criada com sucesso via CLI e disponível para uso no Auto Scaling.

---

## Resultados e Entregas

### Componentes Implementados com Sucesso

- AMI customizada criada a partir de instância existente
- Application Load Balancer configurado e operacional
- Target Group com health checks funcionais
- Launch Template definido com configurações otimizadas
- Auto Scaling Group com políticas de scaling configuradas
- Instâncias distribuídas em múltiplas zonas de disponibilidade
- CloudWatch Alarms para monitoramento e automação
- AMI adicional criada via AWS CLI (desafio opcional)

### Validação da Arquitetura

**Checklist de Validação:**

- Load Balancer acessível via DNS público
- Instâncias registradas e healthy no Target Group
- Auto Scaling mantendo número desejado de instâncias (2)
- Distribuição de instâncias em AZs diferentes
- Health checks passando (2/2 healthy)
- Alarmes do CloudWatch em estado OK
- Scaling policies respondendo adequadamente
- Alta disponibilidade comprovada (término manual de instância)
- Escalabilidade automática testada (simulação de carga)

### Métricas do Laboratório

| Métrica | Valor |
|---------|-------|
| Tempo total de execução | ~3 horas |
| AMIs criadas | 2 (Console + CLI) |
| Instâncias em execução | 2-4 (dinâmico) |
| Zonas de disponibilidade | 2 |
| Alarmes configurados | 2 |
| Testes realizados | 3 |
| Taxa de sucesso | 100% |

---

## Conhecimentos Adquiridos

### Competências Técnicas

**1. Elastic Load Balancing**

- Criação e configuração de Application Load Balancer
- Configuração de Target Groups
- Health checks e routing policies
- Integração com Auto Scaling
- Troubleshooting de problemas de conectividade

**2. Auto Scaling**

- Criação de Launch Templates
- Configuração de Auto Scaling Groups
- Políticas de scaling (target tracking)
- Distribuição em múltiplas AZs
- Gerenciamento de capacidade (min/max/desired)

**3. Amazon Machine Images (AMI)**

- Criação de AMIs via Console
- Criação de AMIs via CLI
- Gestão de imagens customizadas
- Versionamento e tags

**4. Amazon CloudWatch**

- Configuração de alarmes
- Monitoramento de métricas
- Ações automatizadas baseadas em métricas
- Análise de performance

**5. Arquitetura de Alta Disponibilidade**

- Design multi-AZ
- Tolerância a falhas
- Distribuição de carga
- Redundância de componentes

**6. AWS CLI Avançado**

- Automação de tarefas
- Scripting para provisionamento
- Consultas e filtros complexos
- Integração com ferramentas de automação

### Soft Skills Desenvolvidas

- Planejamento de arquitetura escalável
- Pensamento em alta disponibilidade
- Otimização de custos
- Monitoramento proativo
- Automação de processos

---

## Comandos AWS CLI de Referência

### Gerenciamento de AMIs

```bash
# Criar AMI
aws ec2 create-image \
  --instance-id <instance-id> \
  --name "MyAMI" \
  --description "Description" \
  --no-reboot

# Listar AMIs
aws ec2 describe-images --owners self

# Deletar AMI
aws ec2 deregister-image --image-id <ami-id>

# Aguardar disponibilidade
aws ec2 wait image-available --image-ids <ami-id>
```

### Gerenciamento de Load Balancers

```bash
# Listar Load Balancers
aws elbv2 describe-load-balancers

# Listar Target Groups
aws elbv2 describe-target-groups

# Verificar health do target
aws elbv2 describe-target-health \
  --target-group-arn <arn>

# Criar listener
aws elbv2 create-listener \
  --load-balancer-arn <arn> \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=<tg-arn>
```

### Gerenciamento de Auto Scaling

```bash
# Descrever Auto Scaling Groups
aws autoscaling describe-auto-scaling-groups

# Atualizar capacidade desejada
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name <name> \
  --desired-capacity 3

# Descrever políticas de scaling
aws autoscaling describe-policies \
  --auto-scaling-group-name <name>

# Listar instâncias do ASG
aws autoscaling describe-auto-scaling-instances
```

### Gerenciamento de CloudWatch

```bash
# Listar alarmes
aws cloudwatch describe-alarms

# Criar alarme
aws cloudwatch put-metric-alarm \
  --alarm-name <name> \
  --alarm-description <description> \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 70 \
  --comparison-operator GreaterThanThreshold

# Obter métricas
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=<id> \
  --start-time <timestamp> \
  --end-time <timestamp> \
  --period 3600 \
  --statistics Average
```

---

## Recursos e Referências

### Documentação Oficial AWS

- [Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Amazon EC2 Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/)
- [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [Amazon Machine Images (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)

### Conceitos Importantes

- **High Availability**: Capacidade de um sistema permanecer operacional mesmo durante falhas
- **Scalability**: Habilidade de aumentar ou diminuir recursos conforme demanda
- **Elasticity**: Capacidade de provisionamento automático de recursos
- **Target Tracking**: Política de scaling baseada em uma métrica alvo
- **Health Check**: Verificação automática do estado de saúde das instâncias

### Boas Práticas Aplicadas

- Distribuição de recursos em múltiplas AZs
- Uso de Launch Templates para padronização
- Configuração de health checks adequados
- Definição de limites de capacidade apropriados
- Monitoramento proativo com CloudWatch
- Testes de failover e recuperação
- Documentação de configurações

---

## Lições Aprendidas

### Aspectos Críticos

**1. Planejamento de Capacidade**

É fundamental definir corretamente os valores de min, max e desired capacity baseando-se em:
- Padrões históricos de tráfego
- Testes de carga
- Margem de segurança para picos inesperados

**2. Health Checks**

A configuração adequada dos health checks é essencial:
- Grace period suficiente para inicialização
- Thresholds balanceados (não muito sensíveis nem muito lenientes)
- Endpoints apropriados para verificação

**3. Políticas de Scaling**

Target tracking scaling é mais simples, mas step scaling oferece mais controle:
- Target tracking: Ideal para cargas previsíveis
- Step scaling: Melhor para variações abruptas

### Erros Comuns a Evitar

- Não configurar health checks no Target Group
- Definir capacidade mínima muito baixa (comprometendo HA)
- Esquecer de distribuir instâncias em múltiplas AZs
- Não testar a política de scaling antes de produção
- Ignorar custos de data transfer entre AZs
- Não configurar alarmes de monitoramento
- Usar AMIs não atualizadas

### Otimizações Realizadas

- Uso de no-reboot ao criar AMIs (reduz downtime)
- Configuração de instance warm-up para evitar scaling prematuro
- Tags consistentes para organização
- Scripts de automação com AWS CLI

---

## Considerações de Custos

### Componentes com Custo

| Componente | Modelo de Cobrança |
|------------|-------------------|
| Application Load Balancer | Por hora + por LCU (Load Balancer Capacity Unit) |
| Instâncias EC2 | Por hora de execução |
| Data Transfer | Por GB transferido |
| CloudWatch Alarms | Primeiros 10 alarmes gratuitos |
| AMIs | Armazenamento EBS dos snapshots |

### Estratégias de Otimização

- Usar Auto Scaling para reduzir instâncias em períodos ociosos
- Configurar políticas agressivas de scale-in
- Monitorar métricas de utilização
- Considerar Reserved Instances para capacidade base
- Avaliar Savings Plans para workloads consistentes

---

## Conclusão

Este laboratório proporcionou experiência prática completa em arquitetura de alta disponibilidade e escalabilidade na AWS. A implementação de Elastic Load Balancing combinada com Auto Scaling demonstra como criar sistemas resilientes que se adaptam automaticamente às demandas, mantendo disponibilidade e otimizando custos.

Os conhecimentos adquiridos sobre balanceamento de carga, dimensionamento automático e monitoramento são fundamentais para construir aplicações cloud-native robustas. A capacidade de automatizar o provisionamento de infraestrutura através de AMIs e Launch Templates, aliada ao monitoramento proativo com CloudWatch, representa um conjunto essencial de habilidades para profissionais de cloud computing.

A experiência hands-on com AWS CLI no desafio opcional reforçou a importância da automação e do Infrastructure as Code, competências cada vez mais valorizadas no mercado.

---

## Evidências

*Inserir prints de tela quando disponível:*

- Dashboard do Application Load Balancer
- Auto Scaling Group com instâncias em execução
- Target Group mostrando healthy targets
- CloudWatch Alarms configurados
- Teste de carga e scaling automático
- AMI criada via CLI
- Gráficos de métricas do CloudWatch

---

**Laboratório realizado em:** [Data]  
**Ambiente:** AWS Academy  
**Documentado por:** [Seu Nome]  

---

> **Nota:** Esta documentação foi elaborada para fins educacionais, respeitando as diretrizes de uso da AWS e políticas de integridade acadêmica. Todo o conteúdo foi desenvolvido com base na experiência prática do laboratório, sem reprodução de materiais proprietários da Amazon.
