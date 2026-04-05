# ⚓ LH Nauticals — Desafio de Análise de Dados

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?style=for-the-badge&logo=pandas&logoColor=white)
![DuckDB](https://img.shields.io/badge/DuckDB-SQL-FFF000?style=for-the-badge&logo=duckdb&logoColor=black)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-ML-F7931E?style=for-the-badge&logo=scikitlearn&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Viz-11557c?style=for-the-badge)

**Análise completa de dados de vendas, custos de importação e recomendação de produtos para uma empresa do setor náutico.**

</div>

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Datasets](#-datasets)
- [Capítulos da Análise](#-capítulos-da-análise)
  - [Cap. 1 — Análise Exploratória Inicial](#cap-1--análise-exploratória-inicial)
  - [Cap. 2 — Produtos](#cap-2--produtos)
  - [Cap. 3 — Custos de Importação](#cap-3--custos-de-importação)
  - [Cap. 4 — Dados Públicos (API do Banco Central)](#cap-4--dados-públicos-api-do-banco-central)
  - [Cap. 5 — Análise de Clientes](#cap-5--análise-de-clientes)
  - [Cap. 6 — Dimensão de Calendário](#cap-6--dimensão-de-calendário)
  - [Cap. 7 — Previsão de Demanda](#cap-7--previsão-de-demanda)
  - [Cap. 8 — Sistema de Recomendação](#cap-8--sistema-de-recomendação)
- [Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [Como Executar](#-como-executar)
- [Resultados e Insights](#-resultados-e-insights)
- [Autor](#-autor)

---

## 🎯 Sobre o Projeto

Este projeto é a resolução de um **desafio técnico de análise de dados** para a empresa fictícia **LH Nauticals**, que atua no setor de equipamentos náuticos. O objetivo é realizar uma análise completa e estruturada a partir de múltiplas fontes de dados — vendas, produtos, custos de importação, câmbio e CRM de clientes — cobrindo desde a exploração inicial até modelos de Machine Learning.

O notebook `nauticauls.ipynb` está organizado em **8 capítulos**, cada um abordando uma etapa da análise com queries SQL (via DuckDB), manipulação de dados com Pandas, visualizações com Matplotlib/Seaborn e técnicas de ciência de dados.

---

## 📁 Estrutura do Repositório

```
LH Nauticals/
├── datasets/
│   ├── vendas_2023_2024.csv            # Transações de vendas (9.895 registros)
│   ├── produtos_raw.csv                # Catálogo de produtos (157 registros brutos)
│   ├── custos_importacao.json          # Histórico de custos em USD (1.260 entradas)
│   ├── clientes_crm.json              # Dados de CRM dos clientes
│   ├── cambio_bcb_2023_2024.csv       # Taxas de câmbio BCB (cache local)
│   ├── analise_prejuizo_transacoes.csv # Resultado: prejuízo por transação
│   └── analise_prejuizo_produtos.csv  # Resultado: prejuízo agregado por produto
├── nauticauls.ipynb                    # Notebook principal da análise
├── .gitignore
└── README.md
```

---

## 📊 Datasets

| Arquivo | Formato | Registros | Descrição |
|---------|---------|-----------|-----------|
| `vendas_2023_2024.csv` | CSV | 9.895 | Transações de vendas no período de 01/01/2023 a 31/12/2024. Colunas: `id`, `id_client`, `id_product`, `qtd`, `total`, `sale_date` |
| `produtos_raw.csv` | CSV | 157 | Catálogo de produtos com nome, preço e categoria. Contém erros propositais de grafia nas categorias |
| `custos_importacao.json` | JSON | 1.260 | Dados aninhados com histórico de preço de compra em USD por produto |
| `clientes_crm.json` | JSON | — | Informações de CRM dos clientes |
| `cambio_bcb_2023_2024.csv` | CSV | — | Cotação diária do dólar (venda) obtida via API do Banco Central |

---

## 📖 Capítulos da Análise

### Cap. 1 — Análise Exploratória Inicial

> **Objetivo:** Avaliar a qualidade e a confiabilidade dos dados brutos de vendas.

- Quantificação de linhas, colunas e range temporal
- Análise de valores nulos e negativos
- Detecção de **outliers** (média de R$ 263 mil por transação, máximo de R$ 2,2 milhões)
- Identificação de **datas corrompidas** — dois formatos simultâneos (`YYYY-MM-DD` e `DD-MM-YYYY`)

> **Diagnóstico:** Os dados brutos **não são confiáveis** para análises avançadas sem tratamento prévio. Outliers extremos distorcem médias e a inconsistência de datas sabota qualquer análise de sazonalidade.

---

### Cap. 2 — Produtos

> **Objetivo:** Limpar e padronizar o catálogo de produtos.

- Remoção de espaços e normalização de case nas categorias
- **Mapeamento de variantes** com erros de grafia (ex: `Eletronicoz`, `propulção`, `encoragem`) → 3 categorias padronizadas: `eletrônicos`, `propulsão`, `ancoragem`
- Conversão de preços de `string` ("R$ 33122.52") para `float`
- Remoção de duplicatas (157 → 150 produtos únicos)

---

### Cap. 3 — Custos de Importação

> **Objetivo:** Normalizar o JSON aninhado de custos e preparar para joins futuros.

- Uso de `pd.json_normalize()` para achatar a estrutura `historic_data`
- Tipagem correta de `product_id`, `usd_price`, `start_date`
- Resultado: **1.260 entradas** de custo histórico em USD

---

### Cap. 4 — Dados Públicos (API do Banco Central)

> **Objetivo:** Obter câmbio diário USD/BRL e calcular prejuízo por transação.

- Integração com a **API PTAX do BCB** (`olinda.bcb.gov.br`) para cotação de venda do dólar
- Cálculo do custo unitário em BRL: `usd_price × taxa_cambio_data`
- Cálculo do custo total em BRL: `qtd × custo_unitário_brl`
- Identificação de **transações com prejuízo**: quando `custo_total_brl > receita_transação_brl`
- Uso de **DuckDB com SQL analítico** (CTEs, `ROW_NUMBER`, `LEFT JOIN` temporal) para matching de custo vigente por data

#### Principais Queries SQL

- **JOIN temporal** entre vendas, custos e câmbio para encontrar o custo de importação vigente na data da venda
- **Agregação por produto** do prejuízo total e percentual de perda sobre a receita
- **Top 15 produtos com maior prejuízo** visualizado via Lollipop Chart

---

### Cap. 5 — Análise de Clientes

> **Objetivo:** Identificar os clientes mais valiosos (fiéis) e seu comportamento de compra.

- Cálculo de métricas por cliente: `faturamento_total`, `frequência`, `ticket_médio`, `diversidade_categorias`
- Filtro de **clientes de elite**: compras em ≥ 3 categorias distintas
- Ranking dos **Top 10 clientes fiéis** por ticket médio (desempate por `id_client` ASC)
- Categoria mais vendida para os clientes de elite: **propulsão** (6.030 itens)

---

### Cap. 6 — Dimensão de Calendário

> **Objetivo:** Criar uma dimensão temporal e identificar padrões de vendas por dia da semana.

- Construção de uma tabela de calendário via `GENERATE_SERIES` no DuckDB
- Cálculo de vendas diárias e preenchimento de dias sem transação com zero
- Identificação do **dia da semana com menor média de vendas**: **Domingo** (R$ 3.319.503,57)

---

### Cap. 7 — Previsão de Demanda

> **Objetivo:** Construir um baseline de previsão de demanda para um produto específico.

- **Produto-alvo:** Motor de Popa Yamaha Evo Dash 155HP (id=54)
- Construção de **série temporal diária** de quantidade vendida (com preenchimento de zeros)
- Split temporal: **treino** (Jan–Dez 2023) / **teste** (Jan 2024)
- **Baseline:** média móvel de 7 dias (rolling forecast)
- Métrica de avaliação: **MAE = 1.00**

> **Conclusão:** O baseline com média móvel de 7 dias é limitado — não captura sazonalidade, tendências ou eventos especiais que alterem a demanda.

---

### Cap. 8 — Sistema de Recomendação

> **Objetivo:** Recomendar produtos similares com base em padrões de compra.

- **Produto de referência:** GPS Garmin Vortex Maré Drift (id=27)
- Construção de **matriz binária** usuário × produto (presença/ausência de compra)
- Cálculo de **similaridade de cosseno** produto × produto (`cosine_similarity` do scikit-learn)
- **Top 5 produtos mais similares:**

| Ranking | Produto | Similaridade |
|---------|---------|-------------|
| 1 | Motor de Popa Volvo Magnum 276HP | 0.870 |
| 2 | GPS Furuno Swift Leviathan Poseidon | 0.868 |
| 3 | Radar Furuno Swift | 0.854 |
| 4 | Cabo de Nylon Delta Force Magnum Leviathan | 0.850 |
| 5 | Transponder AIS Maré Magnum | 0.850 |

> A recomendação é baseada em uma **abordagem item-item** com filtragem colaborativa simplificada.

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Finalidade |
|------------|------------|
| **Python 3.10+** | Linguagem principal |
| **Pandas** | Manipulação e limpeza de dados |
| **DuckDB** | Queries SQL analíticas in-process |
| **Matplotlib / Seaborn** | Visualização de dados |
| **Scikit-Learn** | Cosine similarity e MAE |
| **NumPy** | Operações numéricas |
| **Requests** | Consumo da API do Banco Central |
| **Jupyter Notebook** | Ambiente de desenvolvimento interativo |

---

## 🚀 Como Executar

### Pré-requisitos

- Python 3.10 ou superior
- `pip` ou `conda` para gerenciamento de pacotes

### Instalação

```bash
# Clonar o repositório
git clone https://github.com/theusnevess/Nauticals_challenge.git
cd Nauticals_challenge

# Criar e ativar ambiente virtual
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/Mac
source .venv/bin/activate

# Instalar dependências
pip install pandas duckdb matplotlib seaborn scikit-learn numpy requests jupyter
```

### Execução

```bash
# Iniciar o Jupyter Notebook
jupyter notebook nauticauls.ipynb
```

> ⚠️ **Nota:** O Cap. 4 realiza chamadas à API do Banco Central para obter dados de câmbio. Certifique-se de ter conexão com a internet.

---

## 💡 Resultados e Insights

- **Dados brutos são problemáticos:** Datas em formatos inconsistentes e outliers extremos invalidam análises sem tratamento prévio.
- **Produtos de propulsão dominam o faturamento** e também concentram o maior volume de prejuízos cambiais.
- **Motor de Popa Volvo Hydro Dash 256HP** lidera com ~R$ 27M em prejuízo (43,5% da receita).
- **Domingos** apresentam a menor média de vendas diárias.
- **Previsão por média móvel** é um ponto de partida limitado; demanda modelos mais sofisticados (ARIMA, Prophet, etc.).
- **Sistema de recomendação item-item** com cosine similarity revela associações cross-category relevantes (GPS → Motor de Popa).

---

## 👤 Autor

**Matheus Neves** — [@theusnevess](https://github.com/theusnevess)

---

<div align="center">
  <sub>Desenvolvido com ☕ e muita análise de dados.</sub>
</div>
