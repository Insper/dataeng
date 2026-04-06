# Airflow e AWS EMR

Nesta aula, vamos explorar como o **Apache Airflow** pode orquestrar tarefas de processamento de dados em grande escala utilizando o **Amazon EMR** (anteriormente chamado de *Amazon Elastic MapReduce*). Esta combinação é muito poderosa para pipelines de dados que precisam de recursos computacionais elásticos e sob demanda.

## O que é o Amazon EMR?

O **Amazon EMR** é um serviço gerenciado da **AWS** que facilita o processamento de grandes volumes de dados utilizando frameworks como **Apache Spark**, **Apache Hadoop**, **Apache Hive**, **Apache HBase**

!!! info "EMR em Engenharia de Dados"
    O EMR é amplamente utilizado em Engenharia de Dados para:
    
    - **Processamento Big Data:** Transformar e analisar terabytes ou petabytes de dados
    - **ETL em Escala:** Executar transformações complexas que seriam inviáveis em máquinas individuais
    - **Machine Learning:** Treinar modelos com grandes datasets utilizando **Spark MLlib**
    - **Análise de Logs:** Processar logs de aplicações, servidores web, etc.

Como principais características do **EMR**, podemos citar:

- **Elasticidade:** Você pode dimensionar clusters (escalonamento horizontal) com base na demanda
- **Gerenciado:** A **AWS** cuida da infraestrutura, instalação e configuração dos frameworks
- **Custo-efetivo:** Pague apenas pelos recursos que usar, quando usar
- **Integração:** Funciona nativamente com outros serviços **AWS** (S3, RDS, Redshift, etc.)

## Por que Airflow + EMR?

A combinação do **Airflow** com o **EMR** oferece vários benefícios:

1. **Orquestração Inteligente:** O **Airflow** pode criar clusters **EMR** sob demanda, executar jobs e depois destruir os clusters, otimizando custos. Uma outra opção é utilizar clusters persistentes, mas que podem ser escalados conforme a necessidade
1. **Gestão de Dependências:** Coordenar múltiplos **jobs Spark** que dependem uns dos outros
1. **Monitoramento Centralizado:** Acompanhar todo o *pipeline* utilizando a interface do **Airflow**
1. **Recuperação de Falhas:** *Retry* automático e notificações em caso de problemas
1. **Escalabilidade:** Ajustar o tamanho do cluster baseado na carga de trabalho de cada etapa

## Preparando o Ambiente

Vamos configurar nosso ambiente para trabalhar com **Airflow** e **EMR**.

Iremos utilizar um ambiente muito parecido com o que usamos na aula anterior de **Airflow**, com algumas modificações para suportar a integração com a **AWS**.

!!! exercise "Exercício"
    Crie uma pasta `airflow-emr`.

    Dentro deste diretório, crie as seguintes subpastas:

    - `dags/`: É aqui que vamos colocar nossos arquivos Python que definem os DAGs. O Airflow irá escanear esta pasta automaticamente.
    - `logs/`: Para armazenar os logs de execução das tarefas.
    - `plugins/`: Para adicionar plugins customizados (não usaremos hoje, mas é bom saber que existe).
    - `config/`: Para configurações adicionais.

    <div class="termy">
    ```bash
    $ mkdir airflow-emr
    $ cd airflow-emr
    $ mkdir -p ./dags ./logs ./plugins ./config
    ```
    </div>

!!! Info "Path!"
    Considere que o restante da aula será feito dentro da pasta `airflow-emr`.

!!! exercise "Exercício"
    Crie um arquivo `.env`.

    Nele, configure o `AIRFLOW_UID` e suas credenciais **AWS**.

    !!! info "Info"
        Após este exercício, deixei um tutorial sobre como criar as credenciais **AWS**.

    !!! warning "Linux/Mac"
        Caso esteja no Linux ou Mac, substitua o `AIRFLOW_UID` pelo resultado do comando `id -u` (execute no terminal).

    !!! danger "Credenciais"
        Nunca compartilhe suas credenciais **AWS** publicamente!

        Se utilizar o Git, adicione o `.env` ao `.gitignore`.

    ```bash { .copy }
    AIRFLOW_UID=50000
    AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY
    AWS_DEFAULT_REGION=us-east-1
    ```

??? "Criar `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY`"
    Para criar as credenciais da **AWS**, siga o passo a passo:

    1. Acesse o **AWS Management Console** e vá para o serviço **IAM**.

    1. No menu lateral, clique em **Users** e depois em **Create user**.

    1. Dê um nome ao usuário (ex: `u_airflow_emr`).

        ![](iam-user-create-1.png)
    
    1. Clique em **Next** e selecione **Attach policies directly**. Procure pelas políticas gerenciadas `AmazonS3FullAccess`, `AmazonElasticMapReduceFullAccess`, `CloudWatchFullAccess` e `AmazonEMRFullAccessPolicy_v2` e selecione-as.

        !!! warning "Atenção"
            Em um ambiente de produção, é recomendável criar uma política personalizada com permissões mínimas necessárias para o usuário.

            Aqui, estamos utilizando políticas gerenciadas para simplificar o processo.

    1. Clique em **Next**. Você deve ver as políticas selecionadas.

        ![](iam-user-policies-2.png)
    
    1. Clique em **Create user**.

    1. Com o usuário criado, clique no nome de usuário. Na seção superior, você deve ver uma opção **Access key 1 Create access key**. Clique nela e selecione a opção **Other**.

        ![](iam-user-access-key-3.png)

    1. Na próxima página você terá a opção de adicionar *tags* (opcional). Clique em **Next**.

    1. Na última página, você verá a **Access Key ID** e a **Secret Access Key**. Copie ambas e utilize no arquivo `.env`.


??? "`docker-compose.yml`:"

    ```yaml { .copy }
    # Basic Airflow cluster configuration for LocalExecutor.
    #
    # WARNING: This configuration is for local development. Do not use it in a production deployment.
    #
    # This configuration supports basic configuration using environment variables or an .env file
    # The following variables are supported:
    #
    # AIRFLOW_IMAGE_NAME           - Docker image name used to run Airflow.
    #                                Default: apache/airflow:3.0.6
    # AIRFLOW_UID                  - User ID in Airflow containers
    #                                Default: 50000
    # AIRFLOW_PROJ_DIR             - Base path to which all the files will be volumed.
    #                                Default: .
    # Those configurations are useful mostly in case of standalone testing/running Airflow in test/try-out mode
    #
    # _AIRFLOW_WWW_USER_USERNAME   - Username for the administrator account (if requested).
    #                                Default: airflow
    # _AIRFLOW_WWW_USER_PASSWORD   - Password for the administrator account (if requested).
    #                                Default: airflow
    # _PIP_ADDITIONAL_REQUIREMENTS - Additional PIP requirements to add when starting all containers.
    #                                Use this option ONLY for quick checks. Installing requirements at container
    #                                startup is done EVERY TIME the service is started.
    #                                A better way is to build a custom image or extend the official image
    #                                as described in https://airflow.apache.org/docs/docker-stack/build.html.
    #                                Default: ''
    #
    # Feel free to modify this file to suit your needs.
    ---
    x-airflow-common:
      &airflow-common
      # In order to add custom dependencies or upgrade provider distributions you can use your extended image.
      # Comment the image line, place your Dockerfile in the directory where you placed the docker-compose.yaml
      # and uncomment the "build" line below, Then run `docker-compose build` to build the images.
      image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:3.0.6}
      # build: .
      environment:
        &airflow-common-env
        AIRFLOW__CORE__EXECUTOR: LocalExecutor
        AIRFLOW__CORE__AUTH_MANAGER: airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager
        AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
        AIRFLOW__CORE__FERNET_KEY: ''
        AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
        AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
        AIRFLOW__CORE__EXECUTION_API_SERVER_URL: 'http://airflow-apiserver:8080/execution/'
        # yamllint disable rule:line-length
        # Use simple http server on scheduler for health checks
        # See https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/logging-monitoring/check-health.html#scheduler-health-check-server
        # yamllint enable rule:line-length
        AIRFLOW__SCHEDULER__ENABLE_HEALTH_CHECK: 'true'
        # WARNING: Use _PIP_ADDITIONAL_REQUIREMENTS option ONLY for a quick checks
        # for other purpose (development, test and especially production usage) build/extend Airflow image.
        _PIP_ADDITIONAL_REQUIREMENTS: ${_PIP_ADDITIONAL_REQUIREMENTS:-apache-airflow-providers-amazon>=8.0.0 boto3>=1.26.0}
        # The following line can be used to set a custom config file, stored in the local config folder
        AIRFLOW_CONFIG: '/opt/airflow/config/airflow.cfg'
        AWS_DEFAULT_REGION: ${AWS_DEFAULT_REGION}
        AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
        AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      volumes:
        - ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags
        - ${AIRFLOW_PROJ_DIR:-.}/logs:/opt/airflow/logs
        - ${AIRFLOW_PROJ_DIR:-.}/config:/opt/airflow/config
        - ${AIRFLOW_PROJ_DIR:-.}/plugins:/opt/airflow/plugins
      user: "${AIRFLOW_UID:-50000}:0"
      depends_on:
        &airflow-common-depends-on
        postgres:
          condition: service_healthy

    services:
      postgres:
        image: postgres:16-alpine
        environment:
          POSTGRES_USER: airflow
          POSTGRES_PASSWORD: airflow
          POSTGRES_DB: airflow
        volumes:
          - postgres-db-volume:/var/lib/postgresql/data
        healthcheck:
          test: ["CMD", "pg_isready", "-U", "airflow"]
          interval: 10s
          retries: 5
          start_period: 5s
        restart: always

      airflow-apiserver:
        <<: *airflow-common
        command: api-server
        ports:
          - "8080:8080"
        healthcheck:
          test: ["CMD", "curl", "--fail", "http://localhost:8080/api/v2/version"]
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 30s
        restart: always
        depends_on:
          <<: *airflow-common-depends-on
          airflow-init:
            condition: service_completed_successfully

      airflow-scheduler:
        <<: *airflow-common
        command: scheduler
        healthcheck:
          test: ["CMD", "curl", "--fail", "http://localhost:8974/health"]
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 30s
        restart: always
        depends_on:
          <<: *airflow-common-depends-on
          airflow-init:
            condition: service_completed_successfully

      airflow-dag-processor:
        <<: *airflow-common
        command: dag-processor
        healthcheck:
          test: ["CMD-SHELL", 'airflow jobs check --job-type DagProcessorJob --hostname "$${HOSTNAME}"']
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 30s
        restart: always
        depends_on:
          <<: *airflow-common-depends-on
          airflow-init:
            condition: service_completed_successfully

      airflow-triggerer:
        <<: *airflow-common
        command: triggerer
        healthcheck:
          test: ["CMD-SHELL", 'airflow jobs check --job-type TriggererJob --hostname "$${HOSTNAME}"']
          interval: 30s
          timeout: 10s
          retries: 5
          start_period: 30s
        restart: always
        depends_on:
          <<: *airflow-common-depends-on
          airflow-init:
            condition: service_completed_successfully

      airflow-init:
        <<: *airflow-common
        entrypoint: /bin/bash
        # yamllint disable rule:line-length
        command:
          - -c
          - |
            if [[ -z "${AIRFLOW_UID}" ]]; then
              echo
              echo -e "\033[1;33mWARNING!!!: AIRFLOW_UID not set!\e[0m"
              echo "If you are on Linux, you SHOULD follow the instructions below to set "
              echo "AIRFLOW_UID environment variable, otherwise files will be owned by root."
              echo "For other operating systems you can get rid of the warning with manually created .env file:"
              echo "    See: https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html#setting-the-right-airflow-user"
              echo
              export AIRFLOW_UID=$$(id -u)
            fi
            one_meg=1048576
            mem_available=$$(($$(getconf _PHYS_PAGES) * $$(getconf PAGE_SIZE) / one_meg))
            cpus_available=$$(grep -cE 'cpu[0-9]+' /proc/stat)
            disk_available=$$(df / | tail -1 | awk '{print $$4}')
            warning_resources="false"
            if (( mem_available < 4000 )) ; then
              echo
              echo -e "\033[1;33mWARNING!!!: Not enough memory available for Docker.\e[0m"
              echo "At least 4GB of memory required. You have $$(numfmt --to iec $$((mem_available * one_meg)))"
              echo
              warning_resources="true"
            fi
            if (( cpus_available < 2 )); then
              echo
              echo -e "\033[1;33mWARNING!!!: Not enough CPUS available for Docker.\e[0m"
              echo "At least 2 CPUs recommended. You have $${cpus_available}"
              echo
              warning_resources="true"
            fi
            if (( disk_available < one_meg * 10 )); then
              echo
              echo -e "\033[1;33mWARNING!!!: Not enough Disk space available for Docker.\e[0m"
              echo "At least 10 GBs recommended. You have $$(numfmt --to iec $$((disk_available * 1024 )))"
              echo
              warning_resources="true"
            fi
            if [[ $${warning_resources} == "true" ]]; then
              echo
              echo -e "\033[1;33mWARNING!!!: You have not enough resources to run Airflow (see above)!\e[0m"
              echo "Please follow the instructions to increase amount of resources available:"
              echo "   https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html#before-you-begin"
              echo
            fi
            echo
            echo "Creating missing opt dirs if missing:"
            echo
            mkdir -v -p /opt/airflow/{logs,dags,plugins,config}
            echo
            echo "Airflow version:"
            /entrypoint airflow version
            echo
            echo "Files in shared volumes:"
            echo
            ls -la /opt/airflow/{logs,dags,plugins,config}
            echo
            echo "Running airflow config list to create default config file if missing."
            echo
            /entrypoint airflow config list >/dev/null
            echo
            echo "Files in shared volumes:"
            echo
            ls -la /opt/airflow/{logs,dags,plugins,config}
            echo
            echo "Change ownership of files in /opt/airflow to ${AIRFLOW_UID}:0"
            echo
            chown -R "${AIRFLOW_UID}:0" /opt/airflow/
            echo
            echo "Change ownership of files in shared volumes to ${AIRFLOW_UID}:0"
            echo
            chown -v -R "${AIRFLOW_UID}:0" /opt/airflow/{logs,dags,plugins,config}
            echo
            echo "Files in shared volumes:"
            echo
            ls -la /opt/airflow/{logs,dags,plugins,config}

        # yamllint enable rule:line-length
        environment:
          <<: *airflow-common-env
          _AIRFLOW_DB_MIGRATE: 'true'
          _AIRFLOW_WWW_USER_CREATE: 'true'
          _AIRFLOW_WWW_USER_USERNAME: ${_AIRFLOW_WWW_USER_USERNAME:-airflow}
          _AIRFLOW_WWW_USER_PASSWORD: ${_AIRFLOW_WWW_USER_PASSWORD:-airflow}
          _PIP_ADDITIONAL_REQUIREMENTS: ''
        user: "0:0"

      airflow-cli:
        <<: *airflow-common
        profiles:
          - debug
        environment:
          <<: *airflow-common-env
          CONNECTION_CHECK_MAX_COUNT: "0"
        # Workaround for entrypoint issue. See: https://github.com/apache/airflow/issues/16252
        command:
          - bash
          - -c
          - airflow
        depends_on:
          <<: *airflow-common-depends-on

    volumes:
      postgres-db-volume:
    ```

!!! exercise "Exercício"
    Crie o arquivo `docker-compose.yml`

??? "Principais modificações no `docker-compose.yml`"
    Em relação à aula passada, as principais modificações foram:

    - Utilização do `LocalExecutor` ao invés do `CeleryExecutor`: Como não precisamos de alta disponibilidade ou escalabilidade horizontal para este exemplo, o `LocalExecutor` é mais simples e fácil de configurar. O principal fornecedor de poder de processamento será o **EMR**, não o **Airflow** (que será executado localmente e ficará responsável pela orquestração).
    - Remoção dos serviços **Flower** e **Redis**: Como não estamos utilizando o `CeleryExecutor`, não precisamos do **Redis** como *broker* de mensagens e nem do **Flower** para monitorar tarefas.
    - Configuração do `environment` do `&airflow-common-env`, contendo as variáveis de ambiente para as credenciais AWS:

    ```yaml { .copy }
    AWS_DEFAULT_REGION: ${AWS_DEFAULT_REGION}
    AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
    AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
    ```
    - Configuração do `_PIP_ADDITIONAL_REQUIREMENTS` para incluir o provedor Amazon, necessário para interagir com o **EMR**.
    - Atualização da versão da imagem `postgres` para `16-alpine` (mais recente): poderíamos manter a `13-alpine`, mas é sempre bom usar versões mais recentes quando possível!

 
!!! exercise "Exercício"
    Inicialize o ambiente **Airflow** para aplicar as mudanças:

    <div class="termy">
    ```bash
    $ docker compose up airflow-init
    $ docker compose up
    ```
    </div>

## Preparando Recursos AWS

Antes de criar nosso **DAG**, precisamos preparar alguns recursos na **AWS**.

### Criando um Bucket S3

!!! exercise "Exercício"
    Crie um bucket **S3** para armazenar dados de entrada e saída. Você pode fazer isso pelo console **AWS** ou via **CLI**:

    !!! warning "Atenção"
        Substitua `INSPER_USERNAME` pelo seu usuário do Insper (sem espaços ou caracteres especiais).

    <div class="termy">
    ```bash
    $ aws s3 mb s3://meu-bucket-emr-INSPER_USERNAME --region us-east-1 --profile dataeng
    ```
    </div>

### Criando Dados de Exemplo

!!! exercise "Exercício"
    Vamos criar um arquivo **CSV** com dados de exemplo para processar no EMR.

    Crie uma pasta `data` e arquivo `data/dados_vendas.csv`:

    !!! info "Info"
        Este é um conjunto de dados fictício e pequeno.

        Em uma situação real, você trabalharia com datasets muito maiores (GBs ou TBs).

    ```csv { .copy }
    data,produto,quantidade,preco_unitario,vendedor
    2025-09-01,Notebook,2,2500.00,Ana Silva
    2025-09-01,Mouse,5,50.00,Ana Silva
    2025-09-01,Teclado,3,150.00,João Santos
    2025-09-02,Monitor,1,800.00,Maria Costa
    2025-09-02,Notebook,1,2500.00,João Santos
    2025-09-02,Mouse,10,45.00,Ana Silva
    2025-09-03,Teclado,5,150.00,Maria Costa
    2025-09-03,Monitor,2,800.00,Ana Silva
    2025-09-03,Notebook,3,2500.00,João Santos
    2025-09-04,Mouse,15,50.00,Maria Costa
    ```

!!! exercise "Exercício"
    Faça *upload* do arquivo para seu bucket **S3**:

    !!! warning "Atenção"
        Substitua `INSPER_USERNAME` pelo seu usuário do Insper (sem espaços ou caracteres especiais).

    <div class="termy">
    ```bash
    $ aws s3 cp data/dados_vendas.csv s3://meu-bucket-emr-INSPER_USERNAME/input/ --profile dataeng
    ```
    </div>

### Criando Script Spark

!!! exercise "Exercício"
    Agora vamos criar uma pasta `scripts` e um script **PySpark** que será executado no **EMR**.

    Crie um arquivo `scripts/processar_vendas.py`:

    ```python { .copy }
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import sum as spark_sum, count, avg
    import sys

    def main():
        # Argumentos do script
        if len(sys.argv) != 3:
            print("Uso: processar_vendas.py <input_path> <output_path>")
            sys.exit(1)
        
        input_path = sys.argv[1]
        output_path = sys.argv[2]
        
        # Criar sessão Spark
        spark = SparkSession.builder.appName("ProcessarVendas").getOrCreate()
        
        # Ler dados do S3
        df = spark.read.option("header", "true").option("inferSchema", "true").csv(input_path)
        
        print("=== Dados originais ===")
        df.show()
        
        # Calcular total de vendas por vendedor
        vendas_por_vendedor = df.groupBy("vendedor").agg(
            spark_sum("quantidade").alias("total_quantidade"),
            spark_sum(df.quantidade * df.preco_unitario).alias("total_receita"),
            count("*").alias("total_transacoes"),
            avg(df.quantidade * df.preco_unitario).alias("ticket_medio")
        ).orderBy("total_receita", ascending=False)
        
        print("=== Vendas por Vendedor ===")
        vendas_por_vendedor.show()
        
        # Salvar resultado no S3
        vendas_por_vendedor.coalesce(1).write.mode("overwrite").option("header", "true").csv(output_path)
        
        print(f"Resultado salvo em: {output_path}")
        
        spark.stop()

    if __name__ == "__main__":
        main()
    ```

!!! exercise "Exercício"
    Faça *upload* do script para o **S3**:

    <div class="termy">
    ```bash
    $ aws s3 cp scripts/processar_vendas.py s3://meu-bucket-emr-INSPER_USERNAME/scripts/ --profile dataeng
    ```
    </div>

!!! exercise "Exercício"
    Liste o conteúdo do *bucket* para verificar se tudo está correto:

    !!! warning "Atenção"
        Substitua `INSPER_USERNAME` pelo seu usuário do Insper.

    !!! tip "Dica"
        Caso prefira, você pode verificar via **console AWS**.

    <div class="termy">

    ```bash
    $ aws s3 ls s3://meu-bucket-emr-INSPER_USERNAME/ --recursive --profile dataeng
    ```

    </div>

## Criando DAG Airflow + EMR

Agora vamos criar um DAG que orquestra todo o processo.

As etapas serão:

1. Criar **cluster EMR**,
1. Executar job Spark,
1. Destruir o cluster.

!!! exercise "Exercício"
    Crie um arquivo `dags/pipeline_emr.py` com o seguinte conteúdo:

    !!! warning "Importante"
        Substitua `INSPER_USERNAME` pelo seu nome de usuário Insper em todas as ocorrências no código abaixo.

    ```python { .copy }
    from __future__ import annotations

    import pendulum
    from airflow.models.dag import DAG
    from airflow.providers.amazon.aws.operators.emr import (
        EmrCreateJobFlowOperator,
        EmrAddStepsOperator,
        EmrTerminateJobFlowOperator,
    )
    from airflow.providers.amazon.aws.sensors.emr import EmrStepSensor

    # Configurações do cluster EMR
    JOB_FLOW_OVERRIDES = {
        "Name": "cluster-airflow-emr",
        "ReleaseLabel": "emr-6.15.0",
        "Applications": [{"Name": "Spark"}],
        "Configurations": [
            {
                "Classification": "spark-env",
                "Configurations": [
                    {
                        "Classification": "export",
                        "Properties": {"PYSPARK_PYTHON": "/usr/bin/python3"},
                    }
                ],
            }
        ],
        "Instances": {
            "InstanceGroups": [
                {
                    "Name": "Primary node",
                    "Market": "ON_DEMAND",
                    "InstanceRole": "MASTER",
                    "InstanceType": "m5.xlarge",
                    "InstanceCount": 1,
                },
                {
                    "Name": "Worker nodes",
                    "Market": "ON_DEMAND",
                    "InstanceRole": "CORE",
                    "InstanceType": "m5.xlarge",
                    "InstanceCount": 2,
                },
            ],
            "KeepJobFlowAliveWhenNoSteps": True,
            "TerminationProtected": False,
        },
        "JobFlowRole": "EMR_EC2_DefaultRole",
        "ServiceRole": "EMR_DefaultRole",
        "LogUri": "s3://meu-bucket-emr-INSPER_USERNAME/logs/",
    }

    # Steps do Spark a serem executados
    SPARK_STEPS = [
        {
            "Name": "processar-vendas-spark",
            "ActionOnFailure": "TERMINATE_CLUSTER",
            "HadoopJarStep": {
                "Jar": "command-runner.jar",
                "Args": [
                    "spark-submit",
                    "--deploy-mode",
                    "cluster",
                    "s3://meu-bucket-emr-INSPER_USERNAME/scripts/processar_vendas.py",
                    "s3://meu-bucket-emr-INSPER_USERNAME/input/dados_vendas.csv",
                    "s3://meu-bucket-emr-INSPER_USERNAME/output/vendas_processadas/",
                ],
            },
        }
    ]

    with DAG(
        dag_id="pipeline_emr",
        start_date=pendulum.datetime(2025, 9, 1, tz="UTC"),
        schedule=None,  # Execução manual
        catchup=False,
        tags=["emr", "spark", "aws"],
        description="Pipeline de processamento de dados usando EMR e Spark",
    ) as dag:

        # Tarefa 1: Criar cluster EMR
        criar_cluster = EmrCreateJobFlowOperator(
            task_id="criar_cluster_emr",
            job_flow_overrides=JOB_FLOW_OVERRIDES,
            aws_conn_id="aws_default",
        )

        # Tarefa 2: Adicionar steps ao cluster
        adicionar_steps = EmrAddStepsOperator(
            task_id="adicionar_steps_spark",
            job_flow_id="{{ task_instance.xcom_pull(task_ids='criar_cluster_emr', key='return_value') }}",
            steps=SPARK_STEPS,
            aws_conn_id="aws_default",
        )

        # Tarefa 3: Aguardar conclusão do step
        aguardar_step = EmrStepSensor(
            task_id="aguardar_step_spark",
            job_flow_id="{{ task_instance.xcom_pull(task_ids='criar_cluster_emr', key='return_value') }}",
            step_id="{{ task_instance.xcom_pull(task_ids='adicionar_steps_spark', key='return_value')[0] }}",
            aws_conn_id="aws_default",
            timeout=60 * 30,  # 30 minutos
            poke_interval=60,  # Verificar a cada minuto
        )

        # Tarefa 4: Terminar cluster
        terminar_cluster = EmrTerminateJobFlowOperator(
            task_id="terminar_cluster_emr",
            job_flow_id="{{ task_instance.xcom_pull(task_ids='criar_cluster_emr', key='return_value') }}",
            aws_conn_id="aws_default",
        )

        # Definir dependências
        criar_cluster >> adicionar_steps >> aguardar_step >> terminar_cluster
    ```

??? "Entendendo o DAG EMR"
    Este DAG demonstra um padrão comum ao trabalhar com **EMR**:

    1. **EmrCreateJobFlowOperator:** Cria um cluster **EMR** com a configuração especificada
    1. **EmrAddStepsOperator:** Adiciona steps (jobs) ao cluster. No nosso caso, um job Spark
    1. **EmrStepSensor:** Monitora a execução do step até sua conclusão
    1. **EmrTerminateJobFlowOperator:** Termina o cluster para evitar custos desnecessários

    **XComs**: Note o uso intensivo de XComs (`xcom_pull`) para passar o ID do cluster entre as tarefas.

## Configurando Roles IAM

Para que o **EMR** funcione, precisamos de  **IAM roles** específicas.

!!! exercise "Exercício"
    Crie as roles IAM necessárias:

    <div class="termy">
    ```bash
    $ aws emr create-default-roles --profile dataeng
    ```
    </div>

    !!! info "Info"
        Este comando cria automaticamente as roles `EMR_DefaultRole`, `EMR_AutoScaling_DefaultRole` e `EMR_EC2_DefaultRole`.

!!! exercise "Exercício"
    Acesse o console **IAM** na AWS e verifique se as roles foram criadas corretamente.

    !!! tip "Dica"
        Caso prefira, faça pelo **CLI**:

        <div class="termy">

        ```bash
        $ aws iam list-roles --profile dataeng | grep EMR
        ```

        </div>  

    !!! answer "Resposta"
        ![](iam-roles-imr.png)

## Executando o Pipeline

Vamos executar nosso **DAG** e monitorar a execução.

!!! danger "Atenção"
    Antes de prosseguir, garanta que você está ciente dos custos associados ao uso do **EMR**.

    **Clusters EMR** podem gerar custos significativos dependendo do tamanho e duração.

    Não faça o restante da aula pela metade nem deixe **clusters** rodando ao final da aula.

!!! warning "Aviso"
    Acesse o menu **EMR** do **AWS Console** e verifique que não há clusters **EMR** rodando antes de iniciar o DAG.

    Utilizaremos o **Console** para monitorar a criação e destruição do cluster.

!!! exercise "Exercício"
    Acesse a interface do Airflow em [http://localhost:8080](http://localhost:8080).

    Encontre o **DAG** `pipeline_emr`.

!!! exercise "Exercício"
    No **Airflow**, utilizando o modo **Graph view**, execute o **DAG** manualmente clicando em **Trigger**.

    Você deve ver o DAG iniciar a execução:

    ![](air-erm-dag-running.png)

    Atualize o **Console AWS EMR** para ver o cluster sendo criado:

    !!! info "Info"
        Pode levar alguns minutos para o cluster ser provisionado!

    ![](aws-emr-starting-cluster.png)

    No **Airflow**, clique na tarefa `aguardar_step_spark` e aproveite para acompanhar os seus **logs** (à direita):

    ![](airflow-emr-logs.png)

!!! exercise "Exercício"
    Após a conclusão, verifique o status do cluster no **Console AWS EMR**.

    !!! answer "Resposta"
        O cluster deve estar em estado **Terminated**.

!!! exercise "Exercício"
    Verifique os resultados no S3:

    <div class="termy">

    ```bash
    $ aws s3 ls s3://meu-bucket-emr-INSPER_USERNAME/output/vendas_processadas/ --profile dataeng
    ```

    </div>

    Faça download do arquivo resultado:

    <div class="termy">

    ```bash
    $ aws s3 cp s3://meu-bucket-emr-INSPER_USERNAME/output/vendas_processadas/ . --recursive --profile dataeng
    ```

    </div>

!!! exercise "Exercício"
    Altere o **DAG**, adicionando uma nova etapa `validar_entrada` no inicio do *pipeline*.

    Esta etapa deve verificar se o arquivo de entrada existe no **S3** antes de criar o cluster **EMR**.

    !!! tip "Dica"
        Utilize o operador [`S3KeySensor`](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/_api/airflow/providers/amazon/aws/sensors/s3/index.html) do **Airflow**.

    Primeiro, configure para um arquivo propositalmente inexistente. Execute o DAG e verifique se a tarefa falha como esperado.

    Na sequência, corrija o caminho do arquivo e execute novamente, garantindo que o DAG funcione corretamente.

## Otimizações

### Uso de Spot Instances

As [**Spot Instances**](https://aws.amazon.com/ec2/spot/use-case/emr/) são uma maneira de economizar custos ao executar *workloads* tolerantes a falhas. Graças à escala da **AWS**, é possível aproveitar a capacidade ociosa do **EC2** com descontos que chegam a 90% em comparação ao preço sob demanda, por meio das instâncias **spot** do Amazon EC2.

!!! info "Info!"
    Embora a AWS possa retomar essa capacidade com um aviso prévio de dois minutos, menos de 5% das cargas de trabalho sofrem interrupções.

!!! exercise "Exercício"
    Modifique a configuração do cluster para usar **Spot Instances** nos workers:

    ```python { .copy }
    # Na configuração JOB_FLOW_OVERRIDES, modifique a seção dos workers:
    {
        "Name": "Worker nodes",
        "Market": "SPOT",  # Mudança aqui
        "BidPrice": "0.10",  # Preço máximo por hora
        "InstanceRole": "CORE",
        "InstanceType": "m5.xlarge",
        "InstanceCount": 2,
    }
    ```

    Execute o **DAG** novamente e monitore a criação do cluster no **Console AWS EMR**.

### Parametrização

!!! exercise "Exercício"
    Torne o DAG mais flexível usando **Variables** do Airflow:

    1. Vá para **Admin → Variables** na interface do Airflow
    1. Adicione as seguintes variáveis:
        - `emr_bucket_name`: nome do seu bucket S3
        - `emr_instance_type`: tipo de instância (ex: `m5.xlarge`)
        - `emr_instance_count`: número de workers (ex: `2`)

    1. Modifique seu DAG para usar essas variáveis:

    ```python { .copy }
    from airflow.models import Variable

    # No início do seu DAG
    BUCKET_NAME = Variable.get("emr_bucket_name")
    INSTANCE_TYPE = Variable.get("emr_instance_type", default_var="m5.xlarge")
    INSTANCE_COUNT = int(Variable.get("emr_instance_count", default_var="2"))
    ```

    Em seguida, utilize essas variáveis na configuração do cluster e nos paths do S3.

    Execute o **DAG** novamente para garantir que tudo funcione corretamente.

## Limpeza e Finalização

!!! exercise "Exercício"
    Para evitar custos desnecessários, limpe os recursos criados:

    1. **Verifique se não há clusters EMR ativos:**

        <div class="termy">
        ```bash
        $ aws emr list-clusters --active --profile dataeng
        ```
        </div>

    1. **Se houver clusters orfãos, termine-os:**

        <div class="termy">
        ```bash
        $ aws emr terminate-clusters --cluster-ids j-XXXXXXXXX --profile dataeng
        ```
        </div>

    1. **Limpe os dados do S3:**

        <div class="termy">
        ```bash
        $ aws s3 rm s3://meu-bucket-emr-INSPER_USERNAME --recursive --profile dataeng
        $ aws s3 rb s3://meu-bucket-emr-INSPER_USERNAME --profile dataeng
        ```
        </div>

## Conclusão

Nesta aula, exploramos como combinar a orquestração do **Apache Airflow** com a capacidade de processamento em escala do **Amazon EMR**. Esta combinação é especialmente útil para casos de uso de **Big Data** em produção!

!!! info "Arquitetura híbrida"
    Esta arquitetura é um exemplo de uma solução que poderia ser escolhida para ambientes híbridos, onde o **Airflow** gerencia a orquestração *on-premises*, enquanto o **EMR** lida com o processamento pesado na nuvem.

    Isso permite aproveitar o melhor dos dois mundos: controle local e poder de processamento escalável.

    Uma opção alternativa seria rodar o **Airflow** também na nuvem (ex: **EC2** ou **MWAA** - *Managed Workflows for Apache Airflow*)!

Caso queira se aprofundar, considere estudar a integração do **Airflow** com outros serviços: **Lambda functions**, **Step Functions**, **Glue**.
