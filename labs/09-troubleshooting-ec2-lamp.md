# Solucionar Problemas com a Criação de uma Instância EC2

## Objetivos do Laboratório

Após concluir este laboratório, sou capaz de:

- Iniciar uma instância do EC2 usando a AWS CLI
- Solucionar problemas em comandos da AWS CLI
- Diagnosticar configurações do serviço Amazon EC2
- Utilizar técnicas básicas de troubleshooting
- Aplicar o utilitário nmap para diagnóstico de rede


## Visão Geral da Atividade

Nesta atividade, utilizei a AWS Command Line Interface (AWS CLI) para iniciar instâncias do Amazon EC2. Durante o processo de criação, consultei um script de dados do usuário (User Data) para configurar automaticamente a instância com os seguintes componentes:

### Pilha LAMP Implementada

- **Linux** - Sistema operacional base (Amazon Linux 2)
- **Apache** - Servidor web HTTP
- **MariaDB** - Sistema gerenciador de banco de dados (fork do MySQL)
- **PHP** - Linguagem de programação server-side

### Funcionalidades do User Data

O script de User Data executou automaticamente:

1. Instalação e configuração do servidor web Apache
2. Instalação e inicialização do banco de dados MariaDB
3. Instalação do PHP e módulos necessários
4. Deploy dos arquivos do site da cafeteria
5. Execução de scripts de configuração do banco de dados
6. Inicialização automática dos serviços

**Resultado:** Uma instância EC2 totalmente configurada hospedando o aplicativo web da cafeteria com backend de banco de dados.

---

## Arquitetura Implementada

```
┌─────────────────────────────────────────────────────────┐
│                        Internet                          │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │   Internet Gateway    │
              └──────────┬───────────┘
                         │
┌────────────────────────┴────────────────────────────────┐
│                      VPC                                 │
│  ┌──────────────────────────────────────────────────┐  │
│  │            Public Subnet                          │  │
│  │  ┌────────────────────────────────────────────┐  │  │
│  │  │     Security Group (Firewall)              │  │  │
│  │  │  - SSH (22)                                │  │  │
│  │  │  - HTTP (80)                               │  │  │
│  │  │  ┌──────────────────────────────────────┐  │  │  │
│  │  │  │      EC2 Instance                    │  │  │  │
│  │  │  │  ┌────────────────────────────────┐  │  │  │  │
│  │  │  │  │     Apache Web Server          │  │  │  │  │
│  │  │  │  ├────────────────────────────────┤  │  │  │  │
│  │  │  │  │     PHP Application            │  │  │  │  │
│  │  │  │  ├────────────────────────────────┤  │  │  │  │
│  │  │  │  │     MariaDB Database           │  │  │  │  │
│  │  │  │  └────────────────────────────────┘  │  │  │  │
│  │  │  └──────────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## Procedimentos Realizados

### 1. Preparação do Ambiente

**Atividades Iniciais:**

- Acesso ao AWS Academy Lab Environment
- Configuração das credenciais da AWS CLI
- Verificação de permissões IAM
- Identificação da região de trabalho (us-east-1)

**Validação da AWS CLI:**

```bash
# Verificar versão da AWS CLI
aws --version

# Verificar configuração atual
aws configure list

# Testar conectividade
aws ec2 describe-regions
```

---

### 2. Análise do Script User Data

Antes de iniciar a instância, analisei o script User Data fornecido que contém:

**Estrutura do Script:**

```bash
#!/bin/bash
# Atualização do sistema
# Instalação de pacotes LAMP
# Configuração do Apache
# Configuração do MariaDB
# Deploy da aplicação
# Inicialização de serviços
```

**Componentes Instalados:**

- `httpd` - Servidor Apache
- `mariadb-server` - Banco de dados
- `php` e módulos PHP necessários
- Arquivos da aplicação web da cafeteria

---

### 3. Provisionamento via AWS CLI

**Comando de Criação da Instância:**

```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxxx \
  --instance-type t2.micro \
  --key-name vockey \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --user-data file://user-data.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=CafeWebServer}]'
```

**Parâmetros Utilizados:**

| Parâmetro | Valor | Descrição |
|-----------|-------|-----------|
| `--image-id` | ami-xxxxxxxxx | Amazon Linux 2 AMI |
| `--instance-type` | t2.micro | Tipo de instância (Free Tier) |
| `--key-name` | vockey | Par de chaves para acesso SSH |
| `--security-group-ids` | sg-xxxxxxxxx | Security Group configurado |
| `--subnet-id` | subnet-xxxxxxxxx | Subnet pública da VPC |
| `--user-data` | file://user-data.sh | Script de configuração automática |

**Verificação da Instância:**

```bash
# Listar instâncias em execução
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=CafeWebServer" \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' \
  --output table

# Obter detalhes específicos
aws ec2 describe-instances --instance-ids i-xxxxxxxxx
```

---

### 4. Troubleshooting e Diagnóstico

#### Problema 1: Instância Não Acessível via HTTP

**Sintoma:**

- Timeout ao tentar acessar o IP público via navegador
- Erro: "ERR_CONNECTION_TIMED_OUT"

**Diagnóstico com nmap:**

```bash
# Escanear portas da instância
nmap -p 22,80,443 <IP-PUBLICO>

# Resultado inicial:
# PORT    STATE    SERVICE
# 22/tcp  open     ssh
# 80/tcp  filtered http
# 443/tcp filtered https
```

**Análise:**

- Porta 22 (SSH): Aberta
- Porta 80 (HTTP): Filtrada (bloqueada por firewall)

**Verificação do Security Group:**

```bash
# Listar regras do Security Group
aws ec2 describe-security-groups \
  --group-ids sg-xxxxxxxxx \
  --query 'SecurityGroups[*].IpPermissions'
```

**Problema Identificado:**

- Security Group sem regra de entrada (inbound) para porta 80

**Solução Aplicada:**

```bash
# Adicionar regra HTTP ao Security Group
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

**Validação:**

```bash
# Verificar novamente com nmap
nmap -p 80 <IP-PUBLICO>

# Resultado após correção:
# PORT   STATE SERVICE
# 80/tcp open  http
```

**Resultado:** Problema resolvido. Aplicação web agora acessível.

---

#### Problema 2: Serviços Não Iniciados Automaticamente

**Sintoma:**

- Instância em execução, mas site não carrega
- Página em branco ou erro 503

**Diagnóstico via SSH:**

```bash
# Conectar à instância
ssh -i vockey.pem ec2-user@<IP-PUBLICO>

# Verificar status do Apache
sudo systemctl status httpd

# Verificar status do MariaDB
sudo systemctl status mariadb

# Analisar logs de inicialização
sudo cat /var/log/cloud-init-output.log
```

**Problema Identificado:**

- Erro no script User Data (sintaxe incorreta)
- Serviços não foram iniciados automaticamente

**Solução Aplicada:**

```bash
# Iniciar serviços manualmente
sudo systemctl start httpd
sudo systemctl start mariadb

# Habilitar inicialização automática
sudo systemctl enable httpd
sudo systemctl enable mariadb

# Verificar se estão rodando
sudo systemctl is-active httpd
sudo systemctl is-active mariadb
```

**Correção do User Data (para próximas instâncias):**

Revisei o script User Data para garantir:

- Sintaxe bash correta
- Comandos `systemctl enable` incluídos
- Tratamento de erros adequado

---

#### Problema 3: Erro no Comando AWS CLI

**Sintoma:**

- Comando `aws ec2 run-instances` retorna erro de validação

**Erro Recebido:**

```
An error occurred (InvalidParameterValue) when calling the RunInstances operation: 
Invalid value for parameter SecurityGroupIds
```

**Diagnóstico:**

```bash
# Listar Security Groups disponíveis
aws ec2 describe-security-groups \
  --query 'SecurityGroups[*].[GroupId,GroupName]' \
  --output table
```

**Problema Identificado:**

- Security Group ID incorreto ou inexistente
- Possível erro de digitação

**Solução Aplicada:**

```bash
# Obter o ID correto do Security Group
SECURITY_GROUP_ID=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=CafeSG" \
  --query 'SecurityGroups[0].GroupId' \
  --output text)

# Usar variável no comando
aws ec2 run-instances \
  --security-group-ids $SECURITY_GROUP_ID \
  # ... outros parâmetros
```

**Resultado:** Problema resolvido. Instância criada com sucesso.

---

### 5. Validação Final da Solução

**Checklist de Validação:**

```bash
# 1. Verificar estado da instância
aws ec2 describe-instances \
  --instance-ids i-xxxxxxxxx \
  --query 'Reservations[*].Instances[*].State.Name'
# Resultado esperado: "running"

# 2. Testar conectividade SSH
ssh -i vockey.pem ec2-user@<IP-PUBLICO>
# Conexão estabelecida

# 3. Verificar serviços em execução
sudo systemctl is-active httpd mariadb
# Ambos retornam: "active"

# 4. Escanear portas com nmap
nmap -p 22,80 <IP-PUBLICO>
# Ambas portas abertas

# 5. Testar aplicação web
curl -I http://<IP-PUBLICO>
# HTTP/1.1 200 OK

# 6. Acessar via navegador
# Site da cafeteria carregando corretamente
```

**Testes Funcionais da Aplicação:**

- Página inicial carrega corretamente
- Imagens e CSS aplicados
- Conexão com banco de dados funcional
- Formulários processam dados
- Sem erros no console do navegador

---

## Resultados e Entregas

### Componentes Implementados

- Instância EC2 provisionada via AWS CLI
- Pilha LAMP completamente funcional
- Servidor Apache rodando e acessível
- Banco de dados MariaDB operacional
- Aplicação web da cafeteria em produção
- Security Groups corretamente configurados
- User Data executado com sucesso
- Troubleshooting documentado

### Métricas do Laboratório

| Métrica | Valor |
|---------|-------|
| Tempo total de execução | 2 horas |
| Problemas encontrados | 3 |
| Problemas resolvidos | 3 (100%) |
| Comandos AWS CLI executados | 15+ |
| Testes de validação | 6 |

---

## Conhecimentos Adquiridos

### Competências Técnicas

**1. AWS CLI Avançado**

- Provisionamento de instâncias EC2
- Gerenciamento de Security Groups
- Consultas com filtros e queries
- Uso de variáveis em scripts

**2. Troubleshooting Sistemático**

- Metodologia estruturada de diagnóstico
- Análise de logs do sistema
- Interpretação de erros da AWS CLI
- Uso de ferramentas de diagnóstico (nmap)

**3. Automação com User Data**

- Criação de scripts de inicialização
- Instalação automatizada de pacotes
- Configuração de serviços
- Tratamento de erros em scripts

**4. Segurança e Rede**

- Configuração de Security Groups
- Regras de firewall (inbound/outbound)
- Diagnóstico de conectividade
- Boas práticas de segurança

**5. Pilha LAMP**

- Instalação e configuração do Apache
- Gerenciamento do MariaDB
- Integração PHP com banco de dados
- Deploy de aplicações web

### Soft Skills Desenvolvidas

- Pensamento analítico
- Resolução de problemas complexos
- Documentação técnica
- Atenção aos detalhes
- Persistência no debugging

---

## Comandos AWS CLI de Referência

### Gerenciamento de Instâncias

```bash
# Criar instância
aws ec2 run-instances --image-id <AMI> --instance-type <TYPE> ...

# Listar instâncias
aws ec2 describe-instances

# Parar instância
aws ec2 stop-instances --instance-ids <ID>

# Iniciar instância
aws ec2 start-instances --instance-ids <ID>

# Terminar instância
aws ec2 terminate-instances --instance-ids <ID>

# Obter IP público
aws ec2 describe-instances \
  --instance-ids <ID> \
  --query 'Reservations[0].Instances[0].PublicIpAddress'
```

### Gerenciamento de Security Groups

```bash
# Listar Security Groups
aws ec2 describe-security-groups

# Ver regras específicas
aws ec2 describe-security-groups --group-ids <SG-ID>

# Adicionar regra de entrada
aws ec2 authorize-security-group-ingress \
  --group-id <SG-ID> \
  --protocol tcp \
  --port <PORT> \
  --cidr 0.0.0.0/0

# Remover regra de entrada
aws ec2 revoke-security-group-ingress \
  --group-id <SG-ID> \
  --protocol tcp \
  --port <PORT> \
  --cidr 0.0.0.0/0
```

### Diagnóstico com nmap

```bash
# Escanear portas específicas
nmap -p 22,80,443 <IP>

# Scan completo
nmap -sV <IP>

# Verificar porta HTTP
nmap -p 80 <IP>
```

---

## Recursos e Referências

### Documentação Oficial AWS

- [AWS CLI Command Reference - EC2](https://docs.aws.amazon.com/cli/latest/reference/ec2/)
- [EC2 User Data Scripts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [Security Groups for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)

### Ferramentas Utilizadas

- [AWS CLI](https://aws.amazon.com/cli/)
- [nmap - Network Mapper](https://nmap.org/)
- [Apache HTTP Server](https://httpd.apache.org/)
- [MariaDB](https://mariadb.org/)

### Conceitos Importantes

- **LAMP Stack**: Conjunto de tecnologias open-source para desenvolvimento web
- **User Data**: Scripts executados na inicialização de instâncias EC2
- **Security Groups**: Firewalls virtuais para controle de tráfego
- **AWS CLI**: Interface de linha de comando para interação com serviços AWS

---

## Lições Aprendidas

### Boas Práticas

1. Sempre validar Security Groups antes de criar instâncias
2. Testar scripts User Data em ambiente controlado primeiro
3. Usar nmap para diagnóstico rápido de conectividade
4. Documentar IDs de recursos (AMI, Security Groups, Subnets)
5. Verificar logs de cloud-init para troubleshooting
6. Utilizar tags para organização de recursos
7. Habilitar serviços para inicialização automática

### Erros Comuns a Evitar

- Esquecer de abrir portas no Security Group
- Usar IDs incorretos nos comandos
- Não validar sintaxe do User Data
- Não verificar se serviços estão habilitados
- Ignorar mensagens de erro da AWS CLI

---

## Conclusão

Este laboratório proporcionou experiência prática essencial em troubleshooting de infraestrutura AWS, combinando conhecimentos de linha de comando (AWS CLI), redes e segurança, sistemas operacionais Linux e serviços de computação em nuvem, além de metodologia estruturada de resolução de problemas.

A capacidade de diagnosticar e resolver problemas rapidamente é fundamental para operações em ambientes de produção. As técnicas aprendidas neste laboratório são aplicáveis não apenas ao EC2, mas a diversos cenários de infraestrutura cloud.

A abordagem sistemática de troubleshooting (identificar, diagnosticar, resolver e validar) é uma competência transferível que será valiosa em toda a carreira em cloud computing.


**Laboratório realizado em:** 16/01/2026
**Ambiente:** AWS Academy  
**Documentado por:** Kaylane Kimberly

---

> **Nota:** Esta documentação foi elaborada para fins educacionais, respeitando as diretrizes de uso da AWS e políticas de integridade acadêmica. Todo o conteúdo foi desenvolvido com base na experiência prática do laboratório.
