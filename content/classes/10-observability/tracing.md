# Tracing com Jaeger

## Distributed Tracing com Jaeger

Nesta seção, vamos abordar o terceiro pilar de observabilidade: **Rastreamento Distribuído (Distributed Tracing)**.

Em sistemas modernos, uma única requisição ou um fluxo de dados pode atravessar dezenas de serviços antes de ser concluído. Um pipeline de dados pode envolver uma API de ingestão, uma fila de mensagens, um processador de *streaming* e, finalmente, um *data warehouse*.

!!! note "Como?"
    Se a latência aumentar, como saber qual componente é o gargalo?
    
    Se ocorrer um erro em uma etapa intermediária, como rastrear a causa raiz a partir da requisição original?

É aqui que o **Distributed Tracing** se torna indispensável. Ele nos permite seguir o caminho completo de uma requisição (um *trace*) à medida que ela passa por múltiplos serviços. Cada unidade de trabalho dentro de um *trace* é chamada de *span*.

!!! important "Conceitos Chave"
    - **Trace**: Representa a jornada completa de uma requisição através de um sistema distribuído. É composto por um ou mais *spans*.
    - **Span**: Representa uma única unidade de trabalho ou operação dentro de um *trace* (ex: uma chamada de banco de dados, uma requisição HTTP). Cada *span* tem um nome, um horário de início e uma duração.
    - **Context Propagation**: O mecanismo que permite que o ID do *trace* e do *span* pai seja passado de um serviço para outro, conectando os *spans* em um único *trace*.

Para a parte prática, usaremos o **Jaeger** (*Jaeger: open source, distributed tracing platform*), um sistema de *tracing* de código aberto criado pela **Uber** e agora um projeto da *Cloud Native Computing Foundation* (**CNCF**).

## Configurando o Ambiente

Para demonstrar o *tracing*, precisamos de uma aplicação com múltiplos serviços. Criaremos uma aplicação simples com dois serviços: um `frontend` que recebe requisições e um `backend` que realiza uma operação.

!!! exercise "Exercício"
    Crie uma nova pasta para esta aula e navegue até ela:

    <div class="termy">

    ```
    $ mkdir -p 11-observability/02-tracing
    $ cd 11-observability/02-tracing
    ```

    </div>

### Instrumentando a Aplicação com OpenTelemetry

Para que uma aplicação gere *traces*, ela precisa ser instrumentada. Usaremos o **OpenTelemetry**, um padrão aberto para instrumentação de telemetria (métricas, logs e *traces*).

!!! exercise "Exercício"
    Crie uma pasta `app` (`11-observability/02-tracing/app`) e, dentro dela, crie dois arquivos: `frontend.py` e `backend.py`.

??? "Código para `app/frontend.py`"
    ```python { .copy }
    from fastapi import FastAPI, Query
    import httpx
    from opentelemetry import trace
    from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
    from opentelemetry.instrumentation.httpx import HTTPXClientInstrumentor
    from opentelemetry.sdk.trace import TracerProvider
    from opentelemetry.sdk.trace.export import BatchSpanProcessor
    from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
    from opentelemetry.sdk.resources import Resource

    # Configuração do OTLP para Jaeger
    otlp_exporter = OTLPSpanExporter(
        endpoint="http://jaeger:4317",
        insecure=True
    )

    # Configuração do OpenTelemetry
    resource = Resource.create({"service.name": "frontend-service"})
    provider = TracerProvider(resource=resource)
    processor = BatchSpanProcessor(otlp_exporter)
    provider.add_span_processor(processor)
    trace.set_tracer_provider(provider)

    tracer = trace.get_tracer(__name__)

    app = FastAPI(title="Frontend Service")
    FastAPIInstrumentor.instrument_app(app)
    HTTPXClientInstrumentor().instrument()

    @app.get("/")
    async def hello(name: str = Query(default="Mundo", description="Nome para cumprimentar")):
        with tracer.start_as_current_span("frontend-request") as span:
            span.set_attribute("user.name", name)

            # Chama o serviço de backend
            async with httpx.AsyncClient() as client:
                response = await client.get(f"http://backend:5001/api?name={name}")
                backend_response = response.text

            span.set_attribute("backend.response", backend_response)
            return f"Olá, {backend_response}!"
    ```

??? "Código para `app/backend.py`"
    ```python { .copy }
    from fastapi import FastAPI, Query
    import time
    import random
    from opentelemetry import trace
    from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
    from opentelemetry.sdk.trace import TracerProvider
    from opentelemetry.sdk.trace.export import BatchSpanProcessor
    from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
    from opentelemetry.sdk.resources import Resource

    # Configuração do OTLP para Jaeger
    otlp_exporter = OTLPSpanExporter(
        endpoint="http://jaeger:4317",
        insecure=True
    )

    # Configuração do OpenTelemetry
    resource = Resource.create({"service.name": "backend-service"})
    provider = TracerProvider(resource=resource)
    processor = BatchSpanProcessor(otlp_exporter)
    provider.add_span_processor(processor)
    trace.set_tracer_provider(provider)

    tracer = trace.get_tracer(__name__)

    app = FastAPI(title="Backend Service")
    FastAPIInstrumentor.instrument_app(app)

    @app.get("/api")
    async def api(name: str = Query(..., description="Nome para processar")):
        with tracer.start_as_current_span("backend-processing") as span:
            span.set_attribute("request.name", name)

            # Simula algum trabalho
            time.sleep(random.uniform(0.1, 0.5))

            return name.capitalize()
    ```

!!! exercise "Exercício"
    Crie um arquivo `app/requirements.txt` com as dependências Python:

    ```txt { .copy }
    fastapi==0.116.2
    uvicorn==0.35.0
    httpx==0.28.1
    opentelemetry-api==1.37.0
    opentelemetry-sdk==1.37.0
    opentelemetry-instrumentation-fastapi==0.58b0
    opentelemetry-instrumentation-httpx==0.58b0
    opentelemetry-exporter-otlp==1.37.0
    ```

!!! exercise "Exercício"
    Crie um `Dockerfile` na raiz do projeto (`11-observability/02-tracing/Dockerfile`):

    ```dockerfile { .copy }
    FROM python:3.12-slim

    WORKDIR /app

    COPY app/requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt

    COPY app/ .

    # A variável de ambiente CMD_TO_RUN será passada pelo docker-compose
    CMD ["python", "-u"]
    ```

### Docker Compose com Jaeger

Agora, vamos criar o `docker-compose.yml` para orquestrar o Jaeger e nossos dois serviços.

!!! exercise "Exercício"
    Crie o arquivo `docker-compose.yml`:

    ```yaml { .copy }
    services:
      jaeger:
        image: jaegertracing/all-in-one:1.73.0
        container_name: jaeger
        ports:
          - "6831:6831/udp" # Agent
          - "16686:16686"   # UI

      frontend:
        build: .
        container_name: frontend
        ports:
          - "5000:5000"
        command: ["uvicorn", "frontend:app", "--host", "0.0.0.0", "--port", "5000"]
        depends_on:
          - jaeger
          - backend

      backend:
        build: .
        container_name: backend
        ports:
          - "5001:5001"
        command: ["uvicorn", "backend:app", "--host", "0.0.0.0", "--port", "5001"]
        depends_on:
          - jaeger
    ```

## Gerando e Visualizando Traces

!!! exercise "Exercício"
    Inicie todos os serviços:

    <div class="termy">

    ```
    $ docker compose build
    $ docker compose up
    ```

    </div>

!!! exercise "Exercício"
    Em um novo terminal, envie algumas requisições para o serviço de `frontend`:

    !!! warning "Atenção"
        Se não tiver o `curl` instalado, utilize seu navegador para acessar `http://localhost:5000/?name=ana` e `http://localhost:5000/?name=maria`.

    <div class="termy">

    ```
    $ curl "http://localhost:5000/?name=ana"
    Olá, Ana!
    $ curl "http://localhost:5000/?name=maria"
    Olá, Maria!
    ```

    </div>

!!! exercise "Exercício"
    Agora, acesse a interface do Jaeger em [http://localhost:16686](http://localhost:16686).

    1.  No menu **Search**, selecione o serviço `frontend` no dropdown **Service**.
    1.  Clique em **Find Traces**. Você deve ver os *traces* correspondentes às suas requisições `curl`.
    1.  Clique em um dos *traces* para ver os detalhes.

!!! exercise text long "Analisando o Trace"
    Na visualização do *trace*, você verá um gráfico de Gantt. Observe a hierarquia:

    - O *span* raiz é do serviço `frontend`.
    - Dentro dele, há um *span* filho para a requisição HTTP feita ao `backend`.
    - O serviço `backend` cria seu próprio *span* para o processamento.

    Clique em cada *span* e explore as abas **Tags** e **Process**. Que informações úteis você consegue encontrar? Consegue ver os atributos que definimos no código, como `user.name`?

## Limpeza

!!! exercise "Exercício"
    Quando terminar, pare os contêineres:

    <div class="termy">

    ```
    $ docker compose down
    ```

    </div>

Com a adição do *tracing*, você agora tem uma visão completa dos três pilares da observabilidade, permitindo não apenas monitorar a saúde do sistema, mas também depurar problemas em ambientes distribuídos.

## Prática: Orquestração e Tracing com Prefect e Jaeger

Nesta seção, orquestraremos um pipeline de **ETL** usando **Prefect** e, ao mesmo tempo, geraremos *traces* para cada etapa usando **OpenTelemetry** e **Jaeger**.

Isso nos dará uma visão onde utilizamos:

- **Prefect**: Para gerenciar o fluxo, dependências, *retries* e agendamento.
- **Jaeger**: Para visualizar a latência de cada tarefa e entender os gargalos de desempenho.

### Configurando o Ambiente

!!! exercise "Exercício"
    Crie uma nova pasta para esta prática e navegue até ela:

    <div class="termy">

    ```
    $ mkdir -p 11-observability/03-prefect-tracing
    $ cd 11-observability/03-prefect-tracing
    ```

    </div>

### Componentes

Nossa prática terá os seguintes componentes:

1.  **Jaeger**: Nosso backend de *tracing*.
2.  **Prefect Server**: O servidor que gerencia os *flows*.
3.  **ETL Runner**: Um contêiner que executa nosso *flow* de **ETL** escrito em Python com **Prefect** e instrumentado com **OpenTelemetry**.

**Código para `docker-compose.yml`**:
```yaml { .copy }
services:
  jaeger:
    image: jaegertracing/all-in-one:1.73.0
    container_name: jaeger
    ports:
      - "6831:6831/udp" # Agent
      - "16686:16686"   # UI

  prefect-server:
    image: prefecthq/prefect:3.4-python3.12
    container_name: prefect_server
    command: ["prefect", "server", "start"]
    environment:
      - PREFECT_SERVER_API_HOST=0.0.0.0
      - PREFECT_API_URL=http://0.0.0.0:4200/api
      - PREFECT_API_DATABASE_CONNECTION_URL=sqlite+aiosqlite:///prefect.db
    ports:
      - "4200:4200"
    volumes:
      - prefect_data:/root/.prefect
    user: "0:0"
    healthcheck:
      test: ["CMD", "sleep", "3"]
      interval: 1s
      timeout: 5s
      retries: 1
      start_period: 5s

  etl-runner:
    build: .
    container_name: etl-runner
    command: python etl_flow.py
    environment:
      - PREFECT_API_URL=http://prefect-server:4200/api
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://jaeger:4317
    depends_on:
      prefect-server:
        condition: service_healthy
      jaeger:
        condition: service_started
    restart: unless-stopped

volumes:
  prefect_data:
```

!!! exercise "Exercício"
    Crie o arquivo `docker-compose.yml`

!!! exercise "Exercício"
    Crie o arquivo `etl_flow.py` que contém nosso *pipeline* de **ETL**:

    ??? "Código para `etl_flow.py`"
        ```python { .copy }
        from prefect import flow, task, get_run_logger
        from opentelemetry import trace
        import time
        import random
        import os

        from opentelemetry.sdk.trace import TracerProvider
        from opentelemetry.sdk.trace.export import BatchSpanProcessor
        from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
        from opentelemetry.sdk.resources import Resource

        # Configuração do OpenTelemetry
        resource = Resource.create({"service.name": "prefect-etl-service"})
        provider = TracerProvider(resource=resource)

        # A URL do endpoint do Jaeger é passada via variável de ambiente
        otlp_exporter = OTLPSpanExporter(
            endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT", "http://localhost:4317"),
            insecure=True
        )

        provider.add_span_processor(BatchSpanProcessor(otlp_exporter))
        trace.set_tracer_provider(provider)

        tracer = trace.get_tracer(__name__)

        @task
        def extract_data():
            """Simula a extração de dados de uma fonte."""
            logger = get_run_logger()
            with tracer.start_as_current_span("extract-data-span") as span:
                logger.info("Iniciando extração de dados...")
                wait_time = random.uniform(1, 3)
                time.sleep(wait_time)
                data = [{"id": i, "value": random.random() * 100} for i in range(10)]
                span.set_attribute("extracted.records", len(data))
                logger.info(f"Extração concluída. {len(data)} registros encontrados.")
                return data

        @task
        def transform_data(data):
            """Simula a transformação dos dados."""
            logger = get_run_logger()
            with tracer.start_as_current_span("transform-data-span") as span:
                logger.info("Iniciando transformação de dados...")
                wait_time = random.uniform(2, 5)
                time.sleep(wait_time)
                transformed_data = []
                for record in data:
                    record['value'] = record['value'] * 2
                    transformed_data.append(record)
                span.set_attribute("transformed.records", len(transformed_data))
                logger.info("Transformação concluída.")
                return transformed_data

        @task
        def load_data(data):
            """Simula o carregamento dos dados em um destino."""
            logger = get_run_logger()
            with tracer.start_as_current_span("load-data-span") as span:
                logger.info("Iniciando carregamento de dados...")
                wait_time = random.uniform(1, 4)
                time.sleep(wait_time)
                span.set_attribute("loaded.records", len(data))
                logger.info(f"Carregamento concluído. {len(data)} registros carregados.")
                return {"status": "success", "records_loaded": len(data)}

        @flow(name="ETL com Tracing")
        def etl_flow():
            """Um flow de ETL simples com tracing integrado."""
            logger = get_run_logger()
            logger.info("Iniciando o pipeline de ETL...")
            
            extracted = extract_data()
            transformed = transform_data(extracted)
            load_result = load_data(transformed)
            
            logger.info(f"Pipeline de ETL concluído com sucesso: {load_result}")
            return load_result

        if __name__ == "__main__":
            etl_flow()
        ```

!!! exercise "Exercício"
    Crie o `requirements.txt` para as dependências:

    ```txt { .copy }
    prefect==3.4.18
    opentelemetry-api==1.37.0
    opentelemetry-sdk==1.37.0
    opentelemetry-exporter-otlp==1.37.0
    ```

!!! exercise "Exercício"
    Finalmente, crie o `Dockerfile` para construir a imagem do nosso `etl-runner`:

    ```dockerfile { .copy }
    FROM python:3.12-slim

    WORKDIR /app

    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt

    COPY etl_flow.py .

    CMD ["python", "-u", "etl_flow.py"]
    ```

### Executando e Observando

!!! exercise "Exercício"
    Inicie todos os serviços:

    <div class="termy">

    ```
    $ docker compose build
    $ docker compose up
    ```

    </div>

!!! exercise "Exercício"
    Aguarde alguns segundos e verifique os logs do `etl-runner` para ver a execução do flow:

    <div class="termy">

    ```
    $ docker logs etl-runner -f
    ```

    </div>

!!! exercise "Exercício"
    Agora, vamos observar os resultados nas interfaces web:

    1.  **Prefect UI**: Acesse [http://localhost:4200](http://localhost:4200).
        -   Vá para a aba **Flow Runs**. Você deve ver a execução do `ETL com Tracing`.
        -   Clique na execução para ver o **DAG**, os logs de cada tarefa e os tempos de execução.

    1.  **Jaeger UI**: Acesse [http://localhost:16686](http://localhost:16686).
        -   No menu **Search**, selecione o serviço `prefect-etl-service`.
        -   Clique em **Find Traces**. Você verá o *trace* da sua execução de ETL.
        -   Clique no *trace* para ver o detalhamento.

!!! exercise "Exercício"
    Com as duas UIs abertas lado a lado, analise a execução:

    -   No **Prefect**, você vê a estrutura lógica do *flow* e o status (sucesso, falha).
    -   No **Jaeger**, você vê a performance real, com um gráfico de Gantt mostrando exatamente onde o tempo foi gasto.
    
    Como a combinação dessas duas ferramentas fornece uma visão mais completa do que cada uma individualmente? Em que cenário o Jaeger seria mais útil? E o Prefect?

### Limpeza

!!! exercise "Exercício"
    Quando terminar, pare e remova os contêineres:

    <div class="termy">

    ```
    $ docker compose down -v
    ```

    </div>
