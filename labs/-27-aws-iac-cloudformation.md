#  AWS CloudFormation Automation Lab

> **Documentação técnica do laboratório prático de automação de infraestrutura com AWS CloudFormation**

---

##  Índice

- [Visão Geral](#visão-geral)
- [Objetivos de Aprendizagem](#objetivos-de-aprendizagem)
- [Arquitetura](#arquitetura)
- [Pré-requisitos](#pré-requisitos)
- [Duração Estimada](#duração-estimada)
- [Estrutura do Laboratório](#estrutura-do-laboratório)
  - [Tarefa 1 — Implantar uma Pilha do CloudFormation](#tarefa-1--implantar-uma-pilha-do-cloudformation)
  - [Tarefa 2 — Adicionar um Bucket do Amazon S3 à Pilha](#tarefa-2--adicionar-um-bucket-do-amazon-s3-à-pilha)
  - [Tarefa 3 — Adicionar uma Instância do Amazon EC2 à Pilha](#tarefa-3--adicionar-uma-instância-do-amazon-ec2-à-pilha)
  - [Tarefa 4 — Excluir a Pilha](#tarefa-4--excluir-a-pilha)
- [Estrutura do Template CloudFormation](#estrutura-do-template-cloudformation)
- [Serviços AWS Utilizados](#serviços-aws-utilizados)
- [Conceitos-Chave](#conceitos-chave)
- [Referências](#referências)
- [Direitos Autorais e Licença](#direitos-autorais-e-licença)

---

## Visão Geral

Implantar infraestrutura de maneira consistente e confiável é um desafio, pois exige que equipes sigam procedimentos documentados sem pegar atalhos. Além disso, provisionamentos fora do horário comercial tornam o processo ainda mais difícil.

O **AWS CloudFormation** resolve esse problema permitindo que a infraestrutura seja definida em um arquivo de template que pode ser implantado de forma totalmente automatizada, repetível e auditável.

Este laboratório oferece experiência prática e interativa na criação, atualização e exclusão de pilhas do CloudFormation, exigindo consulta à documentação oficial da AWS para descobrir como definir cada recurso dentro do template.

---

## Objetivos de Aprendizagem

Ao concluir este laboratório, você será capaz de:

-  Implantar uma pilha do AWS CloudFormation com uma **VPC** e um **Grupo de Segurança**
-  Atualizar uma pilha existente adicionando um **bucket do Amazon S3**
-  Atualizar uma pilha existente adicionando uma **instância do Amazon EC2**
-  Utilizar **parâmetros** e referências (`!Ref`) dentro de templates CloudFormation
-  Usar o **AWS Systems Manager Parameter Store** para recuperar dinamicamente IDs de AMI
-  **Excluir uma pilha** e verificar a remoção automática de todos os recursos associados

---

## Arquitetura

A infraestrutura evolui ao longo das tarefas, conforme ilustrado abaixo:

**Após a Tarefa 1 — VPC e Security Group:**
```
┌──────────────────────────────────────────────┐
│                  AWS Cloud                   │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │             Lab VPC                   │  │
│  │         (10.0.0.0/16)                 │  │
│  │                                       │  │
│  │   ┌───────────────────────────────┐   │  │
│  │   │        Public Subnet          │   │  │
│  │   │       (10.0.0.0/24)           │   │  │
│  │   └───────────────────────────────┘   │  │
│  │                                       │  │
│  │   🔒 App Security Group               │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

**Após a Tarefa 2 — Adição do S3:**
```
┌──────────────────────────────────────────────┐
│  Lab VPC  +   Amazon S3 Bucket             │
└──────────────────────────────────────────────┘
```

**Após a Tarefa 3 — Infraestrutura Completa:**
```
┌──────────────────────────────────────────────────────┐
│                     AWS Cloud                        │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │                   Lab VPC                     │  │
│  │                                               │  │
│  │   ┌───────────────────────────────────────┐   │  │
│  │   │           Public Subnet               │   │  │
│  │   │                                       │   │  │
│  │   │   ┌───────────────────────────────┐   │   │  │
│  │   │   │       App Server              │   │   │  │
│  │   │   │    (EC2 t3.micro)             │   │   │  │
│  │   │   │    Amazon Linux 2             │   │   │  │
│  │   │   └───────────────────────────────┘   │   │  │
│  │   └───────────────────────────────────────┘   │  │
│  │   🔒 App Security Group                       │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│   Amazon S3 Bucket (nome gerado automaticamente)   │
└──────────────────────────────────────────────────────┘
```

---

## Pré-requisitos

- Acesso ao Console de Gerenciamento da AWS com permissões para CloudFormation, VPC, EC2 e S3
- Editor de texto simples (não processador de texto) para editar arquivos `.yaml`
- Conhecimentos básicos de:
  - Infraestrutura como código (IaC)
  - Formato YAML (indentação com 2 espaços, uso de hifens)
  - Conceitos de rede AWS (VPC, Sub-redes, Grupos de Segurança)

---

## Duração Estimada

 **~45 minutos**

---

## Estrutura do Laboratório

### Tarefa 1 — Implantar uma Pilha do CloudFormation

**Objetivo:** Fazer upload do template `task1.yaml` e criar a pilha `Lab` que provisiona uma VPC com sub-rede pública e um grupo de segurança.

**Seções do template analisado:**

| Seção | Função |
|---|---|
| `Parameters` | Define entradas como blocos CIDR da VPC e sub-rede |
| `Resources` | Define os recursos a serem provisionados (VPC, Security Group) |
| `Outputs` | Expõe o ID do Security Group padrão criado |

**Passos executados:**

1. Download do template `task1.yaml`
2. Acesso ao console do **CloudFormation → Criar pilha**
3. Upload do arquivo de template
4. Configuração do nome da pilha: `Lab`
5. Manutenção dos valores CIDR padrão definidos no template
6. Criação da pilha e acompanhamento dos eventos

**Status monitorados:**

```
CREATE_IN_PROGRESS  →  CREATE_COMPLETE 
```

**Recursos criados:**
- `Lab VPC` com bloco CIDR `10.0.0.0/16`
- `Public Subnet` com bloco CIDR `10.0.0.0/24`
- `App Security Group`
- Internet Gateway e Route Table

---

### Tarefa 2 — Adicionar um Bucket do Amazon S3 à Pilha

**Objetivo:** Editar o template para incluir um bucket S3 e atualizar a pilha existente, demonstrando como adicionar recursos sem recriar a infraestrutura existente.

**Alteração realizada no template (`task1.yaml`):**

```yaml
Resources:

  # ... recursos existentes ...

  S3Bucket:
    Type: AWS::S3::Bucket
```

>  **Dica:** Nenhuma propriedade adicional é obrigatória para criação de um bucket S3 básico. O CloudFormation atribuirá um nome aleatório para evitar conflitos.

**Passos executados:**

1. Edição do arquivo `task1.yaml` adicionando o recurso S3
2. Console do CloudFormation → selecionar pilha `Lab` → **Atualizar**
3. Upload do template modificado
4. Revisão do **Change Set** (pré-visualização das alterações)
5. Confirmação da atualização

**Change Set exibido antes da atualização:**

```
Ação: Add
Recurso: AWS::S3::Bucket
Substituição: False   ← Recursos existentes não são afetados
```

**Status monitorados:**

```
UPDATE_IN_PROGRESS  →  UPDATE_COMPLETE 
```

---

### Tarefa 3 — Adicionar uma Instância do Amazon EC2 à Pilha

**Objetivo:** Editar o template para provisionar uma instância EC2 do tipo App Server, utilizando parâmetros dinâmicos do SSM Parameter Store para recuperar o ID da AMI mais recente.

**Parâmetro adicionado na seção `Parameters`:**

```yaml
Parameters:

  AmazonLinuxAMIID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
```

>  Este parâmetro usa o **AWS Systems Manager Parameter Store** para recuperar automaticamente o ID da AMI do Amazon Linux 2 mais recente para a região atual, eliminando a necessidade de atualizar o template manualmente a cada nova AMI.

**Recurso EC2 adicionado na seção `Resources`:**

```yaml
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref AmazonLinuxAMIID
      InstanceType: t3.micro
      SecurityGroupIds:
        - !Ref AppSecurityGroup
      SubnetId: !Ref PublicSubnet
      Tags:
        - Key: Name
          Value: App Server
```

**Uso de `!Ref` para referências internas:**

| Referência | Recurso Apontado |
|---|---|
| `!Ref AmazonLinuxAMIID` | Parâmetro SSM com o ID da AMI |
| `!Ref AppSecurityGroup` | Security Group definido no mesmo template |
| `!Ref PublicSubnet` | Sub-rede pública definida no mesmo template |

**Change Set exibido antes da atualização:**

```
Ação: Add
Recurso: AWS::EC2::Instance
Substituição: False   ← Recursos existentes não são afetados
```

**Status monitorados:**

```
UPDATE_IN_PROGRESS  →  UPDATE_COMPLETE 
```

---

### Tarefa 4 — Excluir a Pilha

**Objetivo:** Excluir a pilha `Lab` e verificar que todos os recursos provisionados foram removidos automaticamente pelo CloudFormation.

**Passos executados:**

1. Console do CloudFormation → selecionar pilha `Lab`
2. Clique em **Excluir** → confirmar

**Status monitorados:**

```
DELETE_IN_PROGRESS  →  (pilha removida da lista) 
```

**Recursos removidos automaticamente:**

-  Amazon S3 Bucket
-  Instância EC2 (App Server)
-  Lab VPC e sub-redes
-  App Security Group
-  Internet Gateway e Route Table

>  **Importante:** O CloudFormation só consegue excluir automaticamente buckets S3 **vazios**. Buckets com objetos requerem esvaziamento manual antes da exclusão da pilha.

---

## Estrutura do Template CloudFormation

Abaixo está a estrutura completa e final do template após as três tarefas:

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Lab VPC + S3 + EC2

Parameters:

  LabVpcCidr:
    Type: String
    Default: 10.0.0.0/16

  PublicSubnetCidr:
    Type: String
    Default: 10.0.0.0/24

  AmazonLinuxAMIID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:

  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref LabVpcCidr

  # Sub-rede Pública
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PublicSubnetCidr

  # Grupo de Segurança
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: App Security Group
      VpcId: !Ref VPC

  # Bucket S3
  S3Bucket:
    Type: AWS::S3::Bucket

  # Instância EC2
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref AmazonLinuxAMIID
      InstanceType: t3.micro
      SecurityGroupIds:
        - !Ref AppSecurityGroup
      SubnetId: !Ref PublicSubnet
      Tags:
        - Key: Name
          Value: App Server

Outputs:
  DefaultSecurityGroup:
    Value: !GetAtt VPC.DefaultSecurityGroup
```

---

## Serviços AWS Utilizados

| Serviço | Uso no Laboratório |
|---|---|
| **AWS CloudFormation** | Criação, atualização e exclusão das pilhas de infraestrutura |
| **Amazon VPC** | Rede virtual isolada com sub-rede pública |
| **Amazon EC2** | Instância App Server provisionada via template |
| **Amazon S3** | Bucket de armazenamento adicionado à pilha |
| **AWS Systems Manager** | Parameter Store para recuperação dinâmica do ID da AMI |

---

## Conceitos-Chave

| Conceito | Descrição |
|---|---|
| **CloudFormation Stack** | Conjunto de recursos gerenciados como uma unidade via template |
| **Template (YAML/JSON)** | Arquivo declarativo que define a infraestrutura desejada |
| **Parameters** | Entradas configuráveis que tornam o template reutilizável |
| **Resources** | Seção principal que define os recursos AWS a provisionar |
| **Outputs** | Valores exportados da pilha para uso externo ou consulta |
| **`!Ref`** | Função que referencia outros recursos ou parâmetros do mesmo template |
| **Change Set** | Pré-visualização das alterações antes de atualizar uma pilha |
| **SSM Parameter Store** | Repositório de parâmetros que permite recuperar valores dinamicamente (ex: AMI IDs) |
| **IaC (Infrastructure as Code)** | Prática de gerenciar infraestrutura através de código versionável e repetível |

---

## Referências

- [Documentação oficial do AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/)
- [Trechos de modelo do Amazon S3](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-s3.html)
- [AWS::EC2::Instance — Referência de recursos](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html)
- [AWS::S3::Bucket — Referência de recursos](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html)
- [Query for latest Amazon Linux AMI IDs using SSM Parameter Store](https://aws.amazon.com/blogs/compute/query-for-the-latest-amazon-linux-ami-ids-using-aws-systems-manager-parameter-store/)
- [Funções intrínsecas do CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

---

## Direitos Autorais e Licença

> [!IMPORTANT]
> **Direitos Autorais:**
>
> - **AWS:** O cenário, arquitetura e scripts originais são propriedade intelectual da **Amazon Web Services, Inc**. Esta documentação é um registro de execução prática de treinamento oficial AWS.
> - **Documentação:** Este relatório técnico, análises de saída e resolução dos desafios foram produzidos por **Kaylane Kimberly**.

---

<div align="center">

**Feito com  por [Kaylane Kimberly](https://github.com/kaylanekimberly)**

*Laboratório baseado no programa de treinamento AWS*

</div>
