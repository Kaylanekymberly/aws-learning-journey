# Lab-AutoScaling-EC2-CLI

##  Sobre

Implementação de infraestrutura escalável na AWS usando **EC2 Auto Scaling** via **CLI**.  
Tempo de conclusão: **40 minutos**.

###  Objetivos
- Criar instância EC2 via AWS CLI
- Gerar AMI customizada
- Configurar Launch Template
- Implementar Auto Scaling Group
- Definir políticas de escalabilidade dinâmica

##  Stack

- **Amazon EC2** - Elastic Compute Cloud
- **EC2 Auto Scaling** - Escalabilidade automática
- **AWS CLI** - Interface de linha de comando
- **Amazon CloudWatch** - Monitoramento
- **AMI** - Amazon Machine Images

##  Comandos Principais

### 1️ Criar Instância EC2
```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxxx \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx
```

### 2️ Criar AMI
```bash
aws ec2 create-image \
  --instance-id i-xxxxxxxxx \
  --name "MyCustomAMI" \
  --description "AMI para Auto Scaling Lab"
```

### 3️ Launch Template
```bash
aws ec2 create-launch-template \
  --launch-template-name MyLaunchTemplate \
  --launch-template-data file://template-config.json
```

### 4️ Auto Scaling Group
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name MyASG \
  --launch-template LaunchTemplateName=MyLaunchTemplate \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2
```

### 5️ Política de Scaling
```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name MyASG \
  --policy-name ScaleUpPolicy \
  --scaling-adjustment 1 \
  --adjustment-type ChangeInCapacity
```

##  Desafios Enfrentados

| **Desafio** | **Solução Aplicada** |
|-------------|----------------------|
| **Sintaxe complexa da CLI** | Validar comandos antes de executar, usar arquivos JSON externos |
| **Dependências entre recursos** | Seguir ordem: instância → AMI → template → ASG |
| **Tempo de propagação** | Usar comandos `describe-*` para verificar status antes de prosseguir |
| **Configurar thresholds** | Balancear sensibilidade das métricas do CloudWatch |
| **Nomenclatura inconsistente** | Definir padrão de nomes desde o início do projeto |

---

##  Distribuição de Tempo

| **Tarefa** | **Tempo Estimado** |
|------------|-------------------|
| Criação de Instância EC2 | 5 minutos |
| Geração de AMI | 8 minutos |
| Configuração de Launch Template | 7 minutos |
| Setup de Auto Scaling Group | 12 minutos |
| Políticas de Scaling e Testes | 8 minutos |
| **TOTAL** | **40 minutos** |

---

##  Como Usar

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/Lab-AutoScaling-EC2-CLI.git
cd Lab-AutoScaling-EC2-CLI

# 2. Configure AWS CLI
aws configure

# 3. Execute os comandos conforme documentação acima
```

---

##  Recursos Adicionais

- **[AWS CLI Documentation](https://docs.aws.amazon.com/cli/)** - Documentação oficial
- **[Auto Scaling Guide](https://docs.aws.amazon.com/autoscaling/)** - Guia completo
- **[EC2 Best Practices](https://docs.aws.amazon.com/ec2/)** - Melhores práticas

---

##  Licença

**© AWS re/Start** - Todos os direitos reservados

Material desenvolvido para fins educacionais como parte do programa AWS re/Start.

---

<div align="center">

**Feito com ☁️ por AWS re/Start**

</div>

<div align="center">

![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![JSON](https://img.shields.io/badge/JSON-000000?style=for-the-badge&logo=json&logoColor=white)

##  Se este projeto te ajudou, considere dar uma estrela! 

*Laboratório realizado em Dezembro de 2025*

</div>
