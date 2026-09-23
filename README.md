## Objetivo

Construir um pipeline de dados no Databricks para analisar os benefícios concedidos pelo INSS, com foco nos registros classificados como afastamento.

O projeto compara diagnósticos associados a transtornos mentais e doenças osteomusculares por características previdenciárias, demográficas, geográficas e, quando disponível, econômicas.

Os resultados representam registros de benefícios e não pessoas ou trabalhadores únicos. Indicadores por CNAE consideram somente o subconjunto com atividade econômica informada.

## Fonte dos Dados

Os microdados utilizados neste projeto foram obtidos manualmente a partir do portal de Dados Abertos do Governo Federal ([dadosabertos.gov.br](https://dadosabertos.gov.br)), especificamente do conjunto de dados de benefícios concedidos pelo INSS.

A coleta **não foi automatizada** devido à indisponibilidade da API no momento da criação do pipeline. O download foi realizado de forma manual e o arquivo CSV resultante foi carregado diretamente na camada Bronze do pipeline.

> **Nota:** Quando a API dos Dados Abertos estiver disponível, recomenda-se substituir a ingestão manual por um processo automatizado (ex.: Auto Loader ou job agendado) para garantir atualizações periódicas e rastreabilidade.

## Arquitetura

- Bronze
- Silver
- Gold

## Tecnologias

- Databricks
- Delta Lake
- PySpark
- Spark SQL
- AI/BI Dashboards (relationship graph, filtros temporais)

## Dashboard

O projeto inclui um dashboard AI/BI publicado no Databricks, com:

- **Relationship graph** conectando a tabela fato (`fato_afastamentos`) a 5 dimensões (`dim_tempo`, `dim_cid`, `dim_especie`, `dim_geografia`, `dim_atividade`) via chaves surrogate
- **Filtro temporal global** (`filter-date-range-picker`) vinculado a `dim_tempo.dt_competencia`, aplicando-se a todos os widgets via relationship graph
- **5 graficos temporais** (line charts) na pagina Visao Geral: evolucao total, por grupo CID, por indicadores de saude, por natureza do afastamento e por sexo
- **6 paginas analiticas:** Visao Geral, Diagnosticos, Natureza do Afastamento, Geografia de Residencia, Perfil dos Registros, Atividade Economica e Qualidade dos Dados
- **Agendamento automatico:** atualizacao diaria (seg-sex, 08:00 BRT)

# afastamento-trabalho-inss
Pipeline de dados no Databricks para análise de afastamentos do trabalho utilizando microdados públicos do INSS.
