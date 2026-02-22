#  AWS re/Start — Laboratório 3.4: Treinamento de um Modelo de Machine Learning

> **Registro técnico de execução prática — Treinamento oficial AWS re/Start**

---

##  Direitos Autorais

> **AWS:** O cenário, arquitetura e scripts originais são propriedade intelectual da **Amazon Web Services, Inc.** Esta documentação é um registro de execução prática de treinamento oficial AWS.
>
> **Documentação:** Este relatório técnico, análises de saída e resolução dos desafios foram produzidos por **Kaylane Kimberly**.

---

##  Índice

1. [Visão Geral do Laboratório](#-visão-geral-do-laboratório)
2. [Objetivos](#-objetivos)
3. [Pré-requisitos](#-pré-requisitos)
4. [Contexto Técnico — Dataset & Algoritmo](#-contexto-técnico--dataset--algoritmo)
5. [Acesso ao Ambiente AWS](#-acesso-ao-ambiente-aws)
6. [Tarefa 1 — Acessar o Amazon SageMaker Notebook](#-tarefa-1--acessar-o-amazon-sagemaker-notebook)
7. [Tarefa 2 — Abrir o Notebook do Laboratório](#-tarefa-2--abrir-o-notebook-do-laboratório)
8. [Conceitos Aplicados](#-conceitos-aplicados)
9. [Pipeline de Machine Learning](#-pipeline-de-machine-learning)
10. [Exemplo de Código — Divisão e Treinamento](#-exemplo-de-código--divisão-e-treinamento)
11. [Arquitetura do Ambiente](#-arquitetura-do-ambiente)
12. [Resultados Esperados](#-resultados-esperados)
13. [Conclusão](#-conclusão)
14. [Recursos de Referência](#-recursos-de-referência)

---

##  Visão Geral do Laboratório

Este laboratório é o **3.4** da trilha de Machine Learning do programa **AWS re/Start**. Ele dá continuidade à exploração do **conjunto de dados de coluna vertebral biomecânica** (*Biomechanical Vertebral Column Dataset*), avançando para as etapas de preparação de dados e treinamento de modelo.

O laboratório é executado dentro do **Amazon SageMaker**, utilizando o ambiente **JupyterLab** para escrever e executar código Python. O algoritmo utilizado para treinar o modelo é o **XGBoost** — um dos algoritmos de boosting mais eficazes e amplamente utilizados em competições e projetos de ciência de dados.

| Informação | Detalhe |
|------------|---------|
| **Número do laboratório** | 3.4 |
| **Duração estimada** | 30 minutos |
| **Tempo ativo do ambiente** | 120 minutos |
| **Serviço principal** | Amazon SageMaker |
| **Algoritmo** | XGBoost |
| **Linguagem** | Python 3 (kernel: `conda_python3`) |
| **Dataset** | Biomechanical Vertebral Column |

---

##  Objetivos

Ao concluir este laboratório, as seguintes habilidades práticas são desenvolvidas:

-  Dividir um dataset em três subconjuntos: **treinamento**, **validação** e **teste**
-  Treinar um modelo de Machine Learning utilizando o algoritmo **XGBoost** no **Amazon SageMaker**

---

##  Pré-requisitos

Para execução do laboratório, os seguintes requisitos são necessários:

- Acesso a um computador com conexão Wi-Fi e sistema operacional **Windows**, **macOS** ou **Linux** (Ubuntu, SUSE ou Red Hat)
- Usuários Windows: **acesso como administrador** ao computador
- Navegador de Internet atualizado: **Chrome**, **Firefox** ou **IE9+**
  >  Versões anteriores ao IE9 não são compatíveis com o console AWS

---

##  Contexto Técnico — Dataset & Algoritmo

### Dataset: Biomechanical Vertebral Column

O dataset utilizado contém características biomecânicas extraídas de pacientes com e sem problemas na coluna vertebral. Os atributos representam medições de ângulos e dimensões da pelve e coluna, sendo utilizados para classificar pacientes como **normais** ou com alguma **patologia ortopédica**.

**Atributos típicos do dataset:**

| Atributo | Descrição |
|----------|-----------|
| `pelvic_incidence` | Incidência pélvica |
| `pelvic_tilt` | Inclinação pélvica |
| `lumbar_lordosis_angle` | Ângulo de lordose lombar |
| `sacral_slope` | Inclinação sacral |
| `pelvic_radius` | Raio pélvico |
| `degree_spondylolisthesis` | Grau de espondilolistese |
| `class` | Rótulo — Normal ou Patológico |

---

### Algoritmo: XGBoost

O **XGBoost** (*Extreme Gradient Boosting*) é um algoritmo supervisionado baseado em **árvores de decisão com boosting por gradiente**. É amplamente reconhecido por sua alta performance, eficiência computacional e capacidade de lidar bem com dados tabulares.

**Principais características:**

- Combina múltiplas árvores de decisão fracas em um modelo forte (*ensemble*)
- Utiliza regularização L1 e L2 para reduzir overfitting
- Suportado nativamente pelo **Amazon SageMaker** como algoritmo built-in
- Excelente desempenho em tarefas de classificação e regressão

---

##  Acesso ao Ambiente AWS

1. Clicar em **"Iniciar laboratório"** e aguardar o status `"Lab status: ready"`
2. Fechar o painel de status e clicar em **"AWS"** para abrir o console em nova aba
3. O login é realizado **automaticamente** com as credenciais do ambiente sandbox

>  **Dica:** Caso o console não abra em nova aba, verificar se o navegador está bloqueando pop-ups e permitir o acesso ao site.

---

##  Tarefa 1 — Acessar o Amazon SageMaker Notebook

### Passo a passo:

**1. Navegar até o Amazon SageMaker:**

No Console de Gerenciamento da AWS, acessar o menu **"Serviços"** e selecionar **Amazon SageMaker**.

**2. Localizar a instância de notebook:**

No painel de navegação à esquerda:
```
Bloco de anotações → Instâncias do bloco de anotações
```

**3. Abrir o JupyterLab:**

Localizar a instância chamada **`MyNotebook`** e clicar em **"Open JupyterLab"** ao final da linha.

> O JupyterLab é o ambiente de desenvolvimento interativo baseado em navegador fornecido pelo SageMaker para execução de notebooks Python.

---

##  Tarefa 2 — Abrir o Notebook do Laboratório

### Passo a passo:

**1. Localizar o arquivo no JupyterLab:**

No painel esquerdo do JupyterLab (navegador de arquivos), localizar o arquivo:
```
pt_br/3_4-machinelearning.ipynb
```

**2. Abrir o notebook:**

Clicar duas vezes no arquivo `3_4-machinelearning.ipynb` para abri-lo.

**3. Selecionar o kernel (se solicitado):**

Caso uma janela de seleção de kernel seja exibida:
```
Kernel: conda_python3  →  Clique em "Selecionar"
```

**4. Executar o notebook:**

Seguir as instruções e células de código presentes no notebook, executando cada célula sequencialmente.

---

##  Conceitos Aplicados

### Divisão do Dataset em 3 Subconjuntos

A divisão do dataset é uma prática fundamental em Machine Learning para garantir que o modelo seja treinado, ajustado e avaliado de forma justa e sem vazamento de dados (*data leakage*):

| Conjunto | Proporção Típica | Finalidade |
|----------|-----------------|------------|
| **Treinamento** | 70–80% | Alimentar o algoritmo para aprender os padrões dos dados |
| **Validação** | 10–15% | Ajustar hiperparâmetros e monitorar overfitting durante o treino |
| **Teste** | 10–15% | Avaliação final do modelo — dados nunca vistos durante o treino |

>  A separação correta dos conjuntos evita **overfitting** (modelo memoriza os dados de treino) e garante que a avaliação de performance seja confiável.

---

### Treinamento com XGBoost no SageMaker

O Amazon SageMaker disponibiliza o XGBoost como **algoritmo built-in**, o que permite treinar modelos de forma gerenciada sem necessidade de configurar servidores. O fluxo típico envolve:

1. Preparar e fazer upload dos dados para o **Amazon S3**
2. Configurar o **Estimator** do SageMaker com o container XGBoost
3. Definir os **hiperparâmetros** do modelo
4. Chamar o método `.fit()` para iniciar o treinamento
5. O SageMaker provisiona a infraestrutura, treina e encerra automaticamente

---

##  Exemplo de Código — Divisão e Treinamento

Os trechos abaixo ilustram o fluxo típico executado no notebook do laboratório:

### 1. Importações e Carregamento dos Dados

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
import boto3
import sagemaker
from sagemaker import get_execution_role
from sagemaker.inputs import TrainingInput
from sagemaker.image_uris import retrieve

# Carregar o dataset
df = pd.read_csv('vertebral_column_data.csv')
print(df.shape)
df.head()
```

### 2. Divisão em Treinamento, Validação e Teste

```python
# Separar features (X) e target (y)
X = df.drop(columns=['class'])
y = df['class']

# Primeira divisão: treino (70%) + temporário (30%)
X_train, X_temp, y_train, y_temp = train_test_split(
    X, y, test_size=0.30, random_state=42, stratify=y
)

# Segunda divisão: validação (15%) + teste (15%)
X_val, X_test, y_val, y_test = train_test_split(
    X_temp, y_temp, test_size=0.50, random_state=42, stratify=y_temp
)

print(f"Treinamento : {X_train.shape[0]} amostras")
print(f"Validação   : {X_val.shape[0]} amostras")
print(f"Teste       : {X_test.shape[0]} amostras")
```

### 3. Preparar Dados para o SageMaker (formato CSV sem header)

```python
# O XGBoost no SageMaker exige que o target seja a primeira coluna
train_data = pd.concat([y_train.reset_index(drop=True),
                        X_train.reset_index(drop=True)], axis=1)
val_data   = pd.concat([y_val.reset_index(drop=True),
                        X_val.reset_index(drop=True)], axis=1)
test_data  = pd.concat([y_test.reset_index(drop=True),
                        X_test.reset_index(drop=True)], axis=1)

# Salvar localmente
train_data.to_csv('train.csv', index=False, header=False)
val_data.to_csv('validation.csv', index=False, header=False)
test_data.to_csv('test.csv', index=False, header=False)
```

### 4. Upload para o Amazon S3

```python
session    = sagemaker.Session()
role       = get_execution_role()
bucket     = session.default_bucket()
prefix     = 'lab34-xgboost'

# Upload dos datasets para o S3
s3_train = session.upload_data('train.csv',      bucket=bucket, key_prefix=f'{prefix}/train')
s3_val   = session.upload_data('validation.csv', bucket=bucket, key_prefix=f'{prefix}/validation')

print(f"Train URI : {s3_train}")
print(f"Val URI   : {s3_val}")
```

### 5. Configurar e Treinar o Modelo XGBoost

```python
# Obter a imagem do container XGBoost gerenciado pelo SageMaker
region         = boto3.Session().region_name
xgboost_image  = retrieve('xgboost', region=region, version='1.5-1')

# Configurar o Estimator
xgb_model = sagemaker.estimator.Estimator(
    image_uri       = xgboost_image,
    role            = role,
    instance_count  = 1,
    instance_type   = 'ml.m4.xlarge',
    output_path     = f's3://{bucket}/{prefix}/output',
    sagemaker_session = session
)

# Definir hiperparâmetros
xgb_model.set_hyperparameters(
    max_depth        = 5,
    eta              = 0.2,
    gamma            = 4,
    min_child_weight = 6,
    subsample        = 0.8,
    objective        = 'binary:logistic',
    num_round        = 100
)

# Configurar canais de entrada
train_input = TrainingInput(s3_train, content_type='text/csv')
val_input   = TrainingInput(s3_val,   content_type='text/csv')

# Iniciar treinamento
xgb_model.fit({'train': train_input, 'validation': val_input})
```

---

##  Arquitetura do Ambiente

```
┌──────────────────────────────────────────────────────────────┐
│                        AWS Account                           │
│                                                              │
│   ┌────────────────────┐       ┌──────────────────────────┐  │
│   │  Amazon SageMaker  │       │       Amazon S3           │  │
│   │                    │       │                          │  │
│   │  ┌──────────────┐  │ upload│  ┌────────────────────┐  │  │
│   │  │  JupyterLab  │──┼──────►│  │ train.csv          │  │  │
│   │  │  (MyNotebook)│  │       │  │ validation.csv     │  │  │
│   │  └──────────────┘  │       │  │ test.csv           │  │  │
│   │                    │       │  └────────────────────┘  │  │
│   │  ┌──────────────┐  │  fit  │                          │  │
│   │  │  XGBoost     │◄─┼───────┤  ┌────────────────────┐  │  │
│   │  │  Training    │  │output │  │  model artifacts   │  │  │
│   │  │  Job         │──┼──────►│  │  (model.tar.gz)    │  │  │
│   │  └──────────────┘  │       │  └────────────────────┘  │  │
│   └────────────────────┘       └──────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

##  Resultados Esperados

Ao executar o notebook com sucesso, os seguintes resultados são obtidos:

**1. Divisão do dataset confirmada no output:**
```
Treinamento : 217 amostras
Validação   :  46 amostras
Teste       :  47 amostras
```

**2. Upload dos dados confirmado no S3:**
```
Train URI : s3://sagemaker-us-east-1-XXXX/lab34-xgboost/train/train.csv
Val URI   : s3://sagemaker-us-east-1-XXXX/lab34-xgboost/validation/validation.csv
```

**3. Logs do treinamento XGBoost durante o `.fit()`:**
```
[0] train-error:0.XX  validation-error:0.XX
[10] train-error:0.XX  validation-error:0.XX
...
[99] train-error:0.XX  validation-error:0.XX
```

**4. Status final do job de treinamento:**
```
Training job completed.
Billable seconds: XX
```

>  O laboratório é considerado **concluído com êxito** quando o job de treinamento do SageMaker finaliza sem erros e os artefatos do modelo são salvos no S3.

---

## 🏁 Conclusão

Neste laboratório foram aplicadas as seguintes habilidades práticas:

-  **Divisão de dados** em três subconjuntos (treinamento, validação e teste) com estratificação para preservar a proporção das classes
-  **Upload dos dados** para o Amazon S3, seguindo o formato esperado pelo algoritmo XGBoost do SageMaker
-  **Configuração de um Estimator** no Amazon SageMaker com hiperparâmetros e canais de entrada definidos
-  **Treinamento gerenciado** de um modelo XGBoost, com o SageMaker provisionando e encerrando automaticamente a infraestrutura de computação
-  **Persistência dos artefatos** do modelo treinado no Amazon S3

Este laboratório consolida a compreensão do **ciclo de vida básico de um modelo de Machine Learning** na AWS, desde a preparação dos dados até o artefato final pronto para ser implantado como endpoint de inferência.

---

##  Recursos de Referência

| Recurso | Link |
|---------|------|
| Amazon SageMaker — Documentação oficial | [docs.aws.amazon.com/sagemaker](https://docs.aws.amazon.com/sagemaker/) |
| XGBoost no SageMaker | [docs.aws.amazon.com/sagemaker/xgboost](https://docs.aws.amazon.com/sagemaker/latest/dg/xgboost.html) |
| Scikit-learn: train_test_split | [scikit-learn.org](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html) |
| AWS Training & Certification | [aws.amazon.com/training](https://aws.amazon.com/training/) |
| UCI Vertebral Column Dataset | [archive.ics.uci.edu](https://archive.ics.uci.edu/ml/datasets/vertebral+column) |

---

<div align="center">

**© 2022 Amazon Web Services, Inc. e suas afiliadas. Todos os direitos reservados.**

*Este trabalho não pode ser reproduzido nem redistribuído, parcial ou integralmente, sem a permissão prévia por escrito da Amazon Web Services, Inc. A cópia, a venda e o empréstimo para fins comerciais são proibidos.*

---

*Documentação técnica produzida por **Kaylane Kimberly** como registro de execução prática do treinamento oficial AWS re/Start.*

</div>
