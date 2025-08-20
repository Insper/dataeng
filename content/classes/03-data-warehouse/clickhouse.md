# ClickHouse

O **ClickHouse** é um sistema de gerenciamento de banco de dados (**SGBD**) analítico columnar (*OLAP*) de código aberto, desenvolvido pela Yandex. É otimizado especificamente para consultas analíticas em tempo real sobre grandes volumes de dados.

!!! info
    O **ClickHouse** é conhecido por sua excepcional velocidade em consultas analíticas.

    Utiliza armazenamento **columnar** e **processamento paralelo massivo** (**MPP**) para alta performance.

## Características principais

- **Columnar storage**: Armazena dados por colunas, otimizando consultas analíticas
- **Compressão avançada**: Algoritmos de compressão específicos para cada tipo de dado
- **SQL completo**: Suporte robusto ao SQL com extensões para análise avançada
- **Distribuído**: Escalabilidade horizontal nativa com *clusters*
- **Real-time**: Inserções e consultas em tempo real
- **Formatos diversos**: Suporte nativo a **Parquet**, **CSV**, **JSON** e outros formatos

## Preparando o ambiente

Vamos configurar um ambiente com **ClickHouse** utilizando Docker e trabalhar com os mesmos dados da aula de **DuckDB**.

!!! exercise
    Crie um diretório para o projeto:

    <div class="termy">

    ```bash
    $ mkdir clickhouse
    $ cd clickhouse
    ```

    </div>

!!! exercise
    Crie um arquivo `clickhouse/docker-compose.yml` para configurar o **ClickHouse**:

    !!! warning "Atenção!"
        Caso seja necessário, **edite as portas** externas!

    ```yaml { .copy }
    services:
      clickhouse:
        image: clickhouse/clickhouse-server:25.7.4.11-alpine
        container_name: clickhouse-server
        ports:
          - "8123:8123"
          - "9000:9000"
        environment:
          CLICKHOUSE_DB: bike_share_clickhouse
          CLICKHOUSE_USER: admin
          CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT: 1
          CLICKHOUSE_PASSWORD: password123
        volumes:
          - clickhouse-data:/var/lib/clickhouse
          - ./dados:/var/lib/clickhouse/user_files
        ulimits:
          nofile:
            soft: 262144
            hard: 262144

    volumes:
      clickhouse-data:
    ```

    !!! info "Info!"
        O **ClickHouse** só permite acesso a arquivos dentro do diretório `user_files` por questões de segurança.
        
        Ao fazer ingestão, acessaremos `./dados` via `/var/lib/clickhouse/user_files/` dentro do container.

    !!! warning "Atenção!"
        Observe que, com a definição do volume `clickhouse-data`, teremos persistência dos dados mesmo após a parada ou remoção do container.

        Confira os volumes com (só irá aparecer após você iniciar o container):

        <div class="termy">

        ```bash
        $ docker volume ls
        ```

        </div>

!!! exercise text long
    O que a seção **ulimits** está configurando?

    !!! answer "Resposta"
        O **ulimits** está configurando o número máximo de arquivos abertos que o **ClickHouse** pode ter, aumentando o limite padrão para 262144.

!!! exercise
    Crie uma pasta `clickhouse/dados` e copie os **CSVs** e **Parquets** criados na aula de [**DuckDB**](../03-data-warehouse/duckdb.md).

!!! exercise
    Inicie o container do **ClickHouse**:

    <div class="termy">

    ```bash
    $ docker compose up -d
    ```

    </div>

!!! exercise
    Verifique se o **ClickHouse** está rodando e se o volume foi criado:

    <div class="termy">

    ```bash
    $ docker logs clickhouse-server
    $ docker volume ls
    ```

    </div>

## Interface de acesso

O **ClickHouse** oferece múltiplas interfaces de acesso: HTTP, cliente nativo, e diversas linguagens de programação (Java, Python, Go, etc.).

### Cliente HTTP

!!! exercise
    Teste a conexão via HTTP:

    !!! info "Info!"
        Esta chamada retorna a versão do **ClickHouse**.

    <div class="termy">

    ```bash
    $ curl 'http://admin:password123@localhost:8123/' -d 'SELECT version()'
    ```

    </div>

    !!! failure "Não tenho `curl`"
        Caso não tenha **curl**, acesse a **URL** diretamente em seu navegador:

        [http://localhost:8123/?query=SELECT%20version()](http://localhost:8123/?query=SELECT%20version())

### Cliente CLI

!!! exercise
    Acesse o cliente **ClickHouse** dentro do container:

    <div class="termy">

    ```bash
    $ docker exec -it clickhouse-server clickhouse-client --user admin --password password123
    ```

    </div>

    Você verá um prompt `clickhouse-server :) ` onde pode executar comandos SQL.

!!! exercise
    Execute a seguinte *query* para ver informações do sistema:

    !!! danger "Atenção!"
        As *queries* (esta e as próximas) devem ser executadas no terminal que está conectado ao **Clickhouse**!

        Aquele que você iniciou com o comando:

        ```bash { .copy }
        docker exec -it clickhouse-server clickhouse-client --user admin --password password123
        ```

    *Query*:

    ```sql { .copy }
    SELECT 
        name,
        value
    FROM system.settings 
    WHERE name LIKE '%thread%'
    LIMIT 5;
    ```

## Obtendo os dados

Vamos utilizar os mesmos dados de bicicletas de São Francisco que usamos na aula do DuckDB.

## Criando databases e tabelas

### Database

!!! exercise
    No cliente **ClickHouse**, crie um database:

    ```sql { .copy }
    CREATE DATABASE IF NOT EXISTS bike_share;
    USE bike_share;
    ```

### Importando dados CSV

Vamos começar criando tabelas e importando os dados CSV.

!!! exercise
    Crie a tabela `station`:

    ```sql { .copy }
    CREATE TABLE station
    (
        id UInt32,
        name String,
        lat Float64,
        long Float64,
        dockcount UInt32,
        landmark String,
        installation String
    )
    ENGINE = MergeTree()
    ORDER BY id;
    ```

!!! exercise
    Importe os dados das estações:

    !!! danger "Atenção!"
        Este comando deve ser rodado em outro terminal **da sua máquina** e não no que está conectado como cliente.

        Ele supõe que seu diretório de trabalho atual é a pasta `clickhouse`.
    
    !!! warning "Path!"
        Antes de executar o comando, garanta que está no *path* correto, que é a raiz da aula!

        Nesta pasta, deve haver uma sub-pasta `dados`.

    <div class="termy">

    ```bash
    $ docker exec -i clickhouse-server clickhouse-client --user admin --password password123 --query="INSERT INTO bike_share.station FORMAT CSVWithNames" < dados/station.csv
    ```

    </div>

!!! exercise
    Verifique os dados importados:

    !!! warning "Atenção!"
        Copie, cole e execute os comandos abaixo, **um por vez**, no terminal que está conectado ao **Clickhouse**!

    ```sql { .copy }
    SHOW TABLES;
    SELECT COUNT(*) FROM station;
    SELECT * FROM station LIMIT 5;
    ```

!!! exercise
    Acesse [http://localhost:8123/dashboard](http://localhost:8123/dashboard) para monitorar o servidor!

    !!! warning "Atenção!"
        Informe o usuário e senha!

!!! exercise
    Crie a tabela `trip` com tipos de dados otimizados:

    ```sql { .copy }
    CREATE TABLE trip
    (
        id UInt64,
        duration UInt32,
        start_date DateTime,
        start_station_name String,
        start_station_id UInt32,
        end_date DateTime,
        end_station_name String,
        end_station_id UInt32,
        bike_id UInt32,
        subscription_type String,
        zip_code String
    )
    ENGINE = MergeTree()
    PARTITION BY toYYYYMM(start_date)
    ORDER BY (start_date, start_station_id);
    ```

!!! info "Particionamento"
    O **ClickHouse** permite particionamento eficiente dos dados. Aqui particionamos por mês da data de início.

!!! exercise
    Importe os dados das viagens. Para isso, no cliente **ClickHouse**, execute a query:

    ```sql { .copy }
    INSERT INTO trip
    SELECT 
        toUInt64(id) as id,
        toUInt32(duration) as duration,
        parseDateTimeBestEffort(start_date) as start_date,
        start_station_name,
        toUInt32(start_station_id) as start_station_id,
        parseDateTimeBestEffort(end_date) as end_date,
        end_station_name,
        toUInt32(end_station_id) as end_station_id,
        toUInt32(bike_id) as bike_id,
        subscription_type,
        zip_code
    FROM file('trip.csv', 'CSVWithNames');
    ```
    
    !!! info "Magic do **ClickHouse**"
        A função `parseDateTimeBestEffort()` consegue interpretar automaticamente formatos de data diversos, incluindo o formato americano MM/DD/YYYY HH:MM.

        <!-- Mas **cuidado**: esta abordagem não fará a conversão automática de datas. Você precisará tratar isso posteriormente. -->
        
    !!! tip "Caminho do arquivo"
        Note que agora usamos apenas `'trip.csv'` (sem caminho) porque o arquivo está mapeado diretamente no diretório `user_files`.

!!! exercise
    Verifique se os dados foram importados corretamente:

    ```sql { .copy }
    SELECT COUNT(*) FROM trip;
    SELECT * FROM trip LIMIT 5;
    ```

## Consultas básicas

### Análise exploratória

!!! exercise
    Execute consultas básicas para entender os dados:

    ```sql { .copy }
    -- Contagem total de viagens
    SELECT COUNT(*) as total_trips FROM trip;

    -- Duração média das viagens
    SELECT AVG(duration) as avg_duration_seconds FROM trip;

    -- Top 5 estações mais utilizadas como ponto de partida
    SELECT 
        start_station_name,
        COUNT(*) as trip_count
    FROM trip 
    GROUP BY start_station_name 
    ORDER BY trip_count DESC 
    LIMIT 5;
    ```

### Funções de data e tempo

O **ClickHouse** possui funções avançadas para manipulação de datas e tempo.

!!! exercise
    Analise padrões temporais:

    ```sql { .copy }
    -- Viagens por dia da semana
    SELECT 
        toDayOfWeek(start_date) as day_of_week,
        COUNT(*) as trip_count
    FROM trip 
    GROUP BY day_of_week 
    ORDER BY day_of_week;

    -- Viagens por hora do dia
    SELECT 
        toHour(start_date) as hour_of_day,
        COUNT(*) as trip_count
    FROM trip 
    GROUP BY hour_of_day 
    ORDER BY hour_of_day;
    ```

## Consultas analíticas avançadas

### Window Functions

O **ClickHouse** suporta window functions para análises sofisticadas.

!!! exercise
    Utilize window functions para análises avançadas:

    ```sql  { .copy }
    -- Ranking das estações mais populares por mês
    SELECT 
        toYYYYMM(start_date) as month,
        start_station_name,
        COUNT(*) as trip_count,
        ROW_NUMBER() OVER (PARTITION BY toYYYYMM(start_date) ORDER BY COUNT(*) DESC) as rank
    FROM trip 
    GROUP BY month, start_station_name 
    ORDER BY month, rank
    LIMIT 20;
    ```

!!! exercise
    Calcule médias móveis:

    ```sql { .copy }
    -- Média móvel de viagens por dia (janela de 7 dias)
    SELECT 
        date,
        daily_trips,
        AVG(daily_trips) OVER (ORDER BY date ROWS 6 PRECEDING) as moving_avg_7days
    FROM (
        SELECT 
            toDate(start_date) as date,
            COUNT(*) as daily_trips
        FROM trip 
        GROUP BY date
        ORDER BY date
    )
    ORDER BY date;
    ```

### Agregações complexas

!!! exercise
    Execute análises com múltiplas agregações:

    ```sql { .copy }
    -- Estatísticas por tipo de assinatura
    SELECT 
        subscription_type,
        COUNT(*) as trip_count,
        AVG(duration) as avg_duration,
        MIN(duration) as min_duration,
        MAX(duration) as max_duration,
        quantile(0.5)(duration) as median_duration,
        quantile(0.95)(duration) as p95_duration
    FROM trip 
    GROUP BY subscription_type;
    ```

## Trabalhando com formatos avançados

### Importando dados Parquet

O **ClickHouse** tem suporte nativo ao formato **Parquet**.

!!! exercise
    Crie uma tabela lendo diretamente do Parquet (com conversão de tipos):

    ```sql { .copy }
    CREATE TABLE trip_parquet
    (
        id String,
        duration Int32,
        start_date DateTime,
        start_station_name String,
        start_station_id Int32,
        end_date DateTime,
        end_station_name String,
        end_station_id Int32,
        bike_id String,
        subscription_type String,
        zip_code String
    )
    ENGINE = MergeTree
    PARTITION BY toYYYYMM(start_date)
    ORDER BY (start_date, start_station_id);

    -- Then insert the transformed data
    INSERT INTO trip_parquet
    SELECT
        id,
        duration,
        parseDateTimeBestEffort(start_date) AS start_date,
        start_station_name,
        start_station_id,
        parseDateTimeBestEffort(end_date) AS end_date,
        end_station_name,
        end_station_id,
        bike_id,
        subscription_type,
        zip_code
    FROM file('parquets/trip.parquet', 'Parquet');
    ```

    !!! info "Conversão de tipos"
        Como o **ClickHouse** pode interpretar colunas de data como String ao ler Parquet, usamos `parseDateTimeBestEffort()` para converter adequadamente os tipos de data tanto na partição quanto na ordenação.

!!! exercise
    Teste a leitura da tabela recém criada com:

    ```sql { .copy }
    SELECT *
    FROM trip_parquet
    LIMIT 10;
    ```

### Formatos de saída

!!! exercise
    O **ClickHouse** suporta múltiplos formatos de saída:

    ```sql { .copy }
    -- JSON
    SELECT * FROM station LIMIT 3 FORMAT JSON;

    -- CSV
    SELECT * FROM station LIMIT 3 FORMAT CSV;

    -- Pretty (formatado para humanos!)
    SELECT * FROM station LIMIT 3 FORMAT Pretty;
    ```

## Análises geoespaciais

O **ClickHouse** possui funções geoespaciais avançadas.

!!! exercise
    Faça análises baseadas em localização:

    ```sql { .copy }
    -- Junção para obter coordenadas das estações
    CREATE TABLE trip_with_coords
    ENGINE = MergeTree
    ORDER BY (start_station_id, end_station_id)
    AS SELECT 
        t.*,
        s1.lat as start_lat,
        s1.long as start_long,
        s2.lat as end_lat, 
        s2.long as end_long
    FROM trip t
    LEFT JOIN station s1 ON t.start_station_id = s1.id
    LEFT JOIN station s2 ON t.end_station_id = s2.id;
    ```

!!! exercise
    Calcule distâncias entre estações:

    ```sql { .copy }
    -- Distância aproximada entre estações (em metros)
    SELECT 
        start_station_name,
        end_station_name,
        COUNT(*) as trip_count,
        geoDistance(start_long, start_lat, end_long, end_lat) as distance_meters
    FROM trip_with_coords 
    WHERE start_lat IS NOT NULL AND end_lat IS NOT NULL
    GROUP BY start_station_name, end_station_name, start_lat, start_long, end_lat, end_long
    HAVING trip_count > 10
    ORDER BY distance_meters DESC
    LIMIT 10;
    ```

## Monitoramento e métricas

### System tables

O **ClickHouse** oferece tabelas de sistema para monitoramento.

!!! exercise
    Explore as tabelas de sistema:

    ```sql { .copy }
    -- Ver queries em execução
    SELECT * FROM system.processes;

    -- Estatísticas das tabelas
    SELECT 
        name,
        rows,
        bytes_on_disk,
        formatReadableSize(bytes_on_disk) as size
    FROM system.parts 
    WHERE database = 'bike_share';

    -- Performance das queries
    SELECT 
        query,
        read_rows,
        read_bytes,
        result_rows,
        memory_usage,
        query_duration_ms
    FROM system.query_log 
    WHERE type = 'QueryFinish'
    ORDER BY query_duration_ms DESC
    LIMIT 5;
    ```

## Integração com Python

!!! exercise
    Crie ou ative um ambiente virtual adequado com **Python 3.12**!

!!! exercise
    Instale o cliente Python para **ClickHouse**:

    ??? "Arquivo `requirements.txt`"
        ```text { .copy }
        clickhouse-connect==0.8.18
        pandas==2.3.1
        python-dotenv==1.0.1
        ```

    <div class="termy">

    ```bash
    $ uv pip install -r requirements.txt
    ```

    </div>

!!! exercise
    Crie um script `src/clickhouse_python.py` para interagir com **ClickHouse**:

    ??? "Arquivo `.env`"
        ```text 
        # ClickHouse Configuration
        CLICKHOUSE_HOST=localhost
        CLICKHOUSE_PORT=8123
        CLICKHOUSE_USERNAME=admin
        CLICKHOUSE_PASSWORD=password123
        CLICKHOUSE_DATABASE=bike_share
        ```

    ```python { .copy }
    import clickhouse_connect
    import pandas as pd
    import os
    from dotenv import load_dotenv

    # Carregar variáveis de ambiente do arquivo .env
    load_dotenv()

    # Conectar ao ClickHouse usando variáveis de ambiente
    client = clickhouse_connect.get_client(
        host=os.getenv('CLICKHOUSE_HOST', 'localhost'),
        port=int(os.getenv('CLICKHOUSE_PORT', 8123)),
        username=os.getenv('CLICKHOUSE_USERNAME', 'admin'), 
        password=os.getenv('CLICKHOUSE_PASSWORD'),
        database=os.getenv('CLICKHOUSE_DATABASE', 'bike_share')
    )

    # Executar consulta e obter DataFrame
    result = client.query_df("""
        SELECT 
            start_station_name,
            COUNT(*) as trip_count,
            AVG(duration) as avg_duration
        FROM trip 
        GROUP BY start_station_name 
        ORDER BY trip_count DESC 
        LIMIT 10
    """)

    print("Top 10 estações mais utilizadas:")
    print(result)

    # Inserir dados via DataFrame
    sample_data = pd.DataFrame({
        'id': [999, 1000],
        'name': ['Test Station 1', 'Test Station 2'], 
        'lat': [37.7849, 37.7849],
        'long': [-122.4194, -122.4094],
        'dockcount': [15, 20],
        'landmark': ['San Francisco', 'San Francisco'],
        'installation': ['2023-01-01', '2023-01-02']
    })

    client.insert_df('station', sample_data)
    print("Dados inseridos com sucesso!")
    ```

!!! exercise
    Rode o script com:

    <div class="termy">

    ```bash
    $ python src/clickhouse_python.py
    ```

    </div>

## Comparação de Performance

!!! exercise
    Execute um benchmark simples comparando diferentes tipos de consulta:

    ```sql { .copy }
    -- Consulta simples de agregação
    SELECT 
        subscription_type, 
        COUNT(*) 
    FROM trip 
    GROUP BY subscription_type;

    -- Consulta com filtro temporal
    SELECT COUNT(*) 
    FROM trip 
    WHERE start_date >= '2013-09-01' AND start_date < '2013-10-01';

    -- Consulta complexa com múltiplas agregações
    SELECT 
        toYYYYMM(start_date) as month,
        start_station_name,
        COUNT(*) as trips,
        AVG(duration) as avg_duration,
        quantile(0.95)(duration) as p95_duration
    FROM trip 
    GROUP BY month, start_station_name
    HAVING trips > 50
    ORDER BY month, trips DESC;
    ```

## Exercícios finais

!!! exercise text long
    Compare o **ClickHouse** com o **DuckDB** em relação aos casos de uso. Quando você usaria cada um?

    !!! answer "Resposta"
        O **ClickHouse** é otimizado para consultas analíticas em grandes volumes de dados, enquanto o **DuckDB** é mais adequado para análises em ambientes locais e em menor escala.
        
        Usaria o **ClickHouse** para análises em tempo real e grandes conjuntos de dados, enquanto o **DuckDB** seria ideal para protótipos rápidos e análises *ad-hoc*.

!!! exercise
    Crie uma análise que:

    1. Identifique os fluxos de viagens mais populares (origem → destino)
    1. Calcule estatísticas temporais (padrões por hora, dia da semana)
    1. Analise a utilização das bicicletas individuais
    1. Exporte os resultados em diferentes formatos

!!! exercise text long
    Quais são as principais vantagens do armazenamento columnar do ClickHouse para consultas analíticas?

!!! tip "Documentação"
    Consulte a [documentação oficial](https://clickhouse.com/docs/) do **ClickHouse** para mais recursos avançados!

## Substituindo PostgreSQL por ClickHouse no Data Warehouse

Agora que você domina os conceitos básicos do **ClickHouse**, vamos aplicá-lo em um cenário prático: substituir o **PostgreSQL** do Data Warehouse da [aula anterior (DW Parte I)](../03-data-warehouse/pratica.md) pelo **ClickHouse**.

### Preparando o ambiente

!!! exercise
    Navegue até o diretório da aula anterior onde você implementou o pipeline ETL.

    Crie uma cópia da pasta `02-etl`.

    <div class="termy">

    ```bash
    $ cp -r /caminho/para/dataeng-03-repo-base/02-etl /caminho/para/dataeng-03-repo-base/02-etl-clickhouse
    ```

    </div>

    !!! warning "Atenção!"
        Certifique-se de que todos os serviços estejam parados antes de iniciar a migração.

### Modificando a arquitetura

!!! exercise
    Edite o arquivo `02-etl-clickhouse/docker-compose.yml` para substituir o **PostgreSQL** pelo **ClickHouse** como Data Warehouse.

### Criando o schema ClickHouse

!!! exercise
    Crie um novo arquivo `02-etl/sql/0001-ddl-clickhouse.sql` com o schema otimizado para **ClickHouse**.

!!! exercise text long
    Quais otimizações foram aplicadas por você no schema do **ClickHouse** em comparação com o **PostgreSQL**?

    !!! answer "Resposta"
        Otimizações que poderiam ser aplicadas (pesquise como fazer):

        1. **LowCardinality**: Para campos com poucos valores únicos (nome, categoria, uf, status)
        2. **Particionamento**: Tabelas particionadas por mês usando `toYYYYMM()`
        3. **Tipos específicos**: UInt32/UInt64 para IDs, Decimal para valores monetários
        4. **Ordenação otimizada**: ORDER BY considera padrões de consulta (data, categoria, etc.)
        5. **Remoção de constraints**: Sem chaves estrangeiras ou checks para melhor performance de inserção

### Adaptando o script de inicialização

!!! exercise
    Crie um novo arquivo `02-etl/src/init_clickhouse.py` para inicializar o **ClickHouse**.

### Modificando o pipeline ETL

!!! exercise
    Crie um novo arquivo `02-etl/src/etl_vendas_clickhouse.py` e faça o processo de **ETL**.

!!! exercise
    Atualize o arquivo `02-etl/requirements.txt` para incluir o cliente **ClickHouse**.

### Testando a nova arquitetura

!!! exercise
    Pare os serviços anteriores (se estiverem rodando):

    !!! warning "Atenção!"
        O `-v` no comando abaixo irá garantir que os volumes criados sejam apagados.

        Utilize isto quando quiser apagar dados persistidos em volumes do docker.

    <div class="termy">

    ```bash
    $ docker compose down -v
    ```

    </div>

!!! exercise
    Construa e inicie os novos serviços:

    <div class="termy">

    ```bash
    $ docker compose build
    $ docker compose up
    ```

    </div>

!!! exercise
    Verifique os logs para confirmar que:

    1. O simulador de vendas está gerando dados no PostgreSQL
    1. O ETL está extraindo dados do PostgreSQL e carregando no **ClickHouse**
    1. Não há erros de conexão ou inserção

### Validação e análises

!!! exercise
    Conecte-se ao **ClickHouse** via cliente **CLI** para validar os dados:

    <div class="termy">

    ```bash
    $ docker exec -it clickhouse-dw clickhouse-client --user dw_user --password dw_pass --database vendas_dw
    ```

    </div>

!!! exercise
    Execute consultas de validação no **ClickHouse**:

    ```sql { .copy }
    -- Verificar contagem de registros por tabela
    SELECT 'cidade' as tabela, count() as total FROM cidade
    UNION ALL
    SELECT 'cliente', count() FROM cliente
    UNION ALL  
    SELECT 'produto', count() FROM produto
    UNION ALL
    SELECT 'venda', count() FROM venda
    UNION ALL
    SELECT 'item_venda', count() FROM item_venda;

    -- Análise temporal das vendas
    SELECT 
        toDate(data) as data,
        count(*) as total_vendas,
        sum(valor_total) as receita_total
    FROM venda
    GROUP BY data 
    ORDER BY data DESC
    LIMIT 10;
    ```

### Exercícios de comparação

!!! exercise text long
    Compare o desempenho do **ClickHouse** com o **PostgreSQL** para consultas analíticas. Quais diferenças você observa?

!!! exercise text long
    Quais são as principais vantagens de usar **ClickHouse** como Data Warehouse em comparação com **PostgreSQL**?

!!! exercise text long
    Em que cenários você ainda preferiria usar **PostgreSQL** em vez de **ClickHouse**?

!!! exercise
    Modifique o *dashboard* (**Jupyter Notebook**) para conectar-se ao **ClickHouse** em vez do **PostgreSQL** e compare os tempos de resposta das consultas.

!!! exercise
    Implemente um mecanismo de monitoramento que compare métricas entre os dois sistemas:
    1. Tempo de resposta das consultas
    2. Uso de CPU e memória
    3. Espaço em disco utilizado
    4. Throughput de inserção de dados


## Limpeza

!!! exercise
    Para parar e remover os *containers* e volumes:

    !!! warning
        O `-v` no comando abaixo irá garantir que os volumes criados sejam apagados.

        Utilize isto quando quiser apagar dados persistidos em volumes do docker.

    <div class="termy">

    !!! danger "Limpeza!"
        Lembre-se que você criou/alterou dois conjuntos de serviços nesta aula!

    ```bash
    $ docker compose down -v
    ```

    </div>
