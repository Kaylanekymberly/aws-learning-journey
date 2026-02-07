#  Lab 17: Monitoramento e Troubleshooting com VPC Flow Logs

[![AWS](https://img.shields.io/badge/AWS-VPC%20Flow%20Logs-FF9900?logo=amazon-aws)](https://aws.amazon.com/)
[![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-1E88E5)](#)
[![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-success)](#)
[![Date](https://img.shields.io/badge/Data-07%2F02%2F2026-blue)](#)

---

##  1. Visão Geral

Neste laboratório, o foco foi a implementação de **visibilidade operacional** sobre o tráfego de rede da VPC. Através da ativação e análise dos **VPC Flow Logs**, foi possível monitorar padrões de tráfego, identificar conexões negadas por Security Groups ou Network ACLs e solucionar problemas de conectividade em tempo real.

**Contexto Profissional:**  
Em ambientes de produção, a análise de Flow Logs é essencial para:
-  Detecção de tentativas de intrusão
-  Diagnóstico rápido de falhas de conectividade
-  Auditoria de conformidade (PCI-DSS, HIPAA, SOC 2)
-  Otimização de custos através da análise de tráfego

---

##  2. Objetivos Alcançados

-  **Habilitação de VPC Flow Logs:** Configuração da coleta de logs de tráfego IP para as interfaces de rede da VPC
-  **Análise de Dados de Rede:** Interpretação dos registros de fluxo para distinguir entre tráfego aceito (`ACCEPT`) e rejeitado (`REJECT`)
-  **Resolução de Problemas (Troubleshooting):** Identificação de falhas de configuração em regras de firewalls (Security Groups e NACLs)
-  **Auditoria:** Uso dos logs para garantir que as políticas de segurança de rede estão sendo aplicadas corretamente

---

##  3. Fluxo de Implementação

### 3.1 Configuração de Logs

Os logs de fluxo foram configurados para serem entregues em um grupo de logs do **Amazon CloudWatch**, permitindo a busca por padrões específicos (como tráfego na porta 80 ou 443).

**Comandos Utilizados:**

```bash
# Criar grupo de logs no CloudWatch
aws logs create-log-group --log-group-name /aws/vpc/flowlogs

# Habilitar Flow Logs na VPC
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-xxxxxxxxx \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name /aws/vpc/flowlogs \
    --deliver-logs-permission-arn arn:aws:iam::ACCOUNT-ID:role/VPCFlowLogsRole
```

### 3.2 Estrutura de um Log de Fluxo

A análise baseou-se na interpretação dos campos fundamentais:

```
<version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> 
<protocol> <packets> <bytes> <start> <end> <action> <log-status>
```

**Campos Críticos para Troubleshooting:**

| Campo | Descrição | Uso no Troubleshooting |
|-------|-----------|------------------------|
| **srcaddr** | IP de origem | Identificar origem de ataques ou tráfego indesejado |
| **dstaddr** | IP de destino | Validar se tráfego está alcançando o destino correto |
| **dstport** | Porta de destino | Essencial para validar serviços (80=HTTP, 443=HTTPS, 22=SSH) |
| **protocol** | Protocolo (6=TCP, 17=UDP, 1=ICMP) | Determinar tipo de conexão |
| **action** | ACCEPT ou REJECT | **Indicador principal de problemas** |
| **packets** | Número de pacotes | Volume baixo + REJECT = problema de configuração |
| **bytes** | Bytes transferidos | Detectar transferências anormalmente grandes |

**Exemplo de Registro Real:**

```
2 123456789012 eni-0a1b2c3d4e5f 10.0.1.50 172.217.14.206 49152 443 6 8 4096 1644242400 1644242460 ACCEPT OK
```

**Interpretação:**
-  Conexão HTTPS (porta 443) bem-sucedida
-  Origem: `10.0.1.50` (instância EC2)
-  Destino: `172.217.14.206` (servidor externo)
-  8 pacotes, 4096 bytes transferidos

---

##  4. Dicas de Troubleshooting - Guia Completo

###  Cenário 1: "Minha instância não consegue acessar a Internet"

#### Sintomas:
```bash
# Dentro da instância EC2
ping 8.8.8.8
# RESULTADO: Request timeout ou Connection timed out
```

#### Análise de Flow Logs:

**Buscar no CloudWatch Logs:**
```bash
aws logs filter-log-events \
    --log-group-name /aws/vpc/flowlogs \
    --filter-pattern '[version, account, eni, source=10.0.1.*, destination, srcport, dstport, protocol, packets, bytes, windowstart, windowend, action=REJECT, flowlogstatus]'
```

**Padrão de Log Encontrado:**
```
2 123456789012 eni-0a1b2c3d 10.0.1.50 8.8.8.8 54321 0 1 5 350 1644242400 1644242460 REJECT OK
```

#### Diagnóstico:
-  **action = REJECT** → Tráfego bloqueado
- **dstport = 0** e **protocol = 1** → Tentativa de ICMP (ping)
- **srcaddr = 10.0.1.50** → Instância em sub-rede privada

#### Possíveis Causas:

| Causa | Como Verificar | Solução |
|-------|----------------|---------|
| **Sem rota para IGW/NAT** | `aws ec2 describe-route-tables` | Adicionar rota `0.0.0.0/0 → NAT Gateway` |
| **Security Group bloqueando saída** | Verificar regras Outbound do SG | Permitir `All Traffic` para `0.0.0.0/0` |
| **NACL bloqueando ICMP** | `aws ec2 describe-network-acls` | Adicionar regra permitindo ICMP outbound |
| **NAT Gateway em subnet errada** | Verificar subnet pública com rota para IGW | Recriar NAT Gateway na subnet pública |

#### Solução Passo a Passo:

```bash
# 1. Verificar tabela de rotas da subnet privada
aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=subnet-private"

# 2. Verificar se existe rota para NAT Gateway
# Se não existir, criar:
aws ec2 create-route \
    --route-table-id rtb-private \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id nat-xxxxxxxxx

# 3. Validar Security Group
aws ec2 describe-security-groups --group-ids sg-xxxxxxxxx
```

---

###  Cenário 2: "SSH funciona às vezes, mas não sempre"

#### Sintomas:
```bash
ssh ec2-user@10.0.1.50
# Às vezes conecta, às vezes: Connection timed out
```

#### Análise de Flow Logs:

**Query CloudWatch Insights:**
```sql
fields @timestamp, srcaddr, dstaddr, srcport, dstport, action
| filter dstport = 22
| stats count() by action, srcaddr
| sort @timestamp desc
```

**Resultado Encontrado:**
```
srcaddr          action   count
203.0.113.25     ACCEPT   15
203.0.113.25     REJECT   45
198.51.100.10    REJECT   100
```

#### Diagnóstico:
-  Mesmo IP tem **ACCEPT e REJECT** → Problema de configuração assimétrica
-  IP `198.51.100.10` só tem REJECT → IP bloqueado (possível blacklist)

#### Possíveis Causas:

**1. NACL com Regras Conflitantes:**
```bash
# Verificar regras da NACL
aws ec2 describe-network-acls --network-acl-ids acl-xxxxxxxxx
```

**Exemplo de Problema Encontrado:**
```yaml
Inbound Rules:
  100   Allow   TCP   22   203.0.113.0/24
  *     Deny    ALL   *    0.0.0.0/0

Outbound Rules:
  100   Deny    TCP   1024-65535   0.0.0.0/0   #  PROBLEMA!
  *     Allow   ALL   *             0.0.0.0/0
```

**Explicação:**  
SSH usa portas **efêmeras** (1024-65535) para respostas. Se o tráfego de saída nessas portas estiver bloqueado, a conexão falha.

**Solução:**
```bash
# Remover regra 100 de Outbound que bloqueia portas efêmeras
aws ec2 delete-network-acl-entry \
    --network-acl-id acl-xxxxxxxxx \
    --rule-number 100 \
    --egress
```

**2. Security Group sem Regra Stateful Adequada:**

Security Groups são **stateful** (resposta automática), mas NACLs são **stateless**.

---

###  Cenário 3: "Aplicação Web não está acessível"

#### Sintomas:
```bash
curl http://10.0.1.50
# RESULTADO: Connection refused ou timeout
```

#### Análise de Flow Logs:

**Buscar tráfego HTTP:**
```bash
aws logs filter-log-events \
    --log-group-name /aws/vpc/flowlogs \
    --filter-pattern '[version, account, eni, source, destination=10.0.1.50, srcport, dstport=80, protocol, packets, bytes, windowstart, windowend, action, flowlogstatus]'
```

**Padrão Encontrado:**
```
2 123456789012 eni-0a1b2c3d 203.0.113.5 10.0.1.50 54321 80 6 0 0 1644242400 1644242460 REJECT OK
```

#### Diagnóstico:
- **packets = 0, bytes = 0** → Conexão rejeitada imediatamente
- **dstport = 80, action = REJECT** → Security Group não permite HTTP

#### Solução:

```bash
# Adicionar regra permitindo HTTP no Security Group
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

**Validar que o serviço está rodando na instância:**
```bash
# Conectar via SSH e verificar
sudo netstat -tulpn | grep :80
```

---

###  Cenário 4: "Alto Tráfego Rejeitado em Porta Suspeita"

#### Descoberta nos Logs:

**Query CloudWatch Insights:**
```sql
fields srcaddr, dstport, action
| filter action = "REJECT" and dstport = 3389
| stats count() as attempts by srcaddr
| sort attempts desc
| limit 10
```

**Resultado:**
```
srcaddr           attempts
198.51.100.99     1500
203.0.113.44      890
192.0.2.15        650
```

#### Diagnóstico:
- **Porta 3389 = RDP (Remote Desktop Protocol)**
-  Múltiplas tentativas de conexão rejeitadas
-  **Possível tentativa de brute force ou port scanning**

#### Ações de Mitigação:

**1. Bloquear IPs Maliciosos via NACL:**
```bash
# Adicionar regra de bloqueio explícita
aws ec2 create-network-acl-entry \
    --network-acl-id acl-xxxxxxxxx \
    --rule-number 10 \
    --protocol tcp \
    --port-range From=3389,To=3389 \
    --cidr-block 198.51.100.99/32 \
    --rule-action deny \
    --ingress
```

**2. Configurar Alarme no CloudWatch:**
```bash
# Criar filtro de métrica
aws logs put-metric-filter \
    --log-group-name /aws/vpc/flowlogs \
    --filter-name RDPBruteForce \
    --filter-pattern '[version, account, eni, source, destination, srcport, dstport=3389, protocol, packets, bytes, windowstart, windowend, action=REJECT, flowlogstatus]' \
    --metric-transformations \
        metricName=RDPAttempts,metricNamespace=Security,metricValue=1

# Criar alarme
aws cloudwatch put-metric-alarm \
    --alarm-name RDP-Brute-Force-Alert \
    --metric-name RDPAttempts \
    --namespace Security \
    --statistic Sum \
    --period 300 \
    --evaluation-periods 1 \
    --threshold 50 \
    --comparison-operator GreaterThanThreshold
```

---

##  5. Queries Úteis para CloudWatch Insights

### Query 1: Top 10 IPs com Mais Conexões Rejeitadas

```sql
fields @timestamp, srcaddr, dstaddr, dstport, action
| filter action = "REJECT"
| stats count() as rejected_count by srcaddr
| sort rejected_count desc
| limit 10
```

### Query 2: Detecção de Port Scanning

```sql
fields srcaddr, dstport
| filter action = "REJECT"
| stats count_distinct(dstport) as unique_ports by srcaddr
| filter unique_ports > 10
| sort unique_ports desc
```
**Interpretação:** IPs tentando acessar mais de 10 portas diferentes podem estar fazendo varredura de rede.

### Query 3: Volume de Tráfego por Serviço

```sql
fields dstport, bytes
| stats sum(bytes) as total_bytes by dstport
| sort total_bytes desc
| limit 10
```

### Query 4: Identificar Tráfego de Saída Suspeito

```sql
fields @timestamp, srcaddr, dstaddr, dstport, bytes, action
| filter action = "ACCEPT" and bytes > 10000000
| sort bytes desc
```
**Uso:** Detectar possível exfiltração de dados (mais de 10MB por conexão).

---

##  6. Boas Práticas Aprendidas

###  Monitoramento Proativo
- Habilitar Flow Logs em **todas as VPCs de produção**
- Configurar alertas para padrões anômalos (>100 rejeições/min)
- Reter logs por **no mínimo 90 dias** para conformidade

###  Troubleshooting Estruturado
1. **Sempre começar pelos Flow Logs** antes de mudar configurações
2. Verificar **action = REJECT** para identificar bloqueios
3. Correlacionar timestamps com eventos de aplicação
4. Usar CloudWatch Insights para análise agregada

###  Segurança
- Monitorar tentativas de acesso a portas sensíveis (3389, 1433, 3306)
- Implementar rate limiting para SSH (porta 22)
- Criar dashboards com métricas de segurança

###  Otimização de Custos
- Usar filtros para capturar apenas tráfego rejeitado em ambientes não-críticos
- Exportar logs antigos para S3 (mais barato que CloudWatch)
- Definir retenção adequada (7-90 dias dependendo da necessidade)

---

##  7. Erros Comuns e Como Evitá-los

| Erro | Sintoma | Prevenção |
|------|---------|-----------|
| **Flow Logs não aparecem** | Nenhum dado no CloudWatch | Aguardar 15min; verificar IAM Role |
| **Muitos REJECTs legítimos** | Logs cheios de tráfego normal bloqueado | Revisar Security Groups; criar filtro para capturar apenas anomalias |
| **Custo elevado de CloudWatch** | Fatura inesperada | Usar S3 como destino; aplicar filtros; reduzir retenção |
| **Logs de múltiplas ENIs misturados** | Difícil identificar origem | Criar Flow Logs separados por subnet ou criar tags |

---

##  8. Conclusão e Insights Técnicos

A capacidade de analisar **VPC Flow Logs** é uma habilidade crítica para um **Arquiteto de Soluções Cloud**. Este laboratório demonstrou que o monitoramento não serve apenas para segurança, mas é a **ferramenta definitiva para Troubleshooting**, evitando "adivinhações" ao configurar regras de rede complexas.

### Principais Competências Desenvolvidas:

 **Análise Forense de Rede** - Capacidade de rastrear conexões e identificar gargalos  
 **Troubleshooting Sistemático** - Abordagem baseada em dados, não em tentativa e erro  
 **Otimização de Segurança** - Detecção de ameaças e vulnerabilidades antes que causem impacto  
 **Conformidade e Auditoria** - Evidências concretas para relatórios regulatórios  

---

##  Direitos Autorais

**Propriedade da Documentação:**  
**Kaylane Kimberly**

**Ambiente:** AWS Cloud  
**Data:** 07/02/2026  
**Status:**  Concluído com Sucesso

---

###  Aviso de Propriedade Intelectual

Este documento é um registro técnico de aprendizado pessoal desenvolvido por **Kaylane Kimberly**. Os conceitos, nomes de serviços e a estrutura original do laboratório são de propriedade intelectual da **Amazon Web Services (AWS)**.

**Todos os direitos reservados à Amazon.com, Inc.**

**AWS®**, **CloudWatch®**, **VPC®** e demais marcas são registradas da Amazon Web Services, Inc. ou suas afiliadas.

---

<div align="center">

[![AWS Certified](https://img.shields.io/badge/AWS-Learning%20Path-orange?logo=amazon-aws)](https://aws.amazon.com/)
[![GitHub](https://img.shields.io/badge/GitHub-Portfolio-black?logo=github)](https://github.com)

**Documentação Técnica | Kaylane Kimberly | 2026**

</div>
