# Pipeline AWS - State of Data
> Pipeline de dados em nuvem, baseada no Amazon Web Services (AWS) para analisar o histórico dos últimos 3 anos da pesquisa State of Data Brazil (2023-2025).

AWS S3 · AWS Glue ETL Jobs · AWS Glue Data Catalog · AWS Glue Notebook · PySpark 
 
## Sobre o projeto

O objetivo do projeto é construir uma pipeline de dados em nuvem, baseada no Amazon Web Services (AWS) para analisar o histórico dos últimos 3 anos da State of Data Brazil (2023-2025), pesquisa realizada anualmente pela Data Hackers em parceria com a Bain & Company. 

Essa pipeline, realiza a ingestão dos dados no serviço AWS S3, sua transformação entre as camadas de uma arquitetura medalhão através do AWS Glue Jobs e a catalogação com o AWS Data Catalog, que disponibiliza a informação para o consumo analítico no AWS Glue Notebook, onde a biblioteca PySpark é utilizada.

## Estrutura do repositório

```text
pipeline_aws_etl_state_of_data/
├── data/            # Origem dos dados
├── diagrama/        # Diagrama da arquitetura do pipeline (PNG)
├── notebooks/       # Glue Notebook com as análises em PySpark
├── results/         # Gráficos gerados
├── src/             # Scripts dos Glue Jobs (bronze_to_silver e silver_to_gold)
├── .gitignore
└── README.md
```

## Diagrama da arquitetura

![Diagrama da arquitetura do pipeline](Diagrama/Diagrama.png)

## Serviços AWS

| Componente | Descrição |
|---|---|
| AWS S3 | `state-of-data-<ID>` (us-east-1) — armazena as camadas bronze, silver e gold |
| Glue ETL Jobs | `bronze_to_silver` e `silver_to_gold` (PySpark) |
| Glue Data Catalog | `db_silver` (1 tabela) e `db_gold` (5 tabelas) |
| AWS Glue Notebook | `state_of_data_brasil_analise` (PySpark) |

## Arquitetura medalhão

### Camada bronze

A fonte de dados da pipeline são 3 arquivos .csv das últimas edições da pesquisa State Of Data (2023-2025), cuja ingestão no bucket bronze do AWS S3 ocorre manualmente.

### Camada Silver

As 3 fontes de dados possuem diferenças de schema, devido ás mudanças que a pesquisa sofreu ao longo dos anos. Diante disso, o objetivo dessa camada é padronizar o schema para permitir o UNION, resultando na `tb_silver`, uma fonte única de dados limpos.

**ETL JOB (`bronze_to_silver`):**
1. Padronização de schema:
- Padronização do cabeçalho: formato snake_case, letras minúsculas e sem caracteres especiais.
- Padronização das respostas que sofreram alterações ortográficas, mas que representam o mesmo conteúdo entre os anos.
- Criação de colunas que foram implementadas em anos posteriores.
- Ordenação alfabética das colunas.
2. Remoção de linhas com tokens duplicados.
3. Criação da coluna ano_pesquisa.
4. União dos dados de 2023,2024 e 2025 e um único dataframe.
5. Armazenamento do resultado em Parquet no bucket silver (S3) e registro da tabela `tb_silver` no `db_silver`, utilizando o AWS Glue Data Catalog.

> [Script completo](src/bronze_to_silver.py)

### Camada Gold

A `tb_silver` possui 420 colunas. Dessa forma, o objetivo da camada gold é organizar o dataframe em tabelas temáticas orientadas às principais dimensões analíticas da pesquisa, mantendo o `token` como identificador único do respondente e o `ano_pesquisa` como referência temporal.

**ETL JOB (`bronze_to_silver`):**
1. Criação das 5 colunas analíticas: `tb_gold_pessoa` | `tb_gold_empresa_atual` | `tb_gold_rotina_profissional` | `tb_gold_ia` | `tb_gold_stack_tecnologico`
2. Armazenamento do resultado em Parquet no bucket gold (S3) e registro das tabelas no  `db_gold`, utilizando o AWS Glue Data Catalog

> [Script completo](src/silver_to_gold.py)

## Análise em PySpark  | Principais insights

**Como está estruturado o mercado brasileiro de Dados?** O mercado está concentrado em quatro principais atuações profissionais: Análise de Dados (26,6%), Gestão (20,8%), Engenharia de Dados (16,2%) e Ciência de Dados (14,0%). Em 2025, O setor financeiro e de tecnologia concentram quase 36% dos respondentes, mais que o dobro do terceiro setor, Consultoria (8,8%).

**Quais perfis profissionais são mais valorizados pelo mercado?** Gestão apresenta a maior valorização financeira, sendo a única atuação com presença relevante nas faixas salariais acima de R$ 20 mil/mês. Entre os perfis técnicos, Ciência e Engenharia de Dados apresentam maior concentração nas faixas salariais mais altas do que Análise de Dados, indicando valorização da especialização técnica.

**Qual é o cenário de diversidade de gênero nas carreiras de dados?** O mercado apresenta forte predominância masculina, que representou mais de 75% dos respondentes nos três anos analisados. Entre mulheres e pessoas de outros gêneros, os principais prejuízos percebidos estão relacionados à atenção às opiniões e ideias e às oportunidades e velocidade de progressão na carreira.

**Quais tecnologias apresentam maior adoção entre os profissionais?** O uso de dados está fortemente concentrado em fontes relacionais e planilhas, ambas com mais de 50% de adoção. Entre as linguagens, Python e SQL superam 50% dos respondentes, enquanto Power BI se destaca entre as ferramentas de visualização, com mais de 35%. Tecnologias específicas de bancos de dados e cloud apresentam adoção mais distribuída, sem nenhuma solução superar 30%.

**Qual é o índice de adoção de Inteligência Artificial e seu impacto?** A adoção de IA avançou significativamente: a parcela de profissionais que não utiliza nenhuma solução caiu de 14,1% em 2023 para 1,3% em 2025. Apesar da ampla adoção, apenas 26,5% dos respondentes relatam resultados efetivos com IA Generativa/LLMs, enquanto 38,4% ainda estão em projetos-piloto, indicando que seu impacto ainda deverá ser mensurado nos próximos anos.

**Existem diferenças relevantes entre regiões, senioridades ou modelos de trabalho?** Há forte concentração de respondentes no Sudeste (+60%) e nos níveis Pleno e Sênior, com uma estrutura de senioridade relativamente homogênea entre regiões. O modelo de trabalho, entretanto, apresenta diferenças regionais: o remoto predomina no Norte, Nordeste e Sul, enquanto o híbrido lidera no Sudeste e o presencial no Centro-Oeste.

**Quais oportunidades e desafios podem ser identificados para empresas que desejam investir em Dados e Inteligência Artificial?** A ampla adoção de IA e a redução da falta de compreensão sobre casos de uso — de 34,3% para 22,0% entre 2024 e 2025 — indicam avanço na utilização da tecnologia. Entretanto, a desconfiança nos resultados cresceu de 17,8% para 25,8%, enquanto, na gestão de dados, o principal desafio passou a ser equilibrar entregas técnicas e gestão (32,6%), reforçando a necessidade de conciliar capacidade técnica, gestão e geração de resultados.

> [Notebook](notebooks\state_of_data_brasil_analise.ipynb)

## Como reproduzir 

**Pré-requisitos:** conta AWS com permissões para S3, Glue e Athena, e uma IAM Role para os Glue Jobs com acesso de leitura e escrita ao bucket.

1. Obter os três arquivos `.csv` da pesquisa State of Data (2023, 2024 e 2025), disponíveis na pasta [`data/`](data/) do repositório.
2. Criar o bucket S3 `state-of-data-<ID>` (região `us-east-1`) e fazer upload dos CSVs para a camada bronze: `s3://<seu-bucket>/bronze/`
3. Criar os databases `db_silver` e `db_gold` no AWS Glue Data Catalog.
4. Executar o Glue Job `bronze_to_silver` (script em [`src/bronze_to_silver.py`](src/bronze_to_silver.py)), que padroniza o schema, une os 3 anos e registra a tabela `tb_silver` no `db_silver`.
5. Executar o Glue Job `silver_to_gold` (script em [`src/silver_to_gold.py`](src/silver_to_gold.py)), que gera as 5 tabelas temáticas em Parquet e as registra no `db_gold`.
6. Executar o Glue Notebook `state_of_data_brasil_analise` para reproduzir a análise em PySpark (versão exportada em [`notebooks/`](notebooks/)).