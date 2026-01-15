# Lab 01: Explorando o Console AWS

##  Data
09 de Dezembro de 2025

##  Objetivo
Familiarizar-me com a interface do Console AWS e entender a navegação básica, além de compreender conceitos fundamentais de infraestrutura em nuvem.

---

##  O que eu fiz

### 1. Login no Console
- ✅ Acessei o Console AWS em https://console.aws.amazon.com
- ✅ Explorei as diferentes regiões disponíveis no seletor superior direito
- ✅ Entendi o conceito de Availability Zones e sua importância para alta disponibilidade
- ✅ Verifiquei minha região padrão (recomendado: **us-east-1** para Free Tier)

### 2. Exploração dos Serviços

Naveguei pelos principais serviços AWS e entendi suas finalidades:

#### **EC2 - Elastic Compute Cloud**
- **O que é:** Servidores virtuais na nuvem
- **Para que serve:** Hospedar aplicações, sites, APIs, processar dados
- **Analogia:** É como alugar um computador na nuvem que você pode ligar/desligar quando quiser
- **Free Tier:** 750 horas/mês de instância t2.micro por 12 meses

#### **S3 - Simple Storage Service**
- **O que é:** Armazenamento de objetos (arquivos) na nuvem
- **Para que serve:** Guardar imagens, vídeos, backups, dados, hospedar sites estáticos
- **Analogia:** É como um "Google Drive/Dropbox infinito" da AWS
- **Free Tier:** 5 GB de armazenamento padrão por 12 meses

#### **RDS - Relational Database Service**
- **O que é:** Banco de dados relacional gerenciado
- **Para que serve:** MySQL, PostgreSQL, SQL Server sem precisar gerenciar o servidor
- **Analogia:** Banco de dados "pronto para usar" sem se preocupar com manutenção
- **Free Tier:** 750 horas/mês de instância db.t3.micro por 12 meses

#### **VPC - Virtual Private Cloud**
- **O que é:** Rede virtual isolada na AWS
- **Para que serve:** Criar sua própria rede privada, controlar acesso, segurança
- **Analogia:** É como ter sua própria "rede local" mas na nuvem da AWS
- **Free Tier:** Totalmente gratuito (só paga por recursos dentro dela)

---

### 3. Conceitos Aprendidos

#### 🌍 Regiões AWS

**O que são?**  
Regiões são localizações geográficas físicas ao redor do mundo onde a AWS possui data centers. Atualmente existem **33 regiões** globalmente.

**Exemplos de Regiões:**
- **us-east-1** (Norte da Virgínia, EUA) - mais popular, mais serviços disponíveis
- **sa-east-1** (São Paulo, Brasil) - menor latência para usuários brasileiros
- **eu-west-1** (Irlanda, Europa) - para usuários europeus
- **ap-southeast-1** (Singapura, Ásia) - para usuários asiáticos

**Por que escolher uma região?**
- ✅ **Latência:** Usuários no Brasil = escolher sa-east-1 (mais rápido)
- ✅ **Compliance:** Leis como LGPD exigem dados no Brasil
- ✅ **Custos:** Regiões diferentes = preços diferentes (us-east-1 geralmente mais barato)
- ✅ **Disponibilidade de serviços:** Nem todos os serviços estão em todas as regiões

**Availability Zones (AZs):**
- Cada região tem **múltiplas AZs** (mínimo 3)
- AZs são data centers fisicamente separados na mesma região
- Exemplo: us-east-1 tem 6 AZs (us-east-1a, us-east-1b, us-east-1c, etc)
- **Benefício:** Se uma AZ cair, suas aplicações continuam nas outras

---

####  Free Tier

**O que é?**  
Programa da AWS que permite usar diversos serviços **gratuitamente** por 12 meses após criar a conta, mais alguns serviços sempre gratuitos.

**Tipos de Free Tier:**

**1. 12 Meses Gratuitos** (a partir da criação da conta):
| Serviço | Limite Gratuito |
|---------|-----------------|
| EC2 | 750 horas/mês de t2.micro |
| RDS | 750 horas/mês de db.t3.micro |
| S3 | 5 GB de armazenamento padrão |
| CloudFront | 50 GB de transferência |
| Lambda | 1 milhão de requisições/mês |

**2. Sempre Gratuito** (não expira):
| Serviço | Limite Gratuito |
|---------|-----------------|
| Lambda | 1 milhão requisições/mês |
| DynamoDB | 25 GB de armazenamento |
| CloudWatch | 10 métricas customizadas |
| SNS | 1 milhão de publicações |

**3. Testes Gratuitos** (curto prazo):
- Amazon SageMaker: 250 horas/mês por 2 meses
- Amazon Redshift: 750 horas/mês por 2 meses

**⚠️ CUIDADO para não sair do Free Tier:**
- Sempre usar instâncias t2.micro (EC2) e db.t3.micro (RDS)
- Não deixar recursos ligados 24/7 sem necessidade
- Configurar **alertas de billing** para ser avisado se passar de $5

---

####  IAM - Identity and Access Management

**O que é?**  
Sistema de gerenciamento de permissões e acessos da AWS. É o "guarda de segurança" da sua conta.

**Conceitos Principais:**

**1. Root User (Usuário Raiz):**
- É a conta criada quando você se registra na AWS
- Tem acesso **total** e **irrestrito** a tudo
- ⚠️ **NUNCA use no dia a dia!** Só para tarefas administrativas críticas
- **SEMPRE habilitar MFA** (autenticação de 2 fatores)

**2. IAM Users (Usuários):**
- Contas individuais para pessoas ou aplicações
- Exemplo: criar usuário "joao-dev" para o desenvolvedor João
- Cada user tem suas próprias credenciais (login/senha ou access keys)

**3. IAM Groups (Grupos):**
- Conjuntos de usuários com permissões similares
- Exemplo: Grupo "Desenvolvedores" com permissões de EC2 e S3
- Facilita gerenciar permissões de várias pessoas

**4. IAM Roles (Funções):**
- Permissões temporárias que podem ser assumidas
- Exemplo: EC2 precisa acessar S3 → criar role com permissão S3
- **Melhor prática:** Use roles ao invés de access keys hardcoded

**5. IAM Policies (Políticas):**
- Documentos JSON que definem permissões
- Exemplo de política que permite listar buckets S3:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:ListBucket",
    "Resource": "*"
  }]
}
```

**Princípio do Menor Privilégio:**
- Sempre dar apenas as permissões **mínimas necessárias**
- Exemplo: Se o usuário só precisa ler S3, NÃO dar permissão de deletar

**Boas Práticas IAM:**
- ✅ Habilitar MFA na conta root
- ✅ Criar usuários IAM individuais (não compartilhar credenciais)
- ✅ Usar grupos para gerenciar permissões
- ✅ Aplicar princípio do menor privilégio
- ✅ Rotacionar credenciais regularmente
- ✅ Usar roles para aplicações (EC2, Lambda, etc)
- ✅ Monitorar atividades com CloudTrail

---

##  Prints

Para adicionar prints de tela no seu repositório GitHub:

**Opção 1: Pasta local**
```
lab01-explorando-console/
├── README.md
└── images/
    ├── console-home.png
    ├── ec2-dashboard.png
    ├── s3-buckets.png
    └── iam-dashboard.png
```

**Opção 2: Hospedagem online**
- [Imgur](https://imgur.com) - Upload de imagens grátis
- Depois linkar no markdown: `![Descrição](https://i.imgur.com/abc123.png)`

---

## ✅ Resultado

✨ **Consegui navegar com confiança pelo console AWS e entender a estrutura básica dos principais serviços!**

**Checklist de Conquistas:**
- [x] Entendi o que são Regiões e Availability Zones
- [x] Explorei os 4 principais serviços (EC2, S3, RDS, VPC)
- [x] Compreendi o funcionamento do Free Tier
- [x] Aprendi conceitos fundamentais de IAM
- [x] Localizei onde criar recursos em cada serviço
- [x] Entendi a importância de MFA e segurança

---

## 📚 O que aprendi

### Principais Descobertas:

1. **A AWS tem 33 regiões globalmente** distribuídas estrategicamente
   - Cada região é completamente independente
   - Dados não atravessam regiões automaticamente (compliance)

2. **Cada região possui múltiplas Availability Zones (AZs)**
   - Mínimo de 3 AZs por região (algumas têm 6+)
   - AZs são conectadas por rede de baixa latência
   - Permite criar aplicações altamente disponíveis

3. **O Free Tier permite praticar sem custos por 12 meses**
   - 750 horas de EC2 t2.micro = deixar 1 instância ligada 24/7
   - 750 horas de RDS db.t3.micro = banco de dados gratuito
   - Precisa monitorar para não ultrapassar os limites

4. **Segurança começa com IAM**
   - Root user = usar apenas para setup inicial
   - MFA = obrigatório em produção
   - Princípio do menor privilégio = fundamental

5. **Console AWS é intuitivo e organizado**
   - Busca no topo encontra qualquer serviço rapidamente
   - Serviços favoritos podem ser fixados
   - Cada serviço tem dashboard próprio com métricas

---

## 🔗 Recursos utilizados

- 🌐 [Console AWS](https://console.aws.amazon.com)
- 📖 [Documentação Oficial AWS](https://docs.aws.amazon.com)
- 🎓 [AWS Free Tier](https://aws.amazon.com/free)
- 🔐 [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- 🌍 [Regiões e AZs](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)
- 📚 Material do curso AWS re/Start

---

## 💡 Próximos passos

### Lab 02: Criar Primeira Instância EC2
- [ ] Criar EC2 t2.micro com Amazon Linux 2023
- [ ] Configurar Security Group (SSH + HTTP)
- [ ] Conectar via SSH
- [ ] Instalar servidor web Apache
- [ ] Testar acesso pelo navegador

### Lab 03: Configurar Bucket S3
- [ ] Criar bucket S3
- [ ] Fazer upload de arquivos
- [ ] Configurar permissões públicas
- [ ] Hospedar site estático

### Lab 04: Explorar IAM na Prática
- [ ] Criar usuário IAM
- [ ] Criar grupo de desenvolvedores
- [ ] Atribuir políticas
- [ ] Habilitar MFA
- [ ] Testar permissões

---

##  Metas de Estudo

**Curto Prazo (1-2 semanas):**
- Completar 5 labs práticos
- Entender os 4 domínios da certificação
- Fazer primeiro simulado

**Médio Prazo (1 mês):**
- Completar todos os labs do AWS Skill Builder
- Revisar Well-Architected Framework
- Fazer 3 simulados (meta: 80%+ de acerto)

**Longo Prazo (2-3 meses):**
-  **AGENDAR PROVA AWS Cloud Practitioner**
- Construir projeto completo documentado no GitHub
- Conseguir a certificação! 🏆

---

**Status:** ✅ Concluído  
**Dificuldade:** ⭐ Iniciante  
**Tempo gasto:** ~45 minutos  
**Confiança:** 😊 Confortável com a interface

---

##  Notas Pessoais

*"Fiquei impressionado com a quantidade de serviços disponíveis! O console é mais intuitivo do que eu pensava. A parte de IAM parece complexa, mas entendi a importância de começar com segurança desde o início. Próximo passo: colocar a mão na massa criando uma instância EC2!"*

---

## 🏷️ Tags
`#AWS` `#CloudPractitioner` `#Console` `#IAM` `#FreeTier` `#Lab01` `#Fundamentos`
