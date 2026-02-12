# Documentação Técnica — Lab 23

## Infraestrutura de Observabilidade, Monitoramento e Conformidade

### Amazon CloudWatch · AWS Systems Manager · AWS Config

> **Data:** 11 de Fevereiro de 2026
> **ID do Lab:** 22
> **Nível:** Intermediário / Avançado
> **Autor:** Kaylane Kimberly
> **Propriedade Intelectual:** Arquitetura e cenários originais © Amazon Web Services (AWS). Execução técnica e documentação © Kaylane Kimberly.

---

## 1. Contexto e Motivação

Em ambientes de nuvem de produção, a ausência de visibilidade operacional é um risco crítico. Uma instância EC2 sobrecarregada, um bucket S3 mal configurado ou uma regra de Security Group aberta indevidamente podem passar despercebidos por horas ou dias sem uma infraestrutura de monitoramento adequada.

Este laboratório endereça esse problema de forma estrutural, implementando três camadas complementares de observabilidade:

- **Camada de Coleta:** instalação e configuração do agente CloudWatch via Systems Manager.
- **Camada de Análise:** centralização de logs e métricas customizadas de sistema operacional.
- **Camada de Governança:** rastreamento contínuo de conformidade com AWS Config.

O objetivo não é apenas "ver o que está acontecendo", mas construir uma infraestrutura que **reage automaticamente** a desvios e **documenta seu próprio estado** ao longo do tempo.

---

## 2. Arquitetura da Solução

```
┌─────────────────────────────────────────────────────────┐
│                    CAMADA DE COLETA                     │
│                                                         │
│   [EC2 Instance]                                        │
│   ├── CloudWatch Agent (instalado via SSM Run Command)  │
│   │   ├── /var/log/application.log ──▶ CloudWatch Logs  │
│   │   ├── Métricas de RAM         ──▶ CloudWatch Metrics │
│   │   ├── Métricas de Disco       ──▶ CloudWatch Metrics │
│   │   └── Métricas de CPU (SO)    ──▶ CloudWatch Metrics │
│   └── SSM Agent (pré-instalado nas AMIs Amazon Linux)   │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   CAMADA DE ANÁLISE                     │
│                                                         │
│   CloudWatch Logs                                       │
│   ├── Log Groups por aplicação/instância                │
│   ├── Metric Filters (ex: padrão "ERROR", "Exception")  │
│   └── Log Insights (queries ad-hoc)                     │
│                                                         │
│   CloudWatch Metrics                                    │
│   ├── Dashboards customizados                           │
│   └── Alarmes com ações automáticas (SNS / Auto Scaling)│
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                  CAMADA DE GOVERNANÇA                   │
│                                                         │
│   AWS Config                                            │
│   ├── Configuration Recorder (estado atual dos recursos)│
│   ├── Config Rules (estado desejado / compliance)       │
│   └── Timeline de mudanças por recurso                  │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Serviços Utilizados e Suas Funções

| Serviço | Papel na Solução | Por que não o alternativo? |
|---|---|---|
| **CloudWatch Logs** | Centralização e análise de logs de aplicação | Alternativa manual (SSH + tail) não escala |
| **CloudWatch Metrics** | Coleta de métricas de SO (RAM, Disco) — não disponíveis por padrão | Métricas padrão da AWS não incluem nível de SO |
| **CloudWatch Alarms** | Alertas e ações automáticas baseadas em limiares | Monitoramento passivo não previne incidentes |
| **Systems Manager (SSM)** | Instalação e gestão do agente sem SSH | Acesso SSH direto é um vetor de segurança e não escala |
| **AWS Config** | Rastreamento de conformidade e histórico de configurações | CloudTrail registra *ações*; Config registra *estado dos recursos* |

---

## 4. Implementação Técnica Detalhada

### Fase 1 — Implantação do Agente via AWS Systems Manager

A instalação do CloudWatch Agent nas instâncias EC2 foi realizada via **SSM Run Command**, sem acesso SSH direto. Essa abordagem tem implicações arquiteturais importantes:

**Por que evitar SSH para gerenciamento de agentes?**
- Em frotas com dezenas ou centenas de instâncias, o SSH manual é inviável e cria inconsistências de versão.
- O SSM mantém um inventário centralizado de todos os agentes instalados.
- Elimina a necessidade de abrir a porta 22 no Security Group, reduzindo a superfície de ataque.

O documento SSM utilizado para instalação padronizada:

```json
{
  "schemaVersion": "2.2",
  "description": "Install and configure CloudWatch Agent",
  "mainSteps": [
    {
      "action": "aws:runDocument",
      "name": "installCWAgent",
      "inputs": {
        "documentType": "SSMDocument",
        "documentPath": "AmazonCloudWatch-ManageAgent",
        "documentParameters": {
          "action": "configure",
          "optionalConfigurationLocation": "AmazonCloudWatch-linux"
        }
      }
    }
  ]
}
```

Após a execução, o status de sucesso/falha em cada instância é visível no painel de **Run Command History** do Systems Manager, permitindo auditoria da implantação.

---

### Fase 2 — Configuração do Agente CloudWatch

O arquivo de configuração do agente define exatamente quais logs e métricas serão coletados:

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "cwagent"
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/application.log",
            "log_group_name": "/cafeteria/application",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 30
          },
          {
            "file_path": "/var/log/httpd/error_log",
            "log_group_name": "/cafeteria/httpd/errors",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 30
          }
        ]
      }
    }
  },
  "metrics": {
    "namespace": "CafeteriaApp/SystemMetrics",
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["disk_used_percent"],
        "resources": ["/"],
        "metrics_collection_interval": 60
      },
      "cpu": {
        "measurement": ["cpu_usage_iowait", "cpu_usage_user", "cpu_usage_system"],
        "metrics_collection_interval": 60,
        "totalcpu": true
      }
    }
  }
}
```

**Por que métricas customizadas são necessárias?**

As métricas padrão da AWS (`AWS/EC2`) coletam dados pela **hypervisor layer**, sem acesso ao sistema operacional convidado. Por isso, métricas críticas como **uso de RAM** e **uso de disco por partição** simplesmente não existem nos namespaces padrão — o agente instalado dentro da instância é o único meio de obtê-las.

---

### Fase 3 — Criação de Metric Filters e Alarmes

#### Metric Filter para detecção de erros

Um *Metric Filter* foi criado no Log Group `/cafeteria/application` para transformar ocorrências do padrão `"[ERROR]"` em uma métrica numérica rastreável:

```
Filter Pattern: [ERROR]
Metric Name:    ApplicationErrorCount
Metric Value:   1
Namespace:      CafeteriaApp/ApplicationLogs
```

Esse filtro permite criar alarmes baseados em frequência de erros — por exemplo, acionar um alerta se mais de 5 erros ocorrerem em um intervalo de 5 minutos.

#### Alarme de Alta Utilização de Memória

```
Metric:     CafeteriaApp/SystemMetrics → mem_used_percent
Threshold:  > 85% por 2 períodos consecutivos de 60s
Action:     Publicar no tópico SNS "ops-alerts"
```

---

### Fase 4 — AWS Config: Governança e Conformidade

O **AWS Config** opera em uma lógica diferente dos demais serviços de monitoramento. Enquanto o CloudWatch monitora *performance* e *comportamento em tempo real*, o Config monitora o **estado de configuração dos recursos** e verifica se estão em conformidade com políticas definidas.

#### Diferença fundamental: CloudTrail vs. AWS Config

| | CloudTrail | AWS Config |
|---|---|---|
| **O que registra** | Ações e chamadas de API (quem fez o quê) | Estado atual e histórico de configuração dos recursos |
| **Pergunta que responde** | "Quem abriu essa regra de Security Group?" | "Esse Security Group está dentro dos padrões?" |
| **Modelo** | Log de eventos (imutável) | Snapshot contínuo + timeline de mudanças |

#### Config Rules implementadas

| Regra | Descrição | Risco Mitigado |
|---|---|---|
| `s3-bucket-public-read-prohibited` | Nenhum bucket pode ter leitura pública habilitada | Exposição acidental de dados sensíveis |
| `restricted-ssh` | Nenhum Security Group pode ter a porta 22 aberta para `0.0.0.0/0` | Acesso SSH irrestrito à internet |
| `ec2-instance-managed-by-ssm` | Todas as instâncias EC2 devem estar registradas no SSM | Instâncias não gerenciadas e não auditáveis |

Quando um recurso viola uma dessas regras, o AWS Config marca o recurso como **NON_COMPLIANT** e pode disparar uma ação corretiva automatizada via **SSM Automation** ou **Lambda**.

---

## 5. Validação e Resultados

| Teste Realizado | Resultado Esperado | Status |
|---|---|:---:|
| Instalação do agente via SSM Run Command | Agente ativo em todas as instâncias sem uso de SSH | ✅ |
| Métricas de RAM e Disco aparecendo no CloudWatch | Namespace `CafeteriaApp/SystemMetrics` populado | ✅ |
| Log Group `/cafeteria/application` recebendo logs | Streams por `instance_id` visíveis em tempo real | ✅ |
| Metric Filter detectando padrão `[ERROR]` | Métrica `ApplicationErrorCount` incrementada nos testes | ✅ |
| Alarme disparado ao simular uso de memória elevado | Notificação SNS recebida em < 60s | ✅ |
| AWS Config identificando bucket S3 público como NON_COMPLIANT | Status atualizado imediatamente após alteração | ✅ |
| AWS Config bloqueando Security Group com porta 22 aberta | Recurso sinalizado e notificação gerada | ✅ |

---

## 6. Análise de Segurança

A solução implementada elimina três vetores de risco comuns em ambientes de nuvem mal configurados:

**1. Falta de visibilidade de recursos internos do SO**
Sem o agente CloudWatch, a equipe de operações é "cega" quanto ao uso real de memória e disco das instâncias. Uma instância com 95% de uso de RAM não geraria nenhum alerta — o problema só seria descoberto após uma interrupção de serviço.

**2. Deriva de configuração (Configuration Drift)**
Sem o AWS Config, mudanças manuais em Security Groups, permissões de buckets ou configurações de instâncias passam desapercebidas. O Config cria um registro imutável de cada mudança e sinaliza desvios em relação ao estado desejado.

**3. Dependência de acesso SSH para gestão de agentes**
O uso do Systems Manager como plano de controle para instalação de agentes permite fechar a porta 22 no Security Group das instâncias gerenciadas, sem perder a capacidade de executar comandos remotamente. Isso segue o princípio de **Zero Trust Networking**.

---

## 7. Conclusão e Relevância para a Carreira

Este laboratório marca uma transição significativa no perfil técnico: de **administrador de recursos** (quem cria e configura serviços) para **Engenheiro de Confiabilidade de Site (SRE)** (quem garante que os serviços operem de forma previsível, segura e auditável).

As três competências consolidadas aqui — **observabilidade**, **gerenciamento de frotas sem SSH** e **governança por código** — são pilares fundamentais de qualquer arquitetura de nuvem de nível empresarial, e aparecem com frequência em certificações como:

- **AWS Certified SysOps Administrator – Associate**
- **AWS Certified DevOps Engineer – Professional**
- **AWS Certified Security – Specialty**

---

## 8. Propriedade Intelectual e Créditos

- **Lab Foundation:** Amazon Web Services Academy.
- **Technical Writing & Engineering:** Kaylane Kimberly.

---

*Documentação gerada como parte de portfólio técnico de estudos em computação em nuvem.*
