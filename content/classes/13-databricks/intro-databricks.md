# Introdução ao Databricks

Na página anterior, discutimos o modelo de **Plataforma como Serviço (PaaS)** e como ele abstrai a complexidade da infraestrutura, permitindo que as equipes de dados foquem em gerar valor. Vimos que, ao usar serviços gerenciados, ganhamos agilidade e escalabilidade.

No entanto, o ecossistema de dados é vasto. Temos ferramentas para orquestração, outras para **ETL**, algumas para **Machine Learning** e outras para **Business Intelligence**.

!!! quote "Engenheiro de Dados Sênior"
    *"Nossa equipe gasta um tempo considerável integrando diferentes serviços. Temos uma ferramenta para ingestão, outra para transformação, uma terceira para versionamento de modelos de ML e uma quarta para criar dashboards. Manter tudo isso coeso e seguro é um desafio constante."*

Essa fragmentação de ferramentas, embora poderosa, cria silos e aumenta a complexidade operacional.

??? "Silos"
    Os silos de dados são **repositórios de informações isolados** e inacessíveis a outras áreas, sistemas ou departamentos de uma organização.

E se houvesse uma plataforma que unificasse todo o ciclo de vida dos dados, da engenharia à inteligência artificial?

## O que é o Databricks?

É para resolver esse desafio que surge o [**Databricks**](https://databricks.com/).

!!! important "Definição: Databricks"
    O **Databricks** é uma **plataforma unificada de análise de dados**, construída pelos criadores do [**Apache Spark**](https://spark.apache.org/).

    Ela combina engenharia de dados, ciência de dados, Machine Learning e *analytics* em um único ambiente colaborativo.

!!! info "Info"
    O **Databricks** combina características de **PaaS** e **SaaS (Software como Serviço)**.

A proposta do **Databricks** é fornecer uma solução única e integrada que atenda às necessidades de diferentes profissionais de dados, como engenheiros, cientistas e analistas, diminuindo a necessidade de múltiplas ferramentas.

## A Arquitetura Lakehouse

O conceito central por trás do **Databricks** é o [**Lakehouse**](../10-data-lakehouse/intro-data-lakehouse.md).

Como vimos em aulas anteriores, os **Data Warehouses** são excelentes para **BI** e análises estruturadas, mas são caros e inflexíveis para dados não estruturados.

Por outro lado, os **Data Lakes** são ótimos para armazenar grandes volumes de dados brutos a baixo custo, mas sofrem com problemas de confiabilidade e performance para análises.

O **Lakehouse** busca unir o melhor dos dois mundos.

!!! info "Relembrando: Lakehouse"
    Uma arquitetura **Lakehouse** é um paradigma de gerenciamento de dados que combina a flexibilidade, o baixo custo e a escala dos **Data Lakes** com a confiabilidade, a estrutura e a performance dos **Data Warehouses**, tudo sobre um armazenamento padronizado.

O **Databricks** implementa o **Lakehouse** utilizando tecnologias *open-source* como o **Delta Lake**, que adiciona uma camada de armazenamento transacional e confiável sobre o seu **Data Lake** (como o **Amazon S3**).

!!! tip
    O **Delta Lake** é uma alternativa similar ao **Apache Iceberg**, vista no curso.

!!! exercise text long
    Com base no que você aprendeu sobre **Data Lakes** e **Data Warehouses**, cite duas principais vantagens de combinar ambas as arquiteturas em um único sistema (o **Lakehouse**)?

    !!! answer "Resposta"
        1.  **Eliminação de Silos de Dados**: Em vez de manter dados estruturados em um **Data Warehouse** e dados brutos/semi-estruturados em um **Data Lake** (e ter que movê-los entre os dois), o **Lakehouse** permite que ambos os tipos de dados coexistam. Isso simplifica a arquitetura e garante que todos usem uma única fonte da verdade.

        2.  **Suporte a Diversos Casos de Uso**: A mesma plataforma pode ser usada para *dashboards* de **BI** (tradicionalmente um caso de uso de **Data Warehouse**) e para treinar modelos de **Machine Learning** sobre dados não estruturados (tradicionalmente um caso de uso de **Data Lake**), aumentando a agilidade e reduzindo custos.

## Componentes e Casos de Uso

O **Databricks** integra diversas ferramentas, muitas delas criadas pela própria empresa e doadas para a comunidade open-source:

-   **Apache Spark**: O motor de processamento distribuído que serve como base para as transformações de dados em larga escala. Foi criada pelos fundadores do **Databricks**.
-   **Delta Lake**: Adiciona transações ACID, versionamento de dados e governança ao Data Lake.
-   **MLflow**: Uma plataforma para gerenciar todo o ciclo de vida do **Machine Learning**, desde a experimentação até o *deploy*. Este conteúdo é abordado na eletiva de **MLOps & Interviews**.
-   **Unity Catalog**: Oferece uma governança de dados unificada para todos os ativos de dados e **IA** na plataforma.

Com essa base unificada, engenheiros de dados podem usar o **Databricks** para:

-   **Construir pipelines de ETL escaláveis**: Utilizando **Spark** e **Delta Lake** para processar dados em *batch* ou *streaming*.
-   **Realizar análises de dados interativas**: Analistas podem usar **SQL** para consultar diretamente os dados no **Lakehouse**.
-   **Desenvolver e implantar modelos de Machine Learning**: Cientistas de dados têm um ambiente colaborativo com notebooks e **MLflow** para treinar e servir modelos.

!!! exercise choice "Question"
    Uma empresa deseja construir um sistema que, ao mesmo tempo, processe terabytes de *logs* de aplicação (dados semi-estruturados) para treinar um modelo de detecção de anomalias e sirva *dashboards* de **BI** para a diretoria com dados de vendas (dados estruturados). Qual arquitetura seria mais adequada para este cenário?

    - [ ] Um Data Warehouse tradicional, pois o foco principal é BI.
    - [ ] Um Data Lake, pois o volume de dados não estruturados é muito grande.
    - [x] Uma arquitetura **Lakehouse**, pois ela suporta nativamente ambos os casos de uso (BI e ML) sobre uma única cópia dos dados.
    - [ ] Dois sistemas separados: um Data Lake para ML e um Data Warehouse para BI.

## Conclusão

Ao unificar as capacidades de **Data Lakes** e **Data Warehouses** e integrar ferramentas para todo o ciclo de vida dos dados, o **Databricks** oferece uma solução que facilita a migração para um modelo *data-driven*.

Para engenheiros de dados, a plataforma simplifica a construção e o gerenciamento de infraestruturas complexas, permitindo focar na entrega de dados limpos, confiáveis e prontos para a análise.