# Configuração AWS CLI

## Configuração de Ambiente

Para seguir com os exemplos práticos desta aula, você precisará configurar seu ambiente de desenvolvimento. Aqui estão as etapas recomendadas:

1. **Ativar Conta na AWS**: Se ainda não ativou sua conta, confira o aviso enviado pelos professores para maior detalhes.

2. **Instalar o AWS CLI**: O AWS Command Line Interface (CLI) é uma das ferramentas que utilizaremos para interagir com os serviços da AWS. Siga as instruções [da primeira aula](../01-intro/warmup.md#aws-cli-command-line-interface) para instalar e configurar o **AWS CLI**.

3. **Configurar Credenciais**: Vamos configurar o acesso à conta **AWS** pelo **SSO** (esquema diferente da primeira aula).

!!! warning "Atenção"
    Execute o seguinte comando:

    !!! danger "Importante"
        Defina a região como `us-east-1`.
        Vamos centralizar recursos em uma região! Assim mantemos uma melhor organização e controle sobre o que criamos durante o curso.

    !!! warning "Login pelo navegador!"
        Durante esta etapa, você será solicitado a fazer login em sua conta **AWS** por meio do navegador.
        
        Siga as instruções na tela para concluir o processo de autenticação.

        Caso solicitado, escolha a conta da disciplina!
    
    !!! warning "Configurações"
        - `SSO session name (Recommended):` pode utilizar `dataeng`
        - `SSO start URL:` utilize a URL de login que está no e-mail de convite
        - `SSO region [None]`: utilize `us-east-1`
        - `SSO registration scopes [sso:account:access]:` apenas aperte ENTER

    <div class="termy">

    ```bash
    $ aws configure sso --profile dataeng
    $ aws sts get-caller-identity --profile dataeng
    $ aws sso login --profile dataeng
    ```

    </div>


!!! danger "Cuidado para não fazer confusão!"
    Sempre confira se o perfil `dataeng` está ativo ao executar comandos da **AWS CLI** nesta disciplina.

    Você pode listar todos os perfis disponíveis com:

    <div class="termy">

    ```bash
    $ aws configure list-profiles
    ```

    </div>

Agora que você configurou seu ambiente, está pronto para começar a trabalhar com a **AWS CLI** e explorar os serviços da **AWS**.