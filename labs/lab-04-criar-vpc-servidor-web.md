# 🌐 Lab 04 - Criar sua VPC e Iniciar um Servidor Web

## 📋 Objetivo do Laboratório

Depois de concluir este laboratório, você deverá ser capaz de:
* ✅ Criar uma Virtual Private Cloud (VPC)
* ✅ Criar sub-redes públicas e privadas
* ✅ Configurar tabelas de rotas
* ✅ Configurar grupos de segurança
* ✅ Iniciar uma instância Amazon EC2 dentro da VPC
* ✅ Configurar um servidor web Apache

**⏱️ Tempo de conclusão:** 1 hora  
**📅 Data de conclusão:** 15/12/2025  
**✨ Status:** ✅ Concluído

---

## 💪 Reflexão Pessoal

Este laboratório foi um verdadeiro teste de **resiliência**! Durante aproximadamente **1 hora**, enfrentei diversos desafios e erros que poderiam ter me feito desistir, mas mantive o foco e a determinação.

**Os desafios enfrentados:**
- ❌ Erros de configuração de sub-redes
- ❌ Problemas com tabelas de rotas
- ❌ Dificuldades na associação correta das sub-redes
- ❌ Instância EC2 sem conectividade inicial

**A lição aprendida:**  
> *"O erro não é o fim, é parte do processo de aprendizado. Cada erro me ensinou algo novo sobre AWS e me tornou mais preparado para os próximos desafios."*

Mesmo com as dificuldades, **não desisti** e consegui completar o laboratório com sucesso! 🎉

---

## 🎯 Cenário

Neste laboratório, você usará a Amazon Virtual Private Cloud (VPC) para criar sua própria VPC e adicionar componentes adicionais para produzir uma rede personalizada para um cliente Fortune 100. Você também criará grupos de segurança para sua instância EC2 e configurará um servidor web.

### 🏗️ Arquitetura Final

A arquitetura implementada inclui:
- 1 VPC (10.0.0.0/16)
- 4 Sub-redes (2 públicas e 2 privadas) em 2 Zonas de Disponibilidade
- 1 Internet Gateway
- 1 NAT Gateway
- Tabelas de rotas configuradas
- Grupo de segurança para servidor web
- 1 Instância EC2 com servidor Apache

---

## 🚀 Tarefa 1: Acessar o Console AWS

### Passo 1.1: Iniciar o Laboratório
1. Na parte superior das instruções, escolha **Start Lab** (Iniciar laboratório)
2. Aguarde até ver a mensagem **"Lab status: ready"**
3. Clique no **X** para fechar o painel

### Passo 1.2: Abrir o Console AWS
1. Clique em **AWS** no topo da página
2. O Console de Gerenciamento AWS abrirá em nova aba
3. O sistema fará login automaticamente

> 💡 **Dica:** Se o pop-up for bloqueado, clique no ícone de bloqueio no navegador e permita pop-ups.

---

## 🌐 Tarefa 2: Criar a VPC

### Passo 2.1: Acessar o Serviço VPC
1. No console AWS, digite **VPC** na barra de pesquisa
2. Selecione **VPC** na lista de serviços

### Passo 2.2: Configurar a VPC
1. Clique em **Criar VPC**
2. Configure as seguintes opções:

**Configurações Gerais:**
- **Recursos a serem criados:** VPC e muito mais
- **Geração automática da etiqueta de nome:** ❌ Desmarcar
- **Etiqueta de nome:** `Lab VPC`
- **IPv4 CIDR:** `10.0.0.0/16`
- **IPv6 CIDR block:** Nenhum bloco CIDR IPv6
- **Tenancy:** Padrão

**Zonas de Disponibilidade e Sub-redes:**
- **Número de Zonas de Disponibilidade (AZs):** 1
- **Número de sub-redes públicas:** 1
- **Número de sub-redes privadas:** 1

### Passo 2.3: Personalizar Blocos CIDR
1. Expanda **Personalizar blocos CIDR de sub-redes**
2. Configure:
   - **Public subnet CIDR block:** `10.0.0.0/24`
   - **Private subnet CIDR block:** `10.0.1.0/24`

**Gateway NAT e Endpoints:**
- **Gateways NAT:** In 1 AZ (Em 1 AZ)
- **Endpoints da VPC:** Nenhum

### Passo 2.4: Nomear os Recursos
No painel de **Visualização**, defina os nomes:

**Sub-redes:**
- Sub-rede pública 1: `Public Subnet 1`
- Sub-rede privada 1: `Private Subnet 1`

**Tabelas de Rota:**
- Tabela pública: `Public Route Table`
- Tabela privada: `Private Route Table`

3. Clique em **Criar VPC**
4. Aguarde a mensagem de **Sucesso**
5. Clique em **Visualizar VPC**

---

## 🔀 Tarefa 3: Criar Sub-redes Adicionais

Agora vamos criar sub-redes em uma segunda Zona de Disponibilidade para **alta disponibilidade**.

### Passo 3.1: Criar Segunda Sub-rede Pública
1. No painel esquerdo, clique em **Sub-redes**
2. Clique em **Criar sub-rede**
3. Configure:
   - **VPC ID:** Lab VPC
   - **Nome da sub-rede:** `Public Subnet 2`
   - **Zona de disponibilidade:** Sem preferências
   - **IPv4 CIDR block:** `10.0.2.0/24`
4. Clique em **Criar sub-rede**

### Passo 3.2: Criar Segunda Sub-rede Privada
1. Clique em **Criar sub-rede**
2. Configure:
   - **VPC ID:** Lab VPC
   - **Nome da sub-rede:** `Private Subnet 2`
   - **Zona de disponibilidade:** Sem preferências
   - **IPv4 CIDR block:** `10.0.3.0/24`
3. Clique em **Criar sub-rede**

---

## 🛣️ Tarefa 4: Associar Sub-redes às Tabelas de Rotas

### Passo 4.1: Configurar Tabela de Rotas Públicas
1. No painel esquerdo, clique em **Tabelas de rotas**
2. Selecione **Public Route Table**
3. Na aba inferior, clique em **Associações de sub-rede**
4. Clique em **Editar associações de sub-rede**
5. Marque a caixa **Public Subnet 2**
6. Clique em **Salvar associações**

### Passo 4.2: Configurar Tabela de Rotas Privadas
1. Selecione **Private Route Table**
2. Na aba inferior, clique em **Associações de sub-rede**
3. Clique em **Editar associações de sub-rede**
4. Marque a caixa **Private Subnet 2**
5. Clique em **Salvar associações**

✅ **Sucesso!** Sua VPC agora tem sub-redes públicas e privadas em 2 Zonas de Disponibilidade!

---

## 🔒 Tarefa 5: Criar Grupo de Segurança

### Passo 5.1: Criar Security Group
1. No painel esquerdo, clique em **Grupos de segurança**
2. Clique em **Criar grupo de segurança**
3. Configure:
   - **Nome do grupo de segurança:** `Web Server SG`
   - **Descrição:** `Security group para servidor web`
   - **VPC:** Lab VPC

### Passo 5.2: Configurar Regras de Entrada
Adicione as seguintes regras:

**Regra 1 - HTTP:**
- **Tipo:** HTTP
- **Porta:** 80
- **Origem:** 0.0.0.0/0
- **Descrição:** `Permitir tráfego HTTP`

**Regra 2 - SSH:**
- **Tipo:** SSH
- **Porta:** 22
- **Origem:** 0.0.0.0/0 (ou seu IP específico)
- **Descrição:** `Permitir acesso SSH`

4. Clique em **Criar grupo de segurança**

---

## 💻 Tarefa 6: Lançar Instância EC2

### Passo 6.1: Iniciar Criação da Instância
1. Digite **EC2** na barra de pesquisa e selecione o serviço
2. Clique em **Executar instância**

### Passo 6.2: Configurar a Instância
**Nome e tags:**
- **Nome:** `Web Server`

**Imagens de aplicação e de sistema operacional:**
- **AMI:** Amazon Linux 2023 AMI (ou Amazon Linux 2)
- **Arquitetura:** 64-bit (x86)

**Tipo de instância:**
- **Tipo:** t2.micro (elegível ao nível gratuito)

**Par de chaves:**
- Selecione um par existente ou crie um novo

**Configurações de rede:**
- **VPC:** Lab VPC
- **Sub-rede:** Public Subnet 1
- **Atribuir IP público automaticamente:** Habilitar
- **Firewall (grupos de segurança):** Selecionar grupo existente
- **Grupo de segurança:** Web Server SG

### Passo 6.3: Configurar User Data
Em **Detalhes avançados**, role até **Dados do usuário** e cole:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Servidor Web funcionando na VPC!</h1>" > /var/www/html/index.html
echo "<p>Instância ID: $(ec2-metadata --instance-id | cut -d ' ' -f 2)</p>" >> /var/www/html/index.html
echo "<p>Zona de Disponibilidade: $(ec2-metadata --availability-zone | cut -d ' ' -f 2)</p>" >> /var/www/html/index.html
```

3. Clique em **Executar instância**

---

## ✅ Tarefa 7: Testar o Servidor Web

### Passo 7.1: Obter IP Público
1. Vá para **Instâncias** no painel EC2
2. Selecione sua instância **Web Server**
3. Copie o **Endereço IPv4 público**

### Passo 7.2: Acessar o Servidor
1. Abra uma nova aba do navegador
2. Cole o endereço IP: `http://SEU-IP-PUBLICO`
3. Você verá a página do servidor web! 🎉

---

## 🧹 Tarefa 8: Limpeza (Opcional)

Para evitar custos, limpe os recursos:

1. **Terminar instância EC2**
2. **Deletar NAT Gateway** (aguardar exclusão)
3. **Liberar Elastic IP**
4. **Deletar VPC** (isso remove automaticamente sub-redes, rotas, IGW)

---

## 📝 Resumo do que foi Criado

| Recurso | Nome | CIDR/Configuração |
|---------|------|-------------------|
| VPC | Lab VPC | 10.0.0.0/16 |
| Sub-rede Pública 1 | Public Subnet 1 | 10.0.0.0/24 (AZ 1) |
| Sub-rede Pública 2 | Public Subnet 2 | 10.0.2.0/24 (AZ 2) |
| Sub-rede Privada 1 | Private Subnet 1 | 10.0.1.0/24 (AZ 1) |
| Sub-rede Privada 2 | Private Subnet 2 | 10.0.3.0/24 (AZ 2) |
| Grupo de Segurança | Web Server SG | HTTP (80) e SSH (22) |
| Instância EC2 | Web Server | t2.micro com Apache |

---

## 🎓 Conceitos Aprendidos

✅ **VPC (Virtual Private Cloud):** Rede virtual isolada na AWS  
✅ **Sub-redes:** Divisões da VPC em redes menores  
✅ **Internet Gateway:** Permite comunicação com a internet  
✅ **NAT Gateway:** Permite instâncias privadas acessarem a internet  
✅ **Tabelas de Rotas:** Definem como o tráfego é direcionado  
✅ **Security Groups:** Firewall virtual para instâncias  
✅ **Alta Disponibilidade:** Recursos em múltiplas AZs  

---

## 💡 Lições Aprendidas

Durante este laboratório, aprendi que:

1. **Erros são parte do aprendizado** - Cada erro me ensinou algo novo
2. **Resiliência é fundamental** - Não desistir diante das dificuldades
3. **Paciência é essencial** - Alguns recursos levam tempo para serem criados
4. **Documentação é sua amiga** - Sempre consulte a documentação AWS
5. **Prática leva à perfeição** - Quanto mais pratico, mais aprendo

---

## 📚 Próximos Passos

- [ ] Implementar Auto Scaling
- [ ] Adicionar Load Balancer
- [ ] Configurar múltiplas instâncias
- [ ] Implementar bastion host
- [ ] Adicionar RDS na sub-rede privada

---

## 🔗 Recursos Úteis

- [Documentação Amazon VPC](https://docs.aws.amazon.com/vpc/)
- [Documentação Amazon EC2](https://docs.aws.amazon.com/ec2/)
- [Best Practices VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)

---

**✨ Lab concluído com sucesso após 1 hora de dedicação e resiliência!**

---

# 📤 Como Adicionar este Lab ao GitHub

Você já completou o lab! Agora é só documentar tudo no GitHub. Vou te guiar passo a passo:

---

## 🌟 MÉTODO RECOMENDADO: Pelo Site do GitHub (Mais Fácil!)

### **PASSO 1: Abra seu repositório**
1. Acesse: `https://github.com/Kaylanekymberli/aws-learning-journey`
2. Clique na pasta **labs**

### **PASSO 2: Crie o arquivo do lab**
1. Clique no botão verde **Add file**
2. Selecione **Create new file**
3. No campo "Name your file", digite: **`lab-04-criar-vpc-servidor-web.md`**

### **PASSO 3: Copie e cole este conteúdo**
1. **Volte para este artifact** (a caixa ao lado)
2. Clique no ícone de **copiar** no canto superior direito do artifact
3. OU selecione TUDO (Ctrl+A ou Cmd+A) → Copie (Ctrl+C ou Cmd+C)
4. **Volte para o GitHub** e cole no editor (Ctrl+V ou Cmd+V)

### **PASSO 4: Faça o commit**
Role até o final da página e preencha:

**Commit message:**
```
Lab 04 concluído - VPC e Servidor Web ✅
```

**Descrição (opcional):**
```
Criação de VPC completa com 4 sub-redes
Servidor web Apache funcionando
Tempo: 1 hora de dedicação
Mantive a resiliência mesmo com os erros iniciais!
```

Depois clique em **Commit new file** (botão verde)

### **PASSO 5: Atualize a tabela do README**
1. Volte para a pasta **labs** (clique em "labs" no topo)
2. Clique no arquivo **README.md**
3. Clique no ícone de **lápis** ✏️ (Edit this file)
4. Procure a tabela de labs
5. Adicione esta linha na tabela:

```markdown
| 04 | Criar VPC e Servidor Web | 15/12/2025 | ✅ Concluído |
```

6. Role até o final e faça o commit:
   - **Mensagem:** `Atualizar README - Lab 04 completo`
   - Clique em **Commit changes**

### **🎉 PRONTO! Está documentado!**

---

## 💻 MÉTODO ALTERNATIVO: Pelo Terminal Git

Se você prefere usar o terminal:

### **PASSO 1: Navegue até o repositório**
```bash
cd caminho/para/aws-learning-journey
cd labs
```

### **PASSO 2: Crie o arquivo**
```bash
# Crie o arquivo
nano lab-04-criar-vpc-servidor-web.md

# Cole TODO o conteúdo deste artifact
# Salve: Ctrl+X → Y → Enter
```

### **PASSO 3: Adicione e faça commit**
```bash
git add lab-04-criar-vpc-servidor-web.md

git commit -m "Lab 04 concluído - VPC e Servidor Web ✅

- Criação de VPC completa com 4 sub-redes
- Servidor web Apache funcionando  
- Tempo: 1 hora de dedicação
- Mantive a resiliência mesmo com os erros iniciais"

git push origin main
```

### **PASSO 4: Atualize o README**
```bash
nano README.md
# Adicione a linha do Lab 04 na tabela
# Salve: Ctrl+X → Y → Enter

git add README.md
git commit -m "Atualizar README - Lab 04 completo"
git push origin main
```

---

## ✅ Checklist de Verificação

Depois de fazer o upload, verifique se:

- [ ] O arquivo `lab-04-criar-vpc-servidor-web.md` está na pasta `labs`
- [ ] O README.md mostra o Lab 04 como **✅ Concluído**
- [ ] A data está correta: **15/12/2025**
- [ ] O commit tem uma mensagem clara e descritiva
- [ ] Você consegue ver o arquivo quando acessa o repositório

---

## 🎯 Resultado Final

Seu repositório vai ficar assim:

```
aws-learning-journey/
├── labs/
│   ├── README.md (atualizado com Lab 04)
│   ├── lab-01-introducao-console-aws.md (já existe - NÃO MEXER)
│   ├── lab-04-criar-vpc-servidor-web.md  ← NOVO! ✨
│   └── ...
```

E na tabela do README, você vai APENAS ADICIONAR esta linha:

| **04** | **Criar VPC e Servidor Web** | **15/12/2025** | **✅ Concluído** |

⚠️ **IMPORTANTE:** NÃO mexa nas outras linhas (Lab 01, 02, 03) - deixe como estão!

---

## 💪 Isso demonstra:

✅ Suas habilidades técnicas em AWS  
✅ Sua capacidade de documentar processos  
✅ Sua resiliência e determinação  
✅ Seu crescimento profissional  
✅ Sua organização e comprometimento  

**Parabéns por não desistir e concluir este laboratório! Continue assim! 🚀**
