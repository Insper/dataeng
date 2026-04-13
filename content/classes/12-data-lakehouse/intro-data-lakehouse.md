
# Data Lakehouse

## Introdução

Vamos fazer uma checagem rápida dos conceitos que você aprendeu até agora.

!!! exercise text long
    Qual o problema resolvido por um **Data Warehouse**?
    Por que precisamos dele?

    !!! answer
        Realizar análise de dados diretamente nos sistemas transacionais (OLTP) pode prejudicar o desempenho desses sistemas. Além disso, os dados nesses sistemas são frequentemente estruturados de maneira diferente (por exemplo, mais de um sistema tem a tabela `cliente`, com atributos diferentes e mesmos atributos com nomenclatura distintas), o que dificulta a integração e a análise.

        Um **Data Warehouse** resolve o problema de integrar dados de diferentes fontes (diferentes sistemas da empresa). Ele armazena dados históricos e estruturados, permitindo uma análise mais fácil e eficiente. Além disso, ele é **otimizado para consultas analíticas**, o que melhora o desempenho das análises de negócio.

!!! exercise text long
    O que é um **Data Lake**? Por que ele é necessário?

    !!! answer
        Um **Data Lake** é um repositório centralizado que permite armazenar grandes volumes de dados em seu formato bruto, sejam eles estruturados, semi-estruturados ou não estruturados. Além disso, os **Data Lakes** oferecem escalabilidade e flexibilidade (algo mais complicado de se obter em um **DW**), permitindo que as organizações cresçam e se adaptem às mudanças nas necessidades de dados.

        Diferente dos **Data Warehouses**, que exigem a transformação e estruturação dos dados antes do armazenamento, os **Data Lakes** permitem que os dados sejam armazenados em seu estado original, proporcionando maior flexibilidade para análises futuras.

!!! exercise text short
    Cite alguns exemplos de aplicações que poderiam consumir dados de um **Data Lake**.

    !!! answer
        AWS Athena, Apache Spark, Trino, ferramentas de BI (ex: Tableau, Power BI).

!!! exercise text short
    Em um **Data Lake**, o que acontece se diversas aplicações tentam escrever o mesmo arquivo ao mesmo tempo?

    !!! answer
        Isso pode levar a conflitos e corrupção de dados, já que não há um controle de concorrência robusto.

Apesar das vantagens, o modelo tradicional de **Data Lake** apresenta desafios importantes: a ausência de controle rigoroso de esquema pode levar à corrupção ou inconsistência dos dados, operações de atualização e exclusão são complexas ou pouco confiáveis, e a governança se torna difícil à medida que o volume e a variedade aumentam.

Como alternativa, introduzimos o conceito de **Data Lakehouse**.

## Data Lakehouse

O termo **Data Lakehouse** surgiu para descrever uma arquitetura híbrida que une as vantagens do **data lake** e do **data warehouse**. Popularizado por volta de 2020, especialmente pela **Databricks**, o conceito busca simplificar o ecossistema de dados, permitindo que diferentes tipos de dados sejam armazenados e analisados em um único repositório.

Imagine que você precisa lidar com dados de sensores, logs, imagens e também tabelas de vendas e clientes. Como garantir flexibilidade, governança e performance analítica sem duplicar dados ou criar silos? O **Data Lakehouse** propõe uma solução para esse dilema.

## Por que surgiu?

Os primeiros **data lakes** eram ótimos para armazenar dados brutos, mas tinham limitações sérias: falta de controle de esquema, dificuldade de atualização/exclusão de registros e pouca governança. Muitas vezes, viravam *data swamps* (pântanos de dados), dificultando a descoberta e o uso confiável dos dados.

O **Data Lakehouse** evolui esse cenário ao adicionar uma camada **transacional** sobre o data lake, trazendo recursos antes exclusivos dos **data warehouses**, como controle de esquema, histórico de tabelas, comandos `DML` (`INSERT, UPDATE, DELETE, MERGE`) e suporte a transações **ACID**.

??? "ACID"
    **ACID** é um acrônimo que representa quatro propriedades fundamentais para garantir a confiabilidade das transações em sistemas de banco de dados:

    - **Atomicidade (*Atomicity*)**: Garante que todas as operações dentro de uma transação sejam concluídas com sucesso ou nenhuma delas seja aplicada. Se qualquer parte da transação falhar, o sistema reverte para o estado anterior, garantindo que dados parciais não sejam salvos.
    - **Consistência (*Consistency*)**: Assegura que uma transação leve o banco de dados de um estado válido para outro estado válido, mantendo todas as regras e restrições definidas (como integridade referencial, tipos de dados, etc.).
    - **Isolamento (*Isolation*)**: Garante que as operações de uma transação sejam isoladas das operações de outras transações concorrentes, evitando interferências e garantindo que o resultado final seja o mesmo como se as transações fossem executadas sequencialmente.
    - **Durabilidade (*Durability*)**: Assegura que, uma vez que uma transação foi confirmada (committed), suas alterações sejam permanentes, mesmo em caso de falhas do sistema, como quedas de energia ou erros de hardware.

!!! tip "Dica!"
    Especialistas preveem que, para a maioria dos casos, o **Data Lakehouse** será a solução intermediária ideal para *analytics*, com *datasets* específicos migrando para **DWs** tradicionais apenas quando necessário.
    A arquitetura **Lakehouse** vem sendo cada vez mais adotada porque combina a flexibilidade e baixo custo do **Data Lake** com funcionalidades típicas de **Data Warehouses** (ACID, controle de esquema, histórico de versões), simplificando a gestão e análise de dados.

    O **Data Lakehouse** combina a flexibilidade e a economia de um **Data Lake** com as capacidades de gestão, estrutura e performance de um **Data Warehouse**.

## Características Fundamentais

1. **Armazenamento Unificado**: Todos os tipos de dados (estruturados, semi-estruturados, não estruturados) ficam em um único **Data Lake**, geralmente em object storage.
2. **Camada Transacional**: Tecnologias como Delta Lake, Apache Iceberg e Apache Hudi permitem operações de banco de dados relacional sobre arquivos, trazendo confiabilidade e segurança.
3. **Transações ACID**: Diferente do data lake tradicional, o lakehouse garante atomicidade, consistência, isolamento e durabilidade nas operações.
4. **Gerenciamento de Esquema**: É possível impor restrições de tipo, nulidade e exclusividade, rejeitando dados incompatíveis.
5. **Histórico e Rollback**: O lakehouse mantém versões antigas dos dados e permite reverter alterações.
6. **Processamento Unificado**: Pipelines de *streaming* e *batch* podem ser implementados sobre o mesmo repositório, simplificando a arquitetura.

!!! exercise text long
    Pense em um cenário onde dados de vendas, logs de acesso e imagens de produtos precisam ser analisados juntos. Quais seriam os desafios de manter esses dados separados em diferentes sistemas? Como o **Data Lakehouse** pode ajudar?

    !!! answer
        Manter dados separados dificulta integrações, aumenta custos e pode gerar inconsistências entre relatórios. O **Data Lakehouse** permite centralizar e governar todos os dados, facilitando análises e reduzindo duplicidade.

## Vantagens e Benefícios

- **Redução da complexidade**: Não é necessário manter dois repositórios (lake e warehouse), simplificando o gerenciamento.
- **Maior confiabilidade**: Uma única cópia dos dados evita inconsistências entre relatórios e ferramentas.
- **Dados mais atuais**: Elimina o problema de dados defasados entre sistemas.
- **Facilidade para ciência de dados**: Cientistas podem trabalhar diretamente com arquivos abertos, sem depender de extrações complexas.
- **Governança aprimorada**: Segurança e qualidade podem ser aplicadas de forma consistente.

!!! info "Importante!"
    Apesar dos avanços, o **Data Lakehouse** ainda enfrenta desafios, especialmente em cenários de BI interativo, onde **DWs** tradicionais podem oferecer consultas mais rápidas (baixa latência). Além disso, a camada de metadados do **Lakehouse** precisa ser bem estruturada para garantir descoberta e relacionamento entre tabelas.

    !!! warning "Atenção!"
        Nem toda carga de trabalho se adapta perfeitamente ao **Data Lakehouse**. Avalie requisitos de performance, governança e integração antes de migrar sistemas críticos.

## Conclusão

O **Data Lakehouse** representa uma evolução importante na arquitetura de dados, combinando flexibilidade, governança e performance analítica em um único ambiente.

Siga para a próxima seção para iniciar a parte prática!
