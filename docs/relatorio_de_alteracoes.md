# Relatório de alterações: preparação dos dados do Projeto AB2 (ICD)

Este relatório descreve como a planilha original foi transformada nas três tabelas tratadas. Todo o
processo está em `notebooks/01_preparacao.ipynb`, que roda do início ao fim ("Reiniciar e executar tudo")
sem intervenção manual. O dicionário das colunas está em `docs/dicionario_de_dados.md`.

## 1. Origem dos dados

* **Arquivo de entrada:** `dados/original/Projeto ICD - Dataset.xlsx`, com 8 abas. Esse arquivo **não é
  alterado**: o notebook só o lê, e todas as correções acontecem em memória.
* **Como foi produzido:** transcrição manual, feita pelo grupo, de tabelas e figuras do relatório do INCA
  *Controle do câncer de mama no Brasil: dados e números 2025* (PDF em
  `dados/original/completo controle do Ca de mama_2025.pdf`). Os números foram digitados com ponto decimal.
* **Correspondência entre abas e fonte:**

| Aba | Tabela/figura do INCA 2025 | Base primária | Níveis | Linhas |
|---|---|---|---|---|
| `incidencia_mama_inca_2023_2025` | Tabela 1 | Estimativa 2023 (INCA 2022a) | UF, região, Brasil | 33 |
| `mortalidade_mama_uf_2023` | Tabela 2 | Atlas on-line de mortalidade (INCA 2025a) | UF, região, Brasil | 33 |
| `cobertura_mamografia_uf_pns_201` | Figura 12 | PNS 2013 e 2019 (IBGE 2021b) | UF, Brasil | 56 |
| `tratamento_ate_60_dias_uf_2020_` | Figura 26 | Painel-Oncologia (Brasil 2025c) | UF | 27 |
| `mamografias_rastreamento_50_69_` | Tabela 4 | SIA/SUS (Brasil 2025f) | UF, região, Brasil | 165 |
| `mamografos_uf_cnes_2024` | Tabela 13 | CNES dez/2024 (Brasil 2025b) | UF, região, Brasil | 33 |
| `cobertura_mamografia_pns2019_an` | Tabela 8 | PNS 2019 (IBGE 2021a; SIDRA 5491, 5493, 5494) | UF, região, Brasil | 462 |
| `nunca_fez_mamografia_pns2019_an` | Tabela 9 | PNS 2019 (IBGE 2021a) | região, Brasil | 90 |

Os nomes das abas e das colunas foram conferidos no notebook com `assert` e correspondem ao planejado.

## 2. Problema de leitura: números gravados como datas

**O que foi encontrado.** Ao abrir a planilha, **303 células numéricas** estavam armazenadas como **datas**.
Durante a digitação, o Excel interpretou textos como `20.04` como "20 de abril" do ano corrente (2026). O
problema afeta 6 das 8 abas e não aparece numa inspeção rápida com `pd.read_excel`, porque as colunas ficam
com tipo misto (texto e data). Os demais números dessas colunas estavam gravados como **texto** (`"16.58"`),
e não como número.

| Aba | Coluna | Células |
|---|---|---|
| `incidencia_mama_inca_2023_2025` | `taxa_bruta` (Roraima), `taxa_ajustada` (Amapá) | 2 |
| `mortalidade_mama_uf_2023` | `taxa_bruta` (2), `taxa_ajustada` (6) | 8 |
| `tratamento_ate_60_dias_uf_2020_` | `pct_tratados_ate_60_dias` (Amapá) | 1 |
| `cobertura_mamografia_pns2019_an` | `pct_mamografia_ultimos_2_anos` (22), `ic95_inferior` (101), `ic95_superior` (1) | 124 |
| `nunca_fez_mamografia_pns2019_an` | `pct_nunca_fez_mamografia` (58), `ic95_inferior` (66), `ic95_superior` (44) | 168 |
| **Total** | | **303** |

**Como foi corrigido.** O formato da célula guarda a forma em que o número foi digitado: `dd.mm` (mês com
dois dígitos, por exemplo `20.04`) ou `d.m` (por exemplo `30.6` ou `7.12`). A planilha é lida com `openpyxl`
e, para cada célula de data, o texto original é reconstruído como `"{dia}.{mês}"`, com o mês em dois dígitos
quando o formato é `dd.mm`. Exemplos:

| Célula lida como | Formato | Valor recuperado | Valor no PDF |
|---|---|---|---|
| 20/04/2026 (incidência, Amapá, taxa ajustada) | `dd.mm` | 20.04 | 20,04 |
| 22/09/2026 (incidência, Roraima, taxa bruta) | `dd.mm` | 22.09 | 22,09 |
| 07/12/2026 (mortalidade, Amapá, taxa ajustada) | `d.m` | 7.12 | 7,12 |
| 30/06/2026 (tratamento, Amapá) | `d.m` | 30.6 | 30,6 (limite inferior da classe "30,6–40,0") |

O notebook verifica, com `assert`, que todas as datas têm um desses dois formatos e hora 00:00. Em seguida,
todas as colunas numéricas são convertidas com `pd.to_numeric(errors="raise")`, de modo que qualquer texto
não numérico interromperia a execução.

**Conferência contra o PDF.** Feita fora do notebook (o PDF não faz parte do pipeline). Extraímos o texto do
PDF e comparamos, após a correção, todos os valores das abas que vêm de tabelas:

| Tabela | O que foi comparado | Resultado |
|---|---|---|
| 1 (incidência) | 33 linhas × 3 valores | 33/33 linhas idênticas |
| 2 (mortalidade) | 33 linhas × 3 valores | 33/33 linhas idênticas |
| 4 (exames) | 165 valores | 165/165 |
| 13 (mamógrafos) | 264 valores | 264/264 |
| 8 (PNS, fez há < 2 anos) | 461 trincas (percentual, IC inf., IC sup.) | 461/461 |
| 9 (PNS, nunca fez) | 90 trincas | 90/90 |

As abas das Figuras 12 e 26 não puderam ser conferidas automaticamente, porque os valores aparecem em
gráfico ou mapa. Nenhuma delas tinha células de data, exceto o Amapá na Figura 26, conferido acima.

## 3. Conferências realizadas (asserts no notebook)

Todas passaram na execução completa do notebook.

1. **Abas e colunas:** os nomes coincidem exatamente com o esperado.
2. **27 UFs:** toda aba com nível UF tem exatamente as 27 siglas, sem duplicatas, em cada recorte (por ano
   nas abas de exames e de cobertura 2013/2019; por subgrupo na Tabela 8). A aba `nunca_fez...` não tem
   nível UF, e a aba de tratamento só tem UF, como previsto.
3. **Identificação consistente:** sigla → nome, código IBGE e região são idênticos nas 8 abas, e linhas de
   região/Brasil não têm sigla.
4. **Totais:** a soma das UFs é igual ao total de cada região e do Brasil, e a soma das regiões é igual ao
   Brasil, em:
   * incidência (`n_casos`);
   * mortalidade (`n_obitos`);
   * exames (`n_mamografias_rastreamento_50_69`, em cada ano de 2020 a 2024);
   * mamógrafos (as 8 colunas, existentes e em uso por tipo e total).
5. **Mamógrafos:** comando simples + estereotaxia + computadorizado = total, para existentes e para em uso,
   e em uso ≤ existentes em todos os tipos e linhas.
6. **IC dos subgrupos:** `ic95_inferior ≤ percentual ≤ ic95_superior` em todas as 551 linhas com IC. A única
   linha sem IC é a do Maranhão, marcada como "IC sem limites na fonte".
7. **Faixas:** todos os percentuais e limites de IC estão entre 0 e 100; taxas e contagens são > 0 e não têm
   valores ausentes.
8. **Ordem de grandeza da população derivada:** a soma de `pop_feminina_estimada` das 27 UFs
   (110.635.860) difere em +0,01% da população implícita na linha "Brasil" da Tabela 1 (110.625.188).
9. **Saídas:** `base_uf` tem 27 linhas e nenhum nulo; `exames_sus` tem 135 linhas, chave (UF, ano) única e
   `exames_sus_2024` coincidente com a `base_uf`; `subgrupos` tem chave única. Cada CSV é relido após a
   gravação para conferir shape e colunas.

## 4. Problemas da fonte (documentados, valores não corrigidos)

* **Incidência: taxas ajustadas regionais repetidas.** As taxas ajustadas de três regiões são idênticas às de
  uma UF da mesma região: Norte = Roraima (27,73), Nordeste = Sergipe (42,11) e Sul = Paraná (41,06). O mesmo
  ocorre no PDF, então provavelmente é um erro do relatório, não da transcrição. O notebook localiza
  exatamente essas três coincidências. **Essas taxas regionais não são usadas**: as saídas não trazem linhas
  de região para incidência, e comparações regionais de incidência ajustada devem ser evitadas ou feitas
  com ressalva.
* **PNS: dígitos soltos.** Em 8 valores dos subgrupos, o PDF mostra um dígito extra ao lado do número (por
  exemplo, "23,7 7"). Foi registrado o valor sem o dígito, e a ocorrência está descrita na coluna
  `observacao`. Os valores registrados passam nas conferências de IC.
* **PNS: valores em cinza.** Para as 4 UFs do Centro-Oeste em "Médio completo e superior incompleto", o PDF
  mostra os valores em cinza, sem legenda que explique o motivo (provavelmente menor precisão). Foram
  mantidos e sinalizados em `observacao`.
* **PNS: Maranhão, "De 3 a 5 SM".** O PDF informa 100,0% "(sem limites)", sem IC. A linha foi **excluída**
  (ver seção 6).
* **Tratamento em até 60 dias.** O Painel-Oncologia exclui os casos sem informação de tratamento (nota da
  Figura 26). Os casos são atribuídos à UF de residência, e o Painel não mede incidência. A coluna
  `classe_mapa` da aba, que é a classe de cor do mapa, não foi levada para as saídas.
* **Períodos diferentes entre indicadores:** PNS 2013 e 2019; exames 2020–2024; tratamento 2020–2024
  (acumulado); mortalidade 2023; incidência estimada para cada ano de 2023–2025; mamógrafos dez/2024. As
  comparações entre indicadores na `base_uf` são, portanto, transversais e aproximadas.
* **Casos estimados arredondados.** O INCA arredonda `n_casos` para múltiplos de 10, o que afeta a precisão
  de `pop_feminina_estimada` nas UFs pequenas (por exemplo, Roraima tem 70 casos).

## 5. Transformações

### 5.1 Comuns

* Leitura célula a célula com `openpyxl`, com correção das datas (seção 2) e descarte de colunas sem cabeçalho.
* Conversão de tipos: contagens, `ano` e `cod_ibge` para inteiro; taxas, percentuais e IC para decimal.

### 5.2 `base_uf.csv` (27 × 21, formato largo)

1. **Filtro:** apenas linhas com `nivel == "uf"` de cada aba.
2. **Base de identificação:** `uf_sigla`, `uf_nome`, `cod_ibge` e `regiao` da aba de incidência, mais
   `grupo_comparacao` (Sul e Sudeste → "Sul/Sudeste"; Norte e Nordeste → "Norte/Nordeste";
   Centro-Oeste → "Centro-Oeste").
3. **Seleção e renomeação:**

| Aba | Coluna original | Coluna na `base_uf` |
|---|---|---|
| incidência | `n_casos`, `taxa_bruta`, `taxa_ajustada` | `incid_casos`, `incid_taxa_bruta`, `incid_taxa_ajustada` |
| mortalidade | `n_obitos`, `taxa_bruta`, `taxa_ajustada` | `mort_obitos_2023`, `mort_taxa_bruta_2023`, `mort_taxa_ajustada_2023` |
| cobertura UF | `pct_mamografia_ultimos_2_anos` (ano 2013 / 2019) | `cobertura_pns_2013` / `cobertura_pns_2019` (**longo → largo** com `pivot`) |
| tratamento | `pct_tratados_ate_60_dias` | `pct_tratados_ate_60_dias` |
| exames | `n_mamografias_rastreamento_50_69` (filtro `ano == 2024`) | `exames_sus_2024` |
| mamógrafos | `existentes_total`, `em_uso_total` | `mamografos_existentes_2024`, `mamografos_em_uso_2024` |

4. **Junções:** `merge` por `uf_sigla` com `how="left"` e `validate="one_to_one"`, partindo das 27 UFs. Ao
   final, nenhuma coluna tem nulos.
5. **Derivados:**

| Variável | Fórmula | Arredondamento |
|---|---|---|
| `pop_feminina_estimada` | `incid_casos / incid_taxa_bruta × 100.000` | inteiro |
| `mamografos_por_100mil_mulheres` | `mamografos_em_uso_2024 / pop_feminina_estimada × 100.000` | 2 casas (calculado com a população não arredondada) |
| `var_cobertura_2013_2019` | `cobertura_pns_2019 − cobertura_pns_2013` (p.p.) | 1 casa |
| `razao_mortalidade_incidencia` | `mort_taxa_ajustada_2023 / incid_taxa_ajustada` | 3 casas |

   `pop_feminina_estimada` é aproximada: corresponde à população feminina de **todas as idades** usada pelo
   INCA na estimativa, não à população-alvo do rastreamento (50–69 anos).
6. **Ordenação:** colunas na ordem do dicionário; linhas por região (Norte, Nordeste, Sudeste, Sul,
   Centro-Oeste) e, dentro de cada região, por sigla.

### 5.3 `subgrupos_pns_2019.csv` (551 × 10, formato longo)

1. As duas abas de subgrupos são **empilhadas** (`concat`) com a nova coluna `indicador`:
   `fez_ultimos_2_anos` (Tabela 8) e `nunca_fez` (Tabela 9).
2. **Renomeação:** `pct_mamografia_ultimos_2_anos` e `pct_nunca_fez_mamografia` → `percentual`.
3. **Colunas mantidas:** `nivel`, `regiao`, `uf_sigla`, `indicador`, `dimensao`, `categoria`, `percentual`,
   `ic95_inferior`, `ic95_superior`, `observacao`. Foram descartadas `uf_nome`, `cod_ibge`, `ano` (sempre 2019)
   e `fonte`. A coluna `observacao` foi mantida sem alteração.
4. **Exclusão:** 1 linha (seção 6). Resultado: 462 + 90 − 1 = 551 linhas.
5. Diferença de estrutura entre os indicadores: `nunca_fez` tem só região e Brasil, e inclui a dimensão
   `total`, que `fez_ultimos_2_anos` não tem.

### 5.4 `exames_sus_2020_2024.csv` (135 × 6, formato longo)

1. **Filtro:** `nivel == "uf"` (as linhas de região e Brasil são descartadas, depois de usadas na
   conferência de totais).
2. Acréscimo de `grupo_comparacao`.
3. **Colunas:** `uf_sigla`, `uf_nome`, `regiao`, `grupo_comparacao`, `ano`, `n_mamografias_rastreamento_50_69`.
4. **Ordenação:** região, sigla e ano.

## 6. Exclusões

| Tabela | Linha excluída | Motivo |
|---|---|---|
| `subgrupos_pns_2019` | `uf`, Nordeste, **MA**, `fez_ultimos_2_anos`, rendimento, "De 3 a 5 SM", percentual 100,0, IC ausente, observação "IC sem limites na fonte" | Sem intervalo de confiança. O valor de 100% indica amostra muito pequena e não é comparável com os demais subgrupos. O notebook confere que exatamente 1 linha foi removida. |

Linhas de região e Brasil também não entram na `base_uf` e na `exames_sus_2020_2024`, por definição dessas
tabelas (são tabelas de UF). Essas linhas foram usadas apenas nas conferências de totais.

## 7. Saídas e reprodutibilidade

| Arquivo | Linhas × colunas |
|---|---|
| `dados/tratados/base_uf.csv` | 27 × 21 |
| `dados/tratados/subgrupos_pns_2019.csv` | 551 × 10 |
| `dados/tratados/exames_sus_2020_2024.csv` | 135 × 6 |

* Formato: CSV UTF-8, separador vírgula, ponto decimal e sem índice.
* Caminhos relativos à pasta `notebooks/`, sem caminhos absolutos.
* Dependências em `requirements.txt`. O notebook de preparação usa apenas `pandas` e `openpyxl`; os demais
  pacotes são para as análises e mapas.
* Para reproduzir: `pip install -r requirements.txt` e, em `notebooks/01_preparacao.ipynb`, "Reiniciar e
  executar tudo". A execução foi testada com Python 3.9, pandas 2.3.3 e openpyxl 3.1.5.

## 8. Ética

* **Dados públicos e agregados:** todos os números vêm de um relatório público do INCA, que por sua vez usa
  bases oficiais de acesso aberto (Estimativa INCA, SIM/Atlas de Mortalidade, SIA/SUS, CNES,
  Painel-Oncologia e PNS/IBGE).
* **Sem dados pessoais:** a menor unidade é a UF (ou o subgrupo populacional da PNS por UF). Não há
  registros individuais, nomes, datas de nascimento nem qualquer dado que permita identificar pessoas. Por
  isso, a LGPD, que trata de dados pessoais, não incide sobre esta base.
* **Uso responsável:** os subgrupos de cor ou raça, escolaridade e rendimento servem para descrever
  desigualdades de acesso, não para atribuir causas a grupos. Diferenças entre UFs refletem também a
  qualidade dos registros (por exemplo, a cobertura do Painel-Oncologia), o que será explicitado nas
  análises. Valores com baixa precisão (IC amplos, valores em cinza, linha do Maranhão excluída) são
  sinalizados em vez de omitidos sem aviso.
* **Fidelidade à fonte:** nenhum valor publicado foi alterado. As únicas correções desfazem um erro de
  digitação no Excel (datas), e cada uma foi conferida contra o PDF.
