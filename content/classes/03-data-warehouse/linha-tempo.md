# Linha do Tempo

A evolução dos bancos de dados é um reflexo direto da crescente demanda por gerenciar volumes de dados cada vez maiores e mais complexos.

Na sequência, apresentamos uma linha do tempo com destaque para os principais marcos.

1. **Décadas de 1960 e 1970**
    - **Modelos de Dados Hierárquicos e de Rede:** Antes dos bancos de dados relacionais, os modelos hierárquicos e de rede dominavam. O *Information Management System* (IMS), da IBM, lançado em 1968, é um exemplo proeminente de banco de dados hierárquico, que organizava dados em uma estrutura de árvore. O modelo de rede, com sua estrutura mais flexível, permitia que um registro tivesse múltiplos "pais".
    - **A Proposta do Modelo Relacional:** Em 1970, Edgar F. Codd, um cientista da computação da IBM, publicou um artigo que propunha o modelo relacional. Esse modelo, baseado na teoria de conjuntos e na álgebra relacional, organizava os dados em **tabelas** (relações), usando **chaves primárias** e **chaves estrangeiras** para estabelecer conexões entre elas. O conceito revolucionou a forma como os dados eram armazenados e acessados, tornando-os mais fáceis de entender e manipular.

2. **Décadas de 1980 e 1990**
    - **O Surgimento dos Bancos de Dados Relacionais:** A década de 1980 viu a ascensão dos *Sistemas de Gerenciamento de Banco de Dados Relacionais* (**RDBMS**). Empresas como a Oracle, IBM (com o DB2) e Microsoft (com o SQL Server) começaram a dominar o mercado. A linguagem SQL (Structured Query Language), que havia sido desenvolvida na década anterior, tornou-se o padrão da indústria para interagir com esses bancos de dados.

    !!! tip
        O modelo relacional **RDBMS** são assuntos principais da disciplina de **Megadados** do Insper!
    
    !!! info
        O projeto **PostgreSQL** teve início em 1986, sob a direção do Professor Michael Stonebreaker, da Universidade da Califórnia, Berkeley.

        Já o **MySQL** teve seu lançamento em 1995.

    - **O Conceito de Data Warehousing:** No final dos anos 1980, Bill Inmon propôs o conceito de **Data Warehouse**, definindo-o como um repositório centralizado de dados integrados, organizados por assunto, não-voláteis e variantes no tempo, especificamente projetado para suporte à tomada de decisões. Este conceito revolucionou a forma como as empresas armazenavam e analisavam dados para *business intelligence*.
    - **Bancos de Dados Orientados a Objetos:** Surge o conceito de bancos de dados orientados a objetos, que tentavam preencher a lacuna entre a programação orientada a objetos e o armazenamento de dados. No entanto, sua adoção foi limitada e eles não conseguiram substituir a popularidade dos **RDBMS**.

3. **Década de 2000**
    - **Bancos de Dados para a Web e a Explosão de Dados:** Com a popularização da internet e o crescimento das plataformas online, a necessidade de lidar com grandes volumes de dados (**Big Data**) se tornou mais evidente. A estrutura rígida dos **RDBMS** nem sempre era a mais adequada para os dados não estruturados gerados pela web.
    - **O Surgimento do NoSQL:** Em resposta a essas novas necessidades, o movimento **NoSQL** (Not Only SQL) ganhou força. Essa classe de bancos de dados se desviava do modelo relacional, oferecendo diferentes modelos de dados (como chave-valor, documentos, colunas e grafos) para maior flexibilidade e escalabilidade horizontal, especialmente para aplicações web.

4. **Década de 2010**
    - **O Advento do Big Data e Data Lakes:** A proliferação de dados de diversas fontes (redes sociais, sensores, IoT) levou à criação do conceito de **Data Lake**. Diferente de um **Data Warehouse**, que armazena dados já estruturados e processados, um **Data Lake** é um repositório centralizado que armazena grandes volumes de dados brutos e de diversos formatos. O objetivo é guardar os dados "crus" para futura análise, sem a necessidade de pré-definição de um esquema.
    - **A Evolução dos Data Warehouses:** Os **Data Warehouses**, que já existiam desde os anos 80, evoluíram para lidar com **Big Data**. Soluções na nuvem, como o **Amazon Redshift** e o **Google BigQuery**, tornaram-se populares, oferecendo escalabilidade e performance para análise.

5. **Década de 2020**
    - **Lakehouses: O Melhor de Dois Mundos:** O conceito de **Lakehouse** emerge para unir as melhores características dos **Data Lakes** e dos **Data Warehouses**. Um **Lakehouse** é construído sobre um **Data Lake** e oferece a estrutura e as funcionalidades de um **Data Warehouse**, permitindo análises de *BI* e *machine learning* diretamente sobre os dados brutos. Isso elimina a necessidade de mover os dados entre sistemas, garantindo mais consistência e eficiência.

!!! info "Importante!"
    Os bancos de dados relacionais continuam sendo a espinha dorsal de muitas aplicações, mas a diversidade de ferramentas e abordagens, como o *NoSQL* e o *Lakehouse*, demonstra a necessidade de soluções mais flexíveis para os desafios de dados do mundo moderno.

Nas próximas aulas, conceitos como **Data Lakes** serão explorados com mais detalhes. Nosso foco hoje será em **Data Warehouses**.