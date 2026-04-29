# API REST

Quando processos (programas em execução) precisam trocar informações em uma rede, existem diversas arquiteturas para organizar essa comunicação. O modelo mais comum envolve dois papeis: **clientes** e **servidores**.

Os servidores disponibilizam uma **Interface de Programação de Aplicações (API)**, que pode ser acessada pela rede pelos clientes.

!!! info "Info!"
    Essencialmente, a **API** que o servidor expõe é o que se define como o **serviço** em questão.

O **FastAPI** é um *framework* muito utilizado para construção de **APIs** web utilizando **Python**. Você já trabalhou com **FastAPI** na disciplina de **Megadados** e agora iremos considerar mais detalhes sobre esse *framework*.

!!! tip "Dica"
    O **FastAPI** facilita a criação desses *endpoints* **RESTful**.

    **REST** é um estilo de arquitetura que define um conjunto de restrições para a criação de serviços web, sendo as mais notáveis o uso de métodos HTTP (como `GET`, `POST`, `PUT`, `DELETE`) e a manipulação de **recursos** (dados) por meio de URLs.

## Configurar Ambiente

Vamos configurar o ambiente necessário para a aula. Utilizaremos a mesma pasta da aula anterior.

!!! exercise "Exercício"
    Navegue até a pasta `15-query-opt` da aula anterior.
    
    Então, clone o repositório com os arquivos base para a aula:

    <div class="termy">

    ```
    $ cd 15-query-opt
    $ git clone https://github.com/macielcalebe/dataeng-sfbikeshare-api.git api
    ```

    </div>

## Configurar Variáveis de Ambiente

Vamos configurar as variáveis de ambiente para o **PostgreSQL**.

!!! exercise "Exercício"
    Crie um arquivo `.env` a partir do arquivo `api/.env.example` e altere as variáveis de ambiente.

    **OBS**: apenas as senhas são de alteração obrigatória.

## Iniciar Serviços

!!! exercise "Exercício"
    Inicie os serviços com o **Docker Compose**:

    !!! danger "Atenção!"
        Garanta que você está na pasta `15-query-opt/api` antes de executar o comando abaixo.

    <div class="termy">

    ```
    $ docker compose up
    ```

    </div>

    !!! info "Info!"
        Este processo pode levar alguns minutos para executar.

## Testar API

Após iniciar os serviços, a API estará disponível no endereço `http://localhost:8000/`.

!!! exercise "Exercício"
    Acesse a documentação interativa da API no endereço [http://localhost:8000/docs](http://localhost:8000/docs).

    Explore os *endpoints* disponíveis e teste alguns deles diretamente na interface.

!!! exercise text long "Exercício"
    No **Swagger UI**, utilize o *endpoint* `GET /api/v1/status/` para listar os status.

    Para que servem os parâmetros `skip` e `limit`?

    !!! answer "Resposta"
        Os parâmetros `skip` e `limit` são utilizados para **paginação** dos resultados retornados pela API.

        - `skip`: Indica o número de registros a serem ignorados no início da lista. Isso é útil para pular um certo número de resultados, permitindo que você acesse páginas subsequentes de dados.
        
        - `limit`: Define o número máximo de registros a serem retornados na resposta. Isso ajuda a controlar a quantidade de dados recebidos em uma única solicitação, evitando sobrecarga e melhorando o desempenho.

        Juntos, esses parâmetros permitem que os clientes da API naveguem eficientemente através de grandes conjuntos de dados, solicitando apenas as partes necessárias em cada chamada.

!!! exercise text long "Exercício"
    Considerando o *endpoint* `GET /api/v1/status/`, como seria a URL para listar os status da quarta página, considerando que cada página deve conter 100 status?

    !!! answer "Resposta"
        Para listar os status da quarta página, considerando que cada página contém 100 status, você deve calcular o valor do parâmetro `skip` como `(página - 1) * tamanho_da_página`. 

        Portanto, para a quarta página:
        
        - `skip = (4 - 1) * 100 = 300`
        - `limit = 100`

        A URL completa seria:

        ```
        http://localhost:8000/api/v1/status/?skip=300&limit=100
        ```

## Testes com `ab`

O **Apache Benchmark (ab)** é uma ferramenta útil para realizar testes de carga em servidores web.

Vamos utilizá-la para testar o desempenho do nosso *endpoint* `GET /api/v1/status/`.


!!! exercise "Exercício"
    Abra um novo terminal e execute o seguinte comando para realizar 1000 requisições com uma concorrência de 10:

    <div class="termy">

    ```
    $ docker run --rm --network host jordi/ab -n 1000 -c 10 http://127.0.0.1:8000/api/v1/status/?skip=0&limit=100
    ```

    </div>

    Analise os resultados fornecidos pelo **ab**, como o tempo médio por requisição e a quantidade de requisições por segundo.

    !!! tip "Dica"
        Você pode ajustar os parâmetros `-n` (número total de requisições) e `-c` (número de requisições concorrentes) conforme necessário para simular diferentes cargas no servidor.

!!! info "Documentação"
    Para mais informações sobre o **Apache Benchmark**, consulte a [documentação oficial](https://httpd.apache.org/docs/2.4/programs/ab.html).


!!! exercise text long "Exercício"
    Agora teste variações do teste anterior:

    - Pulando as primeiras `100000` (cem mil) linhas
    - Pulando as primeiras `1000000` (um milhão) de linhas

    <div class="termy">

    ```
    $ docker run --rm --network host jordi/ab -n 1000 -c 10 http://127.0.0.1:8000/api/v1/status/?skip=100000&limit=100
    $ docker run --rm --network host jordi/ab -n 1000 -c 10 http://127.0.0.1:8000/api/v1/status/?skip=1000000&limit=100
    ```

    </div>

    O que aconteceu?
    
    Repita o teste no **Swagger UI** (duas ou três vezes) fazendo `skip` nas primeiras `50000000` (cinquenta milhões) de linhas. O que aconteceu?

Vamos tentar entender o que aconteceu.

!!! exercise "Exercício"
    Altere o código da **API** no arquivo `app/core/database.py`, na criação do `engine` do **SQLAlchemy**, alterando o parâmetro `echo` para `True`.

!!! exercise "Exercício"
    Considerando o *endpoint* `GET /api/v1/status/`, faça novamente o teste no **Swagger UI** fazendo `skip` nas primeiras `1000000` (um milhão) de linhas.

    Copie a *query* SQL que foi exibida no terminal onde a **API** está rodando.

!!! exercise text long "Exercício"
    Acesse o **DBeaver** e conecte-se ao banco de dados do **PostgreSQL**.

    Abra o **Query Tool** e execute a query recém copiada:

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT
        status.station_id AS status_station_id,
        status.bikes_available AS status_bikes_available,
        status.docks_available AS status_docks_available,
        status.time AS status_time,
        status.category1 AS status_category1,
        status.category2 AS status_category2
    FROM
        status
    LIMIT 100 OFFSET 50000000;
    ```

    O que está acontecendo?

    !!! answer "Resposta"
        A consulta SQL está utilizando o comando `LIMIT` para limitar o número de registros retornados a 100, e o comando `OFFSET` para pular os primeiros 50.000.000 registros.

        O problema com essa abordagem é que o banco de dados ainda precisa processar todos os registros até o ponto do `OFFSET`, o que pode ser muito ineficiente, especialmente quando o valor do `OFFSET` é grande. Isso pode resultar em tempos de resposta lentos, pois o banco de dados tem que percorrer uma grande quantidade de dados antes de retornar os resultados desejados.

## Alternativa ao uso de OFFSET

Uma alternativa mais eficiente seria utilizar um critério de ordenação e um marcador (como um ID ou timestamp) para buscar os registros a partir de um ponto específico, evitando assim a necessidade de pular um grande número de registros.

Este conceito é conhecido como **paginação baseada em cursor** (*cursor-based pagination* ou *cursor-based sorting*) e pode melhorar significativamente o desempenho das consultas em grandes conjuntos de dados.

Suponha que queremos paginar os status com base no campo `time`. Ao invés de buscarmos a décima página utilizando `OFFSET`, poderíamos buscar os status onde o campo `time` é maior que o último valor de `time` da página anterior (nona página).

Considere o seguinte exemplo:

=== "Com OFFSET"

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT
        status.station_id AS status_station_id,
        status.bikes_available AS status_bikes_available,
        status.docks_available AS status_docks_available,
        status.time AS status_time,
        status.category1 AS status_category1,
        status.category2 AS status_category2
    FROM
        status
    LIMIT 100 OFFSET 50000000;
    ```

=== "Com cursor"

    ```sql { .copy }
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT
        status.station_id AS status_station_id,
        status.bikes_available AS status_bikes_available,
        status.docks_available AS status_docks_available,
        status.time AS status_time,
        status.category1 AS status_category1,
        status.category2 AS status_category2
    FROM
        status
    WHERE time > '2015-01-01 10:00:00.000'
    ORDER BY time
    LIMIT 100
    ```

!!! exercise "Exercício"
    No **DBeaver**, execute a query com cursor acima, substituindo o valor `'2015-01-01 10:00:00.000'` por um valor adequado que represente o último `time` da página anterior.

    Compare o plano de execução e o tempo de resposta com a consulta que utiliza `OFFSET`.

    !!! tip "Dica"
        Utilize a seguinte *query* para encontrar um valor adequado para o campo `time`:

        ```sql { .copy }
        SELECT MIN(time), MAX(time)
        FROM status;
        ```

    O que você observa?

    !!! answer "Resposta"
        A consulta utilizando o cursor geralmente apresenta um plano de execução mais eficiente e um tempo de resposta significativamente menor em comparação com a consulta que utiliza `OFFSET`.

        Isso ocorre porque a consulta com cursor evita a necessidade de percorrer todos os registros até o ponto do `OFFSET`, resultando em menos leituras de dados e melhor desempenho geral.

**Vantagens da paginação baseada em cursor:**

- ✅ Performance constante
- ✅ Escalável para milhões de registros
- ✅ Menor uso de recursos do banco

**Desvantagens da paginação baseada em cursor:**

- ❌ Não permite pular páginas diretamente
- ❌ Cliente precisa manter o último cursor

!!! exercise text long "Exercício"
    Teste a requisição com diferentes variações para o valor do cursor (campo `time`).

    Você confirma que a performance é constante? Como isto é possível? Como a ordenação é realizada de forma tão rápida?

    !!! answer "Resposta"
        Sim, a performance permanece constante independentemente do valor do cursor utilizado.

        Isso é possível porque a consulta com cursor utiliza um critério de ordenação e um marcador para buscar os registros a partir de um ponto específico, evitando a necessidade de pular um grande número de registros.

        Como resultado, o banco de dados não precisa processar todos os registros anteriores ao cursor, o que mantém o tempo de resposta estável mesmo quando navegamos por grandes conjuntos de dados.

        !!! info "Info!"
            A performance constante é alcançada porque a consulta com cursor se beneficia do índice **BTree** no atributo `time`, permitindo acesso direto aos registros relevantes sem a sobrecarga de percorrer registros desnecessários.

        !!! tip "WooooW"
            Árvores **BTree** são naturalmente ordenadas, o que as torna ideais para operações de busca e ordenação em bancos de dados relacionais.

### Paginação por Cursor na API

!!! exercise "Exercício"
    Altere o código da **API** no arquivo `app/routes/status.py` para implementar a paginação por cursor no *endpoint* `GET /api/v1/status/`.

    Utilize o campo `time` como cursor para buscar os status a partir de um ponto específico.

    Atualize também o *endpoint* na documentação interativa da API.

    ??? "Cabeçalho"
        A assinatura da função deve ser alterada para:

        ```python
        @router.get("/", response_model=List[Status], summary="Get all status records")
        def get_status_records(
            limit: int = Query(100, ge=1, le=1000, description="Maximum number of records to return"),
            cursor: datetime = Query(None, description="Cursor for pagination (time field)"),
            station_id: int = Query(None, description="Filter by station ID"),
            db: Session = Depends(get_db)
        ):
        ```

    !!! answer "Resposta"
        ```python
        @router.get("/", response_model=List[Status], summary="Get all status records")
        def get_status_records(
            limit: int = Query(100, ge=1, le=1000, description="Maximum number of records to return"),
            cursor: datetime = Query(None, description="Cursor for pagination (time field)"),
            station_id: int = Query(None, description="Filter by station ID"),
            db: Session = Depends(get_db)
        ):
            """
            Retrieve all status records with cursor-based pagination and optional filtering.
            
            - **limit**: Maximum number of records to return (max 1000)
            - **cursor**: Cursor for pagination based on time field (optional, returns records after this timestamp)
            - **station_id**: Filter by specific station ID (optional)
            """
            query = db.query(StatusModel)
            
            if station_id is not None:
                query = query.filter(StatusModel.station_id == station_id)
            
            if cursor is not None:
                query = query.filter(StatusModel.time > cursor)
            
            status_records = query.order_by(StatusModel.time).limit(limit).all()
            return status_records
        ```

!!! exercise "Exercício"
    Teste o *endpoint* `GET /api/v1/status/` na documentação interativa da API utilizando a paginação por cursor.

    Verifique se os resultados estão corretos e se a performance é satisfatória.

    !!! tip "Dica"
        Utilize o campo `time` do último registro retornado como cursor para a próxima requisição.

    !!! answer "Resposta"
        Após implementar a paginação por cursor, ao testar o *endpoint* `GET /api/v1/status/`, os resultados retornados estão corretos e a performance é significativamente melhorada em comparação com a abordagem anterior utilizando `OFFSET`.

        A utilização do campo `time` como cursor permite que as requisições subsequentes sejam eficientes, mantendo um tempo de resposta constante mesmo ao navegar por grandes conjuntos de dados.

!!! info "Onde se aplica?"
    Este tipo de paginação é frequentemente utilizado em páginas com *scroll infinito* (infinite scroll), onde novos dados são carregados conforme o usuário rola a página para baixo, proporcionando uma experiência de usuário fluida e eficiente.
