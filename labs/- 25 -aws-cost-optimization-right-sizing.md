#  Lab 25 | Cloud Economics: Otimização de Recursos e Redução de Custos (Right Sizing)

**Autora:** Kaylane Kimberly  
**Data:** 14 de Fevereiro de 2026  
**Tecnologias:** `Amazon EC2` `Amazon RDS` `AWS Pricing Calculator` `AWS CLI` `Amazon EBS`

> [!IMPORTANT]
> **Direitos Autorais:**
> - **AWS:** O cenário, arquitetura e scripts originais são propriedade intelectual da **Amazon Web Services, Inc.** Esta documentação é um registro de execução prática de treinamento oficial AWS.
> - **Documentação:** Este relatório técnico, análises de saída e resolução dos desafios foram produzidos por **Kaylane Kimberly**.

---

##  Visão Geral e Caso de Negócio

Após a migração bem-sucedida do banco de dados local para o **Amazon RDS**, a instância EC2 original tornou-se subutilizada — um fenômeno comum em ambientes cloud chamado de **over-provisioning** (superprovisionamento).

O objetivo desta atividade é aplicar o conceito de **Right Sizing**: ajustar o tamanho dos recursos computacionais à demanda real da aplicação, eliminando desperdício e reduzindo custos sem comprometer a performance.

### Objetivos

- Remover componentes legados que aumentam o footprint desnecessariamente
- Redimensionar a instância EC2 para o tipo adequado à nova carga de trabalho
- Quantificar a economia gerada usando a **AWS Pricing Calculator**
- Documentar o processo como referência de governança de custos

---

##  Conceitos Técnicos

### O que é Right Sizing?

**Right Sizing** é a prática de selecionar o tipo e tamanho de instância mais adequado para uma carga de trabalho específica. Na AWS, pagar por recursos que não são utilizados é um dos principais responsáveis por faturas elevadas.

> Exemplo prático: uma instância `t3.small` rodando apenas um servidor web PHP consome 2x mais RAM do que precisa. Migrar para `t3.micro` mantém a performance e reduz o custo em ~50%.

### O que é Over-Provisioning?

Ocorre quando alocamos mais recursos do que a aplicação consome. É comum após migrações, quando o ambiente original foi dimensionado para uma carga que não existe mais — exatamente o cenário deste laboratório.

### Família de Instâncias T3

As instâncias da família **T3** são do tipo *burstable performance* — ideais para cargas de trabalho com uso moderado e intermitente de CPU, como servidores web e ambientes de desenvolvimento.

| Tipo | vCPU | RAM | Uso Ideal |
|---|---|---|---|
| `t3.nano` | 2 | 0,5 GB | Testes e microserviços |
| `t3.micro` | 2 | 1 GB |  Servidor web leve (nosso caso) |
| `t3.small` | 2 | 2 GB | App + banco local (cenário anterior) |
| `t3.medium` | 2 | 4 GB | Aplicações com picos de carga |

### Por que separar App e Banco de Dados?

Quando o banco de dados roda na mesma instância EC2 que a aplicação, ambos competem pelos mesmos recursos de CPU e RAM. Ao migrar o banco para o **Amazon RDS**, cada camada é escalada de forma independente, seguindo o princípio de **separação de responsabilidades** (Separation of Concerns).

---

##  Etapas de Execução

### Fase 1 — Limpeza de Armazenamento (Cleanup)

Com o banco de dados agora gerenciado pelo RDS, o motor local instalado na EC2 tornou-se um processo ocioso consumindo disco, CPU e RAM sem necessidade.

**1.1 — Verificar o estado atual antes de remover:**
```bash
# Checar se o serviço de banco local está rodando
sudo systemctl status mariadb

# Ver uso de disco antes da limpeza
df -h

# Verificar quanto o MariaDB ocupa em disco
sudo du -sh /var/lib/mysql
```

**Saída esperada (exemplo didático):**
```
Filesystem      Size  Used Avail Use%
/dev/xvda1       8G   5.8G  2.2G  73%   ← alto uso de disco
/var/lib/mysql: 1.4G
```

**1.2 — Parar e desabilitar o serviço:**
```bash
# Parar o serviço imediatamente
sudo systemctl stop mariadb

# Desabilitar inicialização automática
sudo systemctl disable mariadb
```

**1.3 — Desinstalar o banco de dados local:**
```bash
# Remover MariaDB e dependências não utilizadas
sudo yum remove mariadb-server -y

# Limpar arquivos residuais de dados
sudo rm -rf /var/lib/mysql

# Confirmar espaço recuperado
df -h
```

**Saída esperada após a limpeza:**
```
Filesystem      Size  Used Avail Use%
/dev/xvda1       8G   4.4G  3.6G  55%   ← ~1.4G recuperados
```

**1.4 — Confirmar que a aplicação continua funcional com o RDS:**
```bash
# Testar conectividade com o endpoint do RDS
mysql -h <rds-endpoint>.rds.amazonaws.com -u admin -p -e "SHOW DATABASES;"

# Verificar se o site ainda responde normalmente
curl -I http://localhost/cafe
```

>  **Por que isso importa?** Cada GB armazenado no EBS tem custo (~$0,10/GB/mês). Remover 1,4 GB de dados desnecessários não é só limpeza — é governança de custos.

---

### Fase 2 — Redimensionamento da Instância (Right Sizing)

Com a carga de banco de dados removida, a instância `t3.small` está operando muito abaixo de sua capacidade. É o momento de aplicar o Right Sizing.

**2.1 — Coletar métricas de uso antes de redimensionar:**
```bash
# Ver uso atual de CPU e memória
top -bn1 | grep "Cpu\|Mem"

# Verificar uso médio de CPU nas últimas 24h (via CloudWatch CLI)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=<instance-id> \
  --start-time $(date -u -d '24 hours ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 3600 \
  --statistics Average \
  --output table
```

>  **Boa prática:** nunca redimensione sem antes coletar métricas reais. Um uso médio de CPU abaixo de 20% é um forte indicador de over-provisioning.

**2.2 — Interromper a instância:**

A instância deve estar no estado **Stopped** para que o tipo possa ser alterado. Isso causa uma breve indisponibilidade — planeje uma janela de manutenção em ambientes de produção.

```bash
# Via AWS CLI
aws ec2 stop-instances --instance-ids <instance-id>

# Aguardar o estado "stopped"
aws ec2 wait instance-stopped --instance-ids <instance-id>

echo "Instância parada. Pronta para redimensionamento."
```

**2.3 — Alterar o tipo da instância:**
```bash
aws ec2 modify-instance-attribute \
  --instance-id <instance-id> \
  --instance-type '{"Value": "t3.micro"}'
```

**2.4 — Reiniciar e validar:**
```bash
# Iniciar a instância
aws ec2 start-instances --instance-ids <instance-id>

# Aguardar estado "running"
aws ec2 wait instance-running --instance-ids <instance-id>

# Confirmar o novo tipo
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[*].Instances[*].InstanceType" \
  --output text
```

**Saída esperada:**
```
t3.micro
```

**2.5 — Teste final de sanidade da aplicação:**
```bash
# Verificar se o site da cafeteria responde corretamente
curl -s -o /dev/null -w "%{http_code}" http://localhost/cafe
# Esperado: 200

# Confirmar conexão com o RDS ainda ativa
mysql -h <rds-endpoint>.rds.amazonaws.com -u admin -p -e "SELECT 1;"
```

---

### Fase 3 — Estimativa de Custos com AWS Pricing Calculator

A **AWS Pricing Calculator** permite simular cenários de custo antes e depois de mudanças arquiteturais. Esta etapa é essencial para comunicar o valor financeiro das decisões técnicas à liderança.

**Comparativo de custo mensal estimado (região us-east-1, On-Demand):**

| Item |  Antes |  Depois | Diferença |
|---|---|---|---|
| **Tipo de Instância EC2** | `t3.small` | `t3.micro` | — |
| **vCPU / RAM** | 2 vCPU / 2 GB | 2 vCPU / 1 GB | -1 GB RAM |
| **Custo EC2/mês** | ~$15,18 | ~$7,59 | **-$7,59** |
| **Armazenamento EBS (gp2, 8GB)** | ~$0,80 | ~$0,80 | = |
| **Banco de dados** | Embutido na EC2 | Amazon RDS `db.t3.micro` | Gerenciado |
| **Custo RDS/mês** | $0 (embutido) | ~$12,41 | +$12,41 |
| **Total mensal estimado** | ~$15,98 | ~$20,80 | +$4,82 |

>  **Importante:** o custo total aumenta levemente com o RDS separado, mas os **benefícios ocultos** justificam: alta disponibilidade automática, backups gerenciados, patches de segurança sem downtime e escalabilidade independente. O custo operacional de manter um banco manual (horas de engenharia) torna o RDS mais econômico no longo prazo.

**Economia isolada apenas na camada EC2:**

| Métrica | Valor |
|---|---|
| Economia mensal na instância EC2 | ~$7,59/mês |
| Economia anual projetada | ~**$91,08/ano** |
| Redução de uso de RAM | 50% |
| Redução no footprint de disco | ~1,4 GB |

---

##  Conclusão e Aprendizados

### O que este laboratório demonstrou

A otimização em nuvem não é um evento único — é um **processo contínuo** de revisão e ajuste. Ambientes que não passam por Right Sizing regularmente acumulam desperdício silencioso que cresce com o tempo.

Neste laboratório, o simples ato de remover um banco de dados local e redimensionar uma instância resultou em:

- **50% de redução** no custo da instância EC2
- **Eliminação de ~1,4 GB** de dados desnecessários em disco
- **Arquitetura mais limpa**, com separação clara entre camada de aplicação e dados
- **Maior resiliência**, com o banco de dados gerenciado pelo RDS

### Habilidades consolidadas

| Área | Competência Desenvolvida |
|---|---|
| **Cloud Economics** | Identificar e quantificar desperdício de recursos em contas AWS |
| **Right Sizing** | Redimensionar instâncias com base em métricas reais de uso |
| **AWS CLI** | Automatizar stop, modify e start de instâncias via linha de comando |
| **AWS Pricing Calculator** | Construir cenários de custo comparativos para tomada de decisão |
| **Boas Práticas AWS** | Separação de camadas (App vs. DB) e princípio do mínimo recurso necessário |

### Reflexão final

> *"Na nuvem, pagar pelo que você não usa é tão problemático quanto não ter recursos suficientes. Right Sizing é o equilíbrio entre performance e eficiência financeira."*

---

*Laboratório concluído com sucesso — Kaylane Kimberly, 14/02/2026*
