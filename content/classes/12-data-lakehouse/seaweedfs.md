# SeaweedFS

Iremos utilizar o **SeaweedFS** como nosso sistema de armazenamento de objetos (*object storage*) para o **Data Lakehouse**.

O **SeaweedFS** é um sistema de armazenamento compatível com a API do AWS S3, que pode ser executado localmente.

## Preparar ambiente

!!! exercise
    Crie uma pasta `12-data-lakehouse/01-seaweedfs` em seu diretório de trabalho e navegue até ela:

    <div class="termy">

    ```
    $ mkdir -p 12-data-lakehouse/01-seaweedfs
    $ cd 12-data-lakehouse/01-seaweedfs
    ```

    </div>

## Configuração de credenciais

O **SeaweedFS** utiliza um arquivo de configuração JSON para definir os usuários e suas credenciais de acesso à API S3.

Crie o arquivo `seaweedfs-s3.json` com o seguinte conteúdo:

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

## Docker compose

Vamos utilizar o Docker Compose para facilitar a execução do **SeaweedFS**.

O arquivo `docker-compose.yml` deve conter:

```yaml { .copy }
services:
  master:
    image: chrislusf/seaweedfs:4.19
    container_name: seaweedfs-master-intro
    networks:
      iceberg_net:
    ports:
      - 9333:9333
    command: "master -ip=master"
  volume:
    image: chrislusf/seaweedfs:4.19
    container_name: seaweedfs-volume-intro
    networks:
      iceberg_net:
    ports:
      - 8090:8080
    command: "volume -mserver=master:9333 -port=8080"
    depends_on:
      - master
  filer:
    image: chrislusf/seaweedfs:4.19
    container_name: seaweedfs-filer-intro
    networks:
      iceberg_net:
    ports:
      - 8889:8888
      - 8333:8333
    volumes:
      - ./seaweedfs-s3.json:/etc/seaweedfs/s3.json:ro
    command: "filer -master=master:9333 -s3 -s3.config=/etc/seaweedfs/s3.json"
    depends_on:
      - master
      - volume

networks:
  iceberg_net:


```

!!! exercise
    Crie os arquivos `seaweedfs-s3.json` e `docker-compose.yml` e inicialize com:

    <div class="termy">

    ```
    $ docker compose up
    ```

    </div>

!!! exercise
    Com o serviço inicializado, acesse a URL [http://localhost:9333](http://localhost:9333) em seu navegador para ver o painel de administração do **SeaweedFS**.

!!! exercise
    Agora acessa a URL [http://localhost:8889](http://localhost:8889) para acessar o painel do **Filer** do **SeaweedFS**.

!!! exercise
    Vamos criar buckets para representar a arquitetura medallion, vista na aula passada.

    Utilize a **AWS CLI** para criar os buckets. Primeiro, configure as variáveis de ambiente:

    <div class="termy">

    ```
    $ export AWS_ACCESS_KEY_ID=admin
    $ export AWS_SECRET_ACCESS_KEY=password
    ```

    </div>

    Em seguida, crie os buckets:

    <div class="termy">

    ```
    $ aws --endpoint-url http://localhost:8333 s3 mb s3://bronze --region us-east-1
    $ aws --endpoint-url http://localhost:8333 s3 mb s3://silver --region us-east-1
    $ aws --endpoint-url http://localhost:8333 s3 mb s3://gold --region us-east-1
    ```

    </div>

!!! exercise
    Faça uploads de um ou mais arquivos **CSV** para testes:

    <div class="termy">

    ```
    $ aws --endpoint-url http://localhost:8333 s3 cp meu_arquivo.csv s3://bronze/
    ```

    </div>

    Verifique o upload listando o conteúdo do bucket:

    <div class="termy">

    ```
    $ aws --endpoint-url http://localhost:8333 s3 ls s3://bronze/
    ```

    </div>

!!! exercise
    Volte para a URL [http://localhost:8889](http://localhost:8889) e confira se o arquivo adicionado está visível.

## Remover containers

!!! exercise
    Para parar e remover os containers, utilize:

    <div class="termy">

    ```
    $ docker compose down
    ```

    </div>