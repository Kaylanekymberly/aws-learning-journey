#  Lab 25 | Governança e Automação: Gerenciamento de Recursos com Marcação (Tags)

**Autora:** Kaylane Kimberly  
**Data:** 14 de Fevereiro de 2026  
**Tecnologias:** `AWS CLI` `Amazon EC2` `Bash` `PHP SDK` `VPC`  
**Duração:** 45 minutos

> [!IMPORTANT]
> **Direitos Autorais:**
> - **AWS:** O cenário, arquitetura e scripts originais são propriedade intelectual da **Amazon Web Services, Inc.** Esta documentação é um registro de execução prática de treinamento oficial AWS.
> - **Documentação:** Este relatório técnico, análises de saída e resolução dos desafios foram produzidos por **Kaylane Kimberly**.

---

##  Visão Geral

Este laboratório foca na utilização de **Tags** como estratégia de governança para gerenciar frotas de instâncias Amazon EC2. Através da **AWS CLI**, exploramos como filtrar, iniciar, interromper e encerrar recursos em massa com base em metadados.

---

##  Cenário da Infraestrutura

| Componente | Descrição |
|---|---|
| **VPC do Laboratório** | Sub-redes públicas e privadas |
| **Command Host** | Instância EC2 Linux pré-configurada com AWS CLI |
| **Frota de Instâncias** | 8 instâncias EC2 com tags personalizadas |

Cada instância da frota possuía tags para identificação de **ambiente** e **função**, servindo como base para toda a automação.

---

##  Execução das Tarefas

### Tarefa 1 — Inspeção de Recursos via CLI

Uso de `describe-instances` com filtros específicos para listar as tags existentes:

```bash
# Listar instâncias com a tag "Environment"
aws ec2 describe-instances \
  --filters "Name=tag-key,Values=Environment" \
  --query "Reservations[*].Instances[*].{ID:InstanceId,Tag:Tags}" \
  --output table
```

>  **Habilidade praticada:** domínio de filtros `--query` e `--filters` na linha de comando AWS CLI.

---

### Tarefa 2 — Automação de Estado (Start/Stop)

Scripts que interagem com a API do EC2 para controle em massa:

**1. Filtragem por tag:**
```bash
# Capturar IDs das instâncias com tag "Plan: Temporary"
INSTANCE_IDS=$(aws ec2 describe-instances \
  --filters "Name=tag:Plan,Values=Temporary" \
            "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].InstanceId" \
  --output text)
```

**2. Ação em massa (stop):**
```bash
aws ec2 stop-instances --instance-ids $INSTANCE_IDS
```

**3. Ação em massa (start):**
```bash
aws ec2 start-instances --instance-ids $INSTANCE_IDS
```

>  **Resultado:** todas as instâncias filtradas são controladas com um único comando, eliminando ações manuais repetitivas.

---

##  Desafio — Resposta a Incidentes de Governança

O objetivo foi criar uma lógica para **encerrar (`terminate`) instâncias fora da política de marcação** da empresa — recursos sem a tag obrigatória `CostCenter`.

**Passo 1 — Listar todas as instâncias em execução:**
```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].{ID:InstanceId,Tags:Tags}" \
  --output json
```

**Passo 2 — Isolar instâncias sem a tag `CostCenter`:**
```bash
# Retorna IDs de instâncias que NÃO possuem a tag CostCenter
UNTAGGED_IDS=$(aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[?!not_null(Tags[?Key=='CostCenter'].Value | [0])].InstanceId" \
  --output text)

echo "Instâncias sem tag CostCenter: $UNTAGGED_IDS"
```

**Passo 3 — Encerrar recursos fora da política:**
```bash
aws ec2 terminate-instances --instance-ids $UNTAGGED_IDS
```

>  **Atenção:** `terminate` é irreversível. Em ambientes reais, adicione uma etapa de confirmação antes de executar.

---

##  Conclusão e Aprendizados

| Área | Aprendizado |
|---|---|
| **Governança de Custos** | Tags permitem identificar exatamente quem está gastando o quê |
| **Automação** | Redução de erro humano ao gerenciar múltiplos recursos via script |
| **Conformidade** | Capacidade de identificar e remover recursos não documentados |

A marcação de recursos é uma das práticas mais simples e de maior impacto na gestão de ambientes AWS — especialmente em organizações com múltiplas equipes e projetos compartilhando a mesma conta.

*Laboratório concluído com sucesso — Kaylane Kimberly, 14/02/2026*
