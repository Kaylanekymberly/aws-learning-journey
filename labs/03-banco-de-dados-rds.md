# Lab 03 - Servidor de Banco de Dados e Integração com Aplicação

## 📌 Descrição do Laboratório
Neste laboratório, configurei um servidor de banco de dados relacional utilizando o **Amazon RDS** e conectei uma aplicação funcional a ele. O foco foi entender como gerenciar bancos de dados gerenciados na nuvem e como permitir a comunicação segura entre um servidor web e o banco de dados.

## 🛠️ Recursos Utilizados
* **Amazon RDS:** Instância de banco de dados MySQL.
* **Grupos de Segurança (Security Groups):** Controle de tráfego para a porta 3306.
* **Subnet Groups:** Isolamento do banco na rede.
* **Aplicação Cliente:** Interface web para teste de CRUD (Create, Read, Update, Delete).



## 📖 Passo a Passo Realizado

### 1. Criação da Instância RDS
* Selecionei o motor MySQL (Free Tier).
* Configurei credenciais mestres e desabilitei o acesso público para maior segurança.

### 2. Configuração de Segurança
* Configurei o **Security Group** do RDS para aceitar conexões apenas da origem correta, garantindo que o banco não ficasse exposto à internet.

### 3. Conexão e Teste
* Utilizei o endpoint do RDS na aplicação para validar a persistência de dados.

## 🧠 Desafios e Superação
Durante a execução, enfrentei algumas dificuldades, principalmente na **comunicação entre a aplicação e o banco de dados**. 

* **O Problema:** Inicialmente, a aplicação não conseguia alcançar o banco de dados (Timeout).
* **A Causa:** Identifiquei que o problema estava nas regras de entrada do **Security Group** e na ausência de uma rota correta na VPC.
* **A Solução:** Revisei cuidadosamente as configurações de rede, ajustei as regras de firewall (Security Group) para permitir a porta 3306 e validei o endpoint. Após esses ajustes, a conexão foi estabelecida com sucesso. 
> *Essa experiência foi fundamental para consolidar meu entendimento sobre como o isolamento de rede funciona na AWS.*

## 🛡️ Conceitos Chave Aprendidos
* **Bancos de Dados Gerenciados:** Delegação de manutenção para a AWS.
* **Endpoints:** Uso de DNS em vez de IPs estáticos.
* **Segurança em Camadas:** O banco deve estar sempre protegido por Security Groups restritivos.

## 🔗 Links Úteis
* [O que é o Amazon RDS?](https://docs.aws.amazon.com/pt_br/AmazonRDS/latest/UserGuide/Welcome.html)
