# Trino

## Visão Geral

O **Trino** (anteriormente conhecido como PrestoSQL) é um mecanismo de consulta **SQL** distribuído e de alto desempenho, projetado para consultar dados onde quer que eles estejam armazenados.

!!! tip "Trino"
    O **Trino** é uma das ferramentas essenciais em arquiteturas de **Data Lakehouse**

Ele permite que você execute consultas analíticas complexas em grandes volumes de dados, sejam eles armazenados em **data lakes**, **data warehouses** ou outros sistemas de armazenamento, sem a necessidade de mover os dados.

O Trino oferece suporte a uma ampla gama de fontes de dados por meio de conectores, permitindo que uma única camada de consulta acesse tanto sistemas de armazenamento quanto bancos de dados. Entre as opções estão object storages como S3, GCS e Azure Blob, além de HDFS, bancos relacionais como PostgreSQL, MySQL, Oracle e SQL Server, e ainda bancos NoSQL, como MongoDB, Cassandra e Elasticsearch. Essa versatilidade torna possível integrar diferentes tecnologias em uma mesma arquitetura de dados.

Além disso, o Trino lida com múltiplos formatos de dados, como Parquet, ORC, Avro, JSON e CSV, garantindo flexibilidade na análise de dados estruturados e semiestruturados. Também se integra a sistemas especializados, como Apache Iceberg, Delta Lake e Apache Hudi para gerenciamento de tabelas em data lakes, bem como a data warehouses em nuvem, como BigQuery e Snowflake, permitindo cenários híbridos e distribuídos de análise em larga escala.

!!! tip "Curiosidade!"
    O **AWS Athena** é baseado no **Trino**!

## Configuração

Vamos atualizar o `docker-compose.yml` que utilizamos para o Iceberg, adicionando o serviço do Trino:

```yaml { .copy }
  trino:
    image: trinodb/trino:latest
    container_name: trino
    depends_on:
      - rest
      - master
      - volume
      - filer
    networks:
      - iceberg_net
    ports:
      - 8082:8080
    volumes:
      - ./catalog:/etc/trino/catalog
```

Precisamos também configurar o **Trino** para se conectar ao **SeaweedFS** e ao **Iceberg**. Para isso, crie a pasta `catalog` no mesmo diretório do `docker-compose.yml` e crie o arquivo `catalog/iceberg.properties` contendo as informações de acesso à API REST do Iceberg e ao SeaweedFS:

```properties { .copy }
connector.name=iceberg
iceberg.catalog.type=rest
iceberg.rest-catalog.uri=http://rest:8181
iceberg.file-format=parquet

fs.native-s3.enabled=true
s3.endpoint=http://filer:8333
s3.path-style-access=true
s3.aws-access-key=admin
s3.aws-secret-key=password
s3.region=us-east-1
```

!!! exercise
    Atualize o arquivo `docker-compose.yml` que você criou na seção do Iceberg, adicionando o serviço do Trino.

    Inicialize todos os serviços.

!!! exercise
    Acesse [http://localhost:8082](http://localhost:8082) para verificar se o Trino está rodando.

    Utilize um usuário qualquer, como `admin`!

Agora, vamos utilizar o **Trino** para consultar as tabelas que criamos no **Iceberg**. Iremos fazer isto utilizando o **DBeaver**.

## DBeaver

Acesse o **DBeaver** e crie uma nova conexão com o banco de dados **Trino**.

!!! exercise
    Crie uma nova conexão com o banco de dados **Trino** no **DBeaver**.

    Utilize as seguintes configurações:

    - Host: `localhost`
    - Port: `8082`
    - User name: `admin`

    Teste a conexão e salve.

!!! exercise
    Crie uma nota aba de **SQL script** e execute algumas *queries* nas tabelas que você criou no Iceberg!

    ```sql { .copy }
    SELECT *
    FROM iceberg.nyc.taxis
    LIMIT 5;
    ```

    ```sql
    SELECT avg(passenger_count) AS average_passenger_count
    FROM iceberg.nyc.taxis;
    ```

!!! exercise
    Acesse [http://localhost:8082](http://localhost:8082) para verificar o *status* do Trino.

    Você consegue ver alguma informação sobre as consultas que você fez?!

!!! exercise
    Qual a distância média das corridas para cada `pulocationid`? Crie uma *query* para responder esta pergunta.

!!! tip "Dica!"
    Tente acessar as views que você criou pelo notebook. Você irá perceber que elas não são acessíveis pelo Trino.

    Crie outras views utilizando o Trino. Agora funciona?! Olha só que legal, você tem dados armazenados em um **Parquet** no **SeaweedFS**, e pode criar *views* para estes dados!