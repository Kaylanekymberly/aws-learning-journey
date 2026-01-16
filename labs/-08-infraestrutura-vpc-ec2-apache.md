# Implementação de Infraestrutura Web na AWS

## Visão Geral

Este laboratório demonstra a configuração completa de uma infraestrutura web na AWS, incluindo a criação de uma VPC personalizada, configuração de rede, e deploy de uma instância EC2 com servidor web Apache.

**Objetivo**: Criar uma arquitetura de rede isolada e implementar um servidor web acessível publicamente.


## Arquitetura Implementada

- **VPC** com bloco CIDR customizado
- **Sub-rede pública** com atribuição automática de IP público
- **Internet Gateway** para conectividade externa
- **Tabela de rotas** configurada para tráfego de internet
- **Instância EC2** executando Amazon Linux com Apache HTTP Server
- **Security Group** com regras de entrada para SSH e HTTP

---

## Parte 1: Configuração da VPC e Sub-rede

### 1.1 Criar a VPC

1. Acesse o serviço **VPC** no console AWS
2. Clique em **Criar VPC**
3. Selecione **Somente VPC**
4. Configure os parâmetros:
   - **Nome**: `Lab-VPC`
   - **Bloco CIDR IPv4**: `10.0.0.0/16`
5. Clique em **Criar VPC**

### 1.2 Criar Sub-rede Pública

1. No menu lateral, selecione **Sub-redes**
2. Clique em **Criar sub-rede**
3. Selecione a VPC criada anteriormente
4. Configure os parâmetros:
   - **Nome**: `Public-Subnet`
   - **Zona de disponibilidade**: Escolha uma zona disponível
   - **Bloco CIDR**: `10.0.1.0/24`
5. Clique em **Criar sub-rede**

### 1.3 Habilitar IP Público Automático

1. Selecione a sub-rede recém-criada
2. Navegue até **Ações > Editar configurações da sub-rede**
3. Marque **Habilitar atribuição automática de endereço IPv4 público**
4. Salve as alterações

---

## Parte 2: Configuração do Internet Gateway

O Internet Gateway permite que recursos na VPC se comuniquem com a internet.

### 2.1 Criar Internet Gateway

1. No menu lateral, selecione **Internet Gateways**
2. Clique em **Criar gateway da Internet**
3. Defina um nome: `Lab-IGW`
4. Clique em **Criar**

### 2.2 Anexar à VPC

1. Selecione o Internet Gateway criado
2. Clique em **Ações > Anexar à VPC**
3. Selecione sua VPC
4. Clique em **Anexar gateway da Internet**

---

## Parte 3: Configuração da Tabela de Rotas

### 3.1 Adicionar Rota para Internet

1. No menu lateral, selecione **Tabelas de rotas**
2. Identifique a tabela de rotas associada à sua VPC
3. Selecione a tabela e navegue até a aba **Rotas**
4. Clique em **Editar rotas**
5. Clique em **Adicionar rota** e configure:
   - **Destino**: `0.0.0.0/0` (representa todo o tráfego de internet)
   - **Alvo**: Selecione o Internet Gateway criado
6. Clique em **Salvar alterações**

---

## Parte 4: Lançamento da Instância EC2

### 4.1 Configurações Básicas

1. Acesse o serviço **EC2** no console AWS
2. Clique em **Executar instâncias**
3. Configure os parâmetros básicos:
   - **Nome**: `Web-Server-Lab`
   - **AMI**: Amazon Linux 2023 ou Amazon Linux 2
   - **Tipo de instância**: `t3.micro` ou `t3.small`
   - **Par de chaves**: Selecione uma chave existente ou crie uma nova

### 4.2 Configurações de Rede

1. Clique em **Editar** nas configurações de rede
2. Configure:
   - **VPC**: Selecione a VPC criada anteriormente
   - **Sub-rede**: Selecione a sub-rede pública
   - **Atribuir IP público**: Habilitar

### 4.3 Configuração do Security Group

Configure as seguintes regras de entrada:

| Tipo | Protocolo | Porta | Origem | Descrição |
|------|-----------|-------|--------|-----------|
| SSH | TCP | 22 | 0.0.0.0/0 | Acesso SSH |
| HTTP | TCP | 80 | 0.0.0.0/0 | Tráfego web |

> ⚠️ **Nota de Segurança**: Em ambiente de produção, restrinja o acesso SSH a IPs específicos.

### 4.4 Dados do Usuário (User Data)

Expanda **Detalhes Avançados** e adicione o seguinte script no campo **Dados do Usuário**:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
```

### 4.5 Executar Instância

Revise todas as configurações e clique em **Executar instância**.

---

## Parte 5: Configuração do Servidor Web

### 5.1 Conectar à Instância

1. No painel EC2, clique em **Visualizar todas as instâncias**
2. Selecione a instância criada
3. Clique em **Conectar**
4. Selecione a aba **EC2 Instance Connect**
5. Clique em **Conectar** para abrir o terminal no navegador

### 5.2 Criar o Arquivo HTML

#### Método 1: Usando o comando `cat` (Recomendado)

```bash
cat <<EOF > projects.html
<!DOCTYPE html>
<html>
<body>
<h1>Project Name - Web Server Lab</h1>
<p>EC2 Instance Challenge Lab</p>
</body>
</html>
EOF
```

#### Método 2: Usando o comando `printf`

```bash
printf '<!DOCTYPE html><html><body><h1>Project Name</h1><p>EC2 Instance Challenge Lab</p></body></html>' | sudo tee /var/www/html/projects.html
```

### 5.3 Mover o Arquivo para o Diretório Web

```bash
sudo mv projects.html /var/www/html/
```

---

## Parte 6: Solução de Problemas Comuns

### 6.1 Verificar e Iniciar o Apache

Se o servidor não estiver respondendo, execute:

```bash
# Verificar status
sudo systemctl status httpd

# Instalar Apache (se necessário)
sudo dnf install -y httpd

# Iniciar o serviço
sudo systemctl start httpd

# Habilitar inicialização automática
sudo systemctl enable httpd
```

### 6.2 Garantir que a Estrutura de Diretórios Existe

```bash
# Criar diretório se não existir
sudo mkdir -p /var/www/html

# Recriar o arquivo HTML
printf '<!DOCTYPE html><html><body><h1>Project Name</h1><p>EC2 Instance Challenge Lab</p></body></html>' | sudo tee /var/www/html/projects.html
```

### 6.3 Verificar Permissões

```bash
# Verificar permissões do arquivo
ls -la /var/www/html/projects.html

# Ajustar permissões se necessário
sudo chmod 644 /var/www/html/projects.html
```

---

## Parte 7: Validação e Testes

### 7.1 Validar Acesso Web

1. Obtenha o IP público da instância no painel EC2
2. Abra um navegador e acesse:
   ```
   http://[SEU-IP-PÚBLICO]/projects.html
   ```

> ⚠️ **Importante**: Digite o endereço **manualmente** usando `http://` (não `https://`). Navegadores modernos podem adicionar `https://` automaticamente, o que causará erro de conexão.

### 7.2 Capturar Log do Sistema

Para documentação técnica:

1. No console EC2, selecione a instância
2. Clique em **Ações > Monitoramento e solução de problemas**
3. Selecione **Obter log do sistema**
4. Capture o screenshot para documentação

---

## Comandos de Verificação

### Verificar Status dos Serviços

```bash
# Status do Apache
sudo systemctl status httpd

# Verificar se a porta 80 está em uso
sudo netstat -tulpn | grep :80

# Testar localmente
curl http://localhost/projects.html
```

### Verificar Conectividade de Rede

```bash
# Verificar interface de rede
ip addr show

# Testar conectividade de saída
ping -c 4 8.8.8.8

# Verificar rotas
ip route show
```

---

## Checklist de Conclusão

- [ ] VPC criada com CIDR 10.0.0.0/16
- [ ] Sub-rede pública configurada com IP público automático
- [ ] Internet Gateway criado e anexado à VPC
- [ ] Tabela de rotas configurada com rota padrão (0.0.0.0/0)
- [ ] Security Group com regras para SSH (22) e HTTP (80)
- [ ] Instância EC2 lançada e em execução
- [ ] Apache HTTP Server instalado e iniciado
- [ ] Arquivo HTML criado e acessível
- [ ] Validação via navegador bem-sucedida
- [ ] Log do sistema capturado para documentação

---

## Lições Aprendidas

### Problema: Atalhos de Teclado no Navegador

**Situação**: Ao usar o EC2 Instance Connect no navegador, atalhos como `Ctrl+O` e `Ctrl+X` podem ser interceptados pelo navegador em vez de serem enviados ao terminal.

**Solução**: Utilize comandos alternativos que não dependam de editores interativos:
- Use `cat <<EOF` ou `printf` para criar arquivos
- Evite o editor `nano` quando possível
- Para navegadores específicos (Opera, Chrome, etc.), considere usar o SSH tradicional via cliente terminal

### Problema: Protocolo HTTPS vs HTTP

**Situação**: Navegadores modernos adicionam automaticamente `https://` por padrão ao acessar URLs.

**Solução**: 
- Digite o endereço completo manualmente incluindo `http://`
- Não use copiar/colar para URLs que devem usar HTTP
- Verifique sempre o protocolo na barra de endereços

### Problema: Security Group não Configurado

**Situação**: Servidor Apache funcionando, mas página não carrega no navegador.

**Solução**: 
- Sempre verifique as regras do Security Group
- Confirme que a porta 80 está aberta para 0.0.0.0/0
- Use `telnet [IP] 80` ou ferramentas online para testar conectividade

---

## Recursos Adicionais

- [Documentação oficial AWS VPC](https://docs.aws.amazon.com/vpc/)
- [Documentação oficial AWS EC2](https://docs.aws.amazon.com/ec2/)
- [Guia de boas práticas de Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

---

## Licença

Este documento foi criado para fins educacionais. AWS e todos os serviços relacionados são marcas registradas da Amazon Web Services, Inc.

**Data de criação**: Janeiro 2026  
**Versão**: 1.0
