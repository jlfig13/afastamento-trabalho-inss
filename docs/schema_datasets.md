# Schema e catálogo de dados: Benefícios Concedidos pelo INSS

## 1. Arquivo de origem

| Item | Valor |
|---|---|
| Nome do arquivo | 38 arquivos CSV (`beneficios_concedidos_AAAAMM.csv`) |
| Fonte | Dados abertos do INSS |
| Competência | jun/2023 a jul/2026 (38 competências) |
| Quantidade de registros | 23.689.332 |
| Quantidade de colunas originais | 27 |
| Delimitador | Ponto e vírgula (`;`) |
| Codificação | UTF-8 |
| Formato original obtido | XLSX |
| Formato utilizado no pipeline | CSV UTF-8 |

## 2. Unidade de análise

O arquivo contém registros de benefícios concedidos pelo INSS.

Neste projeto, cada linha é tratada como um registro de benefício concedido. Entretanto, o arquivo não apresenta um identificador único de benefício ou de pessoa beneficiária que permita confirmar a unicidade individual dos registros.

Consequentemente, o grão adotado no pipeline é:

> Uma linha representa um registro de benefício concedido pelo INSS no período analisado.

As contagens produzidas pelo pipeline representam:

- quantidade de registros de benefícios concedidos;
- quantidade de registros classificados como afastamento;
- distribuição dos registros por espécie, CID, localização, forma de filiação e demais atributos disponíveis.

As contagens não devem ser interpretadas diretamente como:

- quantidade de pessoas únicas;
- quantidade de trabalhadores únicos;
- quantidade de beneficiários únicos;
- incidência de afastamentos na população;
- prevalência de determinada condição de saúde;
- taxa de afastamento por setor econômico;
- percentual de trabalhadores afastados.

Uma mesma pessoa pode, em princípio, estar associada a mais de um registro. Essa possibilidade não pode ser confirmada ou descartada sem um identificador individual validado.

---

## 3. Catálogo das colunas de origem

| # | Nome original | Nome Bronze e Staging | Tipo Bronze | Tipo Silver | Valor sentinela | Decisão de tratamento |
|---:|---|---|---|---|---|---|
| 0 | APS0 | `aps_cod` | string | string | Nenhum identificado | Manter como código da APS |
| 1 | APS1 | `aps_desc` | string | string | Nenhum identificado | Manter como descrição da APS |
| 2 | Competência concessão | `competencia_concessao` | string | string | Nenhum identificado | Manter no formato `AAAAMM` |
| 3 | Espécie3 | `especie_cod` | string | string | Nenhum identificado | Manter como código da espécie |
| 4 | Espécie4 | `especie_desc` | string | string | Nenhum identificado | Manter como descrição da espécie |
| 5 | CID5 | `cid_cod` | string | string | `0`, `Zerados`, `Em Branco` | Converter ausência de informação para `null` |
| 6 | CID6 | `cid_desc` | string | string | `0`, `Zerados`, `Em Branco` | Converter ausência de informação para `null` |
| 7 | Despacho7 | `despacho_cod` | string | string | Nenhum identificado | Manter `0`, pois representa Concessão Normal |
| 8 | Despacho8 | `despacho_desc` | string | string | Nenhum identificado | Manter como descrição do despacho |
| 9 | Dt Nascimento | `dt_nascimento` | string | date | Nenhum identificado | Converter do formato `DD/MM/AAAA` |
| 10 | Sexo. | `sexo` | string | string | Nenhum identificado | Manter os valores informados pela fonte |
| 11 | Clientela | `clientela` | string | string | Nenhum identificado | Manter as categorias Urbano e Rural |
| 12 | Mun Resid | `mun_resid` | string | string | `00000-Zerada` | Converter para `null` e derivar componentes geográficos |
| 13 | Vínculo dependentes | `vinculo_dependentes` | string | string | `Não Informado` | Converter para `null` |
| 14 | Forma Filiação | `forma_filiacao` | string | string | Nenhum identificado | Manter como categoria previdenciária |
| 15 | UF | `uf` | string | string | Nenhum identificado | Manter o nome da unidade federativa |
| 16 | Qt SM RMI | `qt_sm_rmi` | string | double | Nenhum identificado | Substituir vírgula por ponto e converter para decimal |
| 17 | Ramo Atividade | `ramo_atividade` | string | string | Nenhum identificado | Manter `Irrelevante`, pois é uma classificação da fonte |
| 18 | Dt DCB | `dt_dcb` | string | date | `00/00/0000` | Converter data inválida para `null` |
| 19 | Dt DDB | `dt_ddb` | string | date | Nenhum identificado | Converter do formato `DD/MM/AAAA` |
| 20 | Dt DIB | `dt_dib` | string | date | Nenhum identificado | Converter do formato `DD/MM/AAAA` |
| 21 | País de Acordo Internacional | `pais_acordo_internacional` | string | string | `{ñ class}` | Converter para `null` |
| 22 | Classificador PA | `classificador_pa` | string | string | Nenhum identificado | Manter como classificação da fonte |
| 23 | CNAE 2.023 | `cnae_2023` | string | string | `0` | Converter para `null` e manter como código quando informado |
| 24 | CNAE 2.024 | `cnae_2024` | string | string | `{ñ class}` | Converter para `null` e manter como código quando informado |
| 25 | Grau Instrução | `grau_instrucao` | string | string | `Não Informado` | Converter para `null` |
| 26 | Qt Anos Contribuição | `qt_anos_contribuicao` | string | integer | Nenhum identificado | Manter o valor zero como quantidade válida |

### 3.1 Observação sobre códigos

Campos como CID, CNAE, espécie, APS e despacho são códigos de negócio. Por esse motivo, são preservados como `string`, mesmo quando seus valores são compostos apenas por números.

Essa decisão evita:

- remoção de zeros à esquerda;
- conversões numéricas desnecessárias;
- alteração da representação original;
- problemas futuros em relacionamentos com tabelas de domínio.

Foi observado que alguns códigos de APS podem ter sofrido remoção de zero à esquerda durante a conversão do arquivo XLSX para CSV. A descrição da APS deve ser utilizada como apoio de validação quando necessário.

---

## 4. Colunas técnicas da Bronze

| Coluna | Tipo | Descrição |
|---|---|---|
| `_arquivo_origem` | string | Nome do arquivo utilizado na ingestão |
| `_competencia` | string | Competência técnica associada à carga, no formato `AAAAMM` |
| `_data_ingestao` | timestamp | Data e hora da execução da ingestão |

A camada Bronze possui 30 colunas:

- 27 colunas provenientes do arquivo;
- 3 colunas técnicas de rastreabilidade.

---

## 5. Colunas derivadas na Silver

| Coluna | Origem | Tipo | Descrição |
|---|---|---|---|
| `mun_cod` | `mun_resid` | string | Código do município extraído do início do campo |
| `mun_nome` | `mun_resid` | string | Trecho geográfico restante após a remoção do código, atualmente contendo UF e nome do município |
| `cid_cod_invalido` | `cid_cod` | integer | 1 quando o código CID da fonte era inválido (ex.: `N`, `200`, `M5`) e foi anulado; os demais são normalizados (ex.: `M-54` → `M54`, `00F840` → `F840`) |
| `cid_capitulo` | `cid_cod` | string | Letra inicial do código CID |
| `cid_categoria` | `cid_cod` | string | Categoria do CID-10 com 3 caracteres (ex.: `F32` para `F320`) |
| `cid_grupo` | `cid_cod` | string | Grupo analitico: mental, osteomuscular, cardiovascular, respiratorio, lesoes_causas_externas, digestivo, geniturinario, neoplasias, nervoso, endocrino_metabolico, infecciosas, pele, sentidos, gravidez_parto, congenitas, perinatal, causas_externas, fatores_saude, sintomas, especiais, sangue_imunitario (D50–D89), outros ou nao_informado. O capítulo `D` é dividido por faixa: D00–D48 é neoplasia e D50–D89 é sangue e sistema imunitário |
| `cid_subgrupo` | `cid_cod` | string | Detalhamento dos grupos mental (`depressao` F32–F33, `ansiedade` F40–F41, `bipolar` F31, `estresse_adaptacao` F43, `esquizofrenia_psicoses` F20–F29, `uso_substancias` F10–F19, `transtornos_organicos` F00–F09, `outros_humor`, `outros_neuroticos`, `outros_mentais`) e osteomuscular (`dorsopatias` M40–M54 ou `outros_osteomusculares`); `nao_aplicavel` nos demais |
| `cid_status` | `cid_cod` | string | Situação do CID: `informado` ou `nao_informado` |
| `tipo_beneficio` | `especie_cod` | string | Classificação em `afastamento` (auxílio-doença, espécies 31 e 91), `auxilio_acidente` (36, 94, 95), `aposentadoria_invalidez` (32, 92 e legados 4 e 5), `aposentadoria`, `pensao`, `assistencial`, `maternidade`, `reclusao`, `sem_especie_definida` (valores inválidos na fonte) ou `outros` |

### 5.1 Regra de derivação do município

Exemplo do valor original:

```text
21504-SP-São Paulo
```

Resultado atual:

```text
mun_cod  = 21504
mun_nome = SP-São Paulo
```

Embora a coluna derivada tenha recebido o nome `mun_nome`, o conteúdo atual mantém a sigla da UF junto ao nome do município.

Em uma evolução futura, o campo poderá ser decomposto em:

- `mun_cod`
- `mun_uf_sigla`
- `mun_nome`

Até essa evolução, a coluna `uf` original deve ser utilizada como referência principal de unidade federativa.

---

## 6. Colunas derivadas na Gold preparatória

A tabela `afastamento_inss.gold.prep_beneficios_bi` mantém o mesmo grão da Silver e acrescenta atributos semânticos destinados ao consumo analítico.

| Coluna | Origem | Tipo | Descrição |
|---|---|---|---|
| `dt_competencia` | `competencia_concessao` | date | Primeiro dia do mês utilizado como referência técnica da competência |
| `ano_competencia` | `dt_competencia` | integer | Ano da competência |
| `mes_competencia` | `dt_competencia` | integer | Número do mês da competência |
| `ano_mes_competencia` | `dt_competencia` | string | Competência no formato `AAAA-MM` |
| `idade_na_competencia` | `dt_nascimento`, `dt_competencia` | integer | Idade aproximada no primeiro dia do mês da competência |
| `faixa_etaria` | `idade_na_competencia` | string | Classificação da idade em faixas |
| `cid_grupo_desc` | `cid_grupo` | string | Descrição amigável do grupo analítico do CID |
| `cid_status_desc` | `cid_status` | string | Descrição amigável da disponibilidade do CID |
| `cid_subgrupo_desc` | `cid_subgrupo` | string | Descrição amigável do subgrupo do CID |
| `cid_situacao` | `cid_cod`, `tipo_beneficio`, `especie_cod` | string | `informado`; `nao_informado` (deveria existir: afastamento, auxílio-acidente, aposentadoria por invalidez e BPC de pessoa com deficiência sem CID); ou `nao_se_aplica` (pensão, maternidade, aposentadoria por idade/tempo, BPC idoso) |
| `cid_situacao_desc` | `cid_situacao` | string | Descrição amigável da situação do CID |
| `tipo_beneficio_desc` | `tipo_beneficio` | string | Descrição amigável da categoria do benefício |
| `natureza_afastamento` | `especie_cod`, `tipo_beneficio` | string | Natureza previdenciária (31), acidentária (91), auxílio-acidente (36, 94, 95) ou não aplicável |
| `natureza_afastamento_desc` | `natureza_afastamento` | string | Descrição amigável da natureza do afastamento |
| `duracao_beneficio_dias` | `dt_dib`, `dt_dcb` | integer | Diferença em dias entre início e cessação, quando ambas as datas existem |
| `possui_data_cessacao` | `dt_dcb` | integer | Indicador binário de disponibilidade da data de cessação |
| `possui_data_cessacao_desc` | `dt_dcb` | string | Descrição da disponibilidade da data de cessação |
| `qtd_beneficios` | Constante | integer | Valor 1 para permitir contagem aditiva dos registros |
| `ind_afastamento` | `tipo_beneficio` | integer | Valor 1 para registro classificado como afastamento |
| `ind_cid_informado` | `cid_status` | integer | Valor 1 para registro com CID informado |
| `ind_saude_mental` | `cid_grupo` | integer | Valor 1 para registro com CID do capítulo F |
| `ind_osteomuscular` | `cid_grupo` | integer | Valor 1 para registro com CID do capítulo M |
| `ind_acidentario` | `natureza_afastamento` | integer | Valor 1 para registro classificado como acidentário |
| `ind_afastamento_cid_informado` | `tipo_beneficio`, `cid_status` | integer | Valor 1 para afastamento com CID informado; denominador do percentual de afastamentos por diagnóstico sobre os afastamentos com diagnóstico |
| `ind_afastamento_saude_mental` | `tipo_beneficio`, `cid_grupo` | integer | Valor 1 para registro simultaneamente classificado como afastamento e saúde mental |
| `ind_afastamento_osteomuscular` | `tipo_beneficio`, `cid_grupo` | integer | Valor 1 para registro simultaneamente classificado como afastamento e osteomuscular |
| `_data_processamento_gold` | Execução do pipeline | timestamp | Data e hora do processamento da camada Gold |

A tabela Gold preparatória possui 65 colunas:

- 39 colunas provenientes da Silver;
- 26 atributos semânticos, indicadores e metadados adicionados na Gold.

---

## 7. Problemas de qualidade documentados

| Coluna | Problema | Volume observado | Tratamento |
|---|---|---|---|
| `cid_cod` e `cid_desc` | Código CID preenchido e descrição ausente | 656 | Limitação da fonte, sem preenchimento artificial |
| `cid_cod` | Valores sentinela indicando ausência | 258.973 após consolidação | Conversão para `null` |
| `cid_desc` | Valores sentinela e descrições ausentes | 259.629 após consolidação | Conversão para `null` |
| `dt_dcb` | Valor `00/00/0000` | 229.668 | Conversão para `null` com `try_to_date` |
| `mun_resid` | Valor `00000-Zerada` | 28.217 | Conversão para `null` |
| `vinculo_dependentes` | Valor `Não Informado` | 342.017 | Conversão para `null` |
| `grau_instrucao` | Valor `Não Informado` | 273.094 | Conversão para `null` |
| `pais_acordo_internacional` | Valor `{ñ class}` | 463.425 | Conversão para `null` |
| `cnae_2023` | Valor `0` utilizado como ausência de informação | 429.687 | Conversão para `null` |
| `cnae_2024` | Valor `{ñ class}` utilizado como ausência de informação | 429.893 | Conversão para `null` |
| `qt_sm_rmi` | Separador decimal representado por vírgula | Não se aplica | Substituição de vírgula por ponto e conversão para `double` |

### 7.1 Divergência entre código e descrição do CID

Foram identificados 656 registros em que:

```text
cid_cod  = preenchido
cid_desc = null
```

Exemplos de códigos observados incluem:

- `K851`
- `M797`
- `O600`

A descrição não foi inferida nem preenchida artificialmente porque isso introduziria uma fonte externa ou uma regra que não estava presente no arquivo original.

A decisão adotada foi:

- preservar o código;
- manter a descrição como `null`;
- documentar a limitação;
- permitir enriquecimento futuro por meio de uma tabela oficial de domínio CID-10.

---

## 8. Interpretação dos registros classificados como afastamento

A classificação de afastamento foi derivada a partir das espécies de benefício identificadas no arquivo.

Na competência de junho de 2023, a Gold preparatória apresentou:

| Indicador | Quantidade | Percentual sobre os registros de afastamento |
|---|---:|---:|
| Total de registros classificados como afastamento | 185.412 | 100,00% |
| Afastamentos associados a transtornos mentais e comportamentais | 19.572 | 10,56% |
| Afastamentos associados a doenças do sistema osteomuscular | 36.251 | 19,55% |
| Afastamentos de natureza acidentária | 13.428 | 7,24% |

Os percentuais utilizam como denominador os 185.412 registros classificados como afastamento no arquivo analisado.

Esses números são resultados da competência de junho de 2023. Novas competências poderão alterar as quantidades e os percentuais.

A interpretação adequada é:

> Na competência de junho de 2023, 10,56% dos registros classificados como afastamento possuíam CID pertencente ao grupo de transtornos mentais e comportamentais.

Não deve ser utilizada a seguinte interpretação:

> 10,56% dos trabalhadores foram afastados por transtornos mentais e comportamentais.

Da mesma forma, os 36.251 registros osteomusculares representam 19,55% dos registros classificados como afastamento, e não 19,55% dos trabalhadores, das pessoas beneficiárias ou da população.

---

## 9. Ausência de denominador populacional

O dataset contém o numerador, representado pelos registros de benefícios concedidos, mas não contém um denominador de população, segurados ou vínculos de trabalho.

Para calcular taxas de afastamento seria necessário integrar uma fonte externa que forneça, no mesmo período e recorte analítico:

- quantidade de vínculos formais;
- quantidade de trabalhadores;
- população economicamente ativa;
- quantidade de segurados expostos;
- quantidade de vínculos por CNAE;
- quantidade de vínculos por CBO;
- quantidade de vínculos por município ou UF.

Fontes como RAIS e Novo CAGED poderão ser utilizadas em uma fase posterior para calcular indicadores como:

- afastamentos por 1.000 vínculos;
- benefícios acidentários por 1.000 vínculos;
- comparação entre setores com diferentes quantidades de vínculos;
- participação de diagnósticos por atividade econômica.

Enquanto essa integração não for realizada, os indicadores deverão ser apresentados como:

- contagens de registros;
- proporções internas;
- distribuições dos benefícios;
- comparações entre categorias da própria base.

Não deverão ser apresentados como taxas populacionais ou taxas de trabalhadores.

---

## 10. Limitações de interpretação do CNAE

### 10.1 Significado do campo

O CNAE representa a atividade econômica associada ao empregador ou estabelecimento quando essa informação está disponível e é aplicável ao registro.

O CNAE não identifica diretamente a ocupação exercida pela pessoa beneficiária.

A distinção conceitual adotada é:

| Elemento | Significado |
|---|---|
| CNAE | Atividade econômica do estabelecimento ou empregador |
| CBO | Ocupação exercida pela pessoa |
| Forma de filiação | Categoria previdenciária informada no registro |
| Ramo de atividade | Classificação ampla de atividade disponibilizada pelo INSS |

Nenhuma dessas informações, isoladamente, comprova a situação laboral completa da pessoa beneficiária.

### 10.2 Cobertura observada

| Campo | Registros sem informação | Registros com informação | Cobertura aproximada |
|---|---:|---:|---:|
| CNAE 2023 | 429.687 | 33.881 | 7,31% |
| CNAE 2024 | 429.893 | 33.675 | 7,26% |

A baixa cobertura impede o uso do CNAE como uma característica representativa de toda a base.

Análises por CNAE deverão:

- considerar somente os registros com CNAE informado;
- apresentar a quantidade de registros considerados;
- apresentar o percentual de cobertura;
- informar que o resultado representa apenas um subconjunto;
- evitar generalizações para todos os benefícios, pessoas beneficiárias ou trabalhadores;
- evitar comparação de taxas sem denominador de vínculos.

### 10.3 Ausência de CNAE

A ausência de CNAE não deve ser interpretada automaticamente como ausência de trabalho ou atividade profissional.

Um registro pode não possuir CNAE porque:

- o CNAE não é aplicável à espécie do benefício;
- o benefício não está associado a um empregador específico;
- a forma de filiação não pressupõe empregador;
- a pessoa está classificada como desempregada;
- a pessoa está classificada como contribuinte individual;
- a pessoa está classificada como segurado facultativo;
- a pessoa está classificada como segurado especial;
- o registro corresponde a aposentadoria, pensão ou benefício assistencial;
- a informação não foi preenchida;
- a informação não foi disponibilizada na extração;
- existe limitação de cobertura ou integração na fonte.

A classificação prevista para o projeto é:

- CNAE informado
- CNAE não informado

Não será criada uma classificação como:

- Trabalhador
- Não trabalhador

com base apenas na presença ou ausência de CNAE.

### 10.4 Uso analítico do CNAE

Formulação recomendada:

> Distribuição dos registros de afastamento por CNAE entre os benefícios com atividade econômica informada.

Formulações que devem ser evitadas:

- Setores com maior taxa de trabalhadores afastados.
- Setores que mais causam afastamentos.
- Percentual de trabalhadores afastados por CNAE.

A primeira formulação exigiria um denominador de vínculos por CNAE.

A segunda exigiria evidência de causalidade ocupacional.

A terceira exigiria identificação de trabalhadores únicos e uma população de referência.

---

## 11. Forma de filiação e condição laboral

O campo `forma_filiacao` representa uma classificação previdenciária disponível no registro.

Entre as categorias observadas estão:

- Empregado;
- Desempregado;
- Segurado Especial;
- Autônomo;
- Facultativo.

Essas categorias ajudam a contextualizar a relação previdenciária, mas não devem ser interpretadas como uma fotografia completa da situação laboral da pessoa.

Exemplos de interpretação:

- **Empregado** indica que o registro foi classificado dessa forma na fonte, mas não confirma por si só a existência de vínculo ativo em todas as datas relacionadas ao benefício;
- **Desempregado** representa a classificação registrada no processo previdenciário;
- **Autônomo** não pressupõe CNAE associado a um empregador;
- **Facultativo** pode possuir contribuição previdenciária sem atividade remunerada;
- **Segurado Especial** possui um enquadramento previdenciário próprio, frequentemente relacionado à atividade rural.

A forma de filiação deverá ser analisada separadamente do CNAE, do ramo de atividade e da espécie do benefício.

---

## 12. Natureza acidentária e relação com o trabalho

A natureza do afastamento foi derivada principalmente da espécie do benefício.

Neste projeto:

- espécie 31 representa afastamento de natureza previdenciária;
- espécie 91 representa afastamento de natureza acidentária;
- espécies 36, 94 e 95 (auxílio-acidente e suplementar) não são afastamentos: são indenizações pagas após a consolidação de sequelas e recebem `tipo_beneficio = auxilio_acidente` e natureza `auxilio_acidente`;
- benefícios não classificados como afastamento recebem a categoria `nao_aplicavel`.

A classificação acidentária indica que o benefício foi registrado administrativamente como relacionado a acidente ou doença do trabalho.

Não se deve inferir relação ocupacional apenas pela presença de:

- CID de transtorno mental;
- CID osteomuscular;
- CNAE;
- ramo de atividade;
- forma de filiação.

Um registro com CID osteomuscular e CNAE informado não comprova isoladamente que a condição foi causada pelo trabalho.

Uma análise ocupacional mais robusta deverá combinar, quando disponíveis:

- espécie 91;
- natureza acidentária;
- CID;
- CNAE;
- CBO;
- Comunicação de Acidente de Trabalho (CAT);
- agente causador;
- informações do empregador;
- circunstâncias do acidente;
- indicadores de exposição ocupacional.

A integração com os dados de CAT está prevista como evolução do projeto.

---

## 13. Critérios de comunicação dos resultados

### 13.1 Termos recomendados

Utilizar:

- registros de benefícios concedidos;
- benefícios classificados como afastamento;
- registros associados a saúde mental;
- registros associados a doenças osteomusculares;
- registros com CNAE informado;
- registros de natureza acidentária;
- pessoas beneficiárias, quando a interpretação individual for apropriada;
- distribuição dos registros;
- proporção dos registros analisados.

### 13.2 Termos que devem ser evitados

Evitar, sem uma fonte complementar:

- trabalhadores únicos;
- quantidade de trabalhadores afastados;
- percentual da população afastada;
- incidência de doença;
- prevalência de doença;
- taxa de afastamento por setor;
- setor que causa mais afastamentos;
- trabalhador sem atividade profissional por ausência de CNAE;
- quantidade de pessoas únicas.

---

## 14. Escopo analítico

O objetivo analítico do projeto é:

> Analisar os benefícios concedidos pelo INSS, com foco nos registros classificados como afastamento, comparando diagnósticos de saúde mental e doenças osteomusculares por características previdenciárias, demográficas, geográficas e, quando disponível, econômicas.

As análises econômicas considerarão somente os registros com CNAE informado e apresentarão explicitamente a cobertura do campo.

As análises relacionadas ao trabalho utilizarão a natureza acidentária como principal classificação administrativa disponível nesta fase.

A integração com CAT, CBO e denominadores de vínculos empregatícios será tratada como evolução futura.

---

## 15. Regras semânticas da Gold

| Campo | Regra | Interpretação |
|---|---|---|
| `qtd_beneficios` | Valor 1 para cada linha | Quantidade de registros de benefícios |
| `ind_afastamento` | Valor 1 para espécies classificadas como afastamento | Registro de benefício classificado como afastamento |
| `ind_cid_informado` | Valor 1 quando o CID está preenchido | Registro com informação de CID disponível |
| `ind_saude_mental` | Valor 1 para CID cujo capítulo é F | Registro com CID associado a transtorno mental ou comportamental |
| `ind_osteomuscular` | Valor 1 para CID cujo capítulo é M | Registro com CID associado ao sistema osteomuscular |
| `ind_acidentario` | Valor 1 para natureza acidentária | Registro classificado administrativamente como acidentário |
| `ind_afastamento_saude_mental` | Afastamento e CID do capítulo F | Registro de afastamento associado a saúde mental |
| `ind_afastamento_osteomuscular` | Afastamento e CID do capítulo M | Registro de afastamento associado a doença osteomuscular |

### 15.1 Indicadores de CNAE planejados

Os campos abaixo são recomendados para uma evolução da Gold, mas somente deverão ser incorporados ao catálogo após serem implementados no notebook:

| Campo planejado | Origem | Tipo | Descrição |
|---|---|---|---|
| `status_cnae` | `cnae_2023`, `cnae_2024` | string | Indica se existe CNAE disponível no registro |
| `ind_cnae_informado` | `cnae_2023`, `cnae_2024` | integer | Indicador binário de disponibilidade de CNAE |

Esses indicadores representarão apenas a disponibilidade da informação econômica.

Não representarão condição de trabalho, vínculo ativo ou ocupação.

---

## 16. Restrições de uso dos indicadores

Os indicadores derivados são aditivos no nível de registro, mas não representam pessoas únicas.

- A soma de `qtd_beneficios` representa a quantidade de registros da tabela.
- A soma de `ind_afastamento` representa a quantidade de registros classificados como afastamento.
- A soma de `ind_saude_mental` representa a quantidade de registros com CID do capítulo F, independentemente do tipo de benefício.
- A soma de `ind_osteomuscular` representa a quantidade de registros com CID do capítulo M, independentemente do tipo de benefício.
- A soma de `ind_afastamento_saude_mental` representa a quantidade de registros simultaneamente classificados como afastamento e associados a CID do capítulo F.
- A soma de `ind_afastamento_osteomuscular` representa a quantidade de registros simultaneamente classificados como afastamento e associados a CID do capítulo M.
- A soma de `ind_acidentario` representa a quantidade de registros de natureza acidentária conforme a regra administrativa derivada da espécie do benefício.

Nenhuma dessas medidas deve ser apresentada como contagem de pessoas únicas sem uma chave individual validada.

---

## 17. Decisões arquiteturais

| Decisão | Justificativa |
|---|---|
| Bronze com `inferSchema=false` | Preservar formatos originais, códigos e zeros à esquerda |
| Todos os campos da Bronze como `string` | Evitar inferências incorretas antes da análise do schema |
| Padronização dos nomes na Bronze | Atender às restrições do Delta Lake sem alterar o conteúdo dos campos |
| Silver Staging antes da Silver | Separar tratamento de qualidade de regras de negócio |
| Valores sentinela convertidos para `null` | Padronizar a ausência de informação |
| `despacho_cod = 0` mantido | O código representa Concessão Normal |
| Zeros à esquerda removidos de `aps_cod`, `especie_cod` e `despacho_cod` | O mesmo código aparecia com e sem zeros conforme o arquivo (`04` e `4`), duplicando categorias |
| Bronze identifica o layout do CSV pelo cabeçalho | Dois layouts distintos têm 23 colunas; agrupar só pela quantidade desalinhava 10 competências. Ver `docs/layouts_csv.md` |
| Afastamento = auxílio-doença (31 e 91) | Auxílio-acidente (36, 94, 95) é indenização por sequela e aposentadoria por invalidez é benefício permanente |
| `qt_anos_contribuicao = 0` mantido | Zero anos é um valor válido de negócio |
| `ramo_atividade = Irrelevante` mantido | Trata-se de classificação oficial da fonte, não necessariamente ausência |
| Datas convertidas com `try_to_date` | Permitir conversão segura de datas inválidas como `00/00/0000` |
| CID e CNAE mantidos como `string` | São códigos de classificação, não medidas numéricas |
| Gold preparatória sem agregação | Preservar o grão detalhado para consumo flexível no BI |
| Indicadores binários na Gold | Centralizar as regras no pipeline e evitar lógica duplicada nos relatórios |
| Descrições no Unity Catalog | Melhorar governança, descoberta e interpretação dos dados |
| Ausência de CNAE não utilizada para inferir condição laboral | CNAE ausente pode representar não aplicabilidade, indisponibilidade ou limitação da fonte |
| Resultados apresentados como registros | O arquivo não permite identificar pessoas únicas |

---

## Ajustes principais realizados

- **Grão:** formalizado como registro de benefício, sem afirmar pessoa única.
- **CNAE:** cobertura documentada e interpretação limitada ao subconjunto disponível.
- **Silver:** corrigida a descrição de `mun_nome`, que ainda contém UF e município.
- **Gold:** catalogadas as 23 colunas adicionadas à tabela preparatória.
- **Indicadores:** separados os indicadores efetivamente implementados dos indicadores de CNAE ainda planejados.
- **Temporalidade:** os valores de junho de 2023 foram tratados como fotografia da competência, não como valores permanentes.
- **Terminologia:** substituídas afirmações sobre trabalhadores por termos compatíveis com a fonte.
- **Governança:** adicionadas restrições explícitas para uso dos indicadores e comunicação dos resultados.