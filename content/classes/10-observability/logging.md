# Logs com Loki

## Introdução

Até agora, exploramos as **métricas** com Prometheus e a **visualização** com Grafana. Falta um pilar crucial da observabilidade: os **Logs**.

Enquanto as métricas nos dão uma visão agregada e quantitativa do sistema, os logs nos fornecem um registro detalhado, evento por evento, do que está acontecendo. Eles são indispensáveis para a depuração e a análise de causa raiz.

Um log é um registro imutável e com carimbo de data/hora de um evento. Em um sistema distribuído, como um pipeline de dados moderno, os logs são gerados por múltiplos componentes (orquestradores, APIs, bancos de dados, etc.). Centralizar esses logs em um único local para busca e análise é fundamental.

Para esta tarefa, usaremos o **Loki**, um sistema de agregação de logs inspirado no Prometheus.

!!! info "Conheça o Loki"
    O **Loki** é um agregador de logs *multitenant*, altamente escalável e econômico. Desenvolvido pela Grafana Labs, ele se diferencia de outras soluções por não indexar o conteúdo completo dos logs. Em vez disso, ele indexa apenas um pequeno conjunto de metadados (rótulos ou *labels*), assim como o Prometheus. Isso o torna mais simples de operar e mais barato para armazenar grandes volumes de logs.

A "pilha" Loki consiste em:

-   **Loki**: O servidor principal que armazena e processa os logs.
-   **Promtail**: O agente responsável por coletar os logs e enviá-los para o Loki.
-   **Grafana**: A interface de visualização para consultar e exibir os logs (sim, a mesma que usamos para métricas!).

## Configurando o Ambiente

Vamos adicionar Loki e Promtail à nossa stack de observabilidade.

!!! warning "Atenção!"
    Continue trabalhando no diretório `11-observability/01-prometheus` da página anterior.

!!! exercise "Exercício"
    Crie um arquivo de configuração para o **Promtail** chamado `promtail-config.yml`:

    ```yaml { .copy }
    server:
      http_listen_port: 9080
      grpc_listen_port: 0

    positions:
      filename: /tmp/positions.yaml

    clients:
      - url: http://loki:3100/loki/api/v1/push

    scrape_configs:
    - job_name: api_logs
      static_configs:
        - targets:
            - localhost
          labels:
            job: api
            service: avatar-api
            __path__: /app/logs/*.log
    ```

!!! exercise text long "Exercício"
    Analise o arquivo `promtail-config.yml`. O que a seção `scrape_configs` está instruindo o Promtail a fazer? De onde ele coletará os logs? Note que agora estamos focando especificamente nos logs da API ao invés de todos os contêineres Docker.

!!! exercise "Exercício"
    Agora, atualize o `docker-compose.yml` para incluir os serviços `loki`, `promtail` e nossa API de exemplo:

    ```yaml { .copy }
    services:
      prometheus:
        # ... (configuração existente)

      node-exporter:
        # ... (configuração existente)

      grafana:
        # ... (configuração existente)

      avatar-api:
        image: python:3.12-slim
        container_name: avatar-api
        ports:
          - "8000:8000"
        volumes:
          - ./api_avatar:/app
          - ./logs:/app/logs
        working_dir: /app
        command: bash -c "pip install fastapi uvicorn && python main.py"
        depends_on:
          - loki

      loki:
        image: grafana/loki:3.5
        container_name: loki
        ports:
          - "3100:3100"
        command: -config.file=/etc/loki/local-config.yaml

      promtail:
        image: grafana/promtail:3.5
        container_name: promtail
        volumes:
          - ./promtail-config.yml:/etc/promtail/config.yml
          - ./logs:/app/logs
        command: -config.file=/etc/promtail/config.yml
        depends_on:
          - avatar-api
    ```

!!! warning "Atenção"
    O volume `- ./logs:/app/logs` compartilha uma pasta de logs entre a **API** e o **Promtail**, permitindo que o **Promtail** colete especificamente os logs gerados pela nossa aplicação.

## Instrumentando a API para Logs Estruturados

Antes de executar nossa stack, vamos criar uma API simples que gera logs estruturados e informativos.

!!! exercise "Exercício"
    Crie uma pasta `api_avatar` no diretório atual e dentro dela crie o arquivo `main.py`:

    ```python { .copy }
    import json
    import logging
    import time
    from datetime import datetime
    from fastapi import FastAPI, HTTPException, Request
    from typing import Dict, Any
    import uvicorn
    import os

    # Configuração de logs estruturados
    class JSONFormatter(logging.Formatter):
        def format(self, record):
            log_entry = {
                "timestamp": datetime.utcnow().isoformat() + "Z",
                "level": record.levelname,
                "logger": record.name,
                "message": record.getMessage(),
            }
            
            # Adiciona informações extras se disponíveis
            if hasattr(record, 'user_id'):
                log_entry['user_id'] = record.user_id
            if hasattr(record, 'endpoint'):
                log_entry['endpoint'] = record.endpoint
            if hasattr(record, 'response_time'):
                log_entry['response_time_ms'] = record.response_time
            if hasattr(record, 'status_code'):
                log_entry['status_code'] = record.status_code
                
            return json.dumps(log_entry)

    # Configurar logging
    os.makedirs('/app/logs', exist_ok=True)
    logger = logging.getLogger("avatar_api")
    logger.setLevel(logging.INFO)

    # Handler para arquivo
    file_handler = logging.FileHandler('/app/logs/api.log')
    file_handler.setFormatter(JSONFormatter())
    logger.addHandler(file_handler)

    # Handler para console (opcional)
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(JSONFormatter())
    logger.addHandler(console_handler)

    app = FastAPI(title="Avatar API", version="1.0.0")

    # Simulação de banco de dados de usuários
    users_db = {
        "1": {"name": "Alice", "avatar": "avatar1.png", "active": True},
        "2": {"name": "Bob", "avatar": "avatar2.png", "active": True},
        "3": {"name": "Charlie", "avatar": "avatar3.png", "active": False}
    }

    @app.middleware("http")
    async def logging_middleware(request: Request, call_next):
        start_time = time.time()
        
        # Log da requisição
        logger.info(
            "Request received",
            extra={
                "endpoint": str(request.url.path),
                "method": request.method,
                "user_agent": request.headers.get("user-agent", "unknown")
            }
        )
        
        response = await call_next(request)
        
        # Calcular tempo de resposta
        process_time = (time.time() - start_time) * 1000
        
        # Log da resposta
        logger.info(
            "Request completed",
            extra={
                "endpoint": str(request.url.path),
                "status_code": response.status_code,
                "response_time": round(process_time, 2)
            }
        )
        
        return response

    @app.get("/")
    async def root():
        logger.info("Health check requested")
        return {"message": "Avatar API is running"}

    @app.get("/users/{user_id}/avatar")
    async def get_user_avatar(user_id: str):
        logger.info(f"Avatar requested", extra={"user_id": user_id})
        
        if user_id not in users_db:
            logger.warning(f"User not found", extra={"user_id": user_id})
            raise HTTPException(status_code=404, detail="User not found")
        
        user = users_db[user_id]
        
        if not user["active"]:
            logger.warning(f"User is inactive", extra={"user_id": user_id})
            raise HTTPException(status_code=403, detail="User is inactive")
        
        # Simular processamento pesado ocasionalmente
        if user_id == "2":  # Bob tem avatar pesado
            time.sleep(2)  # Simula processamento lento
            logger.info(f"Heavy processing completed", extra={"user_id": user_id})
        
        logger.info(f"Avatar served successfully", extra={"user_id": user_id})
        return {"user": user["name"], "avatar_url": f"/static/{user['avatar']}"}

    @app.get("/users/{user_id}/profile")
    async def get_user_profile(user_id: str):
        logger.info(f"Profile requested", extra={"user_id": user_id})
        
        if user_id not in users_db:
            logger.error(f"Profile request for non-existent user", extra={"user_id": user_id})
            raise HTTPException(status_code=404, detail="User not found")
        
        user = users_db[user_id]
        logger.info(f"Profile served", extra={"user_id": user_id})
        return user

    if __name__ == "__main__":
        logger.info("Starting Avatar API server")
        uvicorn.run(app, host="0.0.0.0", port=8000)
    ```

!!! exercise "Exercício" 
    Crie também a pasta `logs` no diretório atual para armazenar os logs da API:

    <div class="termy">

    ```
    $ mkdir logs
    ```

    </div>

!!! exercise text long "Exercício"
    Analise o código da API. Que tipos de logs estruturados ela está gerando? Quais informações estão sendo capturadas que podem ser úteis para observabilidade?

    **Dica**: Observe as informações extras (`extra`) que estão sendo adicionadas aos logs, como `user_id`, `endpoint`, `response_time`, etc.

## Executando a Stack Completa

!!! exercise "Exercício"
    Inicie todos os serviços com a nova configuração:

    <div class="termy">

    ```
    $ docker compose up -d
    ```

    </div>

## Explorando Logs no Grafana

Assim como fizemos com o Prometheus, precisamos adicionar o Loki como uma fonte de dados no Grafana.

!!! exercise "Exercício"
    Siga os passos para adicionar o Loki como *Data Source*:

    1.  Acesse o Grafana ([http://localhost:3000](http://localhost:3000)).
    2.  Vá para **Connections > Data sources** e clique em **Add new data source**.
    3.  Selecione **Loki** da lista.
    4.  Na URL do servidor, insira `http://loki:3100`.
    5.  Clique em **Save & test**. Você deve ver uma mensagem de sucesso.

### Consultando Logs com LogQL

Agora, vamos para a parte divertida: consultar os logs.

!!! exercise "Exercício"
    1.  No menu lateral do Grafana, clique no ícone **Explore**.
    2.  No topo da página, selecione a fonte de dados **Loki**.
    3.  No campo **Log browser**, você pode começar a explorar os rótulos. Por exemplo, clique em `job` e depois em `api` para ver todos os logs coletados.

O Loki usa uma linguagem de consulta chamada **LogQL**, que é inspirada na PromQL.

!!! exercise "Exercício"
    Antes de fazer as consultas, gere alguns dados de teste acessando a API:

    <div class="termy">

    ```
    $ curl http://localhost:8000/
    $ curl http://localhost:8000/users/1/avatar
    $ curl http://localhost:8000/users/2/avatar  # Este é mais lento
    $ curl http://localhost:8000/users/3/avatar  # Este vai falhar (usuário inativo)
    $ curl http://localhost:8000/users/999/profile  # Este vai falhar (usuário não existe)
    ```

    </div>

!!! exercise "Exercício"
    Agora tente as seguintes consultas LogQL no modo **Explore**:

    !!! warning "Atenção"
        Selecione a opção **Code** no lado direito!

    1.  Mostrar todos os logs da API:
        ```logql
        {job="api"}
        ```

    2.  Mostrar apenas logs de erro (4xx e 5xx):
        ```logql
        {job="api"} | json | status_code >= 400
        ```

    3.  Mostrar logs de requisições específicas para avatares:
        ```logql
        {job="api"} |= "avatar"
        ```

    4.  Mostrar logs com tempo de resposta alto (> 1000ms):
        ```logql
        {job="api"} | json | response_time_ms > 1000
        ```

    5.  Mostrar logs de um usuário específico:
        ```logql
        {job="api"} | json | user_id="2"
        ```

!!! exercise text long "Desafio 1: Melhorando a Instrumentação"
    Modifique o código da API para adicionar mais informações úteis aos logs (escolha uma):

    1. **Rate Limiting**: Adicione um contador de requisições por usuário e log quando um usuário fizer muitas requisições (> 10 por minuto)
    2. **Cache Simulation**: Simule um cache de avatares e log quando há cache hit/miss
    3. **Métricas de Negócio**: Log estatísticas como "avatar mais popular" ou "usuários mais ativos"

    **Dica**: Use campos extras nos logs como `cache_status`, `request_count`, `popular_avatar`, etc.

!!! exercise text long "Desafio 2: Dashboard Integrado"
    Vá para o *dashboard* "Monitoramento do Sistema" que você criou na aula anterior. Adicione painéis que combinam métricas e logs:

    1. **Painel de Logs da API**: Mostre logs em tempo real da API
    2. **Painel de Erros**: Use LogQL para contar erros por minuto: `count_over_time({job="api"} | json | status_code >= 400 [1m])`
    3. **Painel de Latência**: Use LogQL para mostrar tempos de resposta: `quantile_over_time(0.95, {job="api"} | json | unwrap response_time_ms [5m])`

    Como essa correlação entre métricas do sistema e logs da aplicação pode ajudar na depuração?

!!! exercise text long "Desafio 3: Cenários de Falha"
    Modifique a API para simular cenários reais de falha e observe os logs:

    1. **Falha Intermitente**: Faça o endpoint `/users/1/avatar` falhar 20% das vezes
    2. **Lentidão Crescente**: Faça os tempos de resposta aumentarem gradualmente ao longo do tempo
    3. **Circuit Breaker**: Implemente um circuit breaker simples que para de processar quando há muitos erros

    Observe como esses padrões aparecem nos logs e como você pode detectá-los com queries LogQL.

## Limpeza do Ambiente

!!! exercise "Exercício"
    Quando terminar, pare e remova todos os contêineres:

    <div class="termy">

    ```
    $ docker compose down
    ```

    </div>

