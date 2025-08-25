# Explorar Alternativas

Agora que você já consegue pelo menos observar quais arquivos estão disponíveis, iremos avançar até que os dados disponíveis no **S3** possam ser analisados pela empresa.

!!! info
    Já sabemos que eles contém informações sobre [estações de bicicleta em São Francisco](https://www.kaggle.com/datasets/benhamner/sf-bay-area-bike-share/data).

!!! exercise text short
    Qual o formato dos arquivos disponíveis no S3? Você considera este formato adequado? Justifique.

    <div class="termy">

    ```console
    $ aws s3 ls s3://dataeng-warmup --recursive --profile dataeng-warmup
    ```

    </div>

    !!! answer "Resposta"
        O formato dos arquivos disponíveis no S3 é CSV. Esse formato é utilizado para análise de dados e suportado por diversas ferramentas de análise. No entanto, pode não ser o mais eficiente em termos de armazenamento e desempenho para grandes volumes de dados.

Um formato alternativo que poderia ser considerado é o Parquet. O Parquet é um formato de arquivo colunar que oferece melhor compressão e desempenho em consultas, especialmente em cenários de big data.

!!! exercise text short
    Uma outra vantagem do Parquet é a fixação de *schema* dos dados. Por que isto é importante?

    !!! answer "Resposta"
        Isto é importante porque garante que todos os dados sejam armazenados de forma consistente, facilitando a validação e a análise.
        
        Além disso, a fixação de *schema* permite que as ferramentas de processamento de dados otimizem suas operações, resultando em melhor desempenho e eficiência.

Sua próxima tarefa será:

1. Ler todos os CSVs
2. Fixar um *schema* adequado
3. Salvar no **S3** em formato **Parquet** no path `s3://dataeng-warmup/data_processed/insper_username/file.parquet` onde:
    1. `insper_username` deve ser substituído pelo seu nome de usuário do Insper.
    2. `file` deve ser substituído pelo nome do arquivo que está sendo processado.

Antes de começar a realizar as tarefas, leia um pouco mais do handout!

## Ferramenta

Para realizar as tarefas, duas alternativas foram propostas pela empresa:
- Pandas,
- Polars.

!!! exercise text long
    Imagino que você já conheça o `pandas`. Ela seria uma boa escolha para a tarefa?

    !!! answer "Resposta"
        Se os arquivos fossem pequenos, seria. No entanto, para arquivos grandes, o `pandas` pode ter problemas de desempenho e consumo de memória.
        
        Confira o tamanho dos arquivos disponíveis no S3 e considere o uso do `polars`.

!!! exercise
    Faça uma pesquisa breve sobre o `polars` e compare com o `pandas`.

    Você pode solicitar alguns exemplos de código para ilustrar as diferenças entre as duas bibliotecas.

!!! exercise
    Crie um diretório para a aula e inicie um projeto Python nele.

    <div class="termy">

    ```console
    $ mkdir aula01
    $ cd aula01
    ```

    </div>


Para conseguirmos interagir com o **S3** utilizando **Python**, precisamos definir as credenciais de acesso. Isto será realizado no arquivo `.env`. Veja um exemplo abaixo:

```
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1
```

!!! exercise
    Crie o arquivo `.env` na raiz do seu projeto e adicione as credenciais de acesso ao S3 fornecidas pelo professor.

    !!! answer "Resposta"
        Pergunte ao professor caso não saiba onde encontrar esta informação!

Uma outra boa prática é criar um ambiente virtual para aula. Você pode fazer isso utilizando o `venv`, `conda`, `uv` ou outra ferramenta de sua preferência.

!!! exercise
    Crie ou ative seu ambiente virtual

 É adequado criar um arquivo `requirements.txt` para gerenciar as dependências do projeto.

!!! exercise
    Crie o arquivo `requirements.txt` na raiz do seu projeto e adicione as seguintes bibliotecas nele:

    ```bash  { .copy }
    pandas==2.3.1
    polars==1.32.2
    python-dotenv==1.1.1
    s3fs==0.4.2
    ```

!!! exercise
    Instale com:

    !!! danger "Atenção!"
        Lembre de ativar o ambiente virtual!
    
    <div class="termy">

    ```console
    $ pip install -r requirements.txt
    ```

    </div>

Utilize os seguintes códigos base para realizar a leitura dos arquivos CSV:

!!! exercise
    Analise os códigos da sequência e garanta que entendeu o que está acontecendo.

=== "Pandas"
    ```python  { .copy }
    import os
    import s3fs
    import pandas as pd
    from dotenv import load_dotenv

    load_dotenv(override=True)

    # Definir path do arquivo a ser lido
    csv_path = "dataeng-warmup/data_raw/station.csv"

    aws_access_key = os.getenv("AWS_ACCESS_KEY_ID")
    aws_secret_access_key = os.getenv("AWS_SECRET_ACCESS_KEY")
    aws_region = os.getenv("AWS_REGION")

    fs = s3fs.S3FileSystem(
        key=aws_access_key, secret=aws_secret_access_key, client_kwargs={"region_name": aws_region}
    )

    # CSV
    with fs.open(csv_path, "rb") as f:
        df = pd.read_csv(f)

    df.head(2)
    ```

=== "Polars"
    ```python  { .copy }
    import os
    import s3fs
    import polars as pl
    from dotenv import load_dotenv

    load_dotenv(override=True)

    # Definir path do arquivo a ser lido
    csv_path = "dataeng-warmup/data_raw/station.csv"

    aws_access_key = os.getenv("AWS_ACCESS_KEY_ID")
    aws_secret_access_key = os.getenv("AWS_SECRET_ACCESS_KEY")
    aws_region = os.getenv("AWS_REGION")

    fs = s3fs.S3FileSystem(
        key=aws_access_key, secret=aws_secret_access_key, client_kwargs={"region_name": aws_region}
    )

    # Observe que estammos utilizando Lazy Evaluation
    with fs.open(csv_path, "rb") as f:
        df = pl.scan_csv(f)

    df.head(2).collect()
    ```


!!! exercise text long
    O que é *lazy evaluation*?

    !!! answer "Resposta"
        Lazy evaluation é uma técnica onde a avaliação de uma expressão é adiada até que seu valor seja realmente necessário. Isso pode ajudar a melhorar o desempenho e a eficiência, especialmente ao trabalhar com grandes volumes de dados.

        No contexto do `polars`, a lazy evaluation permite que o sistema otimize a execução de operações em um DataFrame, agrupando e minimizando o trabalho necessário para produzir o resultado final.

!!! exercise
    Faça uma versão do código para ler o **DataFrame** utilizando `polars` sem *lazy evaluation*.

    !!! answer "Resposta"
        Altere de `pl.scan_csv(f)` para `pl.read_csv(f)` e remova o '.collect()`.

!!! exercise
    Faça a transformação necessárias nos dados para que o *schema* seja fixado corretamente.

    Para fixar o *schema*, você deve garantir que os tipos de dados das colunas estejam corretos e que não haja valores ausentes ou inconsistentes.
        
    Utilize as funções de transformação do `polars` para ajustar os dados conforme necessário.

    !!! answer "Resposta"
        Nesta etapa, pesquise sobre as funções de transformação do `polars`.

!!! exercise
    Para cada arquivo bruto (`raw`) disponível no S3, crie um arquivo `parquet` no S3 com o mesmo nome, mas no diretório `data_processed/insper_username/`.

    O arquivo Parquet deve ser criado a partir do DataFrame processado.

    Utilize este código como base:

    !!! danger "Atenção!"
        O código base não faz as transformações necessárias!

        Defina seu `insper_username` e o `parquet_path` corretamente. 

    ```python  { .copy }
    import os
    import s3fs
    import polars as pl
    from dotenv import load_dotenv

    load_dotenv(override=True)

    # Definir path do arquivo a ser exportado
    insper_username = ""
    bucket_name = "dataeng-warmup"
    parquet_path = f"{bucket_name}/data_processed/{insper_username}/station.parquet"

    aws_access_key = os.getenv("AWS_ACCESS_KEY_ID")
    aws_secret_access_key = os.getenv("AWS_SECRET_ACCESS_KEY")
    aws_region = os.getenv("AWS_REGION")

    fs = s3fs.S3FileSystem(
        key=os.getenv("AWS_ACCESS_KEY_ID"), secret=os.getenv("AWS_SECRET_ACCESS_KEY")
    )

    # Aqui, o `df` é o DataFrame do Polars que iremos exportar
    # Não utilize collect se não estiver utilizando lazy evaluation
    with fs.open(parquet_path, mode="wb") as f:
        df.collect().write_parquet(f)
    ```

!!! exercise
    Após exportar, confira que você consegue ler o arquivo `parquet` criado.

    Utilize este código como base:

    ```python  { .copy }
    import os
    import s3fs
    import polars as pl
    from dotenv import load_dotenv

    load_dotenv(override=True)

    # Definir path do arquivo a ser lido
    insper_username = "pereira"
    bucket_name = "dataeng-warmup"
    parquet_path = f"{bucket_name}/data_processed/{insper_username}/station.parquet"

    aws_access_key = os.getenv("AWS_ACCESS_KEY_ID")
    aws_secret_access_key = os.getenv("AWS_SECRET_ACCESS_KEY")
    aws_region = os.getenv("AWS_REGION")

    fs = s3fs.S3FileSystem(
        key=aws_access_key, secret=aws_secret_access_key, client_kwargs={"region_name": aws_region}
    )

    with fs.open(parquet_path, "rb") as f:
        # df = pl.read_parquet(f) # Sem lazy evaluation
        df = pl.scan_parquet(f) # Com lazy evaluation

    # df.head(2) # Sem lazy evaluation
    df.head(2).collect() # Com lazy evaluation
    ```

!!! exercise
    Após exportar e garantir que consegue ler o arquivo `parquet` criado, confira que você consegue listar os arquivos no S3. Você deve visualizar, na lista de arquivos, os parquets que você criou e os parquets criados por seus colegas.

    <div class="termy">

    ```console
    $ aws s3 ls s3://dataeng-warmup --recursive --profile dataeng-warmup
    ```

    </div>

## Exemplo de análise

Agora que você exportou e leu os arquivos `parquet`, pode realizar análises sobre os dados. Por exemplo, você pode calcular estatísticas descritivas, criar visualizações ou aplicar modelos de machine learning.

!!! exercise
    Sua tarefa é, utilizando `polars` e o histórico de `status` das estações (quantas bicicletas estão disponíveis em cada estação ao longo do tempo e quantos docks de armazenamento de bicicletas estão livres), responder:

    **"Quais as dez estações que, em média, considerando os intantes de tempo, possuem percentualmente menos bicicletas disponíveis em relação ao número total de docks de armazenamento?"**

## Questões finais

!!! exercise choice
    Segundo o esquema deste *warm up*, onde os dados são armazenados?

    - [ ] No computador dos analistas
    - [X] De forma centralizada no S3

    !!! answer "Resposta"
        Os dados são armazenados de forma centralizada no S3, permitindo que todos os membros da equipe acessem e trabalhem com os mesmos conjuntos de dados.

!!! exercise choice
    Segundo o esquema deste *warm up*, onde os dados são **processados**?

    - [X] No computador dos analistas
    - [ ] No S3

!!! exercise text long
    Como você analisa esta organização? Quais as vantagens e desvantagens dos dados serem processados no computador dos analistas?

    !!! answer "Resposta"
        ✅ **Vantagens**:
        
        - **Flexibilidade**: Os analistas podem escolher as ferramentas e ambientes que melhor atendem às suas necessidades.
        - **Autonomia**: Cada analista pode trabalhar de forma independente, sem depender de uma infraestrutura centralizada.

        ❌ **Desvantagens**:

        - **Consistência**: Pode haver variações nos resultados devido a diferentes ambientes e configurações.
        - **Escalabilidade**: Processar grandes volumes de dados localmente pode ser limitado pela capacidade do hardware dos analistas e pela largura de banda da rede.
        - **Colaboração**: A falta de um ambiente centralizado pode dificultar a colaboração e o compartilhamento de resultados entre os membros da equipe.
