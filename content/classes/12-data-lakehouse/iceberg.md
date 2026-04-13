# Apache Iceberg

## Introdução

O **Apache Iceberg** é uma tecnologia de gerenciamento de tabelas de código aberto, que surgiu como uma das principais opções para implementar a camada de software de armazenamento transacional em arquiteturas **Data Lakehouse**. 

Desenvolvido inicialmente pela **Netflix** e posteriormente doado para a **Apache Software Foundation**, o **Iceberg** é uma alternativa ao **Delta Lake** e **Apache Hudi**, oferecendo características muito semelhantes para aprimorar as funcionalidades de um **data lake** tradicional.

## Principais Características

As principais características e funcionalidades do **Apache Iceberg** incluem:

- **Tecnologia de Gerenciamento de Tabelas**: O **Iceberg** é, fundamentalmente, uma tecnologia que gerencia tabelas, assim como o **Apache Hudi**, proporcionando uma interface consistente para operações de dados.

- **Camada de Armazenamento Transacional**: Ele atua como uma camada de software transacional que executa sobre um **data lake** existente, adicionando recursos semelhantes aos de um banco de dados relacional (RDW - Relational Data Warehouse), o que melhora significativamente a confiabilidade, segurança e desempenho do **data lake**.

- **Rastreamento de Arquivos**: O **Iceberg** pode rastrear todos os arquivos que compõem uma tabela, mantendo um controle preciso sobre a estrutura e organização dos dados.

- **Viagem no Tempo (Time Travel)**: Uma das funcionalidades mais poderosas do **Iceberg** é sua capacidade de rastrear arquivos em cada "snapshot" (instantâneo) da tabela ao longo do tempo, permitindo a funcionalidade de **"viagem no tempo"** (time travel) em um ambiente de **data lake**. Isso possibilita consultar dados em estados anteriores e recuperar versões específicas.

- **Evolução de Esquema**: Suporta a evolução de esquema de forma seamless, o que significa que o esquema dos dados pode ser alterado ao longo do tempo sem quebrar consultas existentes, proporcionando flexibilidade para mudanças nos requisitos de negócio.

- **Escalabilidade**: Projetado para gerenciar tabelas em escala de **petabytes**, atendendo às demandas de grandes volumes de dados empresariais.

- **Serialização Híbrida**: É um exemplo de tecnologia de serialização híbrida, que combina múltiplas técnicas de serialização ou integra a serialização com camadas de abstração adicionais, como o gerenciamento de esquema, otimizando o desempenho e a eficiência.

Em suma, o **Apache Iceberg** é uma solução essencial para transformar um **data lake** tradicional em um **Data Lakehouse** robusto, conferindo-lhe recursos de gerenciamento e confiabilidade que antes eram exclusivos dos **data warehouses** relacionais.

Com o **Iceberg**, organizações podem:
- Manter a flexibilidade e economia de um **data lake**
- Obter a confiabilidade e performance de um **data warehouse**
- Implementar operações **ACID** em ambientes distribuídos
- Facilitar a governança e auditoria de dados

!!! tip "Leitura recomendada"
    [Why did Databricks Acquire Tabular](https://www.dqlabs.ai/blog/why-did-databricks-acquire-tabular/)

Vamos praticar com o Apache Iceberg.

## Preparar ambiente

!!! exercise
    Crie uma pasta `12-data-lakehouse/02-iceberg` em seu diretório de trabalho e navegue até ela:

    <div class="termy">

    ```
    $ mkdir -p 12-data-lakehouse/02-iceberg
    $ cd 12-data-lakehouse/02-iceberg
    ```

    </div>

## Docker compose

Para esta seção da aula, o arquivo `docker-compose.yml` deve conter:

```yaml { .copy }
services:
  spark-iceberg:
    image: tabulario/spark-iceberg
    container_name: spark-iceberg
    build: spark/
    networks:
      iceberg_net:
    depends_on:
      - rest
      - filer
    volumes:
      - ./warehouse:/home/iceberg/warehouse
      - ./notebooks:/home/iceberg/notebooks/notebooks
      - ./spark/conf/spark-defaults.conf:/opt/spark/conf/spark-defaults.conf:ro
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
      - PYICEBERG_CATALOG__DEFAULT__URI=http://rest:8181
      - PYICEBERG_CATALOG__DEFAULT__S3__ENDPOINT=http://filer:8333
      - PYICEBERG_CATALOG__DEFAULT__S3__PATH_STYLE_ACCESS=true
      - PYICEBERG_CATALOG__DEFAULT__S3__ACCESS_KEY_ID=admin
      - PYICEBERG_CATALOG__DEFAULT__S3__SECRET_ACCESS_KEY=password
      - PYICEBERG_CATALOG__DEFAULT__S3__REGION=us-east-1
    ports:
      - 8888:8888
      - 8091:8080
      - 10000:10000
      - 10001:10001
  rest:
    image: apache/iceberg-rest-fixture:1.10.1
    container_name: iceberg-rest
    networks:
      iceberg_net:
    ports:
      - 8181:8181
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
      - CATALOG_WAREHOUSE=s3://warehouse/
      - CATALOG_IO__IMPL=org.apache.iceberg.aws.s3.S3FileIO
      - CATALOG_S3_ENDPOINT=http://filer:8333
      - CATALOG_S3_PATH_STYLE_ACCESS=true
      - CATALOG_S3_PATH__STYLE__ACCESS=true
      - CATALOG_S3FILEIO_PATH_STYLE_ACCESS=true
  master:
    image: chrislusf/seaweedfs:4.19
    container_name: seaweedfs-master
    networks:
      iceberg_net:
    ports:
      - 9333:9333
    volumes:
      - seaweedfs_master_data:/data
    command: "master -ip=master -mdir=/data -volumeSizeLimitMB=20480"
  volume:
    image: chrislusf/seaweedfs:4.19
    container_name: seaweedfs-volume
    networks:
      iceberg_net:
    ports:
      - 8090:8080
    volumes:
      - seaweedfs_volume_data:/data
    command: "volume -mserver=master:9333 -dir=/data -port=8080"
    depends_on:
      - master
  filer:
    image: chrislusf/seaweedfs:4.19
    container_name: seaweedfs-filer
    networks:
      iceberg_net:
    ports:
      - 8889:8888
      - 8333:8333
    volumes:
      - ./seaweedfs-s3.json:/etc/seaweedfs/s3.json:ro
      - seaweedfs_filer_data:/data
    command: "filer -master=master:9333 -s3 -s3.config=/etc/seaweedfs/s3.json -defaultStoreDir=/data"
    depends_on:
      - master
      - volume
  seaweedfs-init:
    depends_on:
      - filer
    image: amazon/aws-cli
    container_name: seaweedfs-init
    networks:
      iceberg_net:
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_DEFAULT_REGION=us-east-1
    entrypoint: |
      /bin/sh -c "
      until (aws --endpoint-url http://filer:8333 s3 ls) do echo '...waiting...' && sleep 1; done;
      aws --endpoint-url http://filer:8333 s3 mb s3://warehouse;
      aws --endpoint-url http://filer:8333 s3api put-bucket-acl --bucket warehouse --acl public-read-write;
      tail -f /dev/null
      "
networks:
  iceberg_net:
volumes:
  seaweedfs_master_data:
  seaweedfs_volume_data:
  seaweedfs_filer_data:
```

!!! exercise
    Crie um arquivo `docker-compose.yml` com o conteúdo acima, mas ainda não inicialize os serviços.

!!! exercise
    Dê uma lida com atenção no arquivo `docker-compose.yml`.

    Chame o professor se não entender alguma coisa.

!!! exercise text long
    Analise os serviços que são inicializados com este `docker-compose.yml`. Quais são eles e qual a função de cada um?

    !!! answer
        1. **spark-iceberg**: Este serviço executa o **Apache Spark** com suporte ao **Apache Iceberg**, permitindo a criação e manipulação de tabelas **Iceberg**.
        2. **rest**: Este serviço é um *fixture* **REST** para o **Apache Iceberg**, que fornece uma **API REST** para interagir com o catálogo **Iceberg**.
        3. **master**, **volume**, **filer**: Estes serviços executam o **SeaweedFS** com a **API S3** habilitada, funcionando como o backend de armazenamento de objetos para os dados do **Iceberg**.
        4. **seaweedfs-init**: Este serviço utiliza a **AWS CLI** para criar e configurar o bucket `warehouse` no **SeaweedFS**.

!!! exercise
    A partir da pasta raiz da aula, crie um arquivo `spark/conf/spark-defaults.conf` com o seguinte conteúdo:

    !!! warning "Atenção!"
        Crie as pastas necessárias!

    ```text { .copy }
    spark.sql.extensions                   org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions
    spark.sql.catalog.demo                 org.apache.iceberg.spark.SparkCatalog
    spark.sql.catalog.demo.type            rest
    spark.sql.catalog.demo.uri             http://rest:8181
    spark.sql.catalog.demo.io-impl         org.apache.iceberg.aws.s3.S3FileIO
    spark.sql.catalog.demo.warehouse       s3://warehouse/wh/
    spark.sql.catalog.demo.s3.endpoint     http://filer:8333
    spark.sql.catalog.demo.s3.path-style-access true
    spark.sql.defaultCatalog               demo
    spark.eventLog.enabled                 true
    spark.eventLog.dir                     /home/iceberg/spark-events
    spark.history.fs.logDirectory          /home/iceberg/spark-events
    spark.sql.catalogImplementation        in-memory
    ```

!!! exercise
    A partir da pasta raiz da aula, crie um arquivo `seaweedfs-s3.json` com as credenciais de acesso:

    ```json { .copy }
    {
      "identities": [
        {
          "name": "admin",
          "credentials": [
            {
              "accessKey": "admin",
              "secretKey": "password"
            }
          ],
          "actions": ["Admin", "Read", "List", "Tagging", "Write"]
        }
      ]
    }
    ```

!!! exercise
    Agora, inicialize os serviços utilizando o comando:

    <div class="termy">

    ```
    $ docker compose up
    ```

    </div>

    Aguarde uns instantes até que os serviços estejam totalmente inicializados.

!!! exercise text short
    Verifique se o bucket `warehouse` foi criado. Como ele foi criado?

    !!! info "Info!"
        Utilize a **AWS CLI** para listar os buckets do SeaweedFS:

        <div class="termy">

        ```
        $ export AWS_ACCESS_KEY_ID=admin
        $ export AWS_SECRET_ACCESS_KEY=password
        $ aws --endpoint-url http://localhost:8333 s3 ls
        ```
        
        </div>

    !!! answer
        Sim, existe um bucket `warehouse` já criado. Ele foi criado automaticamente pelo serviço **seaweedfs-init** na primeira vez que foi executado.

## Criar tabelas

Vamos utilizar o **PyIceberg** para criar e manipular tabelas no Iceberg.

!!! exercise
    A imagem **Docker** que estamos utilizando já contém alguns notebooks de exemplo.

    Acesse o [Jupyter Notebook Getting Started](http://localhost:8888/notebooks/PyIceberg%20-%20Getting%20Started.ipynb).

!!! exercise
    Leia o notebook com atenção e execute os códigos até antes de iniciar a seção **DuckDB**.

!!! exercise text long
    Ao terminar a execução do notebook, verifique se os arquivos da tabela foram criados no **SeaweedFS**:

    <div class="termy">

    ```bash { .copy }
    $ export AWS_ACCESS_KEY_ID=admin
    $ export AWS_SECRET_ACCESS_KEY=password
    $ aws --endpoint-url http://localhost:8333 s3 ls s3://warehouse/default/taxis/ --recursive
    ```

    </div>

    Quais arquivos e pastas foram criados? Qual a função das pastas `data` e `metadata`?

    !!! answer
        A pasta `data` contém os arquivos de dados reais da tabela, enquanto a pasta `metadata` contém os arquivos de metadados que descrevem a estrutura da tabela, incluindo informações sobre partições, esquema e snapshots.

!!! exercise
    Acesse o [Jupyter Notebook PyIceberg - Write support](http://localhost:8888/notebooks/PyIceberg%20-%20Write%20support.ipynb).

    Execute os códigos do notebook. Se tiver dúvidas, chame o professor.

!!! exercise
    Acesse o [Jupyter Notebook Iceberg - View Support](http://localhost:8888/notebooks/Iceberg%20-%20View%20Support.ipynb).

    Execute os códigos do notebook. Se tiver dúvidas, chame o professor.

!!! exercise
    Acesse a URL do **SeaweedFS** [http://localhost:8333](http://localhost:8333) para verificar os arquivos disponíveis no bucket `warehouse`.

Acesse o servidor **Jupyter Notebook** no endereço [http://localhost:8888](http://localhost:8888). Você irá encontrar vários notebooks de exemplo para explorar o **Iceberg**!

Na próxima seção, iremos utilizar o **Trino** para consultar as tabelas que criamos utilizando o formato **Iceberg**.