# AWS Glue

## Introdução

Para praticarmos com **Data Lakes** e **ETL na nuvem**, usaremos o **AWS Glue**. Ele é um serviço **serverless** e totalmente gerenciado de **ETL (Extract, Transform, Load)** que automatiza descoberta, preparo, transformação e carregamento de dados a partir de múltiplas fontes.

!!! tip "Prepare o bolso!"
    Você foca na lógica de negócio, a AWS cuida de infraestrutura (escalonamento, gerenciamento de recursos, monitoração).

Em alto nível, o Glue funciona como a espinha dorsal de preparação de dados em um ecossistema analítico: descobre dados, organiza metadados no **Data Catalog**, gera ou executa **jobs** Spark (Python ou Scala), orquestra dependências e integra-se com serviços analíticos como **Athena**, **Redshift**, **EMR**, **QuickSight** (serviços que ainda não foram explorados no curso).


## Estrutura básica S3
Para explorarmos as características principais do **Glue**, vamos criar um **Data Lake** simples em **S3** e usar o **Glue** para catalogar, transformar e preparar dados para análise.

Vimos que um **Data Lake** geralmente é organizado em camadas para facilitar a gestão e o processamento dos dados. A estrutura básica do *bucket* será a seguinte:

```
meu-data-lake-<INSPER_USERNAME>/
  raw/
  processed/
```
!!! exercise
    Antes de prosseguir, faça login SSO na AWS com o perfil `dataeng`:

    <div class="termy">

    ```
    $ aws sso login --profile dataeng
    ```

    </div>
!!! exercise
    Crie um *bucket* no **S3** com o nome `meu-data-lake-<INSPER_USERNAME>`

    !!! warning "Atenção"
        Defina corretamente a variável de ambiente `INSPER_USERNAME` com seu **usuário insper**, pois o nome do *bucket* deve ser globalmente único!

    === "Linux/Mac"
        <div class="termy">

        ```
        $ INSPER_USERNAME=seu-usuario-insper
        $ aws s3api create-bucket \
            --bucket meu-data-lake-$INSPER_USERNAME \
            --region us-east-1 \
            --profile dataeng
        ```

        </div>
    
    === "Powershell"
        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"

        $ aws s3api create-bucket `
            --bucket "meu-data-lake-$($env:INSPER_USERNAME)" `
            --region us-east-1 `
            --profile dataeng
        ```

        </div>
    
    === "CMD"
        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper

        $ aws s3api create-bucket ^
            --bucket meu-data-lake-%INSPER_USERNAME% ^
            --region us-east-1 ^
            --profile dataeng
        ```

        </div>

!!! exercise
    Dentro do *bucket*, crie as seguintes pastas (prefixos):

    - `raw/`: para dados brutos, diretamente da fonte.
    - `processed/`: para dados limpos e transformados.

    !!! warning "Atenção"
        Caso esteja na mesma sessão do terminal, não precisa redefinir a variável de ambiente `$INSPER_USERNAME`.

    === "Linux/Mac"
        
        <div class="termy">

        ```
        $ INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object \
            --bucket meu-data-lake-$INSPER_USERNAME \
            --key raw/ \
            --profile dataeng
        ```

        </div>
        <br><br>

        <div class="termy">

        ```
        $ INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object \
            --bucket meu-data-lake-$INSPER_USERNAME \
            --key processed/ \
            --profile dataeng
        ```

        </div>
    
    === "Powershell"
        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"
        $ aws s3api put-object `
            --bucket "meu-data-lake-$($env:INSPER_USERNAME)" `
            --key raw/ `
            --profile dataeng
        ```

        </div>

        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"
        $ aws s3api put-object `
            --bucket "meu-data-lake-$($env:INSPER_USERNAME)" `
            --key processed/ `
            --profile dataeng
        ```

        </div>
    
    === "CMD"
        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object ^
            --bucket meu-data-lake-%INSPER_USERNAME% ^
            --key raw/ ^
            --profile dataeng
        ```

        </div>

        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object ^
            --bucket meu-data-lake-%INSPER_USERNAME% ^
            --key processed/ ^
            --profile dataeng
        ```

        </div>

!!! exercise
    Verifique se as pastas foram criadas corretamente.

    === "Linux/Mac"
        <div class="termy">

        ```
        $ INSPER_USERNAME=seu-usuario-insper
        $ aws s3 ls meu-data-lake-$INSPER_USERNAME --recursive --profile dataeng
        ```

        </div>

    === "Powershell"
        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"
        $ aws s3 ls "meu-data-lake-$($env:INSPER_USERNAME)" --recursive --profile dataeng
        ```

        </div>
    
    === "CMD"
        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper
        $ aws s3 ls meu-data-lake-%INSPER_USERNAME% --recursive --profile dataeng
        ```

        </div>

## Base de Dados

Iremos utilizar a base de dados [**Brazilian E-Commerce Public Dataset by Olist**](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), disponível no [**Kaggle**](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

![](modelo-rel.png)
**Fonte da imagem**: [Kaggle](https://i.imgur.com/HRhd2Y0.png)

!!! exercise
    Faça o download da base de dados. Crie uma pasta para a aula e descompacte os arquivos nesta pasta.

Agora vamos subir a base de dados de **itens vendidos** (`olist_order_items`):

!!! exercise
    Faça o upload do arquivo `olist_order_items_dataset.csv` para a pasta `raw/olist_order_items/olist_order_items_dataset.csv`

    !!! warning "Atenção"
        Caso esteja na mesma sessão do terminal, não precisa redefinir a variável de ambiente `$INSPER_USERNAME`.

    === "Linux/Mac"
        <div class="termy">

        ```
        $ INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object \
            --bucket meu-data-lake-$INSPER_USERNAME \
            --key raw/olist_order_items/olist_order_items_dataset.csv \
            --body olist_order_items_dataset.csv \
            --profile dataeng
        ```

        </div>

    === "Powershell"
        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"
        $ aws s3api put-object `
            --bucket "meu-data-lake-$($env:INSPER_USERNAME)" `
            --key "raw/olist_order_items/olist_order_items_dataset.csv" `
            --body "olist_order_items_dataset.csv" `
            --profile dataeng
        ```

        </div>
    
    === "CMD"
        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object ^
            --bucket meu-data-lake-%INSPER_USERNAME% ^
            --key raw/olist_order_items/olist_order_items_dataset.csv ^
            --body olist_order_items_dataset.csv ^
            --profile dataeng
        ```

        </div>

Agora vamos subir a base de dados de **products** (`olist_products`):

!!! exercise
    Faça o upload do arquivo `olist_products_dataset.csv` para a pasta `raw/olist_products/olist_products_dataset.csv`

    !!! warning "Atenção"
        Caso esteja na mesma sessão do terminal, não precisa redefinir a variável de ambiente `$INSPER_USERNAME`.

    === "Linux/Mac"
        <div class="termy">

        ```
        $ INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object \
            --bucket meu-data-lake-$INSPER_USERNAME \
            --key raw/olist_products/olist_products_dataset.csv \
            --body olist_products_dataset.csv \
            --profile dataeng
        ```

        </div>

    === "Powershell"
        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"
        $ aws s3api put-object `
            --bucket "meu-data-lake-$($env:INSPER_USERNAME)" `
            --key "raw/olist_products/olist_products_dataset.csv" `
            --body "olist_products_dataset.csv" `
            --profile dataeng
        ```

        </div>
    
    === "CMD"
        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper
        $ aws s3api put-object ^
            --bucket meu-data-lake-%INSPER_USERNAME% ^
            --key raw/olist_products/olist_products_dataset.csv ^
            --body olist_products_dataset.csv ^
            --profile dataeng
        ```

        </div>

!!! exercise
    Verifique se os arquivos foram criados corretamente.

    === "Linux/Mac"
        <div class="termy">

        ```
        $ INSPER_USERNAME=seu-usuario-insper
        $ aws s3 ls meu-data-lake-$INSPER_USERNAME --recursive --profile dataeng
        ```

        </div>

    === "Powershell"
        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"
        $ aws s3 ls "meu-data-lake-$($env:INSPER_USERNAME)" --recursive --profile dataeng
        ```

        </div>
    
    === "CMD"
        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper
        $ aws s3 ls meu-data-lake-%INSPER_USERNAME% --recursive --profile dataeng
        ```

        </div>

## Glue: Características Principais

Os componentes mais usados do **Glue** são:

### Data Catalog & Metadados

Repositório centralizado de metadados (tabelas, bancos lógicos, partições, esquemas, localização no S3, formatos, propriedades). Dentre outras características, ele:

- Permite descoberta de dados por múltiplos serviços (Athena, Redshift Spectrum, EMR, Glue Jobs).
- Armazena evolução de esquemas (*schema versioning*) e suporta alteração incremental.

!!! info "Info!"
    Ao registrar tabelas no **Data Catalog**, elas tornam-se imediatamente consultáveis por **Athena** (SQL serverless) ou via **Redshift Spectrum**, evitando duplicação de metadados.

### Crawlers

Processos que inspecionam fontes (S3, JDBC, etc.) para inferir automaticamente esquemas e criar/atualizar entradas no **Data Catalog**. Benefícios:

- Automatizam descoberta e atualização de partições (por exemplo, dados particionados por `ano=2025/mes=09/`).
- Reduzem erro humano em definição manual de tipos.
- Podem ser agendados ou executados sob demanda.

## Criar um Crawler

Vamos criar um **crawler** para descobrir e catalogar a tabela `olist_order_items` que acabamos de subir para o **S3**.

Inicialmente, criaremos um **database** no **Glue Data Catalog** para organizar nossas tabelas:

!!! exercise
    Crie Database no Glue Data Catalog com o nome `meu-db-olist`. Execute este comando no terminal:

    === "Linux/Mac"
        <!-- <div class="termy"> -->

        ```
        aws glue create-database \
            --profile dataeng \
            --region us-east-1 \
            --database-input '{
                "Name": "meu-db-olist",
                "Description": "Database de exemplo para Glue com base Olist",
                "Parameters": {
                "CreatedBy": "AWS CLI"
                }
            }'
        ```

        <!-- </div> -->
    === "Powershell"
        Crie um arquivo `db.json` com o seguinte conteúdo:

        ```json
        {
            "Name": "meu-db-olistasd",
            "Description": "Database de exemplo para Glue com base Olist",
            "Parameters": { "CreatedBy": "AWS CLI" }
        }
        ```

        Rode:

        <div class="termy">

        ```
        $ aws glue create-database `
            --profile dataeng `
            --region us-east-1 `
            --database-input file://db.json
        ```

        </div>
    
    === "CMD"
        Crie um arquivo `db.json` com o seguinte conteúdo:

        ```json
        {
            "Name": "meu-db-olistasd",
            "Description": "Database de exemplo para Glue com base Olist",
            "Parameters": { "CreatedBy": "AWS CLI" }
        }
        ```

        Rode:

        <div class="termy">

        ```
        $ aws glue create-database ^
            --profile dataeng ^
            --region us-east-1 ^
            --database-input file://db.json
        ```

        </div>

!!! exercise
    Acesse o console do **AWS Glue** e verifique se o database `meu-db-olist` foi criado corretamente.

    Utilize o menu esquerdo **Data catalog > Databases**.

Agora precisamos criar uma **role IAM** para o **Glue** ter permissão de ler os dados do **S3** e escrever no **Data Catalog**.

!!! exercise text long
    Crie os arquivos `glue_etl_crawler_s3_role.json` e `glue_etl_crawler_s3_policy.json`

    ??? "`glue_etl_crawler_s3_role.json`"
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                "Service": "glue.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
            ]
        }
        ```

    !!! danger "Atenção"
        Atualize o arquivo `glue_etl_crawler_s3_policy.json`, substituindo `meu-data-lake-xxx` pelo nome do seu *bucket* criado anteriormente.

    ??? "`glue_etl_crawler_s3_policy.json`"
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                "Effect": "Allow",
                "Action": [
                    "glue:*",
                    "s3:GetBucketLocation",
                    "s3:ListBucket",
                    "s3:ListAllMyBuckets",
                    "s3:GetBucketAcl",
                    "ec2:DescribeVpcEndpoints",
                    "ec2:DescribeRouteTables",
                    "ec2:CreateNetworkInterface",
                    "ec2:DeleteNetworkInterface",
                    "ec2:DescribeNetworkInterfaces",
                    "ec2:DescribeSecurityGroups",
                    "ec2:DescribeSubnets",
                    "ec2:DescribeVpcAttribute",
                    "iam:ListRolePolicies",
                    "iam:GetRole",
                    "iam:GetRolePolicy",
                    "cloudwatch:PutMetricData"
                ],
                "Resource": "*"
                },
                {
                "Effect": "Allow",
                "Action": [
                    "s3:CreateBucket"
                ],
                "Resource": [
                    "arn:aws:s3:::aws-glue-*"
                ]
                },
                {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject",
                    "s3:DeleteObject"
                ],
                "Resource": [
                    "arn:aws:s3:::aws-glue-*/*",
                    "arn:aws:s3:::*/*aws-glue-*/*"
                ]
                },
                {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject"
                ],
                "Resource": [
                    "arn:aws:s3:::crawler-public*",
                    "arn:aws:s3:::aws-glue-*"
                ]
                },
                {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": [
                    "arn:aws:logs:*:*:*:/aws-glue/*"
                ]
                },
                {
                "Effect": "Allow",
                "Action": [
                    "ec2:CreateTags",
                    "ec2:DeleteTags"
                ],
                "Condition": {
                    "ForAllValues:StringEquals": {
                    "aws:TagKeys": [
                        "aws-glue-service-resource"
                    ]
                    }
                },
                "Resource": [
                    "arn:aws:ec2:*:*:network-interface/*",
                    "arn:aws:ec2:*:*:security-group/*",
                    "arn:aws:ec2:*:*:instance/*"
                ]
                },
                {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject"
                ],
                "Resource": [
                    "arn:aws:s3:::meu-data-lake-xxx/*"
                ]
                }
            ]
            }
        ```

        Analise a política e liste algumas permissões concedidas.

!!! exercise
    Crie a role `role-glue-meu-data-lake-s3` com a política anexada `policy-glue-meu-data-lake-s3`.

    <div class="termy">

    ```
    $ aws iam create-role \
        --role-name role-glue-meu-data-lake-s3 \
        --assume-role-policy-document file://glue_etl_crawler_s3_role.json \
        --profile dataeng
    ```

    </div>
    <br>

    <div class="termy">

    ```
    $ aws iam put-role-policy \
        --role-name role-glue-meu-data-lake-s3 \
        --policy-name policy-glue-meu-data-lake-s3 \
        --policy-document file://glue_etl_crawler_s3_policy.json \
        --profile dataeng
    ```

    </div>

!!! exercise
    Para criar o **crawler**, utilize:

    === "Linux/Mac"
        <div class="termy">

        ```
        $ aws glue create-crawler \
            --profile dataeng \
            --region us-east-1 \
            --name meu-crawler-olist \
            --role role-glue-meu-data-lake-s3 \
            --database-name meu-db-olist \
            --targets "S3Targets=[{Path=\"s3://meu-data-lake-$INSPER_USERNAME/raw/\"}]" \
            --schema-change-policy UpdateBehavior=UPDATE_IN_DATABASE,DeleteBehavior=DEPRECATE_IN_DATABASE
        ```

        </div>

    === "Powershell"
        <div class="termy">

        ```
        $ $env:INSPER_USERNAME = "seu-usuario-insper"
        $ aws glue create-crawler `
            --profile dataeng `
            --region us-east-1 `
            --name meu-crawler-olist `
            --role role-glue-meu-data-lake-s3 `
            --database-name meu-db-olist `
            --targets "S3Targets=[{Path='s3://meu-data-lake-$($env:INSPER_USERNAME)/raw/'}]" `
            --schema-change-policy UpdateBehavior=UPDATE_IN_DATABASE,DeleteBehavior=DEPRECATE_IN_DATABASE
        ```

        </div>
    
    === "CMD"
        <div class="termy">

        ```
        $ set INSPER_USERNAME=seu-usuario-insper
        $ aws glue create-crawler ^
            --profile dataeng ^
            --region us-east-1 ^
            --name meu-crawler-olist ^
            --role role-glue-meu-data-lake-s3 ^
            --database-name meu-db-olist ^
            --targets "S3Targets=[{Path=\"s3://meu-data-lake-%INSPER_USERNAME%/raw/\"}]" ^
            --schema-change-policy UpdateBehavior=UPDATE_IN_DATABASE,DeleteBehavior=DEPRECATE_IN_DATABASE
        ```

        </div>
    
!!! exercise
    Acesse o console do **AWS Glue** e verifique se o crawler `meu-crawler-olist` foi criado corretamente.

    Utilize o menu esquerdo **Data catalog > Crawlers**.

!!! exercise
    No menu esquerdo **Data catalog > Crawlers**, selecione seu crawler `meu-crawler-olist` e clique no botão **Run crawler**.

    !!! tip "Dica!"
        Caso queira fazer por **AWS CLI**, utilize:

        <div class="termy">

        ```
        $ aws glue start-crawler \
            --profile dataeng \
            --region us-east-1 \
            --name meu-crawler-olist
        ```

        </div>

!!! exercise
    Espere o término da execução do crawler. Atualize a página e verifique se o status mudou para **Ready**.

!!! exercise
    No menu esquerdo **Data catalog > Databases > Tables**, verifique se as tabelas foram criadas corretamente.

    !!! warning "Atenção"
        Caso não obtenha sucesso, nos **Crawler runs**, verifique os logs (**AWS Cloudwatch**) para entender o que ocorreu.

## Athena

Como desejamos consultar os dados catalogados, vamos acessar o **Athena**.

!!! exercise
    Acesse o console do **AWS Athena** e clique no botão **Iniciar consulta**.

    ![Athena Home](athena-home.png)

Em seguida, no menu esquerdo, escolha o database `meu-db-olist` criado anteriormente.

![](athena-menu.png)

No editor de consultas, teste algumas consultas envolvendo as duas tabelas.

!!! warning "Query result location"
    Caso o **Athena** solicite uma pasta para salvar os resultados das consultas, informe a pasta `athena-results/` dentro do seu *bucket*.

    Por exemplo: `s3://meu-data-lake-<INSPER_USERNAME>/athena-results/`

```sql
SELECT COUNT(*) AS qtde_itens_vendidos,
      SUM(price) AS vlr_total_vendido
FROM olist_order_items;

SELECT * FROM olist_order_items LIMIT 5;
```

!!! tip "Dica!"
    Caso digite múltiplas *queries*, **selecione** a *query* que deseja executar e clique no botão **Run again** ou aperte **Ctrl + Enter**.

![](athena-query.png)

Pronto! Desta maneira, conseguimos catalogar e consultar dados **brutos** no **S3** usando o **Glue** e o **Athena**.

!!! exercise text short
    Quantas linhas tem cada uma das duas tabelas catalogadas?

    !!! answer "Resposta"
        - `olist_order_items`: 112650 linhas
        - `olist_products`: 32951 linhas

        ```sql
        SELECT COUNT(*) FROM olist_order_items; -- 112650
        SELECT COUNT(*) FROM olist_products; -- 32951
        ```

!!! exercise text long
    Quais as cinco categorias de produtos mais vendidas (maior quantidade de itens vendidos)?

    !!! answer "Resposta"
        ```sql
        SELECT p.product_category_name, COUNT(*) AS qtde_itens_vendidos
        FROM olist_order_items oi
        JOIN olist_products p ON oi.product_id = p.product_id
        GROUP BY p.product_category_name
        ORDER BY qtde_itens_vendidos DESC
        LIMIT 5;
        ```

        | product_category_name       | qtde_itens_vendidos |
        |-----------------------------|---------------------|
        | cama_mesa_banho         | 11115               |
        | beleza_saude                  | 9670               |
        | esporte_lazer             | 8641               |
        | moveis_decoracao                   | 8334               |
        | informatica_acessorios      | 7827               |

## Jobs de ETL


Os jobs de **ETL** no **AWS Glue** são processos automatizados que permitem extrair dados de diferentes fontes, realizar transformações necessárias e carregar os resultados em destinos apropriados, como **data lakes** ou **data warehouses**. Eles são essenciais para organizar, limpar e preparar grandes volumes de dados de forma eficiente, sem a necessidade de gerenciar infraestrutura manualmente.

Utilizamos **jobs de ETL** do **Glue** quando precisamos integrar dados de múltiplos sistemas, padronizar formatos, enriquecer informações ou criar pipelines analíticos que exigem escalabilidade e automação. Cenários comuns incluem a conversão de arquivos brutos em formatos otimizados para análise, a atualização incremental de tabelas, ou a preparação de dados para visualização e exploração por ferramentas como **Athena** ou **Redshift**.

!!! info "Vendedor da AWS!"
    O Glue facilita o trabalho de engenharia de dados, tornando o processo mais ágil, seguro e integrado ao ecossistema AWS.

Os **jobs de ETL** no **Glue** leem, transformam e escrevem dados. São programados em **Python** (**PySpark**) ou **Scala**.

Vamos criar um *job** que lê a tabela `olist_order_items` e grava o resultado em formato **Parquet** na pasta `processed/`.

Este passo será realizado de forma manual, pelo editor visual do **Glue Studio**.

!!! exercise
    Acesse o console do **Glue** e clique em **ETL jobs > Visual ETL**.

    A direta, clique em **Create job** (Visual ETL).

![Glue Visual ETL](glue-visual-etl.png)

!!! exercise
    Clique no lápis para editar o nome do **job**

As **sources** representam as fontes de dados de entrada que o **job** irá processar.

!!! exercise text short
    Clique no **+** (**Add nodes**) e explore as opções de configuração de **sources** do **job**.

    Cite uma ou duas que chamaram sua atenção!

!!! exercise
    Explore as demais abas (Transforms, Targets) do **Add nodes**.

!!! exercise
    Adicione uma **source** do tipo **S3**.

    Dê dois cliques na caixa que representa a **source** para editar suas propriedades.

    Configure a origem para ler a tabela `olist_order_items` do **Data Catalog**:
    
    ![](glue-etl-job-s3-source.png)

!!! exercise
    Em seguida, adicione um **Transform - Change schema**.

    Configure tipos adequados para as colunas e remova a coluna `seller_id` (marque a opção "Drop column" para simular que ela não é relevante para nossa análise).

!!! exercise
    Em seguida, adicione um **Transform - Filter**.

    Configure o filtro para manter apenas os itens com `price > 100` (filtragem *pushdown predicate* para otimizar leitura).

    ![](glue-etl-job-filter.png)

!!! exercise
    Adicione também um **Data Target - S3 Bucket**.

    Configure o destino para gravar os resultados na pasta `processed/` em formato **Parquet**.

    !!! warning "Atenção"
        Lembre-se de substituir `<INSPER_USERNAME>` pelo seu **usuário do Insper**.

    **S3 Target Location**: `s3://meu-data-lake-<INSPER_USERNAME>/processed/order-items/`

    ![](glue-etl-job-filter.png)

!!! exercise
    Na aba **Job details**, selecione a **IAM role** criada anteriormente.

    Pode utilizar a mesma role criada para o **crawler**.

!!! exercise
    Na aba **Job details**, marque a opção **Automatically scale the number of workers**.

    Configure o menor **Worker type** possível: `G.1X`.

    Delimite o número máximo de **workers** para `2`.

    Configure o **Job timeout (minutes)** para `10`.

    ![](glue-visual-etl-job-details.png)

Este é o grafo que representa o fluxo de dados do **job**:
![](glue-visual-etl-job-dag.png)

!!! exercise
    Salve e execute o **job**!
    
    Acompanhe sua execução para verificar se tudo ocorreu bem.

!!! exercise choice
    Confira no **Athena**. Os novos dados em **Parquet** já estão disponíveis para consulta?

    - [ ] Sim
    - [X] Não

    !!! answer "Resposta"
        Não. Precisamos catalogar os novos dados. Mas antes, vamos atualizar nosso **ETL**.

!!! exercise
    Crie um novo **DB** para conter as tabelas processadas.

!!! exercise
    Volte ao editor visual do **job** e adicione o **ETL** para a tabela de **products** (`olist_products`).

    Adicione um **JOIN** entre as duas tabelas e salve o resultado do **JOIN** no **S3** em `processed/order-items-products/`.

    !!! warning "Atenção"
        Perceba que o **JOIN** deve ser feito entre as colunas `product_id` de ambas as tabelas.

        O **JOIN** deve ser do tipo **Inner Join** e ser realizado antes do filtro de `price > 100`.

    !!! danger "Atenção"
        Não execute o **job** antes de ter feito o próximo exercício.

    ![](glue-visual-etl-job-dag-v2.png)

!!! exercise
    Agora, em cada nó **Target - S3 Bucket**, altere o **Data Catalog database** para o novo **DB**, criado anteriormente.

    Escolha a opção para criar uma nova tabela, selecione o **DB** adequado e defina nomes apropriados para as tabelas.

    ![](glue-visual-etl-job-data-catalog-table.png)

!!! exercise
    Rode o **job** e verifique se as novas tabelas foram criadas corretamente.

!!! exercise
    No **Athena**, consulte a nova tabela que contém o resultado do **JOIN**.

    !!! answer "Resposta"
        ```sql
        SELECT COUNT(*) FROM order_items_products;
        ```

!!! exercise text long
    Resuma, em suas palavras, os principais objetivos do uso de **AWS Glue**.

Por hoje é só!