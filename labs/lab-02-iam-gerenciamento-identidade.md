# Lab 02 - Introdução ao IAM (Identity and Access Management)

## 📌 Descrição do Laboratório
Neste laboratório, explorei os fundamentos do gerenciamento de identidades na AWS. O objetivo foi aplicar o **Princípio do Menor Privilégio (PoLP)**, garantindo que usuários tenham apenas as permissões necessárias para realizar suas tarefas.

##  Recursos Utilizados
* **Usuários IAM:** Criação de identidades para pessoas.
* **Grupos IAM:** Organização de usuários com permissões comuns.
* **Políticas (Policies):** Definição de permissões usando JSON.
* **Políticas Gerenciadas pela AWS:** Uso de permissões pré-definidas (ex: `ReadOnlyAccess`).

##  Passo a Passo Realizado

### 1. Criação de Grupos e Usuários
* Criei um grupo chamado `Suporte-Nivel-1`.
* Criei um usuário chamado `usuario-teste` e o adicionei ao grupo.
* Configurei o acesso ao Console de Gerenciamento da AWS para este usuário.

### 2. Atribuição de Permissões
* Anexei a política `ReadOnlyAccess` ao grupo. 
* Isso permite que os membros do grupo visualizem recursos, mas não criem ou excluam nada.

### 3. Teste de Acesso (Verificação)
* Fiz login com o `usuario-teste`.
* Tentei visualizar instâncias EC2 (Sucesso ✅).
* Tentei criar um novo bucket S3 (Erro de Permissão Negada ❌ - Conforme esperado).

##  Conceitos Chave Aprendidos
* **Root User vs IAM User:** Nunca usar o usuário raiz para tarefas diárias.
* **Políticas JSON:** Como a AWS lê permissões (Allow/Deny).
* **Segurança:** A importância de habilitar MFA (Multi-Factor Authentication).

## 🔗 Links Úteis
* [Documentação Oficial do IAM](https://docs.aws.amazon.com/pt_br/iam/)
