# Introdução

## Agendamento

Na [aula 03](../03-data-warehouse/pratica.md) discutimos sobre processos de **ETL** / **ELT**.

No exemplo da aula, implementamos uma **pipeline** de dados que extrai informações de um banco de dados **PostgreSQL**, transforma esses dados e os carrega em um **data warehouse Clickhouse**.

!!! exercise text short "Exercício"
    De quanto em quanto tempo a tarefa de **ETL** era executada?

    !!! answer "Resposta"
        A cada dois minutos.

        Um tempo baixo para que conseguíssemos simular uma rotina diária sem esperar tanto.

!!! exercise text short "Exercício"
    Como isto foi implementado?

    !!! answer "Resposta"
        De forma rudimentar, esperando `120` segundos em um loop infinito.

        ```yaml
        # Recorte de parte do docker-compose.yml
        command: >
            bash -c "
                pip install -r /app/requirements.txt &&
                echo 'Inicializando banco de dados warehouse...' &&
                python src/init_database.py &&
                echo 'Iniciando ETL (executará a cada 2 minutos)...' &&
                while true; do
                echo 'Executando ETL...' &&
                python src/etl_vendas.py &&
                echo 'ETL concluído. Aguardando 2 minutos...' &&
                sleep 120
                done
            "
        ```

!!! exercise text long "Exercício"
    Vê algum problema na forma como fizemos?

    !!! answer "Resposta"
        A *query* de **SQL** utilizada para extrair os dados extraía informações desde o segundo zero do minuto atual. Entretanto, caso o processo de **ETL** demorasse mais que um minuto, parte dos dados não seriam ingeridos/tratados.

        Ainda, o máximo que obtínhamos de controle era o tempo de espera. Não havia como, por exemplo, executar o processo de **ETL** às 3 da manhã todos os dias.

## Simulando a atividade

Vamos simular a execução de um processo que precisa ser agendado. Para isto, vamos partir dos seguintes arquivos:

!!! exercise
    Antes, crie um pasta para esta parte da aula.

    Os próximos arquivos devem ser criados nesta pasta.

    <div class="termy">

    ```bash
    $ mkdir -p 06-intro-orchestration/01-intro
    $ cd 06-intro-orchestration/01-intro
    ```

    </div>

Agora considere os arquivos:

??? "`app.py`"
    ```python { .copy }
    import datetime

    def main():
        print(f"Olá do Docker! A hora atual é: {datetime.datetime.now()}", flush=True)

    if __name__ == "__main__":
        main()
    ```

??? "`Dockerfile`"
    ```dockerfile { .copy }
    # Use uma imagem base oficial do Python
    FROM python:3.12-slim

    # Defina o diretório de trabalho no container
    WORKDIR /app

    # Copie o script Python para o diretório de trabalho
    COPY app.py .

    # Execute o script Python quando o container for iniciado
    CMD ["python", "app.py"]
    ```

??? "`docker-compose.yml`"
    ```yaml { .copy }
    services:
        python-app:
            build: .
            container_name: python-scheduler
            restart: always
    ```

!!! exercise
    Crie cada um dos arquivos na pasta `06-intro-orchestration/01-intro`.

!!! exercise text short
    Faça a inicialização com:

    <div class="termy">

    ```bash
    $ docker compose build
    $ docker compose up
    ```

    </div>

    De quanto em quanto tempo a tarefa é executada (`print`)?

    !!! answer
        Assim que possível!
        
        Após a execução, o processo encerra. Então, instantaneamente o *container* é reiniciado e ocorre novo `print`.

!!! exercise
    Altere o arquivo `app.py` para que o `print` ocorra a cada `5` segundos.

    Pode ser com `time.sleep(5)`!

    !!! answer
        ```python { .copy }
        import datetime
        import time

        def main():
            while True:
                print(f"Olá do Docker! A hora atual é: {datetime.datetime.now()}", flush=True)
                time.sleep(5)

        if __name__ == "__main__":
            main()
        ```

Anteriormente, já discutimos por que o `time.sleep` não é uma boa solução. Vamos buscar uma alternativa melhor!

## Cron
O **cron** é um agendador de tarefas do *Unix* que permite executar *scripts* ou comandos em intervalos regulares.

Ele opera em segundo plano como um *daemon*, verificando a cada minuto um arquivo de configuração chamado `crontab` (**cron table**) para saber quais tarefas devem ser executadas.

!!! info "Daemon"
    Um *daemon* é um tipo especial de programa que roda em segundo plano em sistemas Unix e semelhantes.

    Ele geralmente inicia junto com o sistema e fica aguardando por eventos ou condições específicas para executar suas tarefas (Ex: `sshd`).

As tarefas são definidas usando uma sintaxe de cinco campos para especificar o minuto, a hora, o dia do mês, o mês e o dia da semana em que o comando deve rodar:

```console
* * * * *
| | | | |
| | | | ----- Dia da semana (0 - 7, onde 0 ou 7 é Domingo)
| | | ------- Mês (1 - 12)
| | --------- Dia do mês (1 - 31)
| ----------- Hora (0 - 23)
------------- Minuto (0 - 59)
```

!!! info "Info"
    - Um asterisco (`*`) em um campo significa "todos os valores possíveis" para aquele campo.
    - Valores específicos podem ser listados, separados por vírgulas (ex: `1,15` no campo de minutos significa "no minuto 1 e no minuto 15").
    - Intervalos podem ser especificados com um hífen (ex: `1-5` no campo de horas significa "da 1h às 5h").
    - Passos podem ser definidos com uma barra (ex: `*/10` no campo de minutos significa "a cada 10 minutos").

Isso torna o **cron** extremamente útil para tarefas rotineiras, como backups de banco de dados ou limpeza de arquivos temporários.

!!! exercise
    Antes de continuar, encerre a execução dos *containers* e remova-os.

    <div class="termy">

    ```bash
    $ docker compose down
    ```

    </div>

## Atualização para o uso do cron
Vamos utilizar o **cron** para agendar a execução do nossa tarefa (por enquanto, apenas um `print`).

Para isto, considere os arquivos:

!!! exercise
    Crie uma nova pasta `06-intro-orchestration/02-cron` e copie os arquivos na sequência:

??? "`app.py`"
    Será igual ao primeiro arquivo `app.py` proposto nesta aula.
    ```python { .copy }
        import datetime

        def main():
            print(f"Olá do Docker! A hora atual é: {datetime.datetime.now()}", flush=True)

        if __name__ == "__main__":
            main()
    ```

??? "`crontab.txt`"
    ```cron { .copy }
    * * * * * /usr/local/bin/python3 /app/app.py >> /var/log/cron.log 2>&1

    ```

    !!! info "Info"
        A linha acima agenda a execução do `app.py` a cada minuto (de cada hora, de cada dia, de cada mês, etc.).

        A saída padrão e a saída de erro são redirecionadas para o arquivo `/var/log/cron.log`.

    !!! Warning "Atenção!"
        Deixe uma linha em branco no final do arquivo `crontab.txt`. O **cron** exige isto.

??? "`Dockerfile`"
    Atualizaremos para uso do `cron`:
    ```dockerfile { .copy }
    FROM python:3.12-slim

    # Instala o cron
    RUN apt-get update && apt-get install -y cron && rm -rf /var/lib/apt/lists/*

    WORKDIR /app

    COPY app.py .
    COPY crontab.txt .

    # Adiciona o crontab ao cron do sistema
    RUN crontab crontab.txt

    # Cria um arquivo de log para o cron
    RUN touch /var/log/cron.log

    CMD ["cron", "-f"]
    ```

??? "`docker-compose.yml`"
    De importante, apenas a atualização no nome do container!
    ```yaml { .copy }
    services:
    python-app:
        build: .
        environment:
        - PYTHONUNBUFFERED=1
        container_name: python-scheduler-cron
        restart: always
    ```

!!! exercise
    Faça a reinicialização com:

    !!! warning "Atenção!"
        Garanta que esteja no diretório `06-intro-orchestration/02-cron`!

    <div class="termy">

    ```bash
    $ docker compose build
    $ docker compose up
    ```

    </div>

Se aguardar um minuto ou dois, irá perceber que não terá nenhum *feedback*. Isto ocorre porque os *logs* foram redirecionados para o arquivo `/var/log/cron.log` (dentro do *container*).

!!! exercise text short
    Vamos inspecionar os *logs*.

    Com o o *container* em execução, abra outro terminal e execute:

    <div class="termy">

    ```bash
    $ docker exec python-scheduler-cron cat /var/log/cron.log
    ```

    </div>

    Aproveite e explique o que este comando faz!

    !!! answer
        O comando `docker exec python-scheduler-cron cat /var/log/cron.log` é utilizado para acessar o *container* em execução chamado `python-scheduler-cron` e executar o comando `cat /var/log/cron.log` dentro dele.
        
        Isso permite visualizar o conteúdo do arquivo de log do cron, onde estão registrados os resultados das execuções agendadas.

!!! exercise choice
    Funcionou? A tarefa está sendo executada a cada minuto?

    - [X] Sim
    - [ ] Não

!!! exercise text long
    O que você acha de utilizar o `cron` para agendar tarefas em produção?
    
    Analise, de forma crítica, com foco em engenharia de dados.

    !!! answer
        O **cron** é um agendador **simples**.

        Suas limitações incluem:

        - **Sem dependências**: Não consegue gerenciar dependências entre tarefas
        - **Sem estado**: Não rastreia se tarefas anteriores falharam ou tiveram sucesso
        - **Sem *retry***: Não tem mecanismos automáticos de reexecução em caso de falha
        - **Sem paralelização inteligente**: Executa tarefas de forma isolada
        - **Monitoramento limitado**: *Logging* básico, sem *dashboards* ou alertas
        - **Sem fluxo condicional**: Não pode executar tarefas baseadas em resultados de outras
        - **Sem gestão de recursos**: Não controla uso de CPU, memória ou concorrência

        Com o `cron`, tudo isto precisaria ser implementado pelo *script* iniciado ou por ferramentas externas.
        
        Se quisermos um controle mais avançado sobre a execução das tarefas, precisamos considerar outras soluções.

Em ambientes de produção, o uso do **cron** pode não ser a melhor escolha devido às suas limitações. É importante avaliar outras opções que ofereçam mais robustez e recursos para o agendamento de tarefas.

Vamos discutir algumas alternativas na próxima página!

!!! exercise
    Antes de continuar, encerre a execução dos *containers* e remova-os.

    <div class="termy">

    ```bash
    $ docker compose down
    ```

    </div>