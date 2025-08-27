# Prática IaC

## Prática: Infrastructure as Code com Terraform

## Fixar conceito

**Infrastructure as Code** (Infraestrutura como Código) é uma abordagem que permite **gerenciar e provisionar** recursos de infraestrutura usando **código** e arquivos de configuração, ao invés de processos manuais.

Com **IaC**, você define sua infraestrutura em arquivos de texto que podem ser versionados, revisados e reutilizados. Isso traz benefícios como:

- **Consistência**: A mesma configuração pode ser aplicada múltiplas vezes
- **Versionamento**: Mudanças podem ser rastreadas usando Git
- **Reutilização**: Configurações podem ser compartilhadas entre projetos
- **Automação**: Reduz erros humanos e acelera deployments

!!! info "Comparação com a aula anterior"
    Na aula anterior, criamos uma **VM** manualmente pelo console da AWS e depois fizemos configurações via **SSH**.
    
    Com **Terraform**, faremos tudo isso através de código!

## Terraform

O **Terraform** é uma das ferramentas mais populares para **IaC**.

Desenvolvido pela **HashiCorp** ([2014](https://en.wikipedia.org/wiki/Terraform_(software))), ele permite definir infraestrutura usando a linguagem **HCL** (*HashiCorp Configuration Language*).

O **Terraform** suporta múltiplos provedores de nuvem (AWS, Azure, Google Cloud) e serviços diversos, permitindo criar desde uma simples VM até arquiteturas complexas.

!!! danger "Perigo!"
    Se fizer esta aula pela metade, não deixe **VMs** em execução.

    Confira, ao final do tutorial, o comando para **destruir** os recursos criados.
    
    Você também pode utilizar o **console** para conferir se sobrou algo (para destruir, prefira o terraform, não é boa prática utilizar o console uma vez que introduzimos IaC).

    Qualquer dúvida, entre em contato com o professor.

??? info "OpenTofu: Curiosidade!"
    Há alguns anos, a HashiCorp decidiu mudar a licença do Terraform para um modelo mais restritivo, o que gerou preocupação sobre a liberdade e sustentabilidade do projeto.
    
    Como resposta, a comunidade criou um *fork* aberto para manter a colaboração e garantir transparência no desenvolvimento.

    Inicialmente chamado de OpenTF e depois renomeado para [**OpenTofu**](https://opentofu.org/), o projeto passou a ser mantido pela *Linux Foundation*, assegurando governança aberta e compatibilidade com o Terraform.
    
    Assim, tornou-se uma alternativa livre e confiável para usuários e empresas que dependem de **IaC**.

## Instalação do Terraform

!!! exercise "Exercício"
    Instale a versão mais recente do Terraform seguindo as instruções para seu sistema operacional:

    !!! note "Extra"
        Caso necessário, acesse mais informações diretamente no site do [Terraform](https://developer.hashicorp.com/terraform/install).

    === "Linux (Ubuntu/Debian)"
        <div class="termy">

        ```bash
        $ wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
        $ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
        $ sudo apt update && sudo apt install terraform
        ```

        </div>

    === "macOS"
        <div class="termy">

        ```bash
        $ brew tap hashicorp/tap
        $ brew install hashicorp/tap/terraform
        ```

        </div>

    === "Windows"
        1. Baixe o binário do Terraform em [https://developer.hashicorp.com/terraform/install](https://developer.hashicorp.com/terraform/install)
        2. Extraia o arquivo ZIP em um diretório (ex: `C:\terraform`)
        3. Adicione o diretório ao PATH do sistema [Tutorial 1](https://www.wikihow.com/Change-the-PATH-Environment-Variable-on-Windows) [Tutorial 2](https://www.eukhost.com/kb/how-to-add-to-the-path-on-windows-10-and-windows-11/)

!!! exercise "Exercício"
    Verifique se a instalação foi bem-sucedida:

    <div class="termy">

    ```bash
    $ terraform version
    ```

    </div>

    Você deve ver a versão do Terraform instalada.

## Extensão VSCode

Instale a extensão [Hashicorp Terraform](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform) no VS Code.

## Estrutura do projeto

Vamos criar um projeto Terraform que reproduzirá o deploy da API de filmes que fizemos manualmente na aula anterior.

!!! exercise "Exercício"
    Crie um diretório para o projeto e entre nele:

    <div class="termy">

    ```bash
    $ mkdir movies-api-terraform
    $ cd movies-api-terraform
    ```

    </div>

!!! exercise "Exercício"
    Crie um arquivo `main.tf` com a configuração inicial do provedor AWS:

    ```hcl { .copy }
    terraform {
      required_version = ">= 1.0"
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
      }
    }

    provider "aws" {
      region  = "us-east-1"
      profile = "dataeng"
    }
    ```

### Inicializando o Terraform

!!! exercise "Exercício"
    Garanta que você está com o perfil `dataeng` configurado e logado no **AWS CLI**.

    Para mais detalhes, acesse [a aula passada](../04-cloud/config-aws-cli.md)

    Caso tenha o perfil configurado, faça login com:

    <div class="termy">

    ```bash
    $ aws sso login --profile dataeng
    ```

    </div>


!!! exercise "Exercício"
    Inicialize o diretório do Terraform:

    <div class="termy">

    ```bash
    $ terraform init
    ```

    </div>

    Esse comando baixa os plugins necessários para o provedor AWS.

!!! exercise text short
    O que aconteceu após executar `terraform init`?

    !!! answer "Resposta"
        O Terraform criou um diretório `.terraform/` e baixou o plugin do provedor AWS. Também criou um arquivo `.terraform.lock.hcl` para fixar as versões dos provedores.

## Criando a instância EC2

Vamos começar criando apenas a instância EC2 com Terraform, equivalente ao que fizemos manualmente na aula anterior.

!!! warning "Chave SSH"
    O código abaixo assume que você tem uma chave SSH em `~/.ssh/id_ed25519.pub`. Se não tiver, crie uma com:

    === "Linux / Mac / Windows (Git Bash)"
        <div class="termy">

        ```bash
        $ ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
        ```

        </div>
    
    === "Windows (Power Shell)"
        <div class="termy">

        ```bash
        $ ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\id_ed25519
        ```

        </div>

!!! exercise "Exercício"
    Adicione ao final do arquivo `main.tf` a configuração da instância EC2:

    ```hcl { .copy }
    # Dados da AMI mais recente do Ubuntu
    data "aws_ami" "ubuntu" {
      most_recent = true
      owners      = ["099720109477"] # Canonical (Ubuntu)

      filter {
        name   = "name"
        values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"] # A mesma que utilizamos na aula passada!
      }

      filter {
        name   = "virtualization-type"
        values = ["hvm"]
      }
    }

    # Key Pair para acesso SSH
    resource "aws_key_pair" "movies_api_key" {
      key_name   = "movies-api-key"
      public_key = file("~/.ssh/id_ed25519.pub") # Ajuste o caminho conforme necessário
    }

    # Instância EC2
    resource "aws_instance" "movies_api" {
      ami             = data.aws_ami.ubuntu.id
      instance_type   = "t3a.micro"
      key_name        = aws_key_pair.movies_api_key.key_name
      security_groups = [aws_security_group.movies_api_sg.name]

      tags = {
        Name = "movies-api-server"
      }
    }
    ```
!!! exercise text short
    Qual sistema operacional e versão estão sendo utilizados na instância EC2?

    !!! answer "Resposta"
        Ubuntu 22.04 LTS

!!! exercise text short
    Qual será a configuração (RAM e vCPU) da instância EC2?

    !!! answer "Resposta"
        `t3a.micro` (2 vCPU, 1 GB RAM).

        Veja mais em [AWS EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/).



!!! tip "Dica!"
    Você pode conferir as imagens disponíveis da canonical com:

    !!! info "Info!"
        `099720109477` é o ID do dono oficial das imagens Ubuntu na AWS (canonical).

    <div class="termy">

    ```bash
    $ aws ec2 describe-images --owners 099720109477 --query 'Images[*].[ImageId,Name,CreationDate]' --output table --profile dataeng
    ```

    </div>

!!! exercise "Exercício"
    Adicione também a configuração do Security Group:

    ```hcl { .copy }
    # Security Group
    resource "aws_security_group" "movies_api_sg" {
      name_prefix = "movies-api-sg"

      # SSH
      ingress {
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }

      # HTTP para a API (porta 8000)
      ingress {
        from_port   = 8000
        to_port     = 8000
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }

      # Saída para internet
      egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }

      tags = {
        Name = "movies-api-security-group"
      }
    }
    ```

!!! exercise "Exercício"
    Adicione *outputs* para exibir informações importantes:

    ```hcl { .copy }
    # Outputs
    output "instance_id" {
      description = "ID da instância EC2"
      value       = aws_instance.movies_api.id
    }

    output "instance_public_ip" {
      description = "IP público da instância"
      value       = aws_instance.movies_api.public_ip
    }

    output "instance_public_dns" {
      description = "DNS público da instância"
      value       = aws_instance.movies_api.public_dns
    }

    output "ssh_connection" {
      description = "Comando para conectar via SSH"
      value       = "ssh ubuntu@${aws_instance.movies_api.public_ip} -i ~/.ssh/id_ed25519"
    }
    ```

!!! warning "Acesso SSH"
    Perceba que, na aula passada, utilizamos um arquivo `.pem` para obter acesso à VM.

    Nesta aula, a chave da sua máquina é um arquivo `id_ed25519` e estará autorizada a acessar a instância.

### Planejamento e aplicação

O comando  `terraform plan` é usado no Terraform para gerar e exibir um plano de execução, mostrando quais mudanças serão feitas na infraestrutura antes de aplicá-las.

Ele compara o estado atual dos recursos (registrado no *state file* e no provedor) com a configuração descrita nos arquivos `.tf` e indica o que será **criado**, **alterado** ou **destruído**.

!!! info
    O `terraform plan` serve como uma **prévia segura** para validar se as alterações desejadas estão corretas, permitindo **revisar** e **evitar modificações indesejadas** na infraestrutura antes de executar o `terraform apply`.

!!! exercise text short "Exercício"
    Execute o planejamento para ver o que será criado:

    <div class="termy">

    ```bash
    $ terraform plan
    ```

    </div>

    Analise a saída. O que o Terraform pretende criar?

    !!! answer "Resposta"
        Deve retornar algo contendo, dentre outras informações:
        
        ```bash
        # aws_instance.movies_api will be created
        ...
        # aws_key_pair.movies_api_key will be created
        ...
        # aws_security_group.movies_api_sg will be created
        ...
        Plan: 3 to add, 0 to change, 0 to destroy.
        ...
        ```
!!! exercise "Exercício"
    Aplique a configuração para criar os recursos:

    <div class="termy">

    ```bash
    $ terraform apply
    ```

    </div>

    Digite `yes` quando solicitado.

!!! exercise text short "Exercício"
    Abra o **console** da **AWS** e verifique, no painel da **EC2**, se a instância foi criada.

    Quantas instâncias você vê?

    !!! answer "Resposta"
        Duas!

        - Uma da aula passada (desligada)
        - Uma que acabou de ser criada (ligada)

!!! exercise "Exercício"
    Teste o acesso SSH à instância criada:

    !!! warning "Atenção!"
        Substitua `IP_PUBLICO` pelo **IP público** mostrado no output do Terraform.

    <div class="termy">

    ```bash
    $ ssh ubuntu@IP_PUBLICO -i ~/.ssh/id_ed25519
    ```

    </div>

!!! exercise "Exercício"
    Na **VM**, rode o comando:

    <div class="termy">

    ```bash
    $ cat ~/.ssh/authorized_keys
    ```

    </div>
    <br>

    !!! info "Informação"
        O conteúdo do arquivo `~/.ssh/authorized_keys` deve incluir a chave pública correspondente ao arquivo `id_ed25519` da sua máquina!

        Ou seja, esta linha que permite você realize **SSH** no servidor!

        Cada computador autorizado a realizar **SSH** no servidor terá uma chave pública listada nesse arquivo (uma por linha).

!!! exercise text long
    Compare o processo de criação da VM com Terraform versus a criação manual da aula anterior. Quais são as principais diferenças?

    !!! answer "Resposta"
        - **Velocidade**: Terraform cria todos os recursos de uma vez
        - **Reprodutibilidade**: O mesmo código pode ser executado várias vezes
        - **Documentação**: A infraestrutura fica documentada no código
        - **Versionamento**: Mudanças podem ser rastreadas no Git
        - **Consistência**: Reduz erros de configuração manual

## Automatizando a configuração da aplicação

Agora vamos estender nosso Terraform para instalar e configurar a aplicação automaticamente, eliminando as etapas manuais via SSH.

!!! exercise "Exercício"
    Crie um arquivo `user_data.sh` com o script de inicialização:

    !!! info "info"
        Este será o script de inicialização a ser executado quando a VM for criada.

    ```bash { .copy }
    #!/bin/bash
    set -euxo pipefail

    # Atualizar sistema
    apt-get update
    apt-get upgrade -y
    apt-get install -y git curl build-essential python3-pip

    # Criar usuário
    sudo useradd --system --create-home --home-dir /srv/movies-api --shell /usr/sbin/nologin uapi
    sudo mkdir -p /srv/movies-api/app
    sudo chown -R uapi:uapi /srv/movies-api/app

    # Clonar Repositorio API
    sudo -u uapi git clone https://github.com/macielcalebe/movies-api-example-01.git /srv/movies-api/app

    # Instalar uv system-wide
    curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR=/usr/local/bin sh
    /usr/local/bin/uv --version

    # Criar ambiente virtual Python 3.12 com uv
    sudo -u uapi /usr/local/bin/uv venv --python 3.12 /srv/movies-api/app/.venv

    # Instalar dependências
    if [ -f /srv/movies-api/app/requirements.txt ]; then
        sudo -u uapi /usr/local/bin/uv pip install -r /srv/movies-api/app/requirements.txt --python /srv/movies-api/app/.venv/bin/python
    fi

    # Criar arquivo de serviço systemd
    cat > /etc/systemd/system/movies-api.service << 'EOF'
    [Unit]
    Description=Movies API
    After=network.target

    [Service]
    Type=simple
    User=uapi
    WorkingDirectory=/srv/movies-api/app
    Environment=PATH=/srv/movies-api/app/.venv/bin
    ExecStart=/srv/movies-api/app/.venv/bin/fastapi run src/main.py --host 0.0.0.0 --port 8000
    Restart=always

    [Install]
    WantedBy=multi-user.target
    EOF

    # Iniciar o serviço
    systemctl daemon-reload
    systemctl enable movies-api
    systemctl start movies-api
    ```

!!! exercise text short "Exercício"
    O que é `set -euxo pipefail`? Pesquise sobre!

!!! exercise "Exercício"
    Modifique o recurso `aws_instance` no arquivo `main.tf` para incluir o **script** user data:

    !!! warning
        A única alteração será a adição da linha:
        ```bash { .copy }
        user_data = file("user_data.sh")
        ```

    Deverá ficar assim:

    !!! warning "Atenção!"
        Perceba que liberamos o acesso à porta `22` (**SSH**) de qualquer **IP**.

        Por segurança, não é uma boa prática fazer isto em ambientes de produção.

    ```hcl { .copy }
    resource "aws_instance" "movies_api" {
      ami             = data.aws_ami.ubuntu.id
      instance_type   = "t3a.micro"
      key_name        = aws_key_pair.movies_api_key.key_name
      security_groups = [aws_security_group.movies_api_sg.name]
      
      # Script de inicialização
      user_data = file("user_data.sh")

      tags = {
        Name = "movies-api-server"
      }
    }
    ```

!!! exercise "Exercício"
    Ao final do arquivo, adicione um output para a **URL** da **API**:

    ```hcl { .copy }
    output "api_url" {
      description = "URL da API de filmes"
      value       = "http://${aws_instance.movies_api.public_ip}:8000"
    }

    output "api_docs_url" {
      description = "URL da documentação da API"
      value       = "http://${aws_instance.movies_api.public_ip}:8000/docs"
    }
    ```

## Testando a infraestrutura completa

!!! exercise "Exercício"
    Como fizemos mudanças na configuração, vamos recriar a instância:

    !!! info "Responda `yes`"
        Responda 'yes` para confirmar a destruição da instância e para sua recriação!

    <div class="termy">

    ```bash
    $ terraform destroy
    $ terraform apply
    ```

    </div>

    Digite `yes` quando solicitado para ambos comandos.

!!! tip "Dica!"
    Você pode conferir o *log* de execução do *script* `user_data.sh` em `/var/log/cloud-init-output.log`.

    Faça **SSH** na **VM** e rode:

    <div class="termy">

    ```bash
    $ cat /var/log/cloud-init-output.log
    ```

    </div>

!!! exercise "Exercício"
    Aguarde alguns minutos (3-5 minutos) para que o script de inicialização seja executado completamente.

    Teste a API nos endpoints:
    - `http://<IP_PUBLICO>:8000`
    - `http://<IP_PUBLICO>:8000/docs`
    - `http://<IP_PUBLICO>:8000/filmes/avatar`

!!! exercise "Exercício"
    Verifique se o serviço está rodando corretamente conectando via SSH:

    <div class="termy">

    ```bash
    $ ssh ubuntu@IP_PUBLICO -i ~/.ssh/id_ed25519
    $ sudo systemctl status movies-api
    $ sudo journalctl -u movies-api -f
    ```

    </div>

!!! exercise text short
    Quais são as vantagens de usar user data versus configurar tudo manualmente via SSH?

    !!! answer "Resposta"
        - **Automação completa**: A instância fica pronta sem intervenção manual
        - **Reprodutibilidade**: Sempre teremos a mesma configuração
        - **Escalabilidade**: Facilita a criação de múltiplas instâncias
        - **Documentação**: Todo processo fica documentado no código
        - **Redução de erros**: Elimina passos manuais propensos a erro

## Organizando o código

Para projetos maiores, é uma boa prática organizar o código Terraform em múltiplos arquivos.

!!! exercise "Exercício"
    Vamos conferir o *status* atual!

    Execute:

    <div class="termy">

    ```bash
    $ terraform plan
    ```

    </div>

    Como não realizamos nenhuma modificação nos arquivos de configuração, o esperado é que obtenhamos:
    
    ```bash
    No changes. Your infrastructure matches the configuration.
    ```

    Isto indica que não existem mudanças a serem aplicadas.

Assim, prosseguimos para a reorganização do código!

!!! exercise "Exercício"
    Reorganize o código criando os arquivos:

    **`variables.tf`**:
    ```hcl { .copy }
    variable "region" {
      description = "Região AWS"
      type        = string
      default     = "us-east-1"
    }

    variable "instance_type" {
      description = "Tipo da instância EC2"
      type        = string
      default     = "t3a.micro"
    }

    variable "key_name" {
      description = "Nome da chave SSH"
      type        = string
      default     = "movies-api-key"
    }

    variable "ssh_public_key_path" {
      description = "Caminho para a chave pública SSH"
      type        = string
      default     = "~/.ssh/id_ed25519.pub"
    }
    ```

    **`outputs.tf`**:
    ```hcl { .copy }
    output "instance_id" {
      description = "ID da instância EC2"
      value       = aws_instance.movies_api.id
    }

    output "instance_public_ip" {
      description = "IP público da instância"
      value       = aws_instance.movies_api.public_ip
    }

    output "api_url" {
      description = "URL da API de filmes"
      value       = "http://${aws_instance.movies_api.public_ip}:8000"
    }

    output "ssh_connection" {
      description = "Comando para conectar via SSH"
      value       = "ssh ubuntu@${aws_instance.movies_api.public_ip} -i ~/.ssh/id_ed25519"
    }
    ```

!!! exercise "Exercício"
    Atualize o `main.tf` para usar as variáveis:

    ```hcl { .copy }
    terraform {
      required_version = ">= 1.0"
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
      }
    }

    provider "aws" {
      region  = var.region
      profile = "dataeng"
    }

    data "aws_ami" "ubuntu" {
      most_recent = true
      owners      = ["099720109477"]

      filter {
        name   = "name"
        values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
      }

      filter {
        name   = "virtualization-type"
        values = ["hvm"]
      }
    }

    resource "aws_key_pair" "movies_api_key" {
      key_name   = var.key_name
      public_key = file(var.ssh_public_key_path)
    }

    resource "aws_security_group" "movies_api_sg" {
      name_prefix = "movies-api-sg"

      ingress {
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }

      ingress {
        from_port   = 8000
        to_port     = 8000
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }

      egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }

      tags = {
        Name = "movies-api-security-group"
      }
    }

    resource "aws_instance" "movies_api" {
      ami             = data.aws_ami.ubuntu.id
      instance_type   = var.instance_type
      key_name        = aws_key_pair.movies_api_key.key_name
      security_groups = [aws_security_group.movies_api_sg.name]
      user_data       = file("user_data.sh")

      tags = {
        Name = "movies-api-server"
      }
    }
    ```

!!! exercise "Exercício"
    Teste a reorganização:

    <div class="termy">

    ```bash
    $ terraform plan
    ```

    </div>

    Se não houver mudanças planejadas na infra (apenas nos *outputs*), a reorganização foi bem-sucedida!

    Pode aplicar!

## Limpeza dos recursos

!!! danger "Importante!"
    Sempre remova recursos que não estão em uso para evitar custos desnecessários.

!!! exercise "Exercício"
    Destrua todos os recursos criados:

    <div class="termy">

    ```bash
    $ terraform destroy
    ```

    </div>

    Digite `yes` quando solicitado.

!!! exercise "Exercício"
    Confirme que todos os recursos foram removidos:

    <div class="termy">

    ```bash
    $ aws ec2 describe-instances --query 'Reservations[*].Instances[*].{ID:InstanceId,State:State.Name}' --output table --profile dataeng
    ```

    </div>
