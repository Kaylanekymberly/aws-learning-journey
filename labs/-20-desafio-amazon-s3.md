#  Relatório Técnico — Laboratório de Desafio: Amazon S3

> **Data:** 11 de Fevereiro de 2026
> **Assunto:** Implementação e Gerenciamento de Armazenamento de Objetos na AWS
> **Autor:** Kaylane Kimberly
> **Propriedade Intelectual:** Conteúdo acadêmico original © Amazon Web Services (AWS). Organização, execução e documentação técnica ©Kaylane Kimberly.

---

## 1. Visão Geral

Este documento registra a execução do laboratório de desafio focado no **Amazon Simple Storage Service (Amazon S3)**. O objetivo central foi configurar um ambiente de armazenamento em nuvem escalável, aplicar políticas de acesso granulares e gerenciar objetos tanto pelo Console de Gerenciamento da AWS quanto pela interface de linha de comando (CLI).

---

## 2. Objetivos Alcançados

A conclusão deste laboratório validou as seguintes competências técnicas:

| Competência | Descrição |
|---|---|
| **Provisionamento de Infraestrutura** | Criação e configuração de buckets S3 com nomes globalmente exclusivos |
| **Gestão do Ciclo de Vida de Dados** | Upload e organização de objetos dentro da estrutura de buckets |
| **Segurança e Permissões** | Configuração de políticas de acesso público e controle via ACLs para acesso HTTP/HTTPS |
| **Operações via CLI** | Utilização da AWS CLI para listagem e auditoria de recursos em ambiente de terminal |

---

## 3. Procedimentos Executados

### Etapa 1 — Configuração do Bucket

Foi criado um bucket S3 respeitando as premissas de **unicidade global de nome** e **seleção de região**. Durante esta fase, as configurações de *Block Public Access* foram ajustadas de forma controlada para permitir a exposição seletiva de objetos conforme o escopo do desafio.

### Etapa 2 — Gerenciamento de Objetos

- **Upload:** Inserção de arquivos via Console de Gerenciamento da AWS.
- **Permissões de Objeto:** Ajuste de *Bucket Policies* e/ou *Access Control Lists (ACLs)* para garantir leitura pública dos objetos necessários.

### Etapa 3 — Validação de Acesso

A verificação de disponibilidade pública foi realizada acessando a URL de endpoint do objeto no formato padrão:

```
https://<NOME-DO-BUCKET>.s3.<REGIÃO>.amazonaws.com/<NOME-DO-ARQUIVO>
```

### Etapa 4 — Interação via AWS CLI

Execução de comandos no terminal para listar o conteúdo do bucket, simulando um fluxo de automação ou auditoria:

```bash
aws s3 ls s3://nome-do-seu-bucket
```

---

## 4. Conclusão

O laboratório foi concluído com êxito, demonstrando capacidade de manipular o Amazon S3 tanto por interface gráfica quanto por linha de comando. Após a validação dos resultados, todos os recursos foram devidamente encerrados (*cleanup*), assegurando **eficiência de custos** e **conformidade de segurança** no ambiente.

---

## 5. Propriedade Intelectual e Avisos

- **Amazon Web Services (AWS):** Todos os logotipos, nomes de serviços e a estrutura original do desafio são marcas registradas e propriedade intelectual da Amazon Web Services, Inc.
- **Este documento:** O registro técnico, a análise de execução e a formatação dos resultados são de autoria de **[Seu Nome]**, com direitos autorais reservados sobre esta versão da documentação.

---

*Documentação gerada como parte de portfólio técnico de estudos em computação em nuvem.*
