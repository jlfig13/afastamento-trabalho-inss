# Schema — Benefícios Concedidos INSS

## Arquivo de origem

| Item | Valor |
|---|---|
| Nome | beneficios_concedidos_202306.csv |
| Competência | junho/2023 |
| Linhas | 463.568 |
| Colunas originais | 27 |
| Delimitador | ; |
| Encoding | UTF-8 |

## Catalogo de colunas

| # | Nome original | Nome Bronze/Staging | Tipo Bronze | Tipo Silver | Sentinela | Decisao |
|---|---|---|---|---|---|---|
| 0 | APS0 | aps_cod | string | string | nenhum | manter como codigo |
| 1 | APS1 | aps_desc | string | string | nenhum | manter como descricao |
| 2 | Competência concessão | competencia_concessao | string | string | nenhum | manter como AAAAMM |
| 3 | Espécie3 | especie_cod | string | string | nenhum | manter como codigo |
| 4 | Espécie4 | especie_desc | string | string | nenhum | manter como descricao |
| 5 | CID5 | cid_cod | string | string | 0 -> null | manter como codigo alfanumerico |
| 6 | CID6 | cid_desc | string | string | 0 -> null | manter como descricao |
| 7 | Despacho7 | despacho_cod | string | string | nenhum | 0 = Concessao Normal (valido) |
| 8 | Despacho8 | despacho_desc | string | string | nenhum | manter como descricao |
| 9 | Dt Nascimento | dt_nascimento | string | date | nenhum | converter DD/MM/YYYY |
| 10 | Sexo. | sexo | string | string | nenhum | Feminino / Masculino |
| 11 | Clientela | clientela | string | string | nenhum | Urbano / Rural |
| 12 | Mun Resid | mun_resid | string | string | 00000-Zerada -> null | separar cod e nome na Silver |
| 13 | Vínculo dependentes | vinculo_dependentes | string | string | Nao Informado -> null | manter |
| 14 | Forma Filiação | forma_filiacao | string | string | nenhum | manter |
| 15 | UF | uf | string | string | nenhum | manter |
| 16 | Qt SM RMI | qt_sm_rmi | string | double | nenhum | virgula -> ponto antes de converter |
| 17 | Ramo Atividade | ramo_atividade | string | string | nenhum | Irrelevante = classificacao INSS (valido) |
| 18 | Dt DCB | dt_dcb | string | date | 00/00/0000 -> null | converter DD/MM/YYYY |
| 19 | Dt DDB | dt_ddb | string | date | nenhum | converter DD/MM/YYYY |
| 20 | Dt DIB | dt_dib | string | date | nenhum | converter DD/MM/YYYY |
| 21 | País de Acordo Internacional | pais_acordo_internacional | string | string | {ñ class} -> null | manter |
| 22 | Classificador PA | classificador_pa | string | string | nenhum | manter |
| 23 | CNAE 2.023 | cnae_2023 | string | string | 0 -> null | manter como codigo |
| 24 | CNAE 2.024 | cnae_2024 | string | string | {ñ class} -> null | manter como codigo |
| 25 | Grau Instrução | grau_instrucao | string | string | Nao Informado -> null | manter |
| 26 | Qt Anos Contribuição | qt_anos_contribuicao | string | integer | nenhum | 0 = zero anos (valido) |

## Colunas derivadas na Silver

| Nome | Origem | Tipo | Descricao |
|---|---|---|---|
| cid_capitulo | cid_cod | string | Letra inicial do CID (ex: F, M, K) |
| cid_grupo | cid_cod | string | mental / osteomuscular / outros / nao_informado |
| tipo_beneficio | especie_cod | string | afastamento / aposentadoria / pensao / assistencial / maternidade |
| cid_status | cid_cod | string | informado / nao_informado |
| mun_cod | mun_resid | string | Codigo IBGE extraido de mun_resid |
| mun_nome | mun_resid | string | Nome do municipio extraido de mun_resid |

## Problemas de qualidade documentados

| Coluna | Problema | Volume | Tratamento |
|---|---|---|---|
| cid_cod / cid_desc | cid_cod preenchido mas cid_desc null | 656 registros | Limitacao da fonte. Sem tratamento. |
| dt_dcb | 00/00/0000 para beneficios sem data de cessacao | 229.668 registros | Convertido para null na Silver |
| cnae_2023 vs cnae_2024 | Sentinel diferente entre versoes (0 vs {ñ class}) | 429.687 registros | Ambos convertidos para null |

## Decisoes arquiteturais

| Decisao | Justificativa |
|---|---|
| Bronze com inferSchema=false | Preservar zeros a esquerda e formatos originais |
| Silver Staging antes da Silver | Separar tratamento de qualidade de regras de negocio |
| despacho_cod = 0 mantido | 0 representa Concessao Normal, valor valido de negocio |
| qt_anos_contribuicao = 0 mantido | Zero anos e valor valido para beneficios sem carencia |
| ramo_atividade = Irrelevante mantido | Classificacao oficial do INSS, nao ausencia de dado |