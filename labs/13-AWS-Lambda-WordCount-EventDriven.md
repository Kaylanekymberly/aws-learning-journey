# Documentação de Projeto: Automação de Contagem de Palavras Serverless

## 1. Visão Geral

Este projeto foi desenvolvido como parte de um laboratório prático da **Amazon Web Services (AWS)**. O objetivo é demonstrar a implementação de uma arquitetura orientada a eventos utilizando serviços de computação, armazenamento e mensageria.

## 2. Objetivos do Aprendizado

Ao concluir este laboratório, as seguintes competências foram desenvolvidas:

* **Desenvolvimento de Funções**: Criação de uma função AWS Lambda capaz de processar arquivos de texto e realizar lógica de contagem de palavras.
* **Gatilhos de Armazenamento**: Configuração de um bucket Amazon S3 para atuar como produtor de eventos, invocando automaticamente o Lambda no momento do upload de arquivos.
* **Notificação Escalável**: Criação e configuração de um tópico Amazon SNS para distribuição de resultados via e-mail.

## 3. Arquitetura da Solução

A solução utiliza um fluxo *serverless* de ponta a ponta:

1. **Ingestão**: O usuário realiza o upload de um arquivo `.txt` no bucket **Amazon S3**.
2. **Processamento**: O S3 dispara um evento que executa a função **AWS Lambda**.
3. **Lógica**: O Lambda lê o arquivo, realiza a contagem de palavras e utiliza o SDK `boto3` para se comunicar com o SNS.
4. **Saída**: O **Amazon SNS** envia uma notificação formatada com o resultado para o e-mail confirmado.

## 4. Evidências de Sucesso

* **Implantação de Código**: A função `WordCounterFunction` foi atualizada e implantada com êxito.
* **Confirmação de Assinatura**: O status da assinatura no SNS foi validado como "Confirmado", garantindo a entrega das notificações.
* **Teste de Upload**: O arquivo `teste.txt` foi carregado com sucesso, disparando o fluxo de trabalho.

## 5. Conclusão do Laboratório

Após a validação do recebimento do e-mail com a contagem de palavras, o ciclo de vida dos recursos foi encerrado conforme as boas práticas:

1. Encerramento do laboratório via console para limpeza de recursos.
2. Desativação de gatilhos e funções para evitar custos residuais.

---

> **Nota de Direitos Autorais**:
> Todo o conteúdo educativo, objetivos e estrutura deste laboratório são de propriedade intelectual da **Amazon Web Services, Inc. (AWS)**. Esta documentação serve apenas como registro de conclusão técnica das tarefas propostas pela plataforma.
