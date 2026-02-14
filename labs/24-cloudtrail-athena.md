# 🔍 AWS CloudTrail — Investigação Forense de Segurança

**Autora:** Kaylane Kimberly  
**Data:** 14 de Fevereiro de 2026  
**Tecnologias:** `AWS CloudTrail` `Amazon Athena` `AWS CLI` `Amazon EC2`

---

## 📋 Visão Geral

Investigação de uma violação de segurança no site da cafeteria **"Café"**, utilizando o **AWS CloudTrail** para auditar ações na conta, identificar alterações indevidas em Grupos de Segurança e rastrear a identidade do invasor.

### Objetivos

- Configurar e ativar o AWS CloudTrail
- Analisar logs via CLI (Linux `grep`) e Console AWS
- Utilizar o **Amazon Athena** para consultas SQL em logs de auditoria
- Remediar vulnerabilidades e revogar acessos maliciosos

---

## 🛠️ Execução das Tarefas

### Tarefa 1 — Verificação de Integridade Inicial

Validação de que a instância **Café Web Server** está operacional e o site acessível com o layout correto. Esta etapa estabelece a *baseline* de segurança — o estado íntegro da aplicação antes de qualquer incidente.

---

### Tarefa 2 — Configuração da Trilha (Trail)

1. **Criação da trilha:** logs armazenados em um bucket do **Amazon S3**
2. **Incidente:** após a ativação, o site sofre uma violação — as regras de entrada do Grupo de Segurança foram modificadas, expondo a instância

---

### Tarefa 3 — Análise de Logs via CLI

Uso do `grep` para filtrar eventos específicos de forma rápida:

```bash
grep "eventName" log_file.json | grep "Modify"
```

> 💡 **Insight:** A CLI é ideal para buscas rápidas quando já conhecemos o padrão do evento a investigar.

---

### Tarefa 4 — Investigação Avançada com Amazon Athena

O Athena permite tratar os logs do CloudTrail (JSON no S3) como tabelas SQL, sem mover dados.

| Campo de Pesquisa | Valor Encontrado |
|---|---|
| **Event Name** | `AuthorizeSecurityGroupIngress` |
| **User Agent** | Identificação da ferramenta usada pelo atacante |
| **Source IP** | Endereço de origem da invasão |
| **userIdentity** | Usuário responsável pelas chamadas de API não autorizadas |
| **Horário da Ação** | Timestamp exato registrado no log do CloudTrail |

**Identificação do Atacante:** cruzando os dados de `userIdentity`, identificamos o usuário específico, o horário exato e o IP de origem das chamadas de API não autorizadas.

---

### Tarefa 5 — Remediação

1. **Revogação de Acesso:** chaves de acesso (Access Keys) e permissões do usuário malicioso removidas
2. **Rollback de Segurança:** regras do Grupo de Segurança restauradas para tráfego legítimo (Portas `80`/`443`)
3. **Hardening:** políticas de **Mínimo Privilégio** implementadas para impedir que usuários comuns alterem configurações críticas de rede

---

## 💡 Considerações Finais

A segurança na nuvem não é apenas sobre **prevenir**, mas sobre ter **visibilidade total** das ações. Sem o CloudTrail habilitado, identificar o *"quem, quando e onde"* da invasão seria impossível.

A combinação **CloudTrail + Amazon Athena** forma um sistema de auditoria forense poderoso para resposta rápida a incidentes em ambientes cloud.

---

## ⚖️ Direitos Autorais

- **AWS:** conteúdo original, cenários e arquitetura do laboratório são propriedade intelectual da **Amazon Web Services, Inc.** Esta documentação é um registro de execução prática de treinamento oficial AWS.
- **Documentação:** análises e conclusões elaboradas por **Kaylane Kimberly**. Reprodução integral para fins comerciais não autorizada.

---

*Documentação por Kaylane Kimberly — 14/02/2026*
