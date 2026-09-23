# Orquestração — pipeline_afastamento_inss

## Visão Geral

O job **pipeline_afastamento_inss** (ID: `1006226003442942`) orquestra o pipeline de dados de afastamento do trabalho do INSS, desde a ingestão na camada Bronze até a análise visual na camada final. O pipeline segue a arquitetura medalhão (Bronze → Silver → Gold) com 6 tasks executadas de forma sequencial, onde cada task depende do sucesso da anterior.

- **Formato:** MULTI_TASK
- **Run as:** jlucasfigueiredo@outlook.com
- **Max concurrent runs:** 1
- **Fila:** habilitada
- **Trigger:** manual (sem agendamento configurado)

---
## Dashboard AI/BI

O pipeline alimenta um dashboard AI/BI publicado no Databricks com a seguinte estrutura:

### Relationship Graph

O dashboard utiliza um **relationship graph** com modelo estrela (star schema), conectando a tabela fato a 5 dimensões via chaves surrogate (SK):

| Entidade | Dataset | Tabela UC | Cardinalidade |
| --- | --- | --- | --- |
| `Fato_Afastamentos` | `datasets/fato` | `afastamento_inss.gold.fato_afastamentos` | — |
| `Dim_Tempo` | `datasets/dim_tempo` | `afastamento_inss.gold.dim_tempo` | MANY_TO_ONE |
| `Dim_CID` | `datasets/dim_cid` | `afastamento_inss.gold.dim_cid` | MANY_TO_ONE |
| `Dim_Especie` | `datasets/dim_especie` | `afastamento_inss.gold.dim_especie` | MANY_TO_ONE |
| `Dim_Geografia` | `datasets/dim_geografia` | `afastamento_inss.gold.dim_geografia` | MANY_TO_ONE |
| `Dim_Atividade` | `datasets/dim_atividade` | `afastamento_inss.gold.dim_atividade` | MANY_TO_ONE |

Todos os widgets (exceto a pagina de Qualidade) utilizam `datasetName=""` com expressões entity-qualified, navegando pelo grafo de relacionamentos sem JOINs explicitos.

### Filtro Temporal Global

Um filtro `filter-date-range-picker` na pagina Global Filters e vinculado a `Dim_Tempo.dt_competencia`, aplicando-se automaticamente a todos os widgets de todas as paginas via relationship graph.

### Paginas e Graficos Temporais

| Pagina | Conteudo |
| --- | --- |
| Visao Geral | 6 KPIs, bar CID, 5 line charts temporais (evolucao total, por grupo CID, saude mental vs osteomuscular vs acidentario, por natureza, por sexo) |
| Diagnosticos | Barras CID, saude mental e osteomuscular por faixa etaria e sexo |
| Natureza do Afastamento | Pie por natureza, barras por especie de beneficio |
| Geografia de Residencia | Barras e tabela por UF |
| Perfil dos Registros | Barras por faixa etaria, sexo, filiacao + tabela detalhada |
| Atividade Economica | Counters, barras e tabela por ramo (CNAE) |
| Qualidade dos Dados | Cobertura de indicadores (SQL UNION, fora do relationship graph) |

### Agendamento do Dashboard

- **Schedule:** Atualizacao Diaria — Afastamentos INSS
- **Cron:** `0 0 8 ? * MON-FRI` (seg-sex, 08:00 America/Sao_Paulo)
- **Credenciais:** compartilhadas (run as owner)
- **Materializacao:** habilitada no publish

---
## DAG de Execução

```
bronze_ingestao
      │
      ▼
silver_staging
      │
      ▼
silver_transformacao
      │
      ▼
gold_prep_bi
      │
      ▼
gold_modelo_analitico
      │
      ▼
analise_visualizacao
```

Todas as tasks executam com a condição `ALL_SUCCESS` — se qualquer task falha, todas as tasks downstream são puladas.

---
## Tasks

### 1. bronze_ingestao

| Atributo | Valor |
| --- | --- |
| Task key | `bronze_ingestao` |
| Notebook | `01_bronze_ingestao` |
| Dependências | — |
| Descrição | Ingestão dos dados brutos do INSS (arquivo CSV) para a camada Bronze no Unity Catalog |

**Tabela de saída:** `afastamento_inss.bronze.beneficios_concedidos`

---

### 2. silver_staging

| Atributo | Valor |
| --- | --- |
| Task key | `silver_staging` |
| Notebook | `02_silver_staging` |
| Depende de | `bronze_ingestao` |
| Descrição | Tratamento de qualidade dos dados Bronze (trim, substituição de sentinelas por null) sem aplicar regras de negócio |

**Tabela de saída:** `afastamento_inss.silver.stg_beneficios_concedidos`

**Tratamentos aplicados:**
- Remoção de espaços no início e final (`trim`) em todas as colunas string
- Strings vazias após trim → `null`
- Valores sentinelas (`Zerados`, `Em Branco`, `{ñ class}`, etc.) → `null`
- Atualização do metadado `_data_ingestao` com o timestamp de processamento

---

### 3. silver_transformacao

| Atributo | Valor |
| --- | --- |
| Task key | `silver_transformacao` |
| Notebook | `03_silver` |
| Depende de | `silver_staging` |
| Descrição | Aplicação das regras de negócio e transformações sobre os dados da Silver Staging, gerando a Silver final |

**Tabela de saída:** `afastamento_inss.silver.beneficios_concedidos`

---

### 4. gold_prep_bi

| Atributo | Valor |
| --- | --- |
| Task key | `gold_prep_bi` |
| Notebook | `04_gold_prep_bi` |
| Depende de | `silver_transformacao` |
| Descrição | Preparação dos dados para BI — seleção de colunas, agregações e enriquecimento para camada Gold |

**Tabela de saída:** tabela(s) Gold de preparação para BI

---

### 5. gold_modelo_analitico

| Atributo | Valor |
| --- | --- |
| Task key | `gold_modelo_analitico` |
| Notebook | `05_gold_modelo_analitico` |
| Depende de | `gold_prep_bi` |
| Descrição | Construção do modelo analítico final na camada Gold, com métricas e dimensões prontas para consumo |

**Tabela de saída:** tabela(s) Gold do modelo analítico

---

### 6. analise_visualizacao

| Atributo | Valor |
| --- | --- |
| Task key | `analise_visualizacao` |
| Notebook | `06_analise_visualizacao` |
| Depende de | `gold_modelo_analitico` |
| Descrição | Análises visuais e relatórios finais a partir do modelo analítico Gold |

---

## Estrutura de Diretórios

```
afastamento-trabalho-inss/
├── README.md
├── docs/
│   └── orquestracao.md          ← este arquivo
├── dashboard/
│   └── Afastamentos INSS — Análise de Benefícios por Afastamento (Junho 2023).lvdash.json
└── notebooks/
    ├── 01_bronze_ingestao.ipynb
    ├── 02_silver_staging.ipynb
    ├── 03_silver.ipynb
    ├── 04_gold_prep_bi.ipynb
    ├── 05_gold_modelo_analitico.ipynb
    └── 06_analise_visualizacao.ipynb
```

---

## Catálogo e Schemas (Unity Catalog)

| Camada | Catálogo | Schema | Tabela |
| --- | --- | --- | --- |
| Bronze | `afastamento_inss` | `bronze` | `beneficios_concedidos` |
| Silver Staging | `afastamento_inss` | `silver` | `stg_beneficios_concedidos` |
| Silver | `afastamento_inss` | `silver` | `beneficios_concedidos` |
| Gold | `afastamento_inss` | `gold` | *(definido nos notebooks 04–05)* |

---

## Como Executar

### Execução manual (UI)

Acesse a página do job e clique em **Run Now**.

### Execução via CLI

```bash
databricks jobs run-now 1006226003442942
```

### Reparo de run com falha

```bash
databricks jobs repair-run <run_id> --rerun-all-failed-tasks --rerun-dependent-tasks
```

---

## Histórico de Execução

| Run ID | Estado | Trigger |
| --- | --- | --- |
| 457384643365131 | ✅ SUCCESS | ONE_TIME |
| 303898790877173 | ❌ FAILED (silver_staging) | ONE_TIME |
| 1096534227797980 | ⊘ CANCELLED | ONE_TIME |
| 798331064531159 | ⊘ CANCELLED | ONE_TIME |

---

## Notas

- O pipeline nao possui agendamento (trigger manual). Para automatizar, considere configurar um schedule no job.
- A fila (`queue.enabled = true`) garante que novas execucoes aguardem a conclusao da execução atual.
- Nao ha notificacoes por e-mail ou webhook configuradas.
- Cada task executa em serverless compute (sem cluster dedicado).
- **Fonte dos dados:** o dataset foi baixado manualmente do portal Dados Abertos do Governo Federal (dadosabertos.gov.br). A coleta nao foi automatizada devido a indisponibilidade da API no momento da criacao do pipeline.
- **Dashboard:** o dashboard AI/BI possui agendamento proprio (atualizacao diaria seg-sex as 08:00 BRT), independente do agendamento do job de pipeline.