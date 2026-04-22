# APS 01

## Overview

Na **APS 01**, iremos construir um sumarizador de notícias com **ML**.

Imagine que gestores de empresas assinam seu serviço que, diariamente, envia um resumo das notícias do dia. O objetivo do seu produto é que eles possam se manter atualizados rapidamente, sem precisar ler dezenas de artigos.

## Grupos e Entrega

A APS é individual. Crie seu repositório no [Classroom](https://classroom.github.com/a/NveN67bo).

## Prazo

[Acesse a página de prazos](../../deadlines.md) para conferir o prazo de entrega.

## Requisitos

- Criar tarefa para fazer a coleta de notícias de pelo menos dois portais de notícias (ex: IstoÉDinheiro, MoneyTimes). Nesta tarefa você deve:
    - Fazer *crawling* do HTML da página inicial do portal e armazenar o HTML adequadamente.
    - Extrair os links das notícias.
    - Armazenar a URL dos links das notícias.
    - Fazer o *crawling* de cada notícia. Estes dados devem ser devidamente armazenados.
    - Fazer o *scraping* de cada notícia, extraindo, por exemplo, o título, texto e data. Estas informações devem ser devidamente armazenadas.
- Com base nas notícias coletadas, criar uma tarefa que utilize LLM para gerar um panorama geral das notícias do dia. Este resumo deve ser armazenado.
    - Obs: Caso você queira supor que o gestor tem interesse em um determinado tema, você pode criar um prompt que gere um resumo focado neste tema.
- Criar uma tarefa que envie um e-mail para cada gestor com o resumo do dia.
    - Considere uma base de dados com os e-mails dos gestores.
    - Aqui, você pode utilizar *templates* de e-mail (ex: `Jinja2`).
    - Também é adequado utilizar serviços de terceiros para o envio de e-mails (ex: **resend**, **AWS SES**).
- Todo este processo deve ser representado em um DAG do **Airflow**.
    - O **DAG** deve ser agendado para rodar diariamente.
    - Você pode criar implementações extras, como por exemplo, alertas para notificar a equipe de engenharia em caso de falhas.
    - Você pode criar tarefas para fazer a validação dos dados coletados.
- Gravar um vídeo curto (3 a 5 minutos) explicando o funcionamento do seu projeto e demonstrando que a solução funciona.

**Obs**:

- Os serviços de armazenamento não serão especificados. Você pode utilizar o que achar mais adequado (ex: `PostgreSQL`, `MongoDB`, `S3`, `minio`, etc), desde que consiga justificar sua escolha.

## Códigos de Exemplo

### Crawling e Scraping

=== "Com BeautifulSoup"
    ??? "**`requirements.txt`**:"
        ```text { .copy }
        requests==2.32.5
        beautifulsoup4==4.14.2
        pandas==2.3.3
        tabulate==0.9.0
        ```

    ??? "**`ex_scraping_bs4.py`**:"
        ```python { .copy }
        """Exemplo simples de scraping com BeautifulSoup.

        Este módulo contém funções para baixar páginas da seção de notícias
        do site "IstoÉ Dinheiro" e extrair títulos, descrições e datas das
        notícias. As funções foram mantidas simples para fins didáticos.

        Observações:
        - Funções esperam uma estrutura HTML específica do site alvo.
        - Respeite os termos de uso do site e politicas de scraping antes de
        executar este código em larga escala.
        """

        # para nos comunicarmos com a Web
        import requests

        # para extrair informações de páginas HTML
        import bs4
        from bs4 import BeautifulSoup

        # Para criar um Data Frame
        import pandas as pd

        # Para expressões regulares
        import re

        # Recursos do sistema
        import os

        # Aleatoriedade
        import random

        # Para sleep
        import time

        def sleepy_code():
            """Pausa a execução por um tempo aleatório curto.

            Usa um intervalo uniforme entre 1 e 2 segundos. Essa função serve
            para evitar requisições muito rápidas seguidas ao site alvo, reduzindo
            a probabilidade de bloqueio.

            Não recebe parâmetros e não retorna valor.
            """
            sleep_time = random.uniform(1, 2) 
            time.sleep(sleep_time)


        def download_page(secao = "economia"):
            """Baixa o HTML da página de categoria especificada.

            Parâmetros:
            - secao (str): nome da seção no site (ex.: "economia").

            Retorna:
            - str: conteúdo HTML da página baixada.
            """
            sleepy_code()
            url = f"https://istoedinheiro.com.br/categoria/{secao}/"
            resposta = requests.get(url = url)
            resposta.encoding = "utf-8"
            return resposta.text


        def get_data(secao = "economia"):
            """Extrai listas de título, descrição e data das notícias da seção.

            Parâmetros:
            - secao (str): nome da seção a ser processada (padrão: "economia").

            Retorna uma tupla com três listas (titulos, descricoes, datas):
            - lista_titulo (list[str])
            - lista_desc (list[str])
            - lista_data (list[str])

            A função procura por artigos com os atributos `name` usados pelo
            HTML do site: `individualNew` (com imagem) e `ArticleWithText`
            (somente texto). Para cada artigo encontra o texto de <h1>, <p>
            e <span> correspondentes ao título, descrição e data/hora.
            """
            html = download_page(secao=secao) 
            soup = BeautifulSoup(html, "html.parser")
            
            # Noticias com a thumb image
            lista_tag_individual = soup.find_all("article", attrs={"name": "individualNew"})
            # Notícias sem imagem ao lado do título
            lista_tag_a_text = soup.find_all("article", attrs={"name": "ArticleWithText"})
            # Concatena as duas listas
            lista_tag_noticia = lista_tag_individual + lista_tag_a_text

            lista_titulo = []
            lista_desc = []
            lista_data = []

            for tag_noticia in lista_tag_noticia:

                titulo = tag_noticia.find("h1").text
                titulo = titulo.replace("\n", "") #limpa os ENTERS
                lista_titulo.append(titulo)

                descricao = tag_noticia.find("p").text
                lista_desc.append(descricao)
                
                data_hora = tag_noticia.find("span").text
                lista_data.append(data_hora)

            return lista_titulo, lista_desc, lista_data


        def get_dataframe(secao = "economia"):
            """Cria um DataFrame pandas com as notícias de uma seção.

            Parâmetros:
            - secao (str): nome da seção a consultar (padrão: "economia").

            Retorna:
            - pandas.DataFrame: colunas ['Secao', 'Titulo', 'Descrição', 'Data']
            contendo as notícias extraídas.
            """
            lista_titulo, lista_desc, lista_data = get_data(secao=secao)
            df = pd.DataFrame({"Secao": secao,
                            "Titulo": lista_titulo,
                            "Descrição": lista_desc,
                            "Data": lista_data
                            })
            return df


        def get_news(secoes):
            """Concatena DataFrames de múltiplas seções.

            Parâmetros:
            - secoes (iterable[str]): lista/iterável de nomes de seção a processar.

            Retorna:
            - pandas.DataFrame: concatenação (vertical) dos DataFrames por seção.
            """
            dfs = []
            for secao in secoes:
                dfs.append(get_dataframe(secao=secao))
            return pd.concat(dfs, axis=0)


        if __name__ == "__main__":
            secoes = ["economia"]
            df = get_news(secoes)
            print(df.head(10).to_markdown())
            df.to_csv("noticias.csv", index=False)

        ```

=== "Com Selenium"

    ??? "**`requirements.txt`**:"
        ```text { .copy }
        selenium==4.36.0
        webdriver_manager==4.0.2
        ```

    ??? "**`ex_scraping_bs4.py`**:"
        ```python { .copy }
        from selenium import webdriver
        from selenium.webdriver.chrome.options import Options
        from webdriver_manager.chrome import ChromeDriverManager
        from selenium.webdriver.chrome.service import Service
        from selenium.webdriver.common.by import By
        import time


        driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
        driver.set_page_load_timeout(60)

        url = 'https://atd-insper.s3.us-east-2.amazonaws.com/aula07/horas.html'
        driver.get(url)
        driver.implicitly_wait(10)


        for i in range(10):
            horas = driver.find_element(By.TAG_NAME, 'div')
            print(horas.text)
            time.sleep(1)

        # Agora utilize a API do BeautifulSoup para extrair os dados a partir do HTML (driver.page_source)
        # Ou utilize a API do Selenium para extrair os dados diretamente (driver.find_elements...)
        # Com selenium, você consegue interagir com a página, clicando em botões, preenchendo formulários, etc.
        # Veja a documentação!
        ```

### LLM

=== "Com AWS Bedrock"

    ??? "**`requirements.txt`**:"
        ```text { .copy }
        python-dotenv==1.1.1
        boto3==1.42.93
        ```

    ??? "**`.env.example`**:"

        !!! warning "Atenção"
            Renomeie este arquivo para `.env` e preencha as variáveis de ambiente com os valores corretos.

        ```text { .copy }
        AWS_ACCESS_KEY_ID="AAAAAAAAAAAAAAAAAAAAA"
        AWS_SECRET_ACCESS_KEY="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
        AWS_REGION="us-east-1"
        BEDROCK_MODEL_ID="amazon.nova-lite-v1:0"
        ```

    ??? "**`ex_llm.py`**:"

        !!! info "Info"
            Adaptado do [exemplo oficial da AWS](https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-runtime_example_bedrock-runtime_Converse_AmazonNovaText_section.html)

        ```python { .copy }
        # Use the Conversation API to send a text message to Amazon Nova.

        import os

        import boto3
        from botocore.exceptions import ClientError
        from dotenv import load_dotenv

        load_dotenv()

        aws_region = os.getenv("AWS_REGION", "us-east-1")
        model_id = os.getenv("BEDROCK_MODEL_ID", "amazon.nova-lite-v1:0")

        # If keys are present in .env, boto3 will use them.
        session_kwargs = {}
        aws_access_key_id = os.getenv("AWS_ACCESS_KEY_ID")
        aws_secret_access_key = os.getenv("AWS_SECRET_ACCESS_KEY")
        aws_session_token = os.getenv("AWS_SESSION_TOKEN")

        if aws_access_key_id and aws_secret_access_key:
            session_kwargs["aws_access_key_id"] = aws_access_key_id
            session_kwargs["aws_secret_access_key"] = aws_secret_access_key
        if aws_session_token:
            session_kwargs["aws_session_token"] = aws_session_token

        session = boto3.Session(**session_kwargs)

        # Create a Bedrock Runtime client in the AWS Region you want to use.
        client = session.client("bedrock-runtime", region_name=aws_region)

        # Start a conversation with the user message.
        user_message = "Describe the purpose of a 'hello world' program in one line."
        conversation = [
            {
                "role": "user",
                "content": [{"text": user_message}],
            }
        ]

        try:
            # Send the message to the model, using a basic inference configuration.
            response = client.converse(
                modelId=model_id,
                messages=conversation,
                inferenceConfig={"maxTokens": 512, "temperature": 0.5, "topP": 0.9},
            )

            # Extract and print the response text.
            response_text = response["output"]["message"]["content"][0]["text"]
            print(response_text)

        except (ClientError, Exception) as e:
            print(f"ERROR: Can't invoke '{model_id}'. Reason: {e}")
            exit(1)

        ```

=== "Com AWS Bedrock - OpenAI"

    ??? "**`requirements.txt`**:"
        ```text { .copy }
        python-dotenv==1.1.1
        openai==2.32.0
        ```

    ??? "**`.env.example`**:"

        !!! warning "Atenção"
            Renomeie este arquivo para `.env` e preencha as variáveis de ambiente com os valores corretos.

        ```text { .copy }
        AWS_REGION="us-east-1"
        BEDROCK_MODEL_ID="deepseek.v3.2"
        BEDROCK_API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        ```

    ??? "**`ex_llm.py`**:"

        !!! info "Info"
            Adaptado do [exemplo oficial da AWS](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-chat-completions.html#inference-chat-completions-supported)


        ```python { .copy }
        import os

        from dotenv import load_dotenv
        from openai import OpenAI

        load_dotenv()

        aws_region = os.getenv("AWS_REGION", "us-east-1")
        model_id = os.getenv("BEDROCK_MODEL_ID", "openai.gpt-oss-20b-1:0")
        bedrock_api_key = os.getenv("BEDROCK_API_KEY")

        if not bedrock_api_key:
            raise ValueError("BEDROCK_API_KEY não encontrada no ambiente.")

        client = OpenAI(
            base_url=f"https://bedrock-runtime.{aws_region}.amazonaws.com/openai/v1",
            api_key=bedrock_api_key,
        )

        response = client.chat.completions.create(
            model=model_id,
            messages=[
                {"role": "user", "content": "Olá, como posso usar o Bedrock?"}
            ]
        )

        print(response.choices[0].message.content)
        ```

=== "Com AzureOpenAI"
    ??? "**`requirements.txt`**:"
        ```text { .copy }
        python-dotenv==1.1.1
        langchain-openai==0.3.35
        ```

    ??? "**`.env.example`**:"

        !!! warning "Atenção"
            Renomeie este arquivo para `.env` e preencha as variáveis de ambiente com os valores corretos.

        ```text { .copy }
        AZURE_OPENAI_API_KEY="88888888888888888888888888888888888888888888888888888888888888888888888888888888888888888"
        AZURE_OPENAI_ENDPOINT="https://xxxxxxxxxx.cognitiveservices.azure.com/openai/deployments/xxxxxxxxxxx/chat/completions?api-version=2025-01-01-preview"
        AZURE_OPENAI_API_VERSION="2025-01-01-preview"
        AZURE_OPENAI_DEPLOYMENT="xxxxxxxxxxxxxxxx"
        ```

    ??? "**`ex_llm.py`**:"

        ```python { .copy }
        from langchain_openai import AzureChatOpenAI
        from dotenv import load_dotenv

        load_dotenv(override=True)


        llm = AzureChatOpenAI(
            api_version="2025-01-01-preview",
            temperature=0,
            max_tokens=1000,
            timeout=30,
            max_retries=2,
            model="gpt-4.1-nano",
        )

        messages = [
            (
                "system",
                "You are a helpful assistant that translates English to Portuguese. Translate the user sentence.",
            ),
            ("human", "I love programming."),
        ]

        ai_msg = llm.invoke(messages)

        print(ai_msg)
        ```

## Chave de API

Você deve criar os acessos necessários em sua conta AWS da disciplina. Muito cuidado para não disponibilizar nenhuma API KEY de forma aberta!

## Rubrica

| Conceito | Critérios |
|---------|-----------|
| I (Insuficiente) | Não completou as etapas requeridas, componentes principais ausentes ou a solução não funciona. |
| D (Em desenvolvimento) | Desenvolveu algumas etapas do DAG, mas com erros significativos ou partes faltando; solução incompleta ou pouco confiável. |
| C (Essencial) | Fez o vídeo do projeto; Completou todas as etapas requeridas; a solução funciona mas carece de refinamento ou melhores práticas. |
| B (Acima da média) | Solução robusta e bem documentada (README adequado). |
| A (Excelente) | Demonstra entendimento e esforço extra (mencionar na gravação o que fez além dos requisitos)|
