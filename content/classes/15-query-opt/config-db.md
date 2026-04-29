# Configurar Ambiente

Vamos criar o ambiente necessário para a aula. Precisamos de um servidor **PostgreSQL** e um cliente **Python** para configurar a base.

Faremos esta configuração com o uso do **Docker**.

!!! exercise "Exercício"
    Crie uma nova pasta para esta aula e navegue até ela:

    <div class="termy">

    ```
    $ mkdir -p 15-query-opt/data
    ```

    </div>

!!! exercise "Exercício"
    Copie para a pasta `15-query-opt/data` os arquivos **CSV** da base de dados [Kaggle SF Bay Area Bike Share](https://www.kaggle.com/datasets/benhamner/sf-bay-area-bike-share).

    !!! info "Info"
        Os arquivos são os mesmos da aula 01 e da [aula anterior (Databricks)](../13-databricks/databricks-etl.md).

        Você pode baixá-los no [Kaggle SF Bay Area Bike Share](https://www.kaggle.com/datasets/benhamner/sf-bay-area-bike-share).

    Você terá a seguinte estrutura:

    ```
    15-query-opt/
    └── data
        ├── trip.csv
        ├── station.csv
        ├── weather.csv
        └── status.csv
    ```

!!! exercise "Exercício"
    Acesse a pasta `15-query-opt` e clone o repositório com os arquivos base para a aula:

    <div class="termy">

    ```
    $ git clone https://github.com/macielcalebe/dataeng-query-opt-base.git services
    ```

    </div>

## Configurar Variáveis de Ambiente

Vamos configurar as variáveis de ambiente para o **PostgreSQL**.

!!! exercise "Exercício"
    Crie um arquivo `.env` a partir do arquivo `services/.env.example` e altere as variáveis de ambiente.

    **OBS**: apenas a senha é de alteração obrigatória.

## Iniciar Serviços

!!! exercise "Exercício"
    Inicie os serviços com o **Docker Compose**:

    <div class="termy">

    ```
    $ docker compose up
    ```

    </div>

    !!! warning "Atenção!"
        Este processo pode levar alguns minutos, dependendo do desempenho da sua máquina.

    Após o **PostgreSQL** estar totalmente operacional, um script será executado para criar a base de dados e popular as tabelas.

    Enquanto o processo não termina, siga para os próximos exercícios.

!!! exercise text long "Exercício"
    Abra, no **VSCode**, o arquivo `15-query-opt/services/sql/001-ddl.sql`.

    O que ele faz?

    !!! answer "Resposta"
        O arquivo `001-ddl.sql` contém comandos **SQL** para criar a estrutura do banco de dados, incluindo a criação de tabelas, definição de colunas, tipos de dados e restrições. Ele define o esquema inicial necessário para armazenar os dados que serão inseridos posteriormente.

!!! exercise text long "Exercício"
    Abra, no **VSCode**, o arquivo `15-query-opt/services/sql/002-load-base.sql`.

    1. O que ele faz?
    1. Qual a função do comando `COPY`?
    1. O comando `COPY` executa no cliente (Python) ou no servidor (PostgreSQL)?

    !!! answer "Resposta"
        O arquivo `002-load-base.sql` contém comandos **SQL** para carregar dados nas tabelas do banco de dados a partir de arquivos CSV. Ele utiliza o comando `COPY` para importar os dados de forma eficiente.

        A função do comando `COPY` é transferir dados entre um arquivo e uma tabela do banco de dados. Ele é usado para carregar grandes volumes de dados rapidamente, evitando a necessidade de inserir linha por linha.

        O comando `COPY` é executado no servidor (PostgreSQL), o que permite que ele acesse diretamente os arquivos no sistema de arquivos do servidor, resultando em uma operação mais rápida e eficiente.

        O servidor **PostgreSQL** precisa ter acesso aos arquivos CSV para que o comando `COPY` funcione corretamente. Isto foi configurado no `docker-compose.yml`, onde o diretório `../data` foi montado no contêiner do **PostgreSQL**.

!!! exercise "Exercício"
    Vamos criar uma conexão no **DBeaver** para o banco de dados **PostgreSQL**.

    Abra o **DBeaver** e crie uma nova conexão com o banco de dados PostgreSQL da aplicação de vendas utilizando as informações configuradas no `.env`.

    !!! info "Importante!"
        Durante a criação da conexão, marque a opção **Show all databases** para visualizar todos os bancos de dados disponíveis.

!!! warning "Atenção!"
    Antes de prosseguir, verifique se o script de criação e carga da base terminou de ser executado. **Aguarde até o final do processo!**

    Você deve ver uma mensagem `"Inicialização do banco de dados concluída com sucesso"` no terminal do **Docker Compose**.
