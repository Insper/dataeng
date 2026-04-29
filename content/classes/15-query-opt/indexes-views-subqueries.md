# Mais Recursos

Nesta aula, iremos explorar mais detalhes sobre otimizações no **PostgreSQL** (Índices, views materializadas e subqueries).

Iremos utilizar o banco de dados da aula anterior, `sfbikeshare`.

!!! exercise "Exercício"
    Inicie o ambiente da aula anterior.

    Acesse o **DBeaver** e garanta que o banco de dados `sfbikeshare` esteja disponível.

## Exercícios para Prática

Faça os seguintes exercícios para praticar e analisar o impacto do que foi aprendido na aula anterior.

!!! exercise text long "Exercício"
    Considere as *queries* a seguir:

    **Query 1**:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM public.status s
    WHERE s.category1 = 2;
    ```

    **Query 2**:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM public.status s
    WHERE s.category2 = 2;
    ```

    Qual a diferença entre elas? Execute-as e anote o resultado da sua execução.
    
    !!! answer "Resposta"
        A primeira *query* filtra pela coluna `category1`, enquanto a segunda filtra pela coluna `category2`. Ambas as colunas são do tipo inteiro, da mesma tabela.

!!! exercise text long "Exercício"
    Crie índices **B-Tree** para ambas as colunas e execute novamente as *queries*.

    **Criar índices**:

    ```sql { .copy }
    CREATE INDEX idx_status_btree_category1
    ON public.status USING btree (category1);

    CREATE INDEX idx_status_btree_category2
    ON public.status USING btree (category2);
    ```

    **Query 1**:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM public.status s
    WHERE s.category1 = 2;
    ```

    **Query 2**:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM public.status s
    WHERE s.category2 = 2;
    ```

    Compare os resultados. Alguma diferença significativa? Por quê?

    !!! answer "Resposta"
        A diferença na performance das duas *queries* após a criação dos índices ocorre devido à distribuição dos valores nas colunas `category1` e `category2`.

        O índice será mais eficaz se uma coluna tem uma distribuição mais uniforme dos valores, permitindo que o **PostgreSQL** localize rapidamente as linhas relevantes.

        Por outro lado, se uma coluna tem muitos valores repetidos (baixa cardinalidade), o índice pode não ser tão útil, pois muitas linhas podem corresponder ao mesmo valor, levando a um maior número de leituras de blocos.

        **Distribuição `category1`**:

        ```sql { .copy }
        SELECT s.category1,
               COUNT(*) AS qtde_cat1
        FROM public.status s 
        GROUP BY s.category1 
        ORDER BY s.category1 ASC;
        ```

        **Distribuição `category2`**:

        ```sql { .copy }
        SELECT s.category2,
               COUNT(*) AS qtde_cat2
        FROM public.status s 
        GROUP BY s.category2 
        ORDER BY s.category2 ASC;
        ```

## Nem tudo se resolve com índices

Embora os índices sejam ferramentas poderosas para otimização de consultas, eles não são a solução para todos os problemas de performance.

Em alguns casos, outras estratégias podem ser mais eficazes, como a reestruturação de consultas, particionamento de tabelas ou até mesmo ajustes na configuração do banco de dados.

Vamos explorar um exemplo:

!!! exercise text long "Exercício"
    Considere a *query*:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT t.station_id,
        AVG(t.bikes_available) AS qt_bikes_avaiable_avg
    FROM public.status t
    WHERE t.station_id IN (
        SELECT station_id
        FROM station s
        WHERE s.city = 'Redwood City')
    GROUP BY t.station_id;
    ```

    Antes de executar, responsa:
    
    **O que ela faz?!**

    !!! answer "Resposta"
        A *query* retorna a média de bicicletas disponíveis (`bikes_available`) para cada estação (`station_id`) localizada na cidade de `'Redwood City'`.

!!! exercise text long "Exercício"
    Execute a *query* do exercício anterior e analise o plano de execução.
    
    Quanto tempo ela leva para ser executada?

!!! exercise text short "Exercício"
    Você consegue pensar em uma forma de reescrever essa *query* para melhorar sua performance sem utilizar índices?

    Não precisa implementar, apenas pense em uma alternativa.

    !!! answer "Resposta"
        Uma forma de reescrever a *query* é utilizando um `JOIN` ao invés de uma subconsulta com `IN`. Isso pode melhorar a performance, pois o otimizador de consultas do **PostgreSQL** pode gerar um plano de execução mais eficiente com `JOINs`.

        A *query* reescrita ficaria assim:

        ```sql { .copy }
        EXPLAIN (ANALYZE, BUFFERS)
        SELECT t.station_id,
               AVG(t.bikes_available) AS qt_bikes_avaiable_avg
        FROM public.status t
        JOIN station s ON t.station_id = s.id
        WHERE s.city = 'Redwood City'
        GROUP BY t.station_id;
        ```
!!! exercise text long "Exercício"
    Agora execute a *query* reescrita e compare o plano de execução com o da *query* original.

    Houve alguma melhora na performance? Justifique sua resposta.

    !!! answer "Resposta"
        No computador do professor, a *query* reescrita utilizando `JOIN` executou **20 vezes** mais rápido em comparação com a *query* original que utiliza uma subconsulta com `IN`.

## Views Materializadas

!!! exercise text long "Exercício"
    Explique o que é uma **View**.

    !!! answer "Resposta"
        Uma **View** é uma tabela virtual que resulta de uma consulta **SQL**. Ela não armazena dados fisicamente, mas sim a definição da consulta que gera os dados quando a **View** é acessada.

        As **Views** são usadas para simplificar consultas complexas, encapsular lógica de negócios e fornecer uma camada de abstração sobre as tabelas subjacentes.

Já as **Views materializadas** são uma forma de armazenar o resultado de uma consulta para acesso rápido, evitando a necessidade de recalcular os dados toda vez que a consulta é executada.

!!! info "Importante"
    Diferente de uma **View** comum, as **Views materializadas** armazenam fisicamente os dados resultantes da consulta, o que pode melhorar significativamente o desempenho para consultas complexas ou que envolvem grandes volumes de dados.

    Assim, **Views materializadas** são particularmente úteis em cenários onde os dados não mudam com frequência, permitindo que consultas subsequentes sejam atendidas rapidamente a partir dos dados pré-calculados.

    !!! note "Atualização"
        Se os dados quase nunca mudam, diminui-se a necessidade de atualizar a **View materializada** com frequência, o que pode ser um processo custoso.

Vamos explorar o exemplo do último exercício.

!!! exercise "Exercício"
    Crie uma **View materializada** para a *query* do exercício anterior.

    ```sql { .copy }
    CREATE MATERIALIZED VIEW mv_avg_bikes_redwood_city AS
    SELECT t.station_id,
           AVG(t.bikes_available) AS qt_bikes_avaiable_avg
    FROM public.status t
    JOIN station s ON t.station_id = s.id
    WHERE s.city = 'Redwood City'
    GROUP BY t.station_id;
    ```
!!! exercise "Exercício"
    Agora, execute a consulta na **View materializada**:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM mv_avg_bikes_redwood_city;
    ```

    Compare o tempo de execução com o da *query* original e da *query* reescrita com `JOIN`.

    Qual foi a diferença? Justifique sua resposta.

    !!! answer "Resposta"
        A consulta na **View materializada** executa muito mais rápido do que tanto a *query* original quanto a *query* reescrita com `JOIN`, porque os dados já estão pré-calculados e armazenados fisicamente.

!!! exercise choice "Benefícios da IaC"
    Caso os dados envolvidos na consulta mudem (linhas adicionadas, deletadas ou atualizadas), a **View materializada** irá refletir essas mudanças automaticamente?

    - [ ] Sim
    - [X] Não

    !!! answer "Resposta"
        Não, a **View materializada** não reflete automaticamente as mudanças nos dados subjacentes.

        Ela precisa ser atualizada manualmente para incorporar quaisquer alterações feitas nas tabelas originais.

As **View materializada** precisam ser atualizadas para refletir mudanças nos dados. Isso pode ser feito com o comando:

```sql { .copy }
REFRESH MATERIALIZED VIEW mv_avg_bikes_redwood_city;
```

Esse comando recalcula e atualiza os dados armazenados na **View materializada**. Este processo pode ser custoso dependendo do tamanho dos dados e da complexidade da consulta original.

Idealmente, as **Views materializadas** são atualizadas em horários específicos ou quando necessário, para equilibrar a necessidade de dados atualizados com o desempenho da consulta.

!!! tip "Dica"
    Pesquise sobre `pg_cron`, uma extensão do **PostgreSQL** que permite agendar tarefas, como a atualização de **Views materializadas**, de forma automática e periódica.

    Mas você também pode usar ferramentas externas para agendar essas atualizações, como o `cron` do sistema operacional, ou ainda *scripts* que são parte de *pipelines* orquestrados.

!!! exercise "Exercício"
    Para remover a **View materializada**, execute:

    ```sql { .copy }
    DROP MATERIALIZED VIEW IF EXISTS mv_avg_bikes_redwood_city;
    ```

## Exercícios

Vamos trabalhar com outra tabela.

!!! exercise text long "Exercício"
    Rode a seguinte consulta e analise o resultado:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT COUNT(DISTINCT zip_code)
    FROM public.trip t;
    ```

!!! exercise text long "Exercício"
    Crie um índice **B-Tree** na coluna `zip_code`.

    Execute novamente a consulta e compare os resultados.

!!! exercise text long "Exercício"
    Transforme a consulta em uma **View materializada**.

    Compare a performance da **View materializada** com as execuções anteriores.

    !!! answer "Resposta"
        **Criar View materializada**:

        ```sql { .copy }
        CREATE MATERIALIZED VIEW mv_distinct_zip_codes AS
        SELECT COUNT(DISTINCT zip_code) AS distinct_zip_count
        FROM public.trip t;
        ```

        **Executar consulta na View materializada**:

        ```sql { .copy }
        EXPLAIN (ANALYZE, BUFFERS)
        SELECT * FROM mv_distinct_zip_codes;
        ```

!!! exercise text long "Exercício"
    Cite uma ou mais situações onde o uso de **Views materializadas** seria benéfico.

    Indique um exemplo prático.

    !!! answer "Resposta"
        Alguns cenários onde o uso de **Views materializadas** seria benéfico incluem:

        1. **Relatórios frequentes**: Quando há necessidade de gerar relatórios complexos regularmente, como dashboards de negócios que agregam grandes volumes de dados.

        1. **Consultas complexas**: Quando as consultas envolvem múltiplas junções, agregações ou cálculos que são custosos em termos de tempo de execução.

        1. **Dados estáticos ou pouco dinâmicos**: Quando os dados subjacentes não mudam com frequência, permitindo que a View materializada seja atualizada periodicamente sem impactar a performance.

        1. **Redução da carga no banco de dados**: Em sistemas com alta demanda de leitura, as Views materializadas podem aliviar a carga ao fornecer resultados pré-calculados.

        !!! info "Exemplo prático"
            Um sistema de análise de vendas que gera relatórios sobre o desempenho de produtos, onde os dados de vendas não mudam após o fechamento do dia e o dia atual não é considerado.

            Neste caso, a **View materializada** pode ser atualizada de madrugada, sem sobrecarregar o sistema, permitindo que os relatórios sejam gerados rapidamente durante o dia seguinte.

!!! exercise text long "Exercício"
    Cite uma ou mais situações onde o uso de **Views materializadas** não seria indicado.

    Indique um exemplo prático.

    !!! answer "Resposta"
        Alguns cenários onde o uso de **Views materializadas** não seria indicado incluem:

        1. **Dados altamente dinâmicos**: Quando os dados subjacentes mudam frequentemente, exigindo atualizações constantes da View materializada, o que pode ser custoso e ineficiente.

        1. **Consultas simples**: Quando as consultas são simples e rápidas de executar, o overhead de manter uma View materializada pode superar os benefícios.

        1. **Espaço de armazenamento limitado**: Se o banco de dados tem restrições de espaço, armazenar Views materializadas pode não ser viável.

        1. **Necessidade de dados em tempo real**: Em aplicações que exigem acesso a dados em tempo real, as Views materializadas podem introduzir latência devido à necessidade de atualização periódica.

        !!! info "Exemplo prático"
            Um sistema de monitoramento em tempo real que rastreia transações financeiras, onde os dados mudam constantemente e a precisão imediata é crucial.