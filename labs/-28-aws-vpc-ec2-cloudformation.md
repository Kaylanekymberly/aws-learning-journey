# AWS re/Start — Laboratório de Desafio: Criar uma VPC e Instância EC2 com AWS CloudFormation

> **Registro técnico de execução prática — Treinamento oficial AWS re/Start**

---

##  Direitos Autorais

> **AWS:** O cenário, arquitetura e scripts originais são propriedade intelectual da **Amazon Web Services, Inc.** Esta documentação é um registro de execução prática de treinamento oficial AWS.
>
> **Documentação:** Este relatório técnico, análises de saída e resolução dos desafios foram produzidos por **Kaylane Kimberly**.

---

##  Índice

1. [Visão Geral do Laboratório](#-visão-geral-do-laboratório)
2. [Objetivos e Componentes](#-objetivos-e-componentes)
3. [Restrições do Ambiente](#-restrições-do-ambiente)
4. [Acesso ao Ambiente AWS](#-acesso-ao-ambiente-aws)
5. [Ferramentas Disponíveis](#-ferramentas-disponíveis)
6. [Template CloudFormation](#-template-cloudformation)
7. [Arquitetura Implementada](#-arquitetura-implementada)
8. [Processo de Deploy](#-processo-de-deploy)
9. [Validação dos Recursos](#-validação-dos-recursos)
10. [Conclusão](#-conclusão)

---

##  Visão Geral do Laboratório

Este laboratório faz parte do programa **AWS re/Start** e consiste em um ambiente de desafio prático onde o objetivo é provisionar infraestrutura na AWS de forma automatizada, utilizando o serviço de **Infraestrutura como Código (IaC)** da AWS: o **AWS CloudFormation**.

O laboratório exige a criação de um **template CloudFormation** completo e funcional, capaz de provisionar todos os recursos de rede e computação listados nos requisitos, sem erros de execução.

---

##  Objetivos e Componentes

O template CloudFormation desenvolvido deve provisionar os seguintes recursos na AWS:

| # | Recurso | Descrição |
|---|---------|-----------|
| 1 | **Amazon VPC** | Virtual Private Cloud — rede isolada na AWS |
| 2 | **Internet Gateway** | Gateway de internet conectado à VPC |
| 3 | **Security Group** | Grupo de segurança com permissão de SSH (porta 22) de qualquer origem (`0.0.0.0/0`) |
| 4 | **Sub-rede Privada** | Subnet criada dentro da VPC |
| 5 | **Instância EC2 (T3.micro)** | Instância Amazon EC2 lançada dentro da sub-rede privada |

>  **Observação oficial do laboratório:** Não é necessário acessar a instância EC2 via SSH ou Área de Trabalho Remota para que a solução seja considerada bem-sucedida. O objetivo é que todos os recursos sejam criados sem erros no CloudFormation.

---

##  Restrições do Ambiente

O ambiente de laboratório possui restrições de IAM que limitam o acesso apenas aos serviços necessários para a construção da solução. Serviços fora do escopo do laboratório são bloqueados por política.

---

##  Acesso ao Ambiente AWS

O acesso ao ambiente é feito pelo **AWS Management Console**, acessado diretamente pela plataforma de laboratórios:

1. Clicar em **"Iniciar laboratório"** e aguardar o status `"em criação"`
2. Fechar o painel de status e clicar em **"AWS"** para abrir o console em nova aba
3. O login é realizado **automaticamente** com as credenciais do ambiente sandbox

>  **Dica:** Caso o console não abra em nova aba, verificar se o navegador está bloqueando pop-ups e permitir o acesso ao site.

---

##  Ferramentas Disponíveis

O ambiente disponibiliza um **terminal Linux no navegador** com as seguintes ferramentas pré-configuradas:

### AWS CLI

As credenciais já estão configuradas no terminal. Exemplos de uso:

```bash
# Verificar identidade e número da conta
aws sts get-caller-identity

# Listar instâncias EC2 em execução
aws ec2 describe-instances
```

### AWS SDK for Python (Boto3)

O Python 3 e a biblioteca Boto3 estão disponíveis no terminal:

```bash
$ python3
>>> import boto3
>>> ec2 = boto3.client('ec2', region_name='us-west-2')
>>> ec2.describe_regions()
>>> exit()
```

---

##  Template CloudFormation

O template abaixo em formato **YAML** provisiona toda a infraestrutura exigida pelo laboratório:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: >
  Template CloudFormation para criar uma VPC, Internet Gateway,
  Security Group, Sub-rede privada e uma instância EC2 T3.micro.
  Laboratório AWS re/Start — Kaylane Kimberly.

Resources:

  # ─────────────────────────────────────────────
  # 1. Amazon VPC
  # ─────────────────────────────────────────────
  MinhaVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: MinhaVPC

  # ─────────────────────────────────────────────
  # 2. Internet Gateway
  # ─────────────────────────────────────────────
  MeuInternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: MeuInternetGateway

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MinhaVPC
      InternetGatewayId: !Ref MeuInternetGateway

  # ─────────────────────────────────────────────
  # 3. Sub-rede Privada
  # ─────────────────────────────────────────────
  MinhaSubnetPrivada:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MinhaVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs ""]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: MinhaSubnetPrivada

  # ─────────────────────────────────────────────
  # 4. Security Group (permite SSH de qualquer lugar)
  # ─────────────────────────────────────────────
  MeuSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permite acesso SSH de qualquer origem
      VpcId: !Ref MinhaVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: MeuSecurityGroup

  # ─────────────────────────────────────────────
  # 5. Instância EC2 T3.micro
  # ─────────────────────────────────────────────
  MinhaInstanciaEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: !Sub "{{resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2}}"
      SubnetId: !Ref MinhaSubnetPrivada
      SecurityGroupIds:
        - !Ref MeuSecurityGroup
      Tags:
        - Key: Name
          Value: MinhaInstanciaEC2

Outputs:
  VpcId:
    Description: ID da VPC criada
    Value: !Ref MinhaVPC

  SubnetId:
    Description: ID da Sub-rede Privada
    Value: !Ref MinhaSubnetPrivada

  SecurityGroupId:
    Description: ID do Security Group
    Value: !Ref MeuSecurityGroup

  InstanceId:
    Description: ID da Instância EC2
    Value: !Ref MinhaInstanciaEC2
```

---

##  Arquitetura Implementada

```
┌─────────────────────────────────────────────────┐
│                  AWS Account                    │
│                                                 │
│  ┌──────────────────────────────────────────┐   │
│  │            Amazon VPC                   │   │
│  │          (10.0.0.0/16)                  │   │
│  │                                          │   │
│  │   ┌──────────────────────────────────┐   │   │
│  │   │       Sub-rede Privada           │   │   │
│  │   │         (10.0.1.0/24)            │   │   │
│  │   │                                  │   │   │
│  │   │   ┌──────────────────────────┐   │   │   │
│  │   │   │  EC2 Instance (T3.micro) │   │   │   │
│  │   │   │  + Security Group (SSH)  │   │   │   │
│  │   │   └──────────────────────────┘   │   │   │
│  │   └──────────────────────────────────┘   │   │
│  │                                          │   │
│  │   Internet Gateway ←──────────────────   │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

##  Processo de Deploy

### Via Console AWS (CloudFormation)

1. Acesse o serviço **CloudFormation** no Console AWS
2. Clique em **"Create stack"** → **"With new resources (standard)"**
3. Selecione **"Upload a template file"** e faça o upload do arquivo `.yaml`
4. Defina um nome para a stack (ex: `lab-vpc-ec2`)
5. Avance pelas etapas sem alterar configurações adicionais
6. Clique em **"Create stack"**
7. Aguarde o status mudar para `CREATE_COMPLETE`

### Via AWS CLI

```bash
# Criar a stack diretamente pelo terminal
aws cloudformation create-stack \
  --stack-name lab-vpc-ec2 \
  --template-body file://template.yaml \
  --region us-east-1

# Monitorar o status da criação
aws cloudformation describe-stacks \
  --stack-name lab-vpc-ec2 \
  --query "Stacks[0].StackStatus"
```

---

##  Validação dos Recursos

Após o deploy bem-sucedido, os recursos podem ser verificados:

```bash
# Verificar VPCs criadas
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=MinhaVPC"

# Verificar instâncias EC2
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=MinhaInstanciaEC2" \
  --query "Reservations[*].Instances[*].{ID:InstanceId,State:State.Name,Type:InstanceType}"

# Verificar Security Groups
aws ec2 describe-security-groups \
  --filters "Name=tag:Name,Values=MeuSecurityGroup"

# Verificar outputs da stack
aws cloudformation describe-stacks \
  --stack-name lab-vpc-ec2 \
  --query "Stacks[0].Outputs"
```

>  A solução é considerada **bem-sucedida** quando todos os recursos são criados sem erros e o status da stack é `CREATE_COMPLETE`.

---

##  Recursos de Referência

| Recurso | Link |
|---------|------|
| AWS CloudFormation Docs | [docs.aws.amazon.com/cloudformation](https://docs.aws.amazon.com/cloudformation/) |
| AWS CLI Command Reference | [docs.aws.amazon.com/cli](https://docs.aws.amazon.com/cli/latest/reference/) |
| AWS SDK for Python (Boto3) | [boto3.amazonaws.com/v1/documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) |
| AWS Training & Certification | [aws.amazon.com/training](https://aws.amazon.com/training/) |

---

##  Conclusão

Este laboratório demonstrou na prática o uso do **AWS CloudFormation** para provisionamento de infraestrutura como código (IaC). Foram aplicados conceitos essenciais de redes na AWS, como criação e isolamento de VPC, configuração de sub-redes, controle de tráfego com Security Groups e lançamento de instâncias EC2 de forma automatizada, reproduzível e auditável.

A abordagem com IaC elimina o processo manual de criação de recursos, reduz erros humanos e permite versionamento da infraestrutura — habilidades fundamentais para qualquer profissional de Cloud.

---

<div align="center">

**© 2022 Amazon Web Services, Inc. e suas afiliadas. Todos os direitos reservados.**

*Este trabalho não pode ser reproduzido nem redistribuído, parcial ou integralmente, sem a permissão prévia por escrito da Amazon Web Services, Inc. A cópia, a venda e o empréstimo para fins comerciais são proibidos.*

---

*Documentação técnica produzida por **Kaylane Kimberly** como registro de execução prática do treinamento oficial AWS re/Start.*

</div>
