# Pipeline AWS - State of Data
> Pipeline de dados em nuvem, baseada no Amazon Web Services (AWS) para analisar o histórico dos últimos 3 anos da pesquisa State of Data Brazil (2023-2025)

AWS S3 · AWS Glue ETL Jobs · AWS Glue Data Catalog · AWS Glue Notebook · PySpark 
 
## Sobre o projeto

O objetivo do projeto é construir uma pipeline de dados em nuvem, baseada no Amazon Web Services (AWS) para analisar o histórico dos últimos 3 anos da State of Data Brazil (2023-2025), pesquisa realizada anualmente pela Data Hackers em parceria com a Bain & Company. 

Essa pipeline, realiza a ingestão dos dados no serviço de armazenamento AWS S3, sua transformação entre as camadas de uma arquitetura medalhão através do AWS Glue Jobs e a catalogação com o AWS Data Catalog, que disponibiliza a informação para o consumo analítico no Amazon Athena e AWS Glue Notebook. 

Utilizando PySpark, a análise se concentra nos temas: 
1. Qual a estruturação atual do mercado de dados, incluindo a diversidade de gênero nas carreiras da área; 
2. Quais perfis profissionais e tecnologias são mais valorizados pelas empresas;
3. Qual o índice de adoção de Inteligência Artificial e quais oportunidades e desafios se apresentam para empresas que desejam investir em Dados e IA.
4. Diferenças entre regiões, senioridades ou modelos de trabalho

## Estrutura do repositório


## Diagrama da arquitetura

![alt text](image.png)

## Componentes na AWS

| Componente | Descrição |
|---|---|
| S3 | `state-of-data-<ID>` (us-east-1) — armazena as camadas bronze, silver e gold |
| Glue ETL Jobs | `bronze_to_silver` e `silver_to_gold` (PySpark) |
| Glue Data Catalog | `db_silver` (1 tabela) e `db_gold` |
| AWS Glue Notebook | `state_of_data_brasil_analise` (PySpark) |

## Arquitetura medalhão

### Camada bronze

A fonte de dados da pipeline são 3 arquivos .csv das últimas edições da pesquisa State Of Data (2023-2025), cuja ingestão no bucket da camada bronze do S3 ocorre manualmente.

### Camada Silver

O objetivo dessa etapa foi consolidar os dados das pesquisas de 2023,2024 e 2025 na `tb_silver`, com dados limpos, identificação de ano e schema padronizado. Como a pesquisa não possui a mesma estrutura entre os anos, essa etapa concentra o maior volume de transformações do projeto.

Ações realizadas:
1. Padronização de schema entre os anos:
- Padronização do cabeçalho: formato snake_case, letras minúsculas e sem caracteres especiais.
- Padronização das respostas que sofreram alterações ortográficas, mas que representam o mesmo conteúdo entre os anos.
- Criação de colunas que foram implementadas em anos posteriores.
- Ordenação alfabética das colunas.
2. Remoção de linhas com tokens duplicados.
3. Criação da coluna ano_pesquisa.
4. União dos dados de 2023,2024 e 2025 e um único dataframe.
5. Armazenamento do resultado em Parquet no Amazon S3 e registro da tabela `tb_silver` no `db_silver`, utilizando o AWS Glue Data Catalog.

> Script completo

### Camada Gold

Nesta etapa, o AWS Glue é utilizado para criar um ETL Job que transforma os arquivos Parquet do bucket Silver em 5 tabelas catalogada no AWS Glue Data Catalog, com os dados armazenados no bucket gold (S3) em formato Parquet.

Dado o número elevado de colunas da camada Silver (420), a camada Gold foi organizada em tabelas temáticas orientadas às principais dimensões analíticas da pesquisa, mantendo token como identificador único do respondente. 

1. tb_gold_pessoa
2. tb_gold_empresa_atual
3. tb_gold_rotina_profissional
4. tb_gold_ia
5. tb_gold_stack_tecnologico


## Análise em PySpark


## Como reproduzir 

