#  Documentação Técnica — Lab 21

## Sistema de Governança e Notificação Automatizada: Amazon S3 & SNS

> **Data:** 11 de Fevereiro de 2026
> **ID do Lab:** 21
> **Nível:** Intermediário / Avançado
> **Autor:** Kaylane Kimberly
> **Propriedade Intelectual:** Arquitetura e cenários originais © Amazon Web Services (AWS). Execução e documentação técnica © Kaylane Kimberly.

---

## 1. Contexto de Negócio e Cenário

Uma cafeteria contrata uma agência de mídia externa para atualizar imagens de produtos. O desafio técnico era criar um ambiente que atendesse a três requisitos simultâneos:

1. O usuário externo (`mediacouser`) ter autonomia para gerenciar arquivos sem depender da equipe interna.
2. A empresa manter **controle e auditabilidade total** sobre o bucket sem precisar verificá-lo manualmente.
3. A solução ser **escalável e orientada a eventos** (*Event-Driven Architecture*).

---

## 2. Arquitetura da Solução

```
[mediacouser] ──PUT/DELETE──▶ [Amazon S3]
                                   │
                          s3:ObjectCreated:*
                          s3:ObjectRemoved:*
                                   │
                                   ▼
                            [Amazon SNS]
                        s3NotificationTopic
                                   │
                         ┌─────────┼──────────┐
                         ▼         ▼          ▼
                     [E-mail]  [Lambda]    [SQS]
                     (ativo)  (futuro)   (futuro)
```

| Camada | Serviço | Função |
|---|---|---|
| **Armazenamento** | Amazon S3 | Hospedagem dos objetos de mídia |
| **Segurança** | AWS IAM | Controle de acesso granular por usuário e ação |
| **Notificação** | Amazon SNS | Disparo de alertas via tópico `s3NotificationTopic` |

---

## 3. Especificações da Arquitetura (Deep Dive)

### Camada de Armazenamento — Amazon S3

Em vez de um modelo de *polling* — no qual o administrador consulta periodicamente o bucket para verificar mudanças — a solução adota o modelo **Event Notification (push)**. O S3 emite um evento automaticamente no momento em que qualquer alteração ocorre, eliminando latência e esforço manual.

Os eventos monitorados foram configurados com a seguinte granularidade:

- `s3:ObjectCreated:*` — cobre `PUT`, `POST`, `COPY` e `CompleteMultipartUpload`
- `s3:ObjectRemoved:*` — cobre deleções explícitas e por versionamento

### Camada de Segurança — AWS IAM

O `mediacouser` foi configurado seguindo o **Princípio do Menor Privilégio**: acesso estritamente limitado às ações necessárias para sua função, sem qualquer permissão administrativa sobre o bucket ou outros serviços da conta.

Permissões concedidas:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:DeleteObject",
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::cafeteria-media-storage/*"
}
```

Operações explicitamente bloqueadas: `s3:DeleteBucket`, `s3:PutBucketPolicy`, `s3:PutBucketNotification` — impedindo que o usuário externo altere a própria estrutura de segurança do ambiente.

### Camada de Notificação — Amazon SNS (Fan-out)

O uso do SNS como intermediário (em vez de notificação direta por e-mail) é uma decisão arquitetural deliberada. O padrão **Fan-out** permite que, no futuro, um único evento do S3 alimente múltiplos consumidores de forma paralela e desacoplada:

- **E-mail** para o administrador (implementado neste lab)
- **AWS Lambda** para processamento automático da imagem (ex: geração de thumbnails)
- **Amazon SQS** para enfileiramento e processamento assíncrono

---

## 4. Implementação Técnica Detalhada

### Fase 1 — Provisionamento via AWS CLI

Toda a infraestrutura foi provisionada exclusivamente via linha de comando, simulando um fluxo de automação real e reprodutível:

```bash
# Criação do bucket
aws s3api create-bucket \
  --bucket cafeteria-media-storage \
  --region us-east-1

# Aplicação da política de acesso do mediacouser
aws s3api put-bucket-policy \
  --bucket cafeteria-media-storage \
  --policy file://bucket-policy.json

# Vinculação do bucket ao tópico SNS via arquivo de configuração
aws s3api put-bucket-notification-configuration \
  --bucket cafeteria-media-storage \
  --notification-configuration file://notification.json
```

### Fase 2 — O Desafio da Política do SNS

Um ponto crítico da implementação foi a **Access Policy do tópico SNS**. Por padrão, o SNS não aceita publicações de serviços externos, incluindo o próprio S3. Foi necessário editar a política do tópico `s3NotificationTopic` para autorizar explicitamente a ação `sns:Publish` originada do serviço `s3.amazonaws.com`:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "s3.amazonaws.com"
  },
  "Action": "sns:Publish",
  "Resource": "arn:aws:sns:us-east-1:ACCOUNT_ID:s3NotificationTopic",
  "Condition": {
    "ArnLike": {
      "aws:SourceArn": "arn:aws:s3:::cafeteria-media-storage"
    }
  }
}
```

A condição `aws:SourceArn` restringe a permissão ao bucket específico, evitando que qualquer outro recurso S3 da conta publique no mesmo tópico — uma boa prática de segurança frequentemente negligenciada.

---

## 5. Validação e Resultados

| Teste Realizado | Resultado Esperado | Status |
|---|---|:---:|
| Upload via `mediacouser` | Arquivo armazenado, permissão aceita | ✅ |
| Deleção de objeto | Evento `ObjectRemoved` disparado imediatamente | ✅ |
| Recebimento de e-mail | Notificação com payload JSON do evento em < 30s | ✅ |
| Acesso não autorizado (`PutBucketPolicy`) | Erro `Access Denied` via política IAM | ✅ |

---

## 6. Conclusão e Insights

Este laboratório solidifica o entendimento de **Arquiteturas Orientadas a Eventos (EDA)**, um dos paradigmas mais relevantes em sistemas de nuvem modernos. A principal lição é que processos manuais — como verificar periodicamente se um colaborador fez seu trabalho — podem ser substituídos por processos automáticos, auditáveis e de baixo custo operacional.

O uso exclusivo da CLI prepara para ambientes **DevOps e SRE**, onde scripts de automação substituem completamente a interface gráfica, permitindo versionamento da infraestrutura e integração com pipelines de CI/CD.

---

## 7. Propriedade Intelectual e Créditos

- **Lab Foundation:** Amazon Web Services Academy.
- **Technical Writing & Engineering:** [Seu Nome].

---

*Documentação gerada como parte de portfólio técnico de estudos em computação em nuvem.*
