# Lab 06 - AWS CLI com S3 e Hospedagem de Site Estático

**Autor:** Kaylanekymberly  
**Data:** 10 de janeiro de 2026  
**Status:** ✅ Concluído  
**Repositório:** [aws-learning-journey/labs](https://github.com/kaylanekymberly/aws-learning-journey)

---

##  Índice

1. [Visão Geral](#visão-geral)
2. [Objetivos do Lab](#objetivos-do-lab)
3. [Pré-requisitos](#pré-requisitos)
4. [Arquitetura da Solução](#arquitetura-da-solução)
5. [Passo a Passo Detalhado](#passo-a-passo-detalhado)
6. [Scripts Desenvolvidos](#scripts-desenvolvidos)
7. [Minhas Experiências](#minhas-experiências)
8. [Desafios Enfrentados](#desafios-enfrentados)
9. [Lições Aprendidas](#lições-aprendidas)
10. [Recursos e Referências](#recursos-e-referências)

---

##  Visão Geral

Este laboratório focou na prática com AWS CLI, especificamente trabalhando com **IAM** (Identity and Access Management) e **Amazon S3** (Simple Storage Service). O objetivo principal foi criar e hospedar um site estático no S3, além de desenvolver scripts de automação para upload de arquivos.

### Tarefas Concluídas

- Executar comandos AWS CLI com IAM e S3
- Implantar site estático em bucket S3
- Criar script de upload automático para S3

---

##  Objetivos do Lab

### Objetivos Principais

1. **Dominar AWS CLI com S3**: Aprender comandos essenciais para gerenciamento de buckets e objetos
2. **Configurar Hospedagem Estática**: Transformar bucket S3 em servidor web
3. **Automatizar Deploys**: Criar scripts para sincronização de arquivos
4. **Gerenciar Permissões IAM**: Configurar políticas de acesso adequadas

### Competências Desenvolvidas

- Comandos AWS CLI para S3 (cp, sync, ls, mb, rb)
- Configuração de bucket policies e ACLs
- Hospedagem de sites estáticos no S3
- Scripting bash para automação
- Gestão de permissões e segurança IAM

---

##  Pré-requisitos

### Conhecimentos Necessários

- Comandos básicos de terminal/bash
- Conceitos de HTTP e hospedagem web
- HTML/CSS básico
- Noções de AWS IAM e S3

### Ferramentas Necessárias

```bash
# AWS CLI v2 instalado e configurado
aws --version
# Resultado esperado: aws-cli/2.x.x

# Credenciais configuradas
aws configure list

# Git (opcional, para versionamento)
git --version
```

### Permissões IAM Necessárias

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:ListBucket",
        "s3:GetBucketWebsite",
        "s3:PutBucketWebsite",
        "s3:GetBucketPolicy",
        "s3:PutBucketPolicy",
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:PutObjectAcl"
      ],
      "Resource": [
        "arn:aws:s3:::*",
        "arn:aws:s3:::*/*"
      ]
    }
  ]
}
```

---

## Arquitetura da Solução

### Diagrama da Arquitetura

```
┌─────────────────────────────────────────────────────┐
│                   Internet                           │
└───────────────────┬─────────────────────────────────┘
                    │
                    │ HTTP/HTTPS
                    ▼
        ┌───────────────────────┐
        │   Amazon S3 Bucket    │
        │  (Static Website)     │
        │                       │
        │  ┌─────────────────┐ │
        │  │   index.html    │ │
        │  │   style.css     │ │
        │  │   script.js     │ │
        │  │   images/       │ │
        │  └─────────────────┘ │
        └───────────────────────┘
                    ▲
                    │
            ┌───────┴────────┐
            │                │
    ┌───────▼─────┐  ┌──────▼──────┐
    │  IAM User   │  │  IAM Policy │
    │   (você)    │  │ (permissões)│
    └─────────────┘  └─────────────┘
            │
            │ AWS CLI
            ▼
    ┌─────────────┐
    │ Script de   │
    │   Deploy    │
    └─────────────┘
```

### Componentes da Solução

1. **Bucket S3**: Armazena arquivos do site
2. **Static Website Hosting**: Recurso do S3 que serve arquivos HTML
3. **Bucket Policy**: Define quem pode acessar os arquivos
4. **IAM User/Permissions**: Controla quem pode fazer upload
5. **AWS CLI**: Interface para gerenciar recursos
6. **Script de Deploy**: Automatiza upload de arquivos


##  Passo a Passo Detalhado

### Etapa 1: Executar Comandos AWS CLI com IAM e S3

#### 1.1. Verificar Configuração AWS CLI

```bash
# Verificar identidade atual
aws sts get-caller-identity

# Resultado esperado:
# {
#     "UserId": "AIDAXXXXXXXXXX",
#     "Account": "123456789012",
#     "Arn": "arn:aws:iam::123456789012:user/seu-usuario"
# }
```

#### 1.2. Listar Políticas IAM Anexadas

```bash
# Listar políticas do usuário atual
aws iam list-attached-user-policies --user-name seu-usuario

# Listar todas as políticas S3
aws iam list-policies --scope AWS --query 'Policies[?contains(PolicyName, `S3`)].PolicyName'
```

#### 1.3. Comandos Básicos S3

```bash
# Listar todos os buckets
aws s3 ls

# Listar conteúdo de um bucket específico
aws s3 ls s3://nome-do-bucket/

# Listar com detalhes (tamanho, data)
aws s3 ls s3://nome-do-bucket/ --recursive --human-readable --summarize
```

#### 1.4. Criar Primeiro Bucket

```bash
# Criar bucket (nome deve ser único globalmente)
aws s3 mb s3://meu-site-estatico-$(date +%s)

# Verificar se foi criado
aws s3 ls | grep meu-site-estatico

# Salvar nome do bucket em variável
BUCKET_NAME="meu-site-estatico-1736553600"
echo "Bucket criado: $BUCKET_NAME"
```

---

### Etapa 2: Implantar Site Estático no S3

#### 2.1. Criar Estrutura do Site

```bash
# Criar diretório do projeto
mkdir -p meu-site-s3/{css,js,images}
cd meu-site-s3

# Criar arquivo HTML principal
cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meu Site no S3</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <h1> Meu Primeiro Site no AWS S3</h1>
        <p>Hospedado com sucesso usando AWS CLI!</p>
    </header>
    
    <main>
        <section class="hero">
            <h2>Bem-vindo ao Laboratório S3</h2>
            <p>Este site foi implantado automaticamente usando AWS CLI e está sendo servido diretamente do Amazon S3.</p>
        </section>
        
        <section class="features">
            <div class="feature">
                <h3> Rápido</h3>
                <p>Servido pela infraestrutura global da AWS</p>
            </div>
            <div class="feature">
                <h3> Econômico</h3>
                <p>Pague apenas pelo que usar</p>
            </div>
            <div class="feature">
                <h3> Seguro</h3>
                <p>Controle de acesso via IAM</p>
            </div>
        </section>
    </main>
    
    <footer>
        <p>© 2026 Kaylanekymberly | AWS Learning Journey</p>
    </footer>
    
    <script src="js/script.js"></script>
</body>
</html>
EOF

# Criar arquivo CSS
cat > css/style.css << 'EOF'
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    line-height: 1.6;
    color: #333;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
}

header {
    background: rgba(255, 255, 255, 0.95);
    padding: 2rem;
    text-align: center;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

header h1 {
    color: #667eea;
    font-size: 2.5rem;
    margin-bottom: 0.5rem;
}

main {
    max-width: 1200px;
    margin: 3rem auto;
    padding: 0 2rem;
}

.hero {
    background: white;
    padding: 3rem;
    border-radius: 10px;
    text-align: center;
    margin-bottom: 3rem;
    box-shadow: 0 5px 20px rgba(0,0,0,0.2);
}

.hero h2 {
    color: #667eea;
    font-size: 2rem;
    margin-bottom: 1rem;
}

.features {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
}

.feature {
    background: white;
    padding: 2rem;
    border-radius: 10px;
    text-align: center;
    box-shadow: 0 5px 20px rgba(0,0,0,0.2);
    transition: transform 0.3s ease;
}

.feature:hover {
    transform: translateY(-5px);
}

.feature h3 {
    color: #667eea;
    font-size: 1.5rem;
    margin-bottom: 1rem;
}

footer {
    background: rgba(255, 255, 255, 0.95);
    text-align: center;
    padding: 2rem;
    margin-top: 3rem;
}

@media (max-width: 768px) {
    header h1 {
        font-size: 1.8rem;
    }
    
    .hero h2 {
        font-size: 1.5rem;
    }
}
EOF

# Criar arquivo JavaScript
cat > js/script.js << 'EOF'
// Adicionar data/hora de carregamento
document.addEventListener('DOMContentLoaded', function() {
    const footer = document.querySelector('footer p');
    const now = new Date();
    const loadTime = now.toLocaleString('pt-BR');
    
    footer.innerHTML += `<br><small>Carregado em: ${loadTime}</small>`;
    
    // Animação de fade-in
    document.querySelectorAll('.feature').forEach((feature, index) => {
        setTimeout(() => {
            feature.style.opacity = '0';
            feature.style.animation = 'fadeIn 0.5s ease-in forwards';
        }, index * 200);
    });
});

// CSS para animação (injeta no head)
const style = document.createElement('style');
style.textContent = `
    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(20px); }
        to { opacity: 1; transform: translateY(0); }
    }
`;
document.head.appendChild(style);
EOF

# Criar página de erro 404
cat > error.html << 'EOF'
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>404 - Página Não Encontrada</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <h1> 404 - Página Não Encontrada</h1>
    </header>
    <main>
        <div class="hero">
            <h2>Ops! Esta página não existe</h2>
            <p><a href="index.html">Voltar para a página inicial</a></p>
        </div>
    </main>
</body>
</html>
EOF
```

#### 2.2. Fazer Upload dos Arquivos

```bash
# Upload de todos os arquivos de uma vez
aws s3 sync . s3://$BUCKET_NAME/ \
  --exclude ".git/*" \
  --exclude "*.sh" \
  --exclude "README.md"

# Verificar arquivos enviados
aws s3 ls s3://$BUCKET_NAME/ --recursive
```

#### 2.3. Configurar Bucket para Hospedagem Estática

```bash
# Habilitar hospedagem de site estático
aws s3 website s3://$BUCKET_NAME/ \
  --index-document index.html \
  --error-document error.html

# Verificar configuração
aws s3api get-bucket-website --bucket $BUCKET_NAME
```

#### 2.4. Configurar Política de Acesso Público

```bash
# Desbloquear acesso público no bucket
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
  "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"

# Criar arquivo de política
cat > bucket-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::$BUCKET_NAME/*"
    }
  ]
}
EOF

# Aplicar política ao bucket
aws s3api put-bucket-policy \
  --bucket $BUCKET_NAME \
  --policy file://bucket-policy.json

# Verificar política aplicada
aws s3api get-bucket-policy --bucket $BUCKET_NAME
```

#### 2.5. Acessar o Site

```bash
# Obter URL do site
WEBSITE_URL="http://$BUCKET_NAME.s3-website-$(aws configure get region).amazonaws.com"
echo "Site disponível em: $WEBSITE_URL"

# Testar com curl
curl -I $WEBSITE_URL

# Abrir no navegador (Linux)
xdg-open $WEBSITE_URL

# Abrir no navegador (macOS)
# open $WEBSITE_URL

# Abrir no navegador (Windows)
# start $WEBSITE_URL
```

---

### Etapa 3: Criar Script de Upload Automático

#### 3.1. Script Básico de Deploy

```bash
# Criar script de deploy
cat > deploy-to-s3.sh << 'EOF'
#!/bin/bash

# Script de Deploy Automático para S3
# Autor: Kaylanekymberly
# Data: 2026-01-10

set -euo pipefail

# Cores para output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Funções de logging
log_info() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

log_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

log_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
}

# Configurações
BUCKET_NAME="${1:-}"
SOURCE_DIR="${2:-.}"

# Validações
if [ -z "$BUCKET_NAME" ]; then
    log_error "Nome do bucket não especificado!"
    echo "Uso: $0 <nome-do-bucket> [diretorio-origem]"
    exit 1
fi

if [ ! -d "$SOURCE_DIR" ]; then
    log_error "Diretório '$SOURCE_DIR' não encontrado!"
    exit 1
fi

# Verificar se AWS CLI está instalado
if ! command -v aws &> /dev/null; then
    log_error "AWS CLI não está instalado!"
    exit 1
fi

# Verificar credenciais AWS
log_info "Verificando credenciais AWS..."
if ! aws sts get-caller-identity &> /dev/null; then
    log_error "Credenciais AWS inválidas ou não configuradas!"
    exit 1
fi

# Verificar se bucket existe
log_info "Verificando se bucket existe..."
if ! aws s3 ls "s3://$BUCKET_NAME" &> /dev/null; then
    log_error "Bucket '$BUCKET_NAME' não encontrado!"
    exit 1
fi

# Criar backup antes do deploy
log_info "Criando backup dos arquivos atuais..."
BACKUP_DIR="backups/backup-$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

aws s3 sync "s3://$BUCKET_NAME" "$BACKUP_DIR" \
  --quiet || log_warning "Não foi possível criar backup (bucket pode estar vazio)"

# Fazer upload dos arquivos
log_info "Fazendo upload dos arquivos de '$SOURCE_DIR'..."
aws s3 sync "$SOURCE_DIR" "s3://$BUCKET_NAME/" \
  --delete \
  --exclude ".git/*" \
  --exclude "*.sh" \
  --exclude "backups/*" \
  --exclude "README.md" \
  --exclude ".DS_Store"

# Contar arquivos enviados
FILE_COUNT=$(aws s3 ls "s3://$BUCKET_NAME" --recursive | wc -l)

log_success "Deploy concluído com sucesso!"
log_info "Total de arquivos no bucket: $FILE_COUNT"

# Obter URL do site
REGION=$(aws configure get region)
WEBSITE_URL="http://$BUCKET_NAME.s3-website-$REGION.amazonaws.com"

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
log_success "Site disponível em:"
echo -e "${GREEN}$WEBSITE_URL${NC}"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
EOF

# Tornar executável
chmod +x deploy-to-s3.sh
```

#### 3.2. Script Avançado com Cache Busting

```bash
# Criar script avançado
cat > deploy-advanced.sh << 'EOF'
#!/bin/bash

# Script Avançado de Deploy para S3
# Inclui: Cache busting, validação, rollback

set -euo pipefail

# Configurações
BUCKET_NAME="${1:-}"
SOURCE_DIR="${2:-.}"

# Cores
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log_info() { echo -e "${BLUE}[INFO]${NC} $1"; }
log_success() { echo -e "${GREEN}[SUCCESS]${NC} $1"; }
log_warning() { echo -e "${YELLOW}[WARNING]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1" >&2; }

# Validações
if [ -z "$BUCKET_NAME" ]; then
    log_error "Nome do bucket não especificado!"
    echo "Uso: $0 <nome-do-bucket> [diretorio-origem]"
    exit 1
fi

# Verificar pré-requisitos
log_info "Verificando pré-requisitos..."
command -v aws >/dev/null 2>&1 || { log_error "AWS CLI não instalado"; exit 1; }
aws sts get-caller-identity >/dev/null 2>&1 || { log_error "Credenciais AWS inválidas"; exit 1; }

# Criar backup
log_info "Criando backup..."
BACKUP_DIR="backups/backup-$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
aws s3 sync "s3://$BUCKET_NAME" "$BACKUP_DIR" --quiet

# Validar arquivos HTML
log_info "Validando arquivos HTML..."
for html_file in $(find "$SOURCE_DIR" -name "*.html"); do
    if ! grep -q "</html>" "$html_file"; then
        log_warning "Arquivo pode estar incompleto: $html_file"
    fi
done

# Upload com cache control
log_info "Fazendo upload com cache control..."

# HTML - sem cache (sempre buscar nova versão)
aws s3 sync "$SOURCE_DIR" "s3://$BUCKET_NAME/" \
  --exclude "*" \
  --include "*.html" \
  --cache-control "no-cache, no-store, must-revalidate" \
  --metadata-directive REPLACE

# CSS/JS - cache de 1 ano (com versionamento)
aws s3 sync "$SOURCE_DIR" "s3://$BUCKET_NAME/" \
  --exclude "*" \
  --include "*.css" \
  --include "*.js" \
  --cache-control "public, max-age=31536000, immutable" \
  --metadata-directive REPLACE

# Imagens - cache de 1 mês
aws s3 sync "$SOURCE_DIR" "s3://$BUCKET_NAME/" \
  --exclude "*" \
  --include "*.jpg" \
  --include "*.png" \
  --include "*.gif" \
  --include "*.svg" \
  --cache-control "public, max-age=2592000" \
  --metadata-directive REPLACE

# Outros arquivos
aws s3 sync "$SOURCE_DIR" "s3://$BUCKET_NAME/" \
  --exclude "*.html" \
  --exclude "*.css" \
  --exclude "*.js" \
  --exclude "*.jpg" \
  --exclude "*.png" \
  --exclude "*.gif" \
  --exclude "*.svg" \
  --exclude ".git/*" \
  --exclude "*.sh" \
  --exclude "backups/*" \
  --delete

# Testar site
log_info "Testando site..."
REGION=$(aws configure get region)
WEBSITE_URL="http://$BUCKET_NAME.s3-website-$REGION.amazonaws.com"

HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$WEBSITE_URL")

if [ "$HTTP_CODE" = "200" ]; then
    log_success "Site está acessível! (HTTP $HTTP_CODE)"
else
    log_error "Site retornou código HTTP $HTTP_CODE"
    read -p "Deseja fazer rollback? (s/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Ss]$ ]]; then
        log_warning "Fazendo rollback..."
        aws s3 sync "$BACKUP_DIR" "s3://$BUCKET_NAME/" --delete
        log_success "Rollback concluído!"
    fi
    exit 1
fi

# Estatísticas
FILE_COUNT=$(aws s3 ls "s3://$BUCKET_NAME" --recursive | wc -l)
TOTAL_SIZE=$(aws s3 ls "s3://$BUCKET_NAME" --recursive --summarize | grep "Total Size" | awk '{print $3}')

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
log_success "Deploy concluído com sucesso!"
echo " Arquivos: $FILE_COUNT"
echo " Tamanho total: $(numfmt --to=iec-i --suffix=B $TOTAL_SIZE 2>/dev/null || echo "$TOTAL_SIZE bytes")"
echo " URL: $WEBSITE_URL"
echo " Backup: $BACKUP_DIR"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
EOF

chmod +x deploy-advanced.sh
```

#### 3.3. Usar os Scripts

```bash
# Deploy básico
./deploy-to-s3.sh meu-site-estatico-1736553600

# Deploy avançado com cache
./deploy-advanced.sh meu-site-estatico-1736553600 ./meu-site-s3

# Deploy de diretório específico
./deploy-to-s3.sh meu-bucket ./public
```

#### 3.4. Script de Sincronização Contínua (Watch Mode)

```bash
cat > watch-and-deploy.sh << 'EOF'
#!/bin/bash

# Watch mode - deploya automaticamente quando arquivos mudam
BUCKET_NAME="${1:-}"
SOURCE_DIR="${2:-.}"

if [ -z "$BUCKET_NAME" ]; then
    echo "Uso: $0 <bucket-name> [source-dir]"
    exit 1
fi

echo "Monitorando alterações em '$SOURCE_DIR'..."
echo "Deploy automático para: $BUCKET_NAME"
echo "Pressione Ctrl+C para parar"
echo ""

# Instalar fswatch se não existir (macOS)
# sudo apt install inotify-tools (Linux)

while true; do
    inotifywait -r -e modify,create,delete "$SOURCE_DIR" && \
    echo "Alteração detectada! Fazendo deploy..." && \
    ./deploy-to-s3.sh "$BUCKET_NAME" "$SOURCE_DIR"
    sleep 2
done
EOF

chmod +x watch-and-deploy.sh
```

---

##  Scripts Desenvolvidos

### Script 1: Deploy Simples

**Arquivo:** `deploy-to-s3.sh`

**Funcionalidades:**
-  Validação de parâmetros
-  Verificação de credenciais AWS
-  Backup automático antes do deploy
-  Upload sincronizado
-  Exclusão de arquivos desnecessários
-  Feedback colorido
-  URL do site ao final

**Uso:**
```bash
./deploy-to-s3.sh nome-do-bucket diretorio-local
```

### Script 2: Deploy Avançado

**Arquivo:** `deploy-advanced.sh`

**Funcionalidades:**
-  Todas do script simples +
-  Cache control diferenciado por tipo de arquivo
-  Validação de arquivos HTML
-  Teste de conectividade após deploy
-  Rollback automático em caso de falha
-  Estatísticas de uso

**Uso:**
```bash
./deploy-advanced.sh nome-do-bucket diretorio-local
```

### Script 3: Watch Mode

**Arquivo:** `watch-and-deploy.sh`

**Funcionalidades:**
-  Monitora mudanças em arquivos
-  Deploy automático ao detectar alterações
-  Ideal para desenvolvimento

**Uso:**
```bash
./watch-and-deploy.sh nome-do-bucket diretorio-local
```

---

##  Minhas Experiências

### O Processo de Aprendizado

Este lab foi particularmente interessante porque consegui ver na prática como a AWS simplifica a hospedagem de sites estáticos. O que antigamente exigia configuração de servidor Apache/Nginx, agora é resolvido com alguns comandos CLI.

### Momentos "Aha!"

1. **S3 não é só armazenamento**: Descobri que o S3 pode funcionar como servidor web! Isso abriu minha mente para arquiteturas serverless.

2. **AWS CLI é poderoso**: A capacidade de automatizar tudo via linha de comando é incrível. Posso versionar meus scripts no Git e ter infraestrutura como código.

3. **Cache Control importa**: Entendi que configurar corretamente o cache pode economizar custos e melhorar performance drasticamente.

### O que Funcionou Muito Bem

**Automação com Scripts**
- Criar scripts bash foi mais fácil do que esperava
- A sintaxe do AWS CLI é bem intuitiva
- Poder versionar os scripts junto com o código fonte é excelente

**Hospedagem S3**
- Configuração super rápida (minutos vs horas)
- URL instantânea após configuração
- Não preciso me preocupar com escalabilidade

**Desenvolvimento Local**
- Posso testar tudo localmente antes do deploy
- O comando `sync` sincroniza apenas o que mudou
- Backups automáticos me dão confiança

### Descobertas Interessantes

- **Políticas de Bucket são JSON**: Aprendi JSON na prática!
- **URLs do S3 seguem padrão**: `bucket-name.s3-website-region.amazonaws.com`
- **Delete flag é perigoso**: O `--delete` remove arquivos que não estão no local
- **S3 é case-sensitive**: `Index.html` ≠ `index.html`
- **Backups são essenciais**: Sempre criar backup antes de operações destrutivas

---

##  Desafios Enfrentados

### Desafio 1: Erro "Access Denied" ao Acessar Site

**Situação:** Site configurado, mas ao acessar a URL recebia erro 403 Forbidden

**Sintomas:**
```bash
$ curl http://meu-bucket.s3-website-us-east-1.amazonaws.com
<?xml version="1.0" encoding="UTF-8"?>
<Error>
  <Code>AccessDenied</Code>
  <Message>Access Denied</Message>
</Error>
```

**Causa Raiz:** 
- Bucket tinha bloqueio de acesso público ativo
- Política de bucket não estava configurada corretamente
- Arquivos não tinham permissão de leitura pública

**Solução Passo a Passo:**

```bash
# 1. Remover bloqueio de acesso público
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
  "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"

# 2. Aplicar política de leitura pública
cat > policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::$BUCKET_NAME/*"
  }]
}
EOF

aws s3api put-bucket-policy --bucket $BUCKET_NAME --policy file://policy.json

# 3. Verificar se funcionou
curl -I http://$BUCKET_NAME.s3-website-us-east-1.amazonaws.com
# HTTP/1.1 200 OK
```

**Lição Aprendida:** 
Sempre verificar 3 níveis de permissão no S3:
1. Public Access Block (nível de conta/bucket)
2. Bucket Policy (nível de bucket)
3. ACL (nível de objeto - menos comum hoje em dia)

---

### Desafio 2: Arquivos CSS/JS Não Carregavam

**Situação:** HTML carregava, mas CSS e JavaScript retornavam 404

**Sintomas:**
- Página aparecia sem formatação
- Console do navegador mostrava erros 404
- Arquivos existiam no bucket

**Causa Raiz:**
- Caminhos relativos incorretos no HTML
- Estrutura de diretórios não correspondia aos links
- Maiúsculas/minúsculas diferentes

**Solução:**

```bash
# 1. Verificar estrutura de arquivos no bucket
aws s3 ls s3://$BUCKET_NAME/ --recursive

# Saída esperada:
# 2026-01-10 10:30:00       1234 index.html
# 2026-01-10 10:30:00       2345 css/style.css
# 2026-01-10 10:30:00       1567 js/script.js

# 2. Corrigir links no HTML (verificar paths relativos)
# Antes: <link href="/css/style.css">
# Depois: <link href="css/style.css">

# 3. Garantir case-sensitivity
# Renomear se necessário
aws s3 mv s3://$BUCKET_NAME/CSS/style.css s3://$BUCKET_NAME/css/style.css

# 4. Re-upload com estrutura correta
aws s3 sync . s3://$BUCKET_NAME/ --delete
```

**Lição Aprendida:**
- Sempre usar caminhos relativos (sem barra inicial)
- S3 é case-sensitive (Linux-like)
- Testar links antes do deploy final

---

### Desafio 3: Script de Deploy Apagou Arquivos Importantes

**Situação:** Executei script com flag `--delete` e perdi arquivos importantes

**Sintomas:**
```bash
# Antes do deploy
aws s3 ls s3://$BUCKET_NAME/
- images/logo.png
- images/banner.jpg
- documents/whitepaper.pdf

# Depois do deploy (logo e whitepaper sumiram!)
aws s3 ls s3://$BUCKET_NAME/
- images/banner.jpg
```

**Causa Raiz:**
- Flag `--delete` remove do S3 o que não está no diretório local
- Eu tinha deletado localmente alguns arquivos que queria manter no S3
- Não tinha backup recente

**Solução:**

```bash
# 1. Tentar recuperar de backup (se existir)
aws s3 ls s3://$BUCKET_NAME-backup/

# 2. Se tiver versionamento habilitado
aws s3api list-object-versions --bucket $BUCKET_NAME
aws s3api get-object \
  --bucket $BUCKET_NAME \
  --key images/logo.png \
  --version-id "version-id-aqui" \
  logo.png

# 3. Habilitar versionamento para o futuro
aws s3api put-bucket-versioning \
  --bucket $BUCKET_NAME \
  --versioning-configuration Status=Enabled

# 4. Modificar script para criar backup antes de deletar
# Ver deploy-advanced.sh para referência
```

**Lição Aprendida:**
- **SEMPRE** fazer backup antes de usar `--delete`
- Habilitar versionamento em buckets importantes
- Usar `--dryrun` quando disponível para testar
- Revisar o que será deletado antes de confirmar

**Melhoria no Script:**

```bash
# Adicionar confirmação antes de deletar
echo "Arquivos que serão deletados:"
aws s3 sync $SOURCE_DIR s3://$BUCKET_NAME/ --delete --dryrun | grep "delete:"

read -p "Confirma a operação? (sim/não) " -r
if [[ ! $REPLY == "sim" ]]; then
    echo "Operação cancelada"
    exit 0
fi
```

---

### Desafio 4: Custos Inesperados com GET Requests

**Situação:** Conta AWS mostrou cobranças maiores que o esperado

**Sintomas:**
- Bucket tinha poucos arquivos, mas muitos GET requests
- Custos de $2-3/dia para um site simples

**Causa Raiz:**
- Cache não estava configurado
- Navegadores faziam request para cada visita
- Não tinha CloudFront (CDN)
- Algumas imagens eram muito grandes

**Análise:**

```bash
# Verificar estatísticas de acesso
aws s3api get-bucket-metrics-configuration --bucket $BUCKET_NAME

# Ver tamanho dos objetos
aws s3 ls s3://$BUCKET_NAME/ --recursive --human-readable --summarize

# Descobri:
# - Imagens de 5-10MB cada
# - CSS/JS sendo baixados a cada visita (sem cache)
# - Total: 50MB de arquivos
```

**Solução:**

```bash
# 1. Otimizar imagens
# Usar ferramentas como imagemagick
for img in images/*.jpg; do
    convert "$img" -quality 85 -resize 1920x1080\> "${img%.jpg}_optimized.jpg"
done

# 2. Configurar cache nos headers
aws s3 cp images/ s3://$BUCKET_NAME/images/ \
  --recursive \
  --cache-control "public, max-age=2592000" \
  --metadata-directive REPLACE

# 3. Para reduzir custos ainda mais, considerar CloudFront
# (isso será outro lab!)
```

**Lição Aprendida:**
- Otimizar assets antes do upload
- Configurar cache adequadamente
- Monitorar custos regularmente
- CloudFront pode reduzir custos com GET requests

**Tabela de Cache Recomendado:**

| Tipo de Arquivo | Cache Control | Explicação |
|----------------|---------------|------------|
| HTML | `no-cache` | Sempre buscar nova versão |
| CSS/JS | `max-age=31536000` | Cache 1 ano (com versionamento) |
| Imagens | `max-age=2592000` | Cache 1 mês |
| PDFs | `max-age=604800` | Cache 1 semana |
| Fontes | `max-age=31536000` | Cache 1 ano |

---

### Desafio 5: Nome do Bucket Indisponível

**Situação:** Tentei criar bucket mas recebi erro

**Sintomas:**
```bash
$ aws s3 mb s3://meu-site
An error occurred (BucketAlreadyExists) when calling the CreateBucket operation: 
The requested bucket name is not available.
```

**Causa Raiz:**
- Nomes de bucket S3 são **globalmente únicos**
- Alguém em qualquer região já está usando "meu-site"
- Não posso usar o mesmo nome

**Solução:**

```bash
# Estratégias para nomes únicos:

# 1. Adicionar timestamp
BUCKET_NAME="meu-site-$(date +%s)"
aws s3 mb s3://$BUCKET_NAME

# 2. Adicionar random string
BUCKET_NAME="meu-site-$(openssl rand -hex 4)"
aws s3 mb s3://$BUCKET_NAME

# 3. Adicionar ID da conta AWS
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="meu-site-$ACCOUNT_ID"
aws s3 mb s3://$BUCKET_NAME

# 4. Usar domínio próprio
BUCKET_NAME="static.meudominio.com"
aws s3 mb s3://$BUCKET_NAME

# 5. Testar disponibilidade antes
test_bucket_name() {
    if aws s3 ls "s3://$1" 2>&1 | grep -q 'NoSuchBucket'; then
        echo "Nome disponível: $1"
        return 0
    else
        echo "Nome indisponível: $1"
        return 1
    fi
}

test_bucket_name "meu-site-teste-123"
```

**Lição Aprendida:**
- Sempre ter estratégia para nomes únicos
- Usar convenções de nomenclatura no projeto
- Documentar nomes de recursos criados

---

##  Lições Aprendidas

### Sobre S3 Static Website Hosting

**Prós:**
-  Deploy instantâneo
-  Muito barato (centavos por mês)
-  Escalável automaticamente
-  Infraestrutura gerenciada pela AWS
-  Fácil de monitorar

**Contras:**
-  Não suporta HTTPS nativo (precisa CloudFront)
-  Sem redirecionamentos complexos
-  Sem processamento server-side
-  Configuração de permissões pode ser confusa

### Boas Práticas Desenvolvidas

#### 1. Estrutura de Projeto

```
meu-site-s3/
├── index.html              # Página principal
├── error.html              # Página de erro
├── css/
│   └── style.css
├── js/
│   └── script.js
├── images/
│   └── logo.png
├── deploy-to-s3.sh         # Script de deploy
├── bucket-policy.json      # Política do bucket
└── README.md               # Documentação
```

#### 2. Script de Deploy Robusto

```bash
# Checklist do script ideal:
 Verificar pré-requisitos (AWS CLI, credenciais)
 Validar parâmetros de entrada
 Criar backup antes de modificar
 Usar cores para feedback visual
 Testar conectividade após deploy
 Implementar rollback em caso de erro
 Logging detalhado de operações
 Retornar URL do site ao final
```

#### 3. Comandos AWS CLI Essenciais

```bash
# Criar bucket
aws s3 mb s3://bucket-name

# Listar buckets
aws s3 ls

# Upload de arquivo único
aws s3 cp file.html s3://bucket-name/

# Upload de diretório
aws s3 sync ./local-dir s3://bucket-name/

# Download de bucket
aws s3 sync s3://bucket-name/ ./local-backup/

# Deletar arquivo
aws s3 rm s3://bucket-name/file.html

# Deletar bucket (precisa estar vazio)
aws s3 rb s3://bucket-name --force

# Configurar website hosting
aws s3 website s3://bucket-name/ \
  --index-document index.html \
  --error-document error.html

# Aplicar política
aws s3api put-bucket-policy \
  --bucket bucket-name \
  --policy file://policy.json

# Listar com detalhes
aws s3 ls s3://bucket-name/ --recursive --human-readable --summarize
```

#### 4. Segurança

**Princípios aplicados:**
-  Least Privilege: Apenas permissões necessárias
-  Public apenas o que precisa ser público
-  Documentar todas as políticas aplicadas
-  Revisar permissões periodicamente
-  Considerar encryption at rest para dados sensíveis

**Política IAM mínima para usuário de deploy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::meu-bucket-especifico",
        "arn:aws:s3:::meu-bucket-especifico/*"
      ]
    }
  ]
}
```

#### 5. Otimização de Performance

**Cache Strategy:**
```bash
# HTML: Sem cache (conteúdo dinâmico)
--cache-control "no-cache, no-store, must-revalidate"

# CSS/JS: Cache longo (com versionamento)
--cache-control "public, max-age=31536000, immutable"

# Imagens: Cache médio
--cache-control "public, max-age=2592000"
```

**Compressão:**
```bash
# Habilitar compressão gzip
aws s3 cp file.html s3://bucket/ \
  --content-encoding gzip \
  --content-type text/html
```

### Fluxo de Trabalho Ideal

```
1. Desenvolvimento Local
   ↓
2. Testar localmente (http-server, etc)
   ↓
3. Commit no Git
   ↓
4. Executar script de deploy
   ↓
5. Verificar site em produção
   ↓
6. Monitorar logs/métricas
```

---

##  Recursos e Referências

### Documentação Oficial AWS

- [S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [AWS CLI S3 Commands](https://docs.aws.amazon.com/cli/latest/reference/s3/)
- [S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Ferramentas Úteis

**Desenvolvimento Local:**
- `python -m http.server 8000` - Servidor HTTP simples
- `npx serve ./dist` - Servidor Node.js
- `browser-sync` - Live reload para desenvolvimento

**Otimização:**
- ImageMagick - Otimização de imagens
- `gzip` - Compressão de arquivos
- WebPageTest - Análise de performance

**Monitoramento:**
- AWS CloudWatch - Métricas e logs
- AWS Cost Explorer - Análise de custos
- S3 Server Access Logging - Logs de acesso

### Comandos de Troubleshooting

```bash
# Verificar conectividade
curl -I http://bucket-name.s3-website-region.amazonaws.com

# Debug de políticas
aws s3api get-bucket-policy --bucket bucket-name
aws s3api get-public-access-block --bucket bucket-name

# Ver configuração de website
aws s3api get-bucket-website --bucket bucket-name

# Listar versões de objetos (se versionamento ativo)
aws s3api list-object-versions --bucket bucket-name

# Ver logs de acesso (se configurado)
aws s3 sync s3://bucket-logs/ ./logs/

# Calcular custos estimados
aws ce get-cost-and-usage \
  --time-period Start=2026-01-01,End=2026-01-31 \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --filter file://filter.json
```

### Próximos Passos

**Labs Futuros Relacionados:**
- Lab 07: CloudFront + S3 (CDN + HTTPS)
- Lab 08: Route 53 + S3 (Domínio customizado)
- Lab 09: CI/CD com GitHub Actions + S3
- Lab 10: S3 + Lambda (Processamento de uploads)

**Melhorias para Este Setup:**
-  Adicionar CloudFront para HTTPS
-  Configurar domínio customizado
-  Implementar CI/CD automático
-  Adicionar analytics (CloudWatch, Google Analytics)
-  Criar ambiente de staging separado

---

##  Checklist de Conclusão

-  AWS CLI configurado e testado com IAM
-  Comandos S3 executados com sucesso
-  Bucket S3 criado e configurado
-  Site estático desenvolvido (HTML/CSS/JS)
-  Hospedagem estática habilitada
-  Política de acesso público aplicada
-  Site acessível via URL do S3
-  Script básico de deploy criado
-  Script avançado com cache e rollback
-  Backups automáticos implementados
-  Todos os desafios documentados
-  Lições aprendidas registradas

---

##  Certificado de Conclusão

```
╔════════════════════════════════════════════════════╗
║                                                    ║
║         LAB 06 - AWS CLI e S3 Website             ║
║              CONCLUÍDO COM SUCESSO                 ║
║                                                    ║
║  Participante: Kaylanekymberly                    ║
║  Data: 10 de Janeiro de 2026                      ║
║                                                    ║
║  Habilidades Adquiridas:                          ║
║  ✓ AWS CLI (IAM e S3)                            ║
║  ✓ S3 Static Website Hosting                     ║
║  ✓ Bucket Policies e Segurança                   ║
║  ✓ Automação com Shell Scripts                   ║
║  ✓ Deploy e Rollback Strategies                  ║
║                                                    ║
╚════════════════════════════════════════════════════╝
```

---

##  Notas Finais

Este laboratório foi fundamental para entender como o S3 vai além de simples armazenamento. A capacidade de hospedar sites estáticos de forma simples e barata é incrível!

**Principais Conquistas:**
 Criei meu primeiro site hospedado na AWS
 Automatizei o processo de deploy
 Aprendi sobre custos e otimização
 Entendi melhor segurança no S3
 Documentei tudo para referência futura

**Estatísticas do Lab:**
-  Tempo total: ~4 horas
-  Custo estimado: <$0.50
-  Arquivos criados: 15+
-  Bugs encontrados e corrigidos: 5
-  Comandos AWS CLI aprendidos: 20+
-  Linhas de código (scripts): ~300

**Pensamento Final:**

> "Este lab me mostrou que infraestrutura moderna não precisa ser complicada. Com AWS CLI e S3, posso colocar um site no ar em minutos, não em horas ou dias. A automação que criei aqui será a base para projetos futuros mais complexos."

---

##  Notas de Copyright e Atribuição

**© 2026 Kaylanekymberly - Documentação, experiências e scripts originais**

Esta documentação original, incluindo narrativas pessoais, scripts desenvolvidos, soluções para desafios e organização do conteúdo, é de propriedade do autor e protegida por direitos autorais.

**AWS, Amazon Web Services, Amazon S3 e serviços relacionados** são marcas registradas da Amazon.com, Inc. ou suas afiliadas. Esta documentação é uma obra educacional independente e não é afiliada, endossada ou patrocinada pela AWS.

**Comandos AWS CLI** são utilizados conforme a licença Apache 2.0 do projeto open source AWS CLI.

**Licença de uso:**
-  Permitido: Uso pessoal e educacional
-  Permitido: Citar com atribuição apropriada
-  Não permitido: Reprodução comercial sem autorização
-  Não permitido: Remoção de atribuição ao autor

*Esta documentação faz parte do repositório [aws-learning-journey](https://github.com/kaylanekymberly/aws-learning-journey)*

---

**Última atualização:** 10 de janeiro de 2026  
**Versão do documento:** 1.0  
**Status:**  Completo e revisado
