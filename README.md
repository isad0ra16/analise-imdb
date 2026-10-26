# 🎬 Análise de filmes IMDb: avaliação do público e desempenho financeiro

Projeto de análise de dados desenvolvido no desafio de Análise de Dados da comunidade **First Commit-m**. A pergunta central é: **filmes bem avaliados também são os que mais faturam?**

Preparei a base em **Python (pandas, NumPy)**, explorei os dados com **SQL no Databricks** e apresentei os resultados em um dashboard de quatro páginas no **Power BI**, disponível neste repositório em formato PBIP ([`imdb_dashboard/`](imdb_dashboard/)).

## Resultado em números

| Etapa | Quantidade |
| --- | --- |
| Filmes na base tratada | 9.999 |
| Filmes sem nota válida | 209 |
| Filmes bem avaliados (score ≥ 80) | 429 |
| Registros com receita suspeita | 4.287 |
| Registros com orçamento suspeito | 350 |
| Registros com dados financeiros conflitantes | 168 |
| Filmes com orçamento e receita confiáveis para análise financeira | 5.369 |

43% dos registros tiveram a receita sinalizada pelas regras de triagem. Por isso, as análises financeiras usam apenas os 5.369 filmes com orçamento e receita considerados confiáveis.

## Contexto de negócio

Uma empresa de produção e distribuição de filmes quer entender onde investir: quais características estão associadas a boas avaliações e quais estão relacionadas ao desempenho financeiro? O objetivo é investigar essa relação com evidências, reconhecer as limitações da base e identificar que informações ainda faltam para apoiar decisões de investimento.

## Perguntas de negócio

| Bloco | Pergunta |
| --- | --- |
| Entendendo o cenário | Além do gênero, o que parece influenciar a nota de um filme? |
| Entendendo o cenário | Quais informações estão ausentes e como poderiam ser obtidas? |
| Aceitação do público | Filmes muito bem avaliados têm características em comum além do gênero e do orçamento? |
| Aceitação do público | As informações sobre quem trabalhou nos filmes são suficientes para explicar seu destaque? |
| Aceitação do público | Como investigar padrões de sucesso ligados a diretores, produtores e roteiristas? |
| Retorno financeiro × aceitação | Os filmes de maior receita são também os mais bem avaliados? |
| Retorno financeiro × aceitação | O que pode explicar receita alta com nota mediana ou baixa? |
| Retorno financeiro × aceitação | Filmes da mesma franquia se comportam de forma semelhante em nota e receita? |
| Questão complementar | Qual é a relação entre orçamento e avaliação do público? |

## Principais conclusões

- **Receita alta não garante nota alta.** Grandes sucessos comerciais, como *Avengers: Endgame*, *Avatar* e *Jurassic World*, têm avaliações diferentes entre si.
- **Maior orçamento não garante melhor avaliação.** Os dados não sustentam uma relação automática entre investimento e nota.
- **Os filmes bem avaliados variam por período, idioma e país.** A maior presença nas décadas de 2010 e 2020 precisa ser lida junto com o total de filmes disponível em cada período.
- **A base não permite analisar diretores, roteiristas e produtores.** O campo `crew` traz principalmente atores e personagens.
- **Franquias podem explicar receita alta com avaliação moderada.** É uma hipótese: testá-la exige dados adicionais.

As relações exploradas são descritivas e não demonstram causalidade.

## Preparação dos dados

O script [`limpeza_imdb.py`](imdb_dashboard/limpeza_imdb.py):

1. Padroniza textos, espaços e valores ausentes (inclusive espaços invisíveis que quebravam a separação de gêneros).
2. Converte nota, orçamento e receita para números e trata notas menores ou iguais a zero como ausentes.
3. Converte datas e cria as colunas de ano e década.
4. Remove filmes sem título e duplicados por título e data de lançamento, marcando conflitos financeiros entre os duplicados.
5. Aplica as regras de qualidade financeira descritas abaixo.
6. Cria faixas de nota e orçamento e classifica os filmes bem avaliados.
7. Extrai os atores do campo `crew`, que alterna ator e personagem.
8. Exporta a base tratada em CSV (`utf-8-sig`) para o Power BI.

### Regras de qualidade dos dados financeiros

| Indicador | Regra |
| --- | --- |
| `receita_suspeita` | Mesmo valor de receita repetido em 3 ou mais filmes; mesmo título e receita em 2 ou mais registros; receita com casas decimais; valor inferior a 1.000; ou receita informada para filme não lançado. |
| `orcamento_suspeito` | Orçamento inferior a 1.000 ou com casas decimais. |
| `orcamento_revisar` | Orçamento entre 1.000 e 100.000: permanece na análise, mas fica marcado para revisão. |
| `financeiro_conflitante` | Orçamento ou receita diferentes entre registros com o mesmo título e data. |

Essas regras são **heurísticas de triagem**, não prova de erro. Os valores originais são preservados; as análises financeiras usam `orcamento_analise` e `receita_analise`, que excluem os registros sinalizados.

O valor `AU` aparece como país em quase metade dos filmes (4.755), inclusive em produções japonesas e americanas. Tratei como valor padrão da base: o original fica em `pais` e a coluna `pais_analise` usa `Indefinido`. Essa decisão ainda precisa ser confirmada na documentação do dataset.

### Indicadores derivados

- **Bem avaliado:** score ≥ 80.
- **Financeiro confiável:** orçamento e receita disponíveis após a triagem (não é auditoria externa dos valores).
- **`lucro`:** receita menos orçamento. Diferença simplificada, sem marketing, distribuição ou participação dos exibidores.
- **`roi`:** receita ÷ orçamento, um múltiplo. É diferente do ROI percentual convencional, (receita − orçamento) ÷ orçamento.

## Dashboard

| Página | Foco |
| --- | --- |
| Avaliação dos filmes | Distribuição das notas e comparação entre gêneros |
| Filmes bem avaliados | Filmes com score ≥ 80 por década, idioma e país |
| Sucesso comercial | Relação entre receita e nota; filmes de maior receita |
| Orçamento e avaliação | Relação entre investimento e nota; qualidade dos dados financeiros |

O projeto completo do dashboard está em [`imdb_dashboard/imdb_dashboard.pbip`](imdb_dashboard/imdb_dashboard.pbip), com o relatório (`.Report`) e o modelo semântico (`.SemanticModel`).

As perguntas sobre profissionais de produção e franquias exigem dados complementares; o dashboard não as responde sozinho.

## Limitações e próximos passos

- Obter quantidade de votos para contextualizar as notas.
- Complementar a base com diretores, roteiristas e produtores, usando identificadores dos filmes para evitar confusão entre homônimos.
- Identificar franquias e a ordem das sequências.
- Validar países e valores financeiros na fonte e comparar resultados com e sem as regras de exclusão.
- Considerar inflação, período de lançamento, distribuição e marketing nas comparações financeiras.

## Estrutura do repositório

```text
.
├── README.md
├── requirements.txt
└── imdb_dashboard/
    ├── limpeza_imdb.py                   # preparação dos dados
    ├── imdb_dashboard.pbip               # projeto do Power BI
    ├── imdb_dashboard.Report/
    ├── imdb_dashboard.SemanticModel/
    └── data/
        └── clean/
            └── imdb_filmes_limpos_powerbi.csv
```

As consultas SQL e o notebook do Databricks usados na exploração serão adicionados em `sql/` e `notebooks/`.

## Como executar

1. Baixe o arquivo `imdb_movies.csv` do IMDB Movies Dataset, publicado no Kaggle, e salve em `imdb_dashboard/data/raw/imdb_movies.csv`.
2. Instale as dependências e rode a limpeza:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
python imdb_dashboard/limpeza_imdb.py
```

O script gera `imdb_dashboard/data/clean/imdb_filmes_limpos_powerbi.csv` e mostra no terminal a quantidade de linhas, duplicados removidos e registros sinalizados.

3. Abra `imdb_dashboard/imdb_dashboard.pbip` no Power BI Desktop e atualize o relatório.

## Fonte dos dados

IMDB Movies Dataset, publicado no Kaggle. Os créditos da base pertencem aos seus autores. O endereço exato e a licença devem ser confirmados antes de redistribuir os dados.
