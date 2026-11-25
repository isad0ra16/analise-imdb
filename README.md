<div align="center">

![IMDb Analytics — Bilheteria não é aplauso. Filmes bem avaliados também são os que mais faturam?](assets/01-capa.png)

![Área](https://img.shields.io/badge/%C3%A1rea-data%20analytics-EAB308?style=flat-square) ![Python](https://img.shields.io/badge/Python-pandas%20%26%20NumPy-24201C?style=flat-square) ![SQL](https://img.shields.io/badge/SQL-Databricks-EAB308?style=flat-square) ![Power BI](https://img.shields.io/badge/Power%20BI-dashboard-24201C?style=flat-square)

[Investigação](#o-problema-de-negócio) · [Dados](#origem-dos-dados) · [Método](#preparação-dos-dados) · [Descobertas](#principais-descobertas) · [Dashboard](#dashboard-power-bi) · [Reprodução](#como-reproduzir)

</div>

Projeto desenvolvido no desafio de Análise de Dados da comunidade **First Commit-m**, com o objetivo de investigar a relação entre a avaliação do público e o desempenho comercial de filmes. Preparei os dados em Python, explorei perguntas de negócio com SQL no Databricks e construí um relatório interativo de quatro páginas no Power BI.

| Em números | Valor |
| --- | ---: |
| Filmes na base tratada | 9.999 |
| Filmes com score ≥ 80 | 429 |
| Filmes sem nota válida | 209 |
| Filmes elegíveis para análise financeira | 5.369 |
| Páginas do dashboard | 4 |

O projeto não procura estabelecer uma fórmula universal de sucesso no cinema. A proposta é investigar relações descritivas, entender a qualidade dos dados e identificar quais informações seriam necessárias para apoiar decisões de investimento com maior segurança.

---

## O problema de negócio

![Capítulo 02 — Sucesso comercial é o mesmo que aprovação do público?](assets/02-problema-de-negocio.png)

Uma empresa de produção e distribuição de filmes precisa decidir onde investir. A receita mostra parte do desempenho comercial de uma obra, enquanto a avaliação do público representa outra dimensão do resultado. Um filme pode faturar muito sem receber notas particularmente altas; da mesma forma, uma produção bem avaliada pode não alcançar uma receita elevada.

Estruturei a investigação em três frentes principais e uma questão específica sobre franquias. Em vez de partir de uma conclusão, formulei perguntas que permitissem entender o cenário, explorar as características dos filmes bem avaliados e comparar diferentes dimensões do sucesso comercial.

| Frente | Perguntas investigadas |
| --- | --- |
| Cenário | Quais características além do gênero podem estar associadas à nota? Quais informações faltam para melhorar a análise? |
| Aceitação do público | Filmes com score ≥ 80 apresentam padrões de orçamento, período ou idioma? Os dados de equipe permitem investigar profissionais envolvidos? Como estudar padrões ligados a diretores, produtores e roteiristas? |
| Desempenho financeiro | Filmes de maior receita são também os mais bem avaliados? O que pode explicar receita alta com nota mediana ou baixa? Orçamentos maiores estão associados a notas mais altas? |
| Franquias | É possível explicar diferenças de desempenho entre filmes da mesma franquia? |

A pergunta sobre franquias e as perguntas sobre diretores, produtores e roteiristas exigem identificadores e informações complementares. Não as trato como resultados demonstrados pelo dashboard atual.

---

## Origem dos dados

![Capítulo 03 — 9.999 filmes na base tratada](assets/03-origem-dos-dados.png)

Utilizei o **IMDB Movies Dataset**, disponibilizado no Kaggle.

A unidade de análise adotada é um registro de filme após a deduplicação por título e data de lançamento. Essa combinação é uma chave prática para este conjunto, não um identificador universal do IMDb. A base resultante possui 9.999 linhas e 37 colunas selecionadas pelo script de preparação.

### Variáveis utilizadas

| Campo de origem | Uso |
| --- | --- |
| `names`, `orig_title` | Identificação do filme e título original |
| `date_x` | Data de lançamento, ano e década derivados |
| `score` | Avaliação do público, analisada na escala original da base |
| `budget_x`, `revenue` | Orçamento e receita, sujeitos às regras de qualidade |
| `genre` | Gêneros e gênero principal derivado |
| `orig_lang`, `country` | Idioma original e país informado pelo dataset |
| `crew` | Elenco e personagens; não contém uma identificação suficiente de diretores e produtores |
| `status`, `overview` | Situação do lançamento e sinopse |

O projeto preserva a distinção entre informações originais e variáveis criadas para a análise. Essa decisão é particularmente importante nos campos financeiros e no país, nos quais uma transformação precipitada poderia produzir conclusões enganosas.

---

## Preparação dos dados

![Capítulo 04 — Do CSV à base analítica](assets/04-preparacao-dos-dados.png)

A preparação está centralizada no script [`imdb_dashboard/limpeza_imdb.py`](imdb_dashboard/limpeza_imdb.py), escrito em Python com Pandas e NumPy. O objetivo foi transformar o arquivo original em uma base adequada para a exploração analítica e para o Power BI, preservando as informações necessárias para compreender as decisões de tratamento.

| Etapa | Implementação e motivo |
| --- | --- |
| Leitura | `pd.read_csv` com caminhos relativos à pasta do script, evitando dependência de diretórios pessoais. |
| Textos | Remoção de espaços extras e substituição de espaços não separáveis (`\xa0`) para permitir a separação correta de gêneros e nomes. |
| Tipos numéricos | Conversão de score, orçamento e receita com `pd.to_numeric(errors="coerce")`, transformando valores não interpretáveis em ausentes. |
| Datas | Conversão segundo o formato `%m/%d/%Y` e derivação de ano e década. |
| Identificação | Utilização do título em inglês quando disponível, com o título original como alternativa; registros sem título são removidos. |
| Duplicidade | Agrupamento por título e data para identificar conflitos de orçamento e receita antes de manter uma linha por chave. |
| Variáveis derivadas | Faixas de score e orçamento, gênero principal, atores, flags de disponibilidade e qualidade financeira. |
| Exportação | CSV com codificação `utf-8-sig` para consumo no Power BI. |

Uma das decisões foi não interpretar score igual a zero como avaliação negativa. O script transforma notas não positivas em ausentes:

```python
df["score"] = pd.to_numeric(df["score"], errors="coerce")
df["score"] = df["score"].where(df["score"] > 0, np.nan)
```

Isso evita que filmes sem nota válida sejam incluídos indevidamente nas médias. Também criei uma classificação de filmes bem avaliados a partir do critério operacional `score >= 80`. Esse limite organiza a exploração; não representa uma definição universal de qualidade cinematográfica.

### Elenco e gêneros

O campo `crew` alterna nomes de atores e personagens. A função `extrair_atores` tenta separar essa sequência, tratando sufixos como Jr. e Sr. Quando não consegue obter uma sequência par, devolve um valor ausente. Portanto, não interpretei esse campo como cadastro completo dos profissionais de produção.

Para gêneros, o script extrai o primeiro rótulo como `genero_principal` e contabiliza a quantidade de gêneros. O primeiro gênero é uma convenção do processamento, não uma garantia de que seja a categoria predominante do filme.

---

## Qualidade dos dados

![Capítulo 05 — 5.369 filmes elegíveis para análise financeira](assets/05-qualidade-dos-dados.png)

A etapa de qualidade alterou a maneira de conduzir a investigação financeira. Valores repetidos em títulos distintos, valores muito baixos, casas decimais e divergências entre registros com a mesma chave levantaram suspeitas sobre a confiabilidade de parte das informações de receita e orçamento.

Em vez de remover os filmes da análise geral, mantive os campos de origem no conjunto tratado e criei indicadores de triagem. Assim, um registro pode ser útil para a análise de avaliação mesmo que sua informação financeira não passe pelos critérios adotados.

| Indicador | Registros |
| --- | ---: |
| Receitas sinalizadas | 4.287 |
| Orçamentos suspeitos | 350 |
| Conflitos financeiros | 168 |
| Sem nota válida | 209 |
| Elegíveis para análise financeira | 5.369 |

Cerca de 43% dos registros (4.287 de 9.999) tiveram a receita sinalizada, o que explica por que as comparações financeiras se restringem aos 5.369 filmes elegíveis. As categorias de problemas financeiros podem se sobrepor. Portanto, suas contagens não devem ser somadas para calcular o número de registros excluídos da análise financeira.

### Regras de triagem

| Campo | Critério aplicado pelo script |
| --- | --- |
| `receita_suspeita` | Receita positiva repetida em três ou mais registros; mesmo título e receita em dois ou mais registros; receita com parte decimal; receita positiva inferior a 1.000; ou receita informada para status diferente de `Released`. |
| `orcamento_suspeito` | Orçamento informado inferior a 1.000 ou com parte decimal. |
| `orcamento_revisar` | Orçamento a partir de 1.000 e inferior a 100.000; sinalizado, mas não excluído automaticamente por esse motivo. |
| `financeiro_conflitante` | Valores diferentes de orçamento ou receita entre registros com o mesmo título e data. |

Essas regras são heurísticas de triagem, não uma auditoria externa dos valores financeiros. Não são prova de que um valor esteja errado, nem substituem a validação em fontes oficiais de bilheteria e orçamento. Por exemplo, receitas decimais podem depender do formato ou da conversão monetária da fonte, e orçamentos pequenos podem ser legítimos.

### Separação entre valor informado e valor analítico

O script cria `receita_analise` e `orcamento_analise` a partir das variáveis financeiras preparadas, excluindo valores sinalizados e conflitos. Um filme torna-se elegível para a análise financeira conjunta somente quando as duas variáveis estão disponíveis:

```python
df["financeiro_confiavel"] = np.where(
    df["orcamento_analise"].notna() & df["receita_analise"].notna(),
    "Sim", "Não"
)
```

O nome `financeiro_confiavel` é o identificador usado pelo código. No relatório, prefiro o termo **elegível após triagem**, para não sugerir uma certificação dos valores.

### Um problema de país de origem

O código `AU` aparece em 4.755 registros, incluindo obras de diferentes origens aparentes, como produções japonesas e americanas. Tratei-o como um possível valor padrão da base, mantendo o valor original em `pais` e usando `Indefinido` em `pais_analise`. A hipótese precisa ser confirmada na documentação da fonte. Consequentemente, não uso essa variável para afirmar superioridade de um país na produção cinematográfica.

---

## Análise exploratória

![Capítulo 06 — Python, SQL e Databricks](assets/06-analise-exploratoria.png)

Após a preparação, investiguei as perguntas de negócio utilizando SQL no Databricks e os dados tratados. A exploração considera duas populações distintas: os registros com nota válida para questões de avaliação e os registros com dados financeiros elegíveis para comparações envolvendo receita e orçamento.

| Dimensão | Abordagem |
| --- | --- |
| Avaliação | Distribuição de notas e comparação entre gêneros e faixas de score. |
| Filmes bem avaliados | Recorte de score ≥ 80, com agrupamentos por década, idioma e país informado. |
| Orçamento | Comparação entre faixas de investimento e notas, considerando a validade do orçamento. |
| Receita | Comparação entre maiores receitas e avaliações, sem equiparar faturamento a lucro. |
| Lacunas | Identificação de informações necessárias para estudar profissionais e franquias com maior profundidade. |

As consultas e os notebooks do Databricks fazem parte do ambiente de exploração e não são distribuídos neste repositório. Os resultados dessa etapa estão consolidados no dashboard Power BI, e a base que alimenta tanto a exploração quanto o relatório pode ser reproduzida com o script de preparação. Não apresento consultas ilustrativas como se fossem o código original executado.

### Definições financeiras

O script também calcula dois indicadores derivados, restritos aos registros elegíveis:

| Variável | Fórmula | Interpretação |
| --- | --- | --- |
| `lucro` | receita − orçamento | Diferença simplificada, sem custos de marketing, distribuição ou participação dos exibidores. |
| `roi` | receita ÷ orçamento | Múltiplo de receita sobre orçamento, não ROI percentual convencional. |

O ROI percentual convencional seria `(receita − orçamento) ÷ orçamento × 100`. A distinção entre esses conceitos impede que uma razão entre receita e orçamento seja interpretada como rentabilidade líquida.

---

## Principais descobertas

![Capítulo 07 — Orçamento alto não garante nota alta](assets/07-principais-descobertas.png)

A análise sugere que avaliação do público e desempenho financeiro devem ser estudados como dimensões diferentes do sucesso de um filme.

### Aceitação e faturamento

Grandes sucessos comerciais, como *Avengers: Endgame*, *Avatar* e *Jurassic World*, apresentam avaliações diferentes entre si. O resultado não sustenta uma relação automática entre maior receita e melhor nota.

### Orçamento e avaliação

A comparação entre orçamento e score também não sustenta a afirmação de que um orçamento maior garanta uma nota mais alta. Essa conclusão é descritiva: não mede um efeito causal do investimento sobre a avaliação.

### Período, idioma e país

Identifiquei 429 filmes com score ≥ 80 na base tratada. A presença de filmes bem avaliados nas décadas de 2010 e 2020 precisa ser interpretada em conjunto com a quantidade de títulos disponíveis em cada período. Contagens absolutas não demonstram, sozinhas, que uma década tenha maior proporção de bons filmes.

As análises de idioma e país também dependem de cobertura e qualidade dos metadados. Não generalizo resultados da amostra para toda a indústria cinematográfica.

### O que não foi possível responder

A base não oferece informações suficientes para atribuir desempenho a diretores, roteiristas ou produtores. A hipótese de que franquias ajudam a explicar faturamentos elevados com notas moderadas exige uma fonte complementar que identifique franquias e a ordem das sequências.

Esses limites são parte do resultado: reconhecer o que os dados não conseguem responder é tão importante quanto identificar padrões.

---

## Dashboard Power BI

![Capítulo 08 — Quatro páginas de análise](assets/08-power-bi.png)

Transformei as perguntas de negócio em um relatório interativo organizado em quatro páginas. O projeto está disponível em formato PBIP, com o relatório e o modelo semântico em arquivos separados.

| Página | Objetivo |
| --- | --- |
| Avaliação dos filmes | Explorar a distribuição das notas e comparar gêneros. |
| Filmes bem avaliados | Entender o recorte de score ≥ 80 por década, idioma e país. |
| Sucesso comercial | Comparar avaliação e receita e investigar os filmes com maior faturamento. |
| Orçamento e avaliação | Investigar a relação entre orçamento e nota, considerando a qualidade dos dados financeiros. |

As quatro páginas permitem explorar diferentes dimensões sem transformar uma nota em medida direta de retorno comercial. O visual do relatório utiliza a identidade escura e dourada do projeto.

O projeto completo está em [`imdb_dashboard/imdb_dashboard.pbip`](imdb_dashboard/imdb_dashboard.pbip), com o relatório em [`imdb_dashboard.Report/`](imdb_dashboard/imdb_dashboard.Report/) e o modelo semântico em [`imdb_dashboard.SemanticModel/`](imdb_dashboard/imdb_dashboard.SemanticModel/). Para explorá-lo, abra o arquivo `.pbip` no Power BI Desktop e atualize os dados, conforme descrito em [Como reproduzir](#como-reproduzir). As artes editoriais deste README são ilustrativas e não representam capturas do relatório.

---

## Status do projeto

O IMDb Analytics está **concluído dentro do escopo proposto**: preparação dos dados em Python, exploração das perguntas de negócio com SQL no Databricks e relatório interativo de quatro páginas no Power BI, disponível neste repositório em formato PBIP.

Os itens abaixo não são tarefas pendentes de implementação. Descrevem os limites das conclusões e caminhos possíveis para uma investigação futura.

### Limitações metodológicas

- As relações observadas são descritivas e não demonstram causalidade.
- A base não traz quantidade de votos, o que impede contextualizar notas provenientes de diferentes volumes de usuários.
- O campo `crew` não identifica diretores, roteiristas e produtores, e não há identificadores de franquia.
- As regras de qualidade financeira são heurísticas de triagem, e as conclusões financeiras dependem delas.
- O código de país `AU` foi tratado como possível valor padrão da base, hipótese ainda não confirmada na fonte.
- Valores financeiros não estão ajustados pela inflação e não incluem custos de marketing e distribuição.

### Possibilidades de aprofundamento

- Incorporar a quantidade de votos e identificadores de filmes, profissionais e franquias, evitando confusão entre homônimos.
- Testar a sensibilidade das conclusões às regras de triagem financeira, comparando resultados com e sem as exclusões.
- Ajustar valores pela inflação e considerar período de lançamento, distribuição e marketing nas comparações financeiras.
- Validar países e valores financeiros em fontes complementares.

### Ressalva sobre o dataset

A versão, a autoria, o endereço e a licença do dataset do Kaggle não foram confirmados. Este repositório não declara permissões de redistribuição; consulte as condições de uso da fonte antes de reutilizar o arquivo original ou a base tratada.

## Estrutura do repositório

```text
.
├── README.md
├── assets/                      # identidade editorial do projeto
├── requirements.txt
└── imdb_dashboard/
    ├── limpeza_imdb.py
    ├── imdb_dashboard.pbip
    ├── imdb_dashboard.Report/
    ├── imdb_dashboard.SemanticModel/
    └── data/
        └── clean/
            └── imdb_filmes_limpos_powerbi.csv
```

## Como reproduzir

1. Obtenha a versão correta de `imdb_movies.csv`, respeitando as condições de uso da fonte, e coloque-a em `imdb_dashboard/data/raw/`.
2. Na raiz do repositório, crie um ambiente e instale as dependências.

Windows (PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python imdb_dashboard/limpeza_imdb.py
```

Linux/macOS:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python imdb_dashboard/limpeza_imdb.py
```

O script exporta `imdb_dashboard/data/clean/imdb_filmes_limpos_powerbi.csv` e informa a quantidade de registros, duplicidades removidas e indicadores de qualidade no terminal. Abra o projeto PBIP no Power BI Desktop e atualize o relatório, ajustando os caminhos da fonte, se necessário.

## Tecnologias e autoria

Python · Pandas · NumPy · SQL · Databricks · Power BI · Git/GitHub.

**Isadora Veiga** — projeto individual de portfólio em Análise de Dados e Business Intelligence.

[GitHub](https://github.com/isad0ra16) · [LinkedIn](https://www.linkedin.com/in/isadora-veiga/) · [Portfólio](https://isadora-veiga.vercel.app/)

<div align="center">

**IMDb Analytics · Bilheteria não é aplauso.**

*Uma investigação sobre o que os números do cinema realmente dizem.*

</div>
