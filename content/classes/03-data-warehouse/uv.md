# Conhecendo o `uv`

o **uv** é um gerenciador de pacotes, ambientes e projetos para Python. Ele foi escrito em *Rust* e serve como uma alternativa a como utilizamos `conda` e `venv` nos cursos do Insper.

Para esta aula e as demais, indicamos utilizar o **uv** para gerenciar ambientes virtuais.

!!! attention "Atenção!"
    Utilizar o **uv** é opcional.
    
    Caso prefira, pode ignorar os passos de instalação e seguir com seu gerenciador de pacotes preferido!

## Instalação

=== "Linux e MacOS"
    <div class="termy">
        ```bash
        $ curl -LsSf https://astral.sh/uv/install.sh | sh
        $ uv --version
        ```
    </div>

=== "Windows"
    No **Powershell**, faça:

    <div class="termy">
        ```powershell
        $ powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
        ```
    </div>

Caso tenha problemas, veja mais informações [no site oficial do UV](https://docs.astral.sh/uv/getting-started/installation).

## Criar ambiente virtual

=== "uv"
    === "Linux e MacOS"
        <div class="termy">
            ```bash
            $ uv venv --python 3.12 .olap-env
            $ source .olap-env/bin/activate
            ```
        </div>

    === "Windows"
        <div class="termy">
            ```bash
            $ uv venv --python 3.12 .olap-env
            $ .olap-env\Scripts\Activate.ps1
            ```
        </div>

    !!! tip
        Caso precise, você pode baixar a versão específica de python com:

        <div class="termy">
            ```bash
            $ uv python install 3.12
            ```
        </div>

        Para ver as versões do Python disponíveis e instaladas, utilize:

        <div class="termy">
            ```bash
            $ uv python list
            ```
        </div>

=== "venv"
    No **Powershell**, faça:

    <div class="termy">
        ```powershell
        $ python -m venv .olap-env
        $ .olap-env\Scripts\activate
        ```
    </div>

=== "conda"
    <div class="termy">
        ```powershell
        $ conda create --name olap-env python=3.12
        $ conda activate olap-env
        ```
    </div>