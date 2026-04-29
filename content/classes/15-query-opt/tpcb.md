# TPCB

Com o banco de dados populado, podemos começar a explorar a performance das consultas.

Como primeira opção, iremos utilizar o **TPC-B** (*Transaction Processing Performance Council - Benchmark B*), um benchmark clássico para medir a performance de sistemas de banco de dados transacionais.

??? "O Que é um Benchmark?"
    Um **benchmark** é uma ferramenta ou conjunto de testes padronizados utilizados para avaliar o desempenho de sistemas, componentes ou processos. No contexto de bancos de dados, **benchmarks** são usados para medir a eficiência e a capacidade de resposta do sistema sob diferentes cargas de trabalho.

    Eles fornecem métricas quantitativas que ajudam a comparar o desempenho entre diferentes sistemas de gerenciamento de banco de dados (**SGBDs**) ou configurações específicas, permitindo que desenvolvedores e administradores tomem decisões informadas sobre qual tecnologia ou configuração utilizar.


A **TPC** é uma organização que define padrões para medir o desempenho de sistemas de processamento de transações. O **TPC-B** simula um ambiente de banco de dados transacional, focando em operações típicas como inserções, atualizações e consultas simples.

## Configurar o Banco de Dados

Para executar o **TPC-B**, precisamos configurar o banco de dados.

Vamos acessar o servidor **PostgreSQL** que criamos:

!!! exercise "Exercício"
    Acesse o servidor **PostgreSQL**:

    <div class="termy">

    ```
    $ docker exec -it postgres-sfbikeshare-01 bash
    ```

    </div>

!!! warning "Atenção!"
    Os comandos a seguir devem ser executados dentro do container do **PostgreSQL**.

!!! exercise "Exercício"
    No servidor **PostgreSQL**, crie o banco de dados `pgbench_db` e faça sua inicialização:

    <div class="termy">

    ```
    $ createdb -U postgres pgbench_db
    $ pgbench -i -s 10 -U postgres pgbench_db
    ```

    </div>

    !!! info "Info"
        O parâmetro `-s 10` define o tamanho do banco de dados. Neste caso, ele criará 1 milhão de linhas na tabela `pgbench_accounts`. Você pode ajustar esse valor conforme a capacidade da sua máquina.

!!! exercise "Exercício"
    No **DBeaver**, confira o banco `pgbench_db` e suas tabelas.

## Rodar teste TPC-B

Para executar o teste, utilizaremos a ferramenta `pgbench`, que já vem instalada no container do **PostgreSQL**.

O `pgbench` é uma ferramenta de *benchmarking* para o **PostgreSQL** que simula cargas de trabalho transacionais, permitindo medir a performance do banco de dados sob diferentes condições.

!!! exercise text long "Exercício"
    Rode um primeiro teste:

    <div class="termy">

    ```
    $ pgbench -c 1 -T 30 -b tpcb-like -U postgres pgbench_db
    ```

    </div>

    !!! info "Info"
        O parâmetro `-c 1` define o número de clientes simultâneos (conexões) e o `-T 30` define a duração do teste em segundos.

        O `-b tpcb-like` indica que queremos usar o conjunto de transações similar ao do **TPC-B**.
    
    **Qual o resultado?**

    !!! answer "Resposta"
        O resultado do comando `pgbench` fornecerá várias métricas de desempenho, incluindo:
        
        - **Número de transações por segundo (tps):** Indica quantas transações o banco de dados conseguiu processar por segundo durante o teste.
        - **Número de transações com falha:** Indica quantas transações falharam durante o teste.
        - **Latência:** Mede o tempo de resposta do banco de dados para cada transação.
        - **Número total de transações processadas:** Indica o total de transações processadas durante o período do teste.

!!! tip "Dica"
    Enquanto executa os testes, você pode abrir outra janela de terminal e monitorar o uso de recursos do sistema com o comando `htop` (linux), `top` (macOS) ou o Gerenciador de Tarefas (Windows).

!!! exercise text short "Exercício"
    Agora com 20 conexões simultâneas. Alguma diferença nos resultados?

    <div class="termy">

    ```
    $ pgbench -c 20 -T 30 -b tpcb-like -U postgres pgbench_db
    ```

    </div>

!!! exercise text short "Exercício"
    E com uma queries preparadas?

    !!! info "Info"
        Uma query preparada é uma instrução SQL que é pré-compilada e armazenada no banco de dados para execução posterior. Ela permite que o banco de dados otimize a execução da consulta, especialmente quando a mesma consulta é executada repetidamente com diferentes parâmetros.

        Queries preparadas podem melhorar a performance ao reduzir o tempo de análise e planejamento das consultas repetitivas.

    <div class="termy">

    ```
    $ pgbench -c 20 -T 30 -b tpcb-like -M prepared -U postgres pgbench_db
    ```

    </div>

!!! exercise text short "Exercício"
    Execute o teste com outras variações.

## `pgbench` com SQL customizado

Vamos utilizar o `pgbench` para rodar um script **SQL** customizado e aproveitar para testar nosso banco `sfbikeshare`.

!!! exercise "Exercício"
    Caso não esteja com acesso, acesse o servidor **PostgreSQL**:

    <div class="termy">

    ```
    $ docker exec -it postgres-sfbikeshare-01 bash
    ```

    </div>

!!! exercise text long "Exercício"
    Navegue até os testes e execute um teste de `SELECT` na tabela `station`:

    !!! warning "Atenção!"
        Abra o arquivo `15-query-opt/services/tests/select_station_01.sql` e veja o conteúdo.

    <div class="termy">

    ```
    $ cd /app/tests
    $ pgbench -c 20 -T 30 -U postgres -f select_station_01.sql -n sfbikeshare
    ```

    </div>

    **Anote os resultados**.

!!! exercise text long "Exercício"
    Agora execute os testes de `SELECT` das tabelas `trip` e `status`:

    !!! warning "Atenção!"
        Abra o arquivo `15-query-opt/services/tests/select_trip_01.sql` e veja o conteúdo.

        Abra o arquivo `15-query-opt/services/tests/select_status_01.sql` e veja o conteúdo.

    <div class="termy">

    ```
    $ pgbench -c 20 -T 30 -U postgres -f select_trip_01.sql -n sfbikeshare
    $ pgbench -c 20 -T 30 -U postgres -f select_status_01.sql -n sfbikeshare
    ```

    </div>

    **Compare os resultados** com os testes anteriores. Qual a diferença?

    !!! answer "Resposta"
        Ao comparar os resultados dos testes de `SELECT` nas tabelas `station`, `trip` e `status`, você deve observar diferenças significativas nas métricas de desempenho, como o número de transações por segundo (tps) e latência.

De forma geral (não apenas neste caso), essas diferenças podem ser atribuídas a vários fatores, incluindo o tamanho das tabelas, a complexidade das consultas SQL, a presença ou ausência de índices, e a quantidade de dados que cada consulta precisa processar.


!!! exercise text long "Exercício"
    Agora execute os testes de `INSERT`:

    !!! warning "Atenção!"
        Abra os arquivos e estude seu conteúdo

    <div class="termy">

    ```
    $ pgbench -c 20 -T 30 -U postgres -f insert_trip_01.sql -n sfbikeshare
    $ pgbench -c 20 -T 30 -U postgres -f insert_status_01.sql -n sfbikeshare
    $ pgbench -c 20 -T 30 -U postgres -f insert_station_01.sql -n sfbikeshare
    ```

    </div>

    **Compare os resultados**, entre os atuais e com os testes anteriores. Qual a diferença?

!!! exercise text long "Exercício"
    Agora crie testes de `UPDATE` e `DELETE` para as tabelas `station`, `trip` e `status`.

    Rode e analise os resultados.

    !!! warning "Atenção!"
        Você pode criar os arquivos na pasta `15-query-opt/services/tests/`.

        Mas neste caso, será necessário reiniciar os serviços (você irá perder os dados inseridos).

        Recomendo que crie os arquivos `update_station_01.sql`, `update_trip_01.sql`, `update_status_01.sql`, `delete_station_01.sql`, `delete_trip_01.sql` e `delete_status_01.sql` na sua máquina e copie seu conteúdo para o container utilizando **VI** ou **nano**.