# Dicionário de dados: Projeto AB2 (ICD)

Tabelas geradas por `notebooks/01_preparacao.ipynb` a partir de `dados/original/Projeto ICD - Dataset.xlsx`.
Todas estão em `dados/tratados/`, como CSV UTF-8, separador vírgula, ponto decimal e sem índice. Campo vazio
indica valor ausente, o que só ocorre na coluna `observacao`.

**Fonte geral:** INCA, *Controle do câncer de mama no Brasil: dados e números 2025* (abaixo, "INCA 2025").
Na coluna "Fonte", o nome da aba de origem vem primeiro, seguido da tabela ou figura do INCA 2025 e da base
primária que o próprio relatório cita.

Siglas das bases primárias:

| Sigla | Base |
|---|---|
| Estimativa 2023 | INCA, *Estimativa 2023: incidência de câncer no Brasil* (INCA 2022a) |
| Atlas de Mortalidade | INCA, Atlas on-line de mortalidade (INCA 2025a), dados do SIM |
| SIA/SUS | Sistema de Informações Ambulatoriais do SUS (Brasil 2025f) |
| PNS | IBGE, Pesquisa Nacional de Saúde 2013 e 2019 (IBGE 2021a, 2021b) |
| Painel-Oncologia | DATASUS, Painel-Oncologia Brasil (Brasil 2025c) |
| CNES | Cadastro Nacional de Estabelecimentos de Saúde, competência dez/2024 (Brasil 2025b) |

Abreviações: "p.p." = pontos percentuais; "SM" = salário mínimo; "IC 95%" = intervalo de confiança de 95%.

---

## 1. `base_uf.csv`

Uma linha por Unidade da Federação (27 linhas, 22 colunas), sem valores ausentes. As linhas estão ordenadas
por região (Norte, Nordeste, Sudeste, Sul, Centro-Oeste) e, dentro de cada região, por sigla.

| Nome | Descrição | Tipo | Unidade | Período | Fonte |
|---|---|---|---|---|---|
| `uf_sigla` | Sigla da UF (chave da tabela) | texto | — | — | todas as abas (colunas comuns) |
| `uf_nome` | Nome da UF | texto | — | — | todas as abas (colunas comuns) |
| `cod_ibge` | Código IBGE da UF (2 dígitos) | inteiro | — | — | todas as abas (colunas comuns) |
| `regiao` | Grande região (Norte, Nordeste, Sudeste, Sul, Centro-Oeste) | texto | — | — | todas as abas (colunas comuns) |
| `grupo_comparacao` | Agrupamento do projeto: Sul e Sudeste → "Sul/Sudeste"; Norte e Nordeste → "Norte/Nordeste"; Centro-Oeste → "Centro-Oeste" | texto | — | — | derivado de `regiao` |
| `incid_casos` | Número estimado de casos novos de câncer de mama feminino, **por ano** do triênio (o INCA arredonda para múltiplos de 10) | inteiro | casos/ano | 2023–2025 | `incidencia_mama_inca_2023_2025`; INCA 2025, Tabela 1 (Estimativa 2023) |
| `incid_taxa_bruta` | Taxa bruta estimada de incidência | decimal | casos por 100 mil mulheres | 2023–2025 | `incidencia_mama_inca_2023_2025`; INCA 2025, Tabela 1 (Estimativa 2023) |
| `incid_taxa_ajustada` | Taxa de incidência estimada, ajustada por idade (população padrão mundial) | decimal | casos por 100 mil mulheres | 2023–2025 | `incidencia_mama_inca_2023_2025`; INCA 2025, Tabela 1 (Estimativa 2023) |
| `mort_obitos_2023` | Número de óbitos por câncer de mama em mulheres | inteiro | óbitos | 2023 | `mortalidade_mama_uf_2023`; INCA 2025, Tabela 2 (Atlas de Mortalidade) |
| `mort_taxa_bruta_2023` | Taxa bruta de mortalidade por câncer de mama | decimal | óbitos por 100 mil mulheres | 2023 | `mortalidade_mama_uf_2023`; INCA 2025, Tabela 2 (Atlas de Mortalidade) |
| `mort_taxa_ajustada_2023` | Taxa de mortalidade ajustada por idade (população padrão mundial) | decimal | óbitos por 100 mil mulheres | 2023 | `mortalidade_mama_uf_2023`; INCA 2025, Tabela 2 (Atlas de Mortalidade) |
| `cobertura_pns_2013` | Percentual de mulheres de 50 a 69 anos que fizeram mamografia há menos de dois anos da entrevista (autorrelato, SUS e privado) | decimal | % | 2013 | `cobertura_mamografia_uf_pns_201` (ano = 2013); INCA 2025, Figura 12 (PNS) |
| `cobertura_pns_2019` | Mesmo indicador de `cobertura_pns_2013`, na PNS de 2019 | decimal | % | 2019 | `cobertura_mamografia_uf_pns_201` (ano = 2019); INCA 2025, Figura 12 (PNS) |
| `pct_tratados_ate_60_dias` | Percentual dos casos de câncer de mama com primeiro tratamento em até 60 dias após o diagnóstico, por UF de residência. Exclui casos sem informação de tratamento | decimal | % | 2020–2024 (acumulado) | `tratamento_ate_60_dias_uf_2020_`; INCA 2025, Figura 26 (Painel-Oncologia) |
| `exames_sus_2024` | Número de mamografias de rastreamento em mulheres de 50 a 69 anos realizadas no SUS | inteiro | exames | 2024 | `mamografias_rastreamento_50_69_` (ano = 2024); INCA 2025, Tabela 4 (SIA/SUS) |
| `mamografos_existentes_2024` | Mamógrafos existentes, todos os tipos (comando simples + estereotaxia + computadorizado), total cadastrado no CNES (não restrito ao SUS; os do SUS estão na Tabela 14, não usada) | inteiro | equipamentos | dez/2024 | `mamografos_uf_cnes_2024` (`existentes_total`); INCA 2025, Tabela 13 (CNES) |
| `mamografos_em_uso_2024` | Mamógrafos em uso, todos os tipos | inteiro | equipamentos | dez/2024 | `mamografos_uf_cnes_2024` (`em_uso_total`); INCA 2025, Tabela 13 (CNES) |
| `pop_feminina_estimada` | **Derivado.** População feminina (todas as idades) implícita na estimativa do INCA: `incid_casos / incid_taxa_bruta × 100.000`, arredondada para inteiro. É aproximada, porque os casos são arredondados | inteiro | mulheres | 2023–2025 | derivado de `incid_casos` e `incid_taxa_bruta` |
| `mamografos_por_100mil_mulheres` | **Derivado.** `mamografos_em_uso_2024 / pop_feminina_estimada × 100.000` (com a população não arredondada), 2 casas decimais. O denominador inclui todas as idades | decimal | equipamentos por 100 mil mulheres | dez/2024 ÷ 2023–2025 | derivado |
| `var_cobertura_2013_2019` | **Derivado.** `cobertura_pns_2019 − cobertura_pns_2013`, 1 casa decimal. Positivo = aumento | decimal | p.p. | 2013→2019 | derivado |
| `razao_mortalidade_incidencia` | **Derivado.** `mort_taxa_ajustada_2023 / incid_taxa_ajustada`, 3 casas decimais. Indicador grosseiro de letalidade relativa, com períodos diferentes no numerador e no denominador | decimal | adimensional | 2023 ÷ 2023–2025 | derivado |
| `destaque_alagoas` | `True` apenas para Alagoas (AL); usado para destacar a UF em gráficos e tabelas | booleano | — | — | derivado de `uf_sigla` |

---

## 2. `subgrupos_pns_2019.csv`

Formato longo, 551 linhas e 10 colunas. Cada linha corresponde a um indicador × local × subgrupo. A chave é
(`nivel`, `regiao`, `uf_sigla`, `indicador`, `dimensao`, `categoria`).

| Nome | Descrição | Tipo | Unidade | Período | Fonte |
|---|---|---|---|---|---|
| `nivel` | Nível geográfico: `uf`, `regiao` ou `pais` | texto | — | — | abas `cobertura_mamografia_pns2019_an` e `nunca_fez_mamografia_pns2019_an` |
| `regiao` | Grande região; em `nivel = pais`, vale "Brasil" | texto | — | — | idem |
| `uf_sigla` | Sigla da UF; vazio quando `nivel` é `regiao` ou `pais` | texto | — | — | idem |
| `indicador` | `fez_ultimos_2_anos`: fez mamografia há menos de dois anos da entrevista (UF, região e Brasil). `nunca_fez`: nunca fez mamografia (só região e Brasil) | texto | — | — | `fez_ultimos_2_anos` ← `cobertura_mamografia_pns2019_an` (INCA 2025, Tabela 8); `nunca_fez` ← `nunca_fez_mamografia_pns2019_an` (INCA 2025, Tabela 9) |
| `dimensao` | Dimensão do subgrupo: `escolaridade`, `cor_raca`, `rendimento` (domiciliar per capita) ou `total` (este último só em `nunca_fez`) | texto | — | — | idem |
| `categoria` | Categoria dentro da dimensão. Escolaridade: "Sem instrução e fundamental incompleto", "Fundamental completo e médio incompleto", "Médio completo e superior incompleto", "Superior completo". Cor ou raça: "Branca", "Parda", "Preta". Rendimento: "Sem rendimento até 1/4 SM", "De 1/4 a 1/2 SM", "De 1/2 a 1 SM", "De 1 a 2 SM", "De 2 a 3 SM", "De 3 a 5 SM", "Mais de 5 SM". Total: "Total" | texto | — | — | idem |
| `percentual` | Percentual de mulheres de 50 a 69 anos no subgrupo com o desfecho do `indicador` (renomeado de `pct_mamografia_ultimos_2_anos` / `pct_nunca_fez_mamografia`) | decimal | % | 2019 | idem (PNS 2019; Tabelas SIDRA 5491, 5493, 5494 para a Tabela 8) |
| `ic95_inferior` | Limite inferior do IC 95% do `percentual` | decimal | % | 2019 | idem |
| `ic95_superior` | Limite superior do IC 95% do `percentual` | decimal | % | 2019 | idem |
| `observacao` | Nota da transcrição, mantida sem alteração: "Valor em cinza na fonte" (4 linhas, UFs do Centro-Oeste) ou "Na fonte aparece um dígito extra junto ao valor (...)" (8 linhas). Vazio nas demais | texto | — | — | idem (anotação do grupo) |

---

## 3. `exames_sus_2020_2024.csv`

Formato longo, 135 linhas (27 UFs × 5 anos) e 6 colunas. A chave é (`uf_sigla`, `ano`).

| Nome | Descrição | Tipo | Unidade | Período | Fonte |
|---|---|---|---|---|---|
| `uf_sigla` | Sigla da UF | texto | — | — | `mamografias_rastreamento_50_69_` |
| `uf_nome` | Nome da UF | texto | — | — | `mamografias_rastreamento_50_69_` |
| `regiao` | Grande região | texto | — | — | `mamografias_rastreamento_50_69_` |
| `grupo_comparacao` | Agrupamento do projeto (mesma regra da `base_uf`) | texto | — | — | derivado de `regiao` |
| `ano` | Ano de realização dos exames | inteiro | ano | 2020–2024 | `mamografias_rastreamento_50_69_` |
| `n_mamografias_rastreamento_50_69` | Número de mamografias de rastreamento em mulheres de 50 a 69 anos realizadas no SUS | inteiro | exames | 2020–2024 | `mamografias_rastreamento_50_69_`; INCA 2025, Tabela 4 (SIA/SUS) |
