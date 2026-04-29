
# Pratica com Índices

Nesta seção, vamos trabalhar com o banco de dados `sfbikeshare` para criar índices e analisar seu impacto na performance.

!!! exercise text long "Exercício"
    No **DBeaver**, explore o banco de dados `sfbikeshare` e suas tabelas.

    Quantas linhas cada tabela possui?

## Baseline

Vamos estabelecer uma baseline de performance para nossas consultas. Isso nos permitirá quantificar exatamente o impacto que os índices terão na performance.

!!! exercise "Exercício"
    No **DBeaver**, conecte-se ao banco de dados `sfbikeshare` e execute a seguinte consulta para analisar seu plano de execução:

    !!! warning "Select ALL"
        Utilizar `SELECT *` não é recomendado em produção!

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM trip 
    WHERE start_station_id = 50;
    ```

    Anote os seguintes valores:
    
    1. Tempo de execução (Execution Time)
    1. Tipo de scan utilizado
    1. Número de linhas processadas vs. retornadas
    1. Informações de buffers

!!! exercise text long
    Com base no resultado do exercício anterior, você consegue identificar oportunidades de otimização? O que o plano de execução está revelando sobre como o PostgreSQL está processando esta consulta?

    !!! answer "Resposta"
        O plano de execução provavelmente mostra um Sequential Scan, indicando que o banco está lendo todas as mais de 600.000 linhas da tabela `trip` para encontrar aquelas onde `start_station_id = 50`.

        Isso é ineficiente, especialmente considerando que provavelmente apenas algumas milhares de linhas satisfazem essa condição.

        A criação de um índice na coluna `start_station_id` permitiria ao PostgreSQL localizar diretamente as linhas relevantes sem precisar varrer toda a tabela.

Vamos também examinar esta *query*:

!!! exercise "Exercício"
    Execute a seguinte consulta com análise de plano:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT t.id,
           t.duration,
           t.start_date,
           s.name AS station_name
    FROM trip t
    JOIN station s ON t.start_station_id = s.id
    WHERE t.start_date >= '2015-01-01'
      AND t.start_date < '2015-02-01'
      AND t.duration > 3600;
    ```

    Esta consulta busca viagens que:
    - Começaram em janeiro de 2015
    - Duraram mais de uma hora (3600 segundos)
    - E recupera o nome da estação de origem

    Analise o plano de execução e identifique quais operações estão consumindo mais tempo.

## Criando o Primeiro Índice

Vamos começar criando um índice **B-Tree** na coluna `start_station_id` da tabela `trip`, que usamos na primeira consulta.

!!! exercise "Exercício"
    Crie um índice B-Tree na coluna `start_station_id`:

    ```sql { .copy }
    CREATE INDEX idx_trip_start_station 
    ON trip USING btree (start_station_id);
    ```

    !!! warning "Atenção!"
        O PostgreSQL levará alguns segundos para criar o índice, pois precisa processar todas as mais de **600k** linhas.

!!! info "Info"
    Durante a criação do índice, o PostgreSQL mantém a tabela disponível para leituras, mas bloqueia escritas.

    Em ambientes de produção com tabelas muito grandes, você pode usar `CREATE INDEX CONCURRENTLY` para evitar bloquear escritas, embora isso torne a criação do índice mais lenta.

Agora vamos verificar o impacto do índice:

!!! exercise "Exercício"
    Execute novamente a consulta anterior:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM trip t
    WHERE t.start_station_id = 50;
    ```

    Compare o resultado com a execução anterior (sem índice). O que mudou?

    1. O tipo de scan mudou de Sequential para Index Scan?
    1. O tempo de execução diminuiu? Em quanto?
    1. Como mudou o número de blocos lidos (buffers)?

!!! exercise text long "Exercício"
    Agora execute a consulta a seguir.

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT *
    FROM trip t
    WHERE t.subscription_type = 'Subscriber';
    ```

    Anota os resultados, crie um índice **B-Tree** na coluna `subscription_type`, e execute a consulta novamente.

    Aguma melhora? Como você explicaria o resultado?!

    !!! answer "Resposta"
        **Dica**: Crie uma *query* e analise a distribuição dos valores na coluna `subscription_type` *versus* `start_station_id`.

## Índices para Análises Temporais

A tabela `status` contém *timestamps* na coluna `time`.

Análises temporais são extremamente comuns em engenharia de dados, então vamos explorar como otimizá-las.

!!! exercise "Exercício"
    Execute esta consulta:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT t.station_id,
           AVG(t.bikes_available) AS qt_bikes_avaiable_avg
    FROM public.status t
    WHERE t.time BETWEEN '2013-12-01' AND '2013-12-31'
    GROUP BY t.station_id;
    ```

    Quanto tempo leva? O PostgreSQL está usando algum índice?

!!! exercise "Exercício"
    Crie um índice **B-Tree** na coluna `time`:

    ```sql { .copy }
    CREATE INDEX idx_status_time
    ON status USING btree (time);
    ```

    Execute novamente a consulta de agregação. Houve melhoria? Por que sim ou por que não?

!!! exercise text long
    Em consultas de agregação que precisam varrer uma grande porção da tabela (como neste caso, dois anos de dados), índices podem não trazer tanto benefício quanto em buscas pontuais. Por que você acha que isso acontece?

    !!! answer "Resposta"
        Quando uma consulta precisa processar uma porcentagem significativa da tabela (mais de 5-10%), o PostgreSQL muitas vezes decide que um **Sequential Scan** é mais eficiente que usar o índice.

        Isso porque usar um índice requer duas operações de **I/O**: ler o índice e depois ler os blocos da tabela, que podem estar espalhados fisicamente no disco.

        Para grandes porções de dados, ler a tabela sequencialmente é mais eficiente que pular entre diferentes blocos.

        Índices são mais efetivos para consultas seletivas que retornam uma pequena fração dos dados.

!!! exercise "Exercício"
    Faça **DROP** no índice anterior e crie um índice **BRIN** na coluna `time`:

    ```sql { .copy }
    DROP INDEX idx_status_time;

    CREATE INDEX idx_status_time_brin
        ON status USING brin(time);
    ```

    Este índice deve ser criado muito mais rapidamente que um **B-Tree** seria.

!!! exercise "Exercício"
    Execute novamente a consulta e compare com os resultados anteriores:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT t.station_id,
           AVG(t.bikes_available) AS qt_bikes_avaiable_avg
    FROM public.status t
    WHERE t.time BETWEEN '2013-12-01' AND '2013-12-31'
    GROUP BY t.station_id;
    ```

!!! exercise "Exercício"
    Para verificar o tamanho dos índices da tabela `status`, execute:

    ```sql { .copy }
    SELECT
    pg_size_pretty(pg_relation_size('public.status')) AS table_data_size,
    pg_size_pretty(pg_indexes_size('public.status')) AS indexes_size,
    pg_size_pretty(pg_total_relation_size('public.status')) AS total_size;
    ```

!!! exercise "Exercício"
    Utilize o `pgbench` para realizar testes de performance com as tabelas contendo índices.

    Garanta que os índices existem e atualize as *queries* de teste. O ideal é testar com seleção, inserção, atualização e deleção.

## Exercício

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

        O índice será mais eficaz se uma coluna tem uma distribuição mais uniforme dos valores, permitindo que o PostgreSQL localize rapidamente as linhas relevantes.

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

Por hoje é só! Bons estudos!