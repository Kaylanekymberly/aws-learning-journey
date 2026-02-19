#  AWS CloudFormation Troubleshooting Lab

> **Documentação técnica do laboratório prático de resolução de problemas com AWS CloudFormation**

---

##  Índice

- [Visão Geral](#visão-geral)
- [Objetivos de Aprendizagem](#objetivos-de-aprendizagem)
- [Arquitetura](#arquitetura)
- [Pré-requisitos](#pré-requisitos)
- [Estrutura do Laboratório](#estrutura-do-laboratório)
  - [Tarefa 1 — JMESPath e Consultas JSON](#tarefa-1--jmespath-e-consultas-json)
  - [Tarefa 2 — Criação e Solução de Problemas de Pilha](#tarefa-2--criação-e-solução-de-problemas-de-pilha)
  - [Tarefa 3 — Detecção de Desvios (Drift Detection)](#tarefa-3--detecção-de-desvios-drift-detection)
  - [Tarefa 4 — Exclusão da Pilha e Desafio Final](#tarefa-4--exclusão-da-pilha-e-desafio-final)
- [Serviços AWS Utilizados](#serviços-aws-utilizados)
- [Conceitos-Chave](#conceitos-chave)
- [Referências](#referências)
- [Direitos Autorais e Licença](#direitos-autorais-e-licença)

---

## Visão Geral

Este laboratório foi desenvolvido para praticar a **resolução de problemas em implantações do AWS CloudFormation**, abordando cenários reais que vão desde falhas na criação de pilhas até problemas na exclusão de recursos com dependências. O laboratório também explora o uso do **JMESPath** para consulta de dados JSON e a **Detecção de Desvios** (Drift Detection) para identificar alterações manuais em recursos gerenciados pelo CloudFormation.

O contexto de negócio é baseado em uma solicitação da equipe de liderança da **cafeteria Café**, que demanda a automação e o gerenciamento confiável de infraestrutura em nuvem.

---

## Objetivos de Aprendizagem

Ao concluir este laboratório, você será capaz de:

-  Utilizar **JMESPath** para consultar documentos em formato JSON de forma eficiente
-  **Solucionar falhas** na criação de pilhas do AWS CloudFormation via AWS CLI
-  Analisar **arquivos de log** em instâncias EC2 Linux para identificar a causa raiz de erros em `create-stack`
-  Identificar e corrigir problemas em operações de **`delete-stack`** com falha
-  Executar e interpretar resultados de **Drift Detection** no CloudFormation
-  Gerenciar recursos dependentes (como buckets S3 com conteúdo) durante a exclusão de pilhas

---

## Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Cloud                            │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                       VPC2                           │   │
│  │                                                      │   │
│  │   ┌─────────────────────────────────────────────┐    │   │
│  │   │           Public Subnet                     │    │   │
│  │   │                                             │    │   │
│  │   │   ┌──────────────────┐                      │    │   │
│  │   │   │   CLI Host       │                      │    │   │
│  │   │   │  (EC2 Instance)  │──── SSH ────► Usuário│    │   │
│  │   │   │                  │                      │    │   │
│  │   │   │  AWS CLI         │                      │    │   │
│  │   │   └──────────────────┘                      │    │   │
│  │   └─────────────────────────────────────────────┘    │   │
│  │                                                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌───────────────────────────┐   ┌──────────────────────┐   │
│  │   CloudFormation Stack    │   │     Amazon S3        │   │
│  │   (Recursos provisionados)│   │  (Bucket com objetos)│   │
│  └───────────────────────────┘   └──────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Componentes principais:**

| Componente | Descrição |
|---|---|
| **VPC2** | Rede virtual privada que hospeda o ambiente do laboratório |
| **CLI Host** | Instância EC2 usada como estação de trabalho via SSH |
| **AWS CLI** | Interface de linha de comando para interagir com os serviços AWS |
| **CloudFormation Stack** | Pilha de recursos provisionados via template |
| **Amazon S3 Bucket** | Bucket criado pela pilha, depois populado com objetos |

---

## Pré-requisitos

- Conta AWS com permissões para CloudFormation, EC2, S3 e IAM
- Cliente SSH instalado (ex: Terminal, PuTTY, OpenSSH)
- Conhecimentos básicos de:
  - Linha de comando Linux
  - Conceitos de infraestrutura como código (IaC)
  - JSON e estruturas de dados

---

## Estrutura do Laboratório

### Tarefa 1 — JMESPath e Consultas JSON

**Objetivo:** Dominar a linguagem de consulta JMESPath para filtrar e transformar saídas JSON retornadas pela AWS CLI.

**Conceitos abordados:**
- Sintaxe básica do JMESPath (seletores de campo, filtros, projeções)
- Uso da flag `--query` na AWS CLI para refinar saídas
- Extração de valores específicos de estruturas JSON aninhadas

**Exemplo de uso:**
```bash
# Filtrando instâncias EC2 pelo estado "running"
aws ec2 describe-instances \
  --query "Reservations[*].Instances[?State.Name=='running'].{ID:InstanceId,Type:InstanceType}" \
  --output table
```

---

### Tarefa 2 — Criação e Solução de Problemas de Pilha

**Objetivo:** Conectar-se ao CLI Host via SSH e utilizar a AWS CLI para criar uma pilha CloudFormation, identificando e corrigindo a falha inicial.

**Fluxo da tarefa:**

```
Conectar via SSH ao CLI Host
        │
        ▼
Executar aws cloudformation create-stack
        │
        ▼
   Falha detectada ◄──────────────────┐
        │                             │
        ▼                             │
Analisar logs no EC2 Linux            │
        │                             │
        ▼                             │
Identificar causa raiz                │
        │                             │
        ▼                             │
Corrigir o problema ──────────────────┘
        │
        ▼
Pilha criada com sucesso 
```

**Comandos relevantes:**
```bash
# Criar uma pilha
aws cloudformation create-stack \
  --stack-name minha-pilha \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

# Verificar status da pilha
aws cloudformation describe-stacks \
  --stack-name minha-pilha \
  --query "Stacks[0].StackStatus"

# Listar eventos da pilha para diagnóstico
aws cloudformation describe-stack-events \
  --stack-name minha-pilha \
  --query "StackEvents[?ResourceStatus=='CREATE_FAILED']"
```

**Análise de logs no Linux:**
```bash
# Verificar logs do sistema
sudo cat /var/log/cloud-init-output.log
sudo journalctl -u amazon-ssm-agent
```

---

### Tarefa 3 — Detecção de Desvios (Drift Detection)

**Objetivo:** Modificar manualmente um recurso criado pela pilha e usar o CloudFormation para detectar a inconsistência entre o estado real e o estado desejado (template).

**O que é Drift Detection?**

> **Drift** ocorre quando o estado atual de um recurso difere do estado definido no template CloudFormation. Isso acontece quando recursos são modificados manualmente fora do controle do CloudFormation.

**Fluxo da tarefa:**

```
Modificar recurso manualmente (ex: alterar configuração no console)
        │
        ▼
Iniciar detecção de desvio no CloudFormation
        │
        ▼
Aguardar conclusão da análise
        │
        ▼
Visualizar relatório de desvios
        │
        ▼
Interpretar diferenças: Expected vs Actual
```

**Comandos relevantes:**
```bash
# Iniciar detecção de desvio
aws cloudformation detect-stack-drift \
  --stack-name minha-pilha

# Verificar resultado da detecção
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id <detection-id>

# Ver detalhes dos recursos com desvio
aws cloudformation describe-stack-resource-drifts \
  --stack-name minha-pilha \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

---

### Tarefa 4 — Exclusão da Pilha e Desafio Final

**Objetivo:** Tentar excluir a pilha CloudFormation, solucionar os erros encontrados e concluir o desafio de excluir a pilha preservando o bucket S3 com seus objetos.

**Problema:** A exclusão falha porque o bucket S3 criado pela pilha contém objetos. O CloudFormation não pode excluir um bucket S3 não vazio por padrão.

**Desafio:** Excluir a pilha com sucesso **mantendo** o bucket S3 e seu conteúdo.

**Estratégia de solução:**

```
Tentativa de delete-stack falha
        │
        ▼
Identificar recurso bloqueante (S3 Bucket com objetos)
        │
        ▼
Opção A: Esvaziar o bucket e deletar a pilha normalmente
        │
        OU
        │
Opção B: Usar --retain-resources para manter o bucket
        │
        ▼
Pilha excluída com sucesso 
Bucket S3 preservado com seus objetos 
```

**Comandos relevantes:**
```bash
# Tentativa inicial de exclusão
aws cloudformation delete-stack --stack-name minha-pilha

# Excluindo pilha e retendo recursos específicos
aws cloudformation delete-stack \
  --stack-name minha-pilha \
  --retain-resources MeuBucketS3

# Listar objetos no bucket S3
aws s3 ls s3://nome-do-bucket --recursive

# Esvaziar o bucket (alternativa)
aws s3 rm s3://nome-do-bucket --recursive
```

---

## Serviços AWS Utilizados

| Serviço | Uso no Laboratório |
|---|---|
| **AWS CloudFormation** | Provisionar, gerenciar e excluir pilhas de infraestrutura |
| **Amazon EC2** | Instância CLI Host para execução de comandos |
| **Amazon S3** | Bucket criado pela pilha, usado no cenário de exclusão |
| **Amazon VPC** | Rede isolada (VPC2) que contém o ambiente do laboratório |
| **AWS CLI** | Interface de linha de comando para todos os comandos do laboratório |

---

## Conceitos-Chave

| Conceito | Descrição |
|---|---|
| **CloudFormation Stack** | Conjunto de recursos AWS provisionados e gerenciados como uma unidade |
| **Template** | Arquivo YAML/JSON que define os recursos da pilha |
| **Stack Events** | Histórico de eventos de criação/atualização/exclusão para diagnóstico |
| **Drift Detection** | Mecanismo de identificação de alterações manuais em recursos gerenciados |
| **JMESPath** | Linguagem de consulta para filtrar e transformar estruturas JSON |
| **CAPABILITY_IAM** | Confirmação explícita necessária quando o template cria recursos IAM |
| **--retain-resources** | Flag para preservar recursos específicos ao excluir uma pilha |

---

## Referências

- [Documentação oficial do AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/)
- [AWS CLI — Referência do CloudFormation](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/)
- [JMESPath — Documentação oficial](https://jmespath.org/)
- [AWS CloudFormation — Drift Detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
- [AWS CloudFormation — Troubleshooting](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html)

---

## Direitos Autorais e Licença

> **Important**
>
> **Direitos Autorais:**
>
> - **AWS:** O cenário, arquitetura e scripts originais são propriedade intelectual da **Amazon Web Services, Inc**. Esta documentação é um registro de execução prática de treinamento oficial AWS.
> - **Documentação:** Este relatório técnico, análises de saída e resolução dos desafios foram produzidos por **Kaylane Kimberly**.

---

<div align="center">

**Feito com  por [Kaylane Kimberly](https://github.com/kaylanekimberly)**

*Laboratório baseado no programa de treinamento AWS*

</div>
