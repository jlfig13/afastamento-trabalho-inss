# Layouts dos CSVs de benefícios concedidos

Os arquivos mensais do INSS **não mantêm o mesmo layout ao longo do tempo**. A Bronze (`01_bronze_ingestao`) identifica o layout de cada arquivo pelo **cabeçalho** e aplica o mapeamento posicional correspondente.

> Agrupar apenas pela quantidade de colunas não funciona: existem dois layouts diferentes com 23 colunas. Foi esse o erro que desalinhou 10 competências (jul–dez/2024 e fev–mai/2025) antes da correção.

## Layouts identificados

| Layout | Colunas | Competências | Diferenças em relação ao layout completo |
|---|---|---|---|
| `original_27` | 27 | 2023-06 a 2024-05 e 2025-11 a 2026-07 | Layout completo |
| `alfabetico_23` | 23 | 2024-06 | Colunas em **ordem alfabética**; traz apenas as descrições de APS e CID (sem os códigos); inclui `Segurado MEI`; coluna vazia no final; competência em formato timestamp (`2024-06-01 00:00:00`) |
| `original_23` | 23 | 2024-07 a 2024-12 e 2025-02 a 2025-05 | Ordem original, sem `CNAE 2.0` (x2), `Grau Instrução` e `Qt Anos Contribuição` |
| `original_22` | 22 | 2025-01 | Como `original_23`, mas o despacho vem só com a descrição (sem o código) |
| `original_26` | 26 | 2025-06 a 2025-10 | Sem `Qt Anos Contribuição` |

Colunas ausentes em cada layout são preenchidas com `null` na Bronze.

## Tratamento do layout `alfabetico_23`

- `aps_cod` é extraído do início de `aps_desc` (ex.: `02001050-Aps Maceio...` → `02001050`);
- `cid_cod` é extraído do início de `cid_desc` (ex.: `F72   Retardo Mental Grave` → `F72`; `F31.8` → `F318`);
- `Segurado MEI` e a coluna vazia final são descartados.

## Outras variações entre arquivos

| Variação | Tratamento |
|---|---|
| Datas no formato `yyyy-MM-dd HH:mm:ss` (e `00/00/0000` para ausência) | Silver converte os dois formatos; `00/00/0000` vira `null` |
| `aps_cod`, `especie_cod` e `despacho_cod` com e sem zeros à esquerda (`02001050` e `2001050`, `04` e `4`) | Silver remove os zeros à esquerda |
| `cid_cod = 000000` (2024–2025) e `cid_cod = NULL` como texto (2026) | Staging trata como ausência de informação |
| `cid_cod` malformado (`M-54`, `H54.`, `00F840`) | Silver normaliza; o que continuar inválido vira `null` e é marcado em `cid_cod_invalido` |
| Despacho `67` sem descrição (arquivos de 2026) | Mantido; a descrição não é informada pela fonte |

## Validações automáticas na Bronze

Antes de gravar a tabela, a seção 4.1 do notebook verifica:

1. **Alinhamento por arquivo:** competência coerente com o nome do arquivo, `especie_cod` numérico e `sexo` válido, com limiar de 99% de linhas válidas por arquivo;
2. **Continuidade das competências:** nenhum mês pode faltar entre a primeira e a última competência ingeridas.

Se qualquer uma falhar, a execução é interrompida e a Bronze não é gravada.
