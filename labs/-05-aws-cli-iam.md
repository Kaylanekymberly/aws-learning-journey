# Introdução ao IAM (Identity and Access Management)

> 📚 **Créditos**: Este laboratório foi desenvolvido baseado no programa **AWS Academy Cloud Foundations**. Todos os direitos do conteúdo educacional pertencem à Amazon Web Services (AWS).

## 🚀 Descrição do Laboratório

Neste laboratório prático, explorei os fundamentos do gerenciamento de identidades e acessos na AWS, com foco na **AWS CLI** (Command Line Interface). O objetivo principal foi aplicar o **Princípio do Menor Privilégio (PoLP)** e desenvolver habilidades de automação através da linha de comando.

### Objetivo de Aprendizado

Demonstrar a capacidade de gerenciar recursos IAM via AWS CLI em ambiente Linux, extraindo e analisando políticas de segurança diretamente pela linha de comando - uma competência essencial para profissionais de cloud computing e DevOps.

---

## 🛠️ Recursos e Tecnologias Utilizadas

### Conceitos IAM Explorados
- **Usuários IAM**: Criação de identidades para pessoas
- **Grupos IAM**: Organização de usuários com permissões comuns
- **Políticas (Policies)**: Definição de permissões usando JSON
- **Políticas Gerenciadas pela AWS**: Uso de permissões pré-definidas (ex: `ReadOnlyAccess`)

### Stack Tecnológico
- **Cloud Provider**: Amazon Web Services (AWS)
- **Instância**: EC2 (Red Hat Enterprise Linux)
- **Ferramentas**: AWS CLI v2, SSH (PuTTY/OpenSSH)
- **Linguagens**: JSON, Bash
- **Protocolos**: SSH para conexão remota segura

---

## 🔒 Segurança da Informação (Best Practices)

Em conformidade com as melhores práticas de segurança em nuvem, **todos os dados sensíveis** nesta documentação foram **ofuscados**:

- ✅ IDs de conta AWS
- ✅ Chaves de acesso (Access Keys)
- ✅ Chaves secretas (Secret Keys)
- ✅ Endereços IP públicos e privados
- ✅ ARNs completos

---

## 📋 Passo a Passo do Laboratório

### Tarefa 1: Conexão Segura via SSH 🔐

**Objetivo**: Estabelecer conexão remota segura com a instância EC2.

#### Processo Realizado

**Para Windows (PuTTY)**:
1. Download do arquivo `.ppk` contendo a chave privada
2. Configuração do PuTTY com o endereço IP público da instância
3. Autenticação via par de chaves

**Para Linux/macOS**:
```bash
# Ajustar permissões da chave privada
chmod 400 labsuser.pem

# Conectar via SSH
ssh -i labsuser.pem ec2-user@<PUBLIC_IP>
```

#### Primeiro Desafio Encontrado ⚠️

Durante a primeira conexão, recebi o alerta de segurança do SSH sobre a autenticidade do host:

```
The authenticity of host 'XX.XXX.XXX.XXX' can't be established.
ECDSA key fingerprint is SHA256:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.
Are you sure you want to continue connecting (yes/no)?
```

**Solução**: Após validar que era o servidor correto, digitei `yes` para adicionar o host aos conhecidos (`known_hosts`).

![Conexão SSH](./images/ssh-connection.png)
*Figura 1: Primeira conexão SSH com validação de fingerprint (dados sensíveis ofuscados)*

---

### Tarefa 2: Instalação da AWS CLI v2 📦

**Objetivo**: Instalar a interface de linha de comando da AWS no Red Hat Linux.

#### Comandos Executados

```bash
# Download do instalador
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Descompactar o arquivo
unzip -u awscliv2.zip

# Executar instalação com privilégios de superusuário
sudo ./aws/install

# Verificar instalação
aws --version
```

#### Saída Esperada
```
aws-cli/2.7.24 Python/3.8.8 Linux/4.14.133-113.105.amzn2.x86_64 botocore/2.4.5
```

**Observação**: Os números de versão podem variar dependendo da data de execução do laboratório.

---

### Tarefa 3: Auditoria do IAM via Console Web 🔍

**Objetivo**: Observar a estrutura de políticas IAM no Console de Gerenciamento antes de migrar para a CLI.

#### Análise Realizada

1. Acessei o serviço **IAM** no Console AWS
2. Naveguei até **Users** → `awsstudent`
3. Na aba **Permissions**, localizei a política `lab_policy`
4. Visualizei o documento JSON da política

**Aprendizado**: Essa visualização prévia foi fundamental para entender o formato esperado na extração via CLI posteriormente.

---

### Tarefa 4: Configuração de Credenciais ⚙️

**Objetivo**: Configurar a AWS CLI com credenciais de acesso à conta.

#### Comando de Configuração
```bash
aws configure
```

#### Parâmetros Fornecidos
```
AWS Access Key ID [None]: AKIAXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: us-west-2
Default output format [None]: json
```

![Configuração AWS CLI](./images/aws-configure.png)
*Figura 2: Processo de configuração da AWS CLI (credenciais ofuscadas por segurança)*

#### Segundo Desafio Encontrado ⚠️

Inicialmente, **digitei a região errada** (`us-east-1` ao invés de `us-west-2`). 

**Consequência**: Os comandos subsequentes retornavam erros de recursos não encontrados.

**Solução**: Reconfigurei executando `aws configure` novamente e corrigindo a região. Aprendi que a região deve corresponder exatamente ao ambiente do laboratório.

---

### Tarefa 5: Gerenciamento de Políticas via CLI 🎯

**Objetivo**: Extrair a política `lab_policy` em formato JSON usando apenas a linha de comando.

#### Desafio da Atividade

Este foi o **maior desafio do laboratório** - recriar via CLI o que havia visto no Console, sem poder consultá-lo novamente.

#### Processo de Resolução (Troubleshooting)

**Passo 1**: Listar todas as políticas gerenciadas pelo cliente
```bash
aws iam list-policies --scope Local
```

**Dificuldade Encontrada**: A saída JSON era **extensa** e difícil de localizar o ARN correto visualmente.

**Solução**: Usei o comando `grep` para filtrar:
```bash
aws iam list-policies --scope Local | grep -A 5 "lab_policy"
```

**Passo 2**: Obter detalhes da política específica
```bash
aws iam get-policy --policy-arn arn:aws:iam::XXXXXXXXXXXX:policy/lab_policy
```

**Terceiro Desafio Encontrado** ⚠️: O comando acima retornava **metadados** da política, mas não o documento JSON em si.

**Aprendizado**: Aprendi que existem dois comandos distintos:
- `get-policy` → retorna metadados (ARN, data de criação, versão)
- `get-policy-version` → retorna o **documento JSON real**

**Passo 3**: Extrair a versão específica do documento de política
```bash
aws iam get-policy-version \
  --policy-arn arn:aws:iam::XXXXXXXXXXXX:policy/lab_policy \
  --version-id v1 > lab_policy.json
```

**Observação**: O `--version-id` foi obtido na saída do comando anterior (campo `DefaultVersionId`).

![Extração da Política](./images/policy-extraction.png)
*Figura 3: Comando de extração bem-sucedido (Account ID ofuscado)*

#### Validação do Resultado
```bash
cat lab_policy.json | jq .
```

O arquivo continha exatamente o mesmo JSON visualizado no Console! 🎉

---

## ✅ Resultados e Conquistas

### Principais Aprendizados

| Competência | Descrição |
|-------------|-----------|
| **Domínio da AWS CLI** | Capacidade de gerenciar IAM sem depender da interface gráfica |
| **Troubleshooting** | Resolução de erros de região, permissões e sintaxe de comandos |
| **Segurança Operacional** | Compreensão profunda de credenciais, chaves e princípio do menor privilégio |
| **Automação** | Preparação para Infrastructure as Code (IaC) com scripts |
| **Análise de Políticas JSON** | Interpretação de documentos de permissões complexos |

### Habilidades Técnicas Desenvolvidas

- ✅ Gerenciamento de identidades e acessos na AWS
- ✅ Utilização avançada da AWS CLI (filtros, queries, pipe)
- ✅ Conexão remota segura via SSH
- ✅ Manipulação de arquivos JSON em Linux
- ✅ Boas práticas de segurança em cloud computing
- ✅ Leitura e interpretação de documentação técnica

---

## 🎓 Reflexões sobre os Desafios

### O Que Aprendi com os Obstáculos

1. **Região Incorreta**: Ensinou-me a **sempre validar** configurações antes de executar comandos em produção.

2. **Diferença entre `get-policy` e `get-policy-version`**: Reforçou a importância de **ler a documentação** com atenção - nem sempre o comando óbvio é o correto.

3. **Validação de Fingerprint SSH**: Compreendi melhor os mecanismos de **segurança de conexão** e ataques Man-in-the-Middle.

### Metodologia de Aprendizado

> "Falhar rápido, aprender rápido" - Os erros cometidos foram **oportunidades de aprendizado** valiosas. Cada troubleshooting fortaleceu minha capacidade de diagnosticar problemas em ambientes cloud.

---

## 🎯 Conclusão

Este laboratório demonstrou que a **AWS CLI** é uma ferramenta indispensável para profissionais de cloud, permitindo:

- 🔄 **Automação** de tarefas repetitivas
- 📊 **Auditorias** rápidas de permissões
- 🔄 **Replicação** de configurações entre ambientes
- 📝 **Versionamento** de políticas como código (GitOps)
- ⚡ **Velocidade** em operações que seriam lentas via Console

### Impacto Profissional

As competências adquiridas são diretamente aplicáveis em:
- Gestão de infraestrutura multi-conta
- Pipelines CI/CD com automação AWS
- Auditoria de segurança e compliance
- Disaster recovery e backup de configurações

---

## 📚 Referências Utilizadas

Durante o laboratório, consultei ativamente:

- [Documentação Oficial AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/)
- [AWS CLI Command Reference - IAM](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/index.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Conectando-se a instâncias Linux via SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

---

## 🏆 Certificação

Este laboratório faz parte do programa **AWS Academy Cloud Foundations**, preparando para a certificação:

- 📜 **AWS Certified Cloud Practitioner** (CLF-C02)

---

## 👤 Autor

**Kaylane Kimberly**

🎯 Entusiasta de Cloud Computing | Estudante AWS  
🔗 [LinkedIn](https://www.linkedin.com/feed/) | 💼 [Portfolio](https://github.com/Kaylanekymberly/Kaylanekymberly) | 📧 [Email](kaylanekymberly123@gmail.com)

---

## 📄 Licença e Direitos Autorais

- **Conteúdo do Laboratório**: © Amazon Web Services (AWS) - AWS Academy
- **Documentação e Análise**: Desenvolvida como material de estudo pessoal
- **Imagens**: Capturadas durante a execução do laboratório com dados sensíveis ofuscados

---

<div align="center">

**⭐ Se este projeto te ajudou, considere dar uma estrela!**

*Laboratório realizado em Dezembro de 2025*

![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![JSON](https://img.shields.io/badge/JSON-000000?style=for-the-badge&logo=json&logoColor=white)

</div>
