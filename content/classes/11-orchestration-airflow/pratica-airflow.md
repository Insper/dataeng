# Pipeline com Airflow

Nesta seção, vamos utilizar **Airflow** para construir, agendar e monitorar um *pipeline* de dados.

## Ambiente de Desenvolvimento

!!! exercise "Exercício"
    Crie uma pasta `airflow`.

    Dentro deste diretório, crie as seguintes subpastas:

    - `dags/`: É aqui que vamos colocar nossos arquivos Python que definem os DAGs. O Airflow irá escanear esta pasta automaticamente.
    - `logs/`: Para armazenar os logs de execução das tarefas.
    - `plugins/`: Para adicionar plugins customizados (não usaremos hoje, mas é bom saber que existe).
    - `config/`: Para configurações adicionais.

    <div class="termy">
    ```bash
    $ mkdir airflow
    $ cd airflow
    $ mkdir -p ./dags ./logs ./plugins ./config
    ```
    </div>

!!! exercise
    Defina a variável de ambiente `AIRFLOW_UID` com o ID do seu usuário atual. Isso ajuda a evitar problemas de permissão com os arquivos criados pelo Airflow dentro dos containers Docker.

    === "Linux/Mac"

        <div class="termy">
        ```bash
        $ echo -e "AIRFLOW_UID=$(id -u)" > .env
        ```
        </div>

    === "Windows"
        Crie um arquivo `.env` na raiz do projeto com o seguinte conteúdo:

        !!! info "Info"
            Caso esteja no Windows, você pode definir qualquer número para o `AIRFLOW_UID`, como `50000`.

        ```text { .copy }
        AIRFLOW_UID=50000
        ```

!!! exercise "Exercício"
    A comunidade Airflow mantém uma imagem oficial e um arquivo `docker-compose.yml` que configura todos os componentes necessários
    
    Baixe o arquivo `docker-compose.yml` oficial. Você pode fazer isso via `curl` ou copiar o **yaml** disponível na sequência:

    <div class="termy">
    ```bash
    $ curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.0.6/docker-compose.yaml'
    ```
    </div>

    Para saber mais, verifique a [documentação oficial](https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html)

`docker-compose.yml`:
```yaml { .copy }
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

# Basic Airflow cluster configuration for CeleryExecutor with Redis and PostgreSQL.
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
    AIRFLOW__CORE__EXECUTOR: CeleryExecutor
    AIRFLOW__CORE__AUTH_MANAGER: airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager
    AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
    AIRFLOW__CELERY__BROKER_URL: redis://:@redis:6379/0
    AIRFLOW__CORE__FERNET_KEY: ''
    AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
    AIRFLOW__CORE__LOAD_EXAMPLES: 'true'
    AIRFLOW__CORE__EXECUTION_API_SERVER_URL: 'http://airflow-apiserver:8080/execution/'
    # yamllint disable rule:line-length
    # Use simple http server on scheduler for health checks
    # See https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/logging-monitoring/check-health.html#scheduler-health-check-server
    # yamllint enable rule:line-length
    AIRFLOW__SCHEDULER__ENABLE_HEALTH_CHECK: 'true'
    # WARNING: Use _PIP_ADDITIONAL_REQUIREMENTS option ONLY for a quick checks
    # for other purpose (development, test and especially production usage) build/extend Airflow image.
    _PIP_ADDITIONAL_REQUIREMENTS: ${_PIP_ADDITIONAL_REQUIREMENTS:-}
    # The following line can be used to set a custom config file, stored in the local config folder
    AIRFLOW_CONFIG: '/opt/airflow/config/airflow.cfg'
  volumes:
    - ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags
    - ${AIRFLOW_PROJ_DIR:-.}/logs:/opt/airflow/logs
    - ${AIRFLOW_PROJ_DIR:-.}/config:/opt/airflow/config
    - ${AIRFLOW_PROJ_DIR:-.}/plugins:/opt/airflow/plugins
  user: "${AIRFLOW_UID:-50000}:0"
  depends_on:
    &airflow-common-depends-on
    redis:
      condition: service_healthy
    postgres:
      condition: service_healthy

services:
  postgres:
    image: postgres:13
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

  redis:
    # Redis is limited to 7.2-bookworm due to licencing change
    # https://redis.io/blog/redis-adopts-dual-source-available-licensing/
    image: redis:7.2-bookworm
    expose:
      - 6379
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 30s
      retries: 50
      start_period: 30s
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

  airflow-worker:
    <<: *airflow-common
    command: celery worker
    healthcheck:
      # yamllint disable rule:line-length
      test:
        - "CMD-SHELL"
        - 'celery --app airflow.providers.celery.executors.celery_executor.app inspect ping -d "celery@$${HOSTNAME}" || celery --app airflow.executors.celery_executor.app inspect ping -d "celery@$${HOSTNAME}"'
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s
    environment:
      <<: *airflow-common-env
      # Required to handle warm shutdown of the celery workers properly
      # See https://airflow.apache.org/docs/docker-stack/entrypoint.html#signal-propagation
      DUMB_INIT_SETSID: "0"
    restart: always
    depends_on:
      <<: *airflow-common-depends-on
      airflow-apiserver:
        condition: service_healthy
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

  # You can enable flower by adding "--profile flower" option e.g. docker-compose --profile flower up
  # or by explicitly targeted on the command line e.g. docker-compose up flower.
  # See: https://docs.docker.com/compose/profiles/
  flower:
    <<: *airflow-common
    command: celery flower
    profiles:
      - flower
    ports:
      - "5555:5555"
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:5555/"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s
    restart: always
    depends_on:
      <<: *airflow-common-depends-on
      airflow-init:
        condition: service_completed_successfully

volumes:
  postgres-db-volume:    
```

!!! exercise "Exercício"
    !!! info "Importante:"
        Antes de iniciar, precisamos inicializar o ambiente.

        Isso criará o usuário administrador e as estruturas de banco de dados necessárias. 
        
    Rode o seguinte comando pela primeira vez:

    <div class="termy">
    ```bash
    $ docker compose up airflow-init
    ```
    </div>

!!! exercise "Exercício"
    Agora, inicie o Airflow:

    <div class="termy">
    ```bash
    $ docker compose up
    ```
    </div>

    Isso iniciará todos os serviços.
    
    !!! warning "Atencão!"
        Pode levar alguns minutos.
    
    Quando estiver pronto, você poderá acessar a interface do Airflow em [http://localhost:8080](http://localhost:8080).

    -   **Login:** `airflow`
    -   **Senha:** `airflow`

Você deverá ver a tela principal do Airflow, com vários **DAGs** de exemplo.

## Criar DAG

Vamos criar um **DAG** para entender a estrutura básica do **Airflow**.

!!! exercise "Exercício"
    Dentro da sua pasta `dags/`, crie um arquivo chamado `meu_primeiro_dag.py` com o seguinte código:

    ```python { .copy }
    from __future__ import annotations

    import pendulum

    from airflow.models.dag import DAG
    from airflow.operators.bash import BashOperator

    # Define o DAG
    with DAG(
        dag_id="meu_primeiro_dag",
        start_date=pendulum.datetime(2026, 1, 1, tz="UTC"),
        schedule="@daily",
        catchup=False,
        tags=["exemplo"],
    ) as dag:
        # Define a primeira tarefa
        tarefa_1 = BashOperator(
            task_id="diz_ola",
            bash_command="echo 'Olá Airflow!'",
        )

        # Define a segunda tarefa
        tarefa_2 = BashOperator(
            task_id="data_atual",
            bash_command="date",
        )

        # Define a ordem de execução
        tarefa_1 >> tarefa_2
    ```

??? "Entendendo o Código"
    - `with DAG(...) as dag:`: Este é o gerenciador de contexto que define nosso **DAG**.
        - `dag_id`: O nome único do seu **DAG**.
        - `start_date`: A data a partir da qual o **DAG** pode começar a ser agendado. É uma boa prática usar uma data no passado.
        - `schedule`: Com que frequência o **DAG** deve rodar. Usamos `@daily`, mas poderia ser uma expressão **cron** como `'0 0 * * *'`. Veja mais na Documentação do [Scheduler](https://airflow.apache.org/docs/apache-airflow/3.0.6/administration-and-deployment/scheduler.html) e [Dag Runs](https://airflow.apache.org/docs/apache-airflow/3.0.6/core-concepts/dag-run.html).
        - `catchup=False`: Se `True`, o Airflow tentaria rodar o **DAG** para todos os intervalos de agendamento perdidos desde a `start_date`. Geralmente, mantemos `False` durante o desenvolvimento.
    - `BashOperator`: Usamos este operador para executar comandos no *shell*.
        - `task_id`: O nome único da tarefa dentro do **DAG**.
        - `bash_command`: O comando a ser executado.
    - `tarefa_1 >> tarefa_2`: Esta é a forma concisa de definir a dependência. Dizemos que `tarefa_2` só pode começar depois que `tarefa_1` for concluída.

### Visualizando na UI

Volte para a UI do Airflow ([http://localhost:8080](http://localhost:8080)).

!!! exercise "Exercício"
    Aguarde e atualize a página.

    Você deverá ver `meu_primeiro_dag` na lista (para facilitar, utilize o menu de busca).
    
    !!! info "Info"
        Pode levar um minuto para o *Scheduler* detectar o novo arquivo.

    Caso não veja, verifique os logs do *Scheduler* para identificar possíveis erros.

!!! exercise "Exercício"
    Clique no **DAG** `meu_primeiro_dag` e clique no Botão **Trigger** para que seja executado manualmente.

    Após, explore as opções **Grid** e **Graph** (abaixo do nome do **DAG**) e as abas **Overview**, **Runs**, **Tasks**, **Backfills**, **Events**, **Code**, **Details**.

    !!! tip "Dica!"
        A visão **Graph** é especialmente útil para visualizar as dependências.

!!! exercise "Exercício"
    Altere o arquivo `meu_primeiro_dag.py` para alterar o `schedule` para `*/2 * * * *` (a cada dois minutos) e salve.

!!! exercise "Exercício"
    Ative o **DAG** clicando no *toggle button* à direita do nome `meu_primeiro_dag`.

    Aguarde alguns minutos para o **DAG** ser executado automaticamente.

    Após, explore as opções **Grid** e **Graph** (abaixo do nome do **DAG**) e as abas **Overview**, **Runs**, **Tasks**, **Backfills**, **Events**, **Code**, **Details**.

!!! exercise "Exercício"
    Na **UI**, com seu **DAG** selecionado na visão **Grid**, clique no botão **Trigger** novamente (execução manual).
    
    Observe o que acontece na visão **Grid**. As tarefas devem mudar de cor à medida que são executadas.

    Clique em uma das tarefas executadas (um quadrado verde) e depois em **Logs** para ver a saída do seu comando `echo` ou `date`.

!!! exercise "Exercício"
    Repita o exercício anterior, agora na visão **Graph**.

## Pipeline de Dados

Agora, vamos criar um **DAG** um pouco mais realista, simulando um processo de **ETL**.

Nosso *pipeline* fará o seguinte:

1.  **Extract:** Baixar dados de uma API pública (usaremos a [JSONPlaceholder API](https://jsonplaceholder.typicode.com/)).
2.  **Transform:** Processar o JSON para extrair apenas os campos de interesse.
3.  **Load:** Salvar os dados processados em um arquivo.

Para isso, usaremos a **TaskFlow API**, que é uma forma mais moderna e "Pythonica" de escrever DAGs.

!!! exercise "Exercício"
    Crie um novo arquivo em sua pasta `dags/` chamado `pipeline_dados_ex.py`.

    ??? "`pipeline_dados_ex.py`"

        ```python
        from __future__ import annotations

        import json
        import pendulum

        from airflow.models.dag import DAG
        from airflow.decorators import task

        @task
        def extrair_dados():
            """
            Simula a extração de dados de uma API.
            Retorna uma lista de dicionários.
            """
            import requests
            
            url = "https://jsonplaceholder.typicode.com/users"
            response = requests.get(url)
            response.raise_for_status()  # Lança um erro se a requisição falhar
            return response.json()

        @task
        def transformar_dados(users: list[dict]):
            """
            Transforma os dados, selecionando apenas alguns campos.
            """
            usuarios_transformados = []
            for user in users:
                usuario = {
                    "nome": user["name"],
                    "email": user["email"],
                    "cidade": user["address"]["city"]
                }
                usuarios_transformados.append(usuario)
            return usuarios_transformados

        @task
        def carregar_dados(usuarios_transformados: list[dict]):
            """
            Salva os dados transformados em um arquivo JSON.
            """
            with open("/opt/airflow/dags/usuarios.json", "w") as f:
                json.dump(usuarios_transformados, f, indent=4)
            print(f"{len(usuarios_transformados)} usuários salvos com sucesso.")


        with DAG(
            dag_id="pipeline_dados_ex",
            start_date=pendulum.datetime(2026, 1, 1, tz="UTC"),
            schedule=None, # Não agendado, apenas manual
            catchup=False,
            tags=["exemplo", "etl"],
        ) as dag:
            dados_brutos = extrair_dados()
            dados_transformados = transformar_dados(dados_brutos)
            carregar_dados(dados_transformados)
        ```

??? "Entendendo a TaskFlow API"
    -   `@task`: Decorador que transforma uma função Python em uma tarefa do **Airflow**!
    -   **Passagem de Dados (XComs):** Perceba que não estamos salvando arquivos intermediários. O retorno da função `extrair_dados()` (`dados_brutos`) é passado diretamente como argumento para a função `transformar_dados()`. O **Airflow** gerencia essa passagem de dados nos bastidores usando um mecanismo chamado **XComs** (*cross-communications*).
    -   **Dependências Implícitas:** O **Airflow** infere automaticamente as dependências. Como `transformar_dados` usa o resultado de `extrair_dados`, o **Airflow** sabe que `extrair_dados` deve ser executado primeiro. A sintaxe fica muito mais limpa!

### Testando o Pipeline

!!! exercise "Exercício"
    Acesse a **UI** do **Airflow** (em modo **Graph view**).

    Encontre o **DAG** `pipeline_dados_ex` e execute-o manualmente.

!!! exercise "Exercício"
    Observe o grafo de execução.

    Você verá as três tarefas rodando em sequência.

!!! exercise "Exercício"
    Verifique os logs da tarefa `carregar_dados`.

    Você deve ver a mensagem de sucesso.

!!! info "Info!"
    O arquivo `usuarios.json` foi salvo na pasta `dags/` dentro do container do Airflow.

    Como esta pasta está mapeada para a pasta `dags/` do seu host, você pode encontrá-lo lá! Confira o conteúdo do arquivo.

!!! exercise "Exercício"
    Modifique o DAG `pipeline_dados_ex` para adicionar uma quarta tarefa.

    1.  Crie uma nova função decorada com `@task` chamada `contar_usuarios`.
    1.  Esta função deve receber a lista de usuários transformados.
    1.  Ela deve simplesmente retornar uma string formatada, como: `"Total de usuários processados: 10"`.
    1.  Chame esta função no final do seu DAG.
    1.  Verifique o resultado nos logs da nova tarefa.

    !!! answer "Resposta"

        ```python { .copy }
        # ... (código anterior) ...

        @task
        def contar_usuarios(usuarios_transformados: list[dict]):
            """
            Conta o número de usuários e retorna uma mensagem.
            """
            total = len(usuarios_transformados)
            return f"Total de usuários processados: {total}"

        # ... (dentro do with DAG(...)) ...
        dados_brutos = extrair_dados()
        dados_transformados = transformar_dados(dados_brutos)
        carregar_dados(dados_transformados)
        mensagem_final = contar_usuarios(dados_transformados) # Adicione esta linha
        ```

        Note que `carregar_dados` e `contar_usuarios` podem rodar em paralelo, pois ambas dependem de `transformar_dados`, mas não uma da outra. O Airflow gerencia isso para você!

Por hoje é só!
