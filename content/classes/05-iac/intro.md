# Introdução

## Introdução à Infrastructure as Code (IaC)

Na era moderna do desenvolvimento de software e engenharia de dados, a infraestrutura que suporta nossas aplicações se tornou tão crítica quanto o próprio código. Tradicionalmente, o provisionamento e gerenciamento de infraestrutura era um **processo manual**, propenso a erros e inconsistências entre diferentes ambientes.

Imagine o seguinte cenário: você desenvolveu um *pipeline* de dados que funciona perfeitamente no seu ambiente local. Ao tentar replicá-lo no ambiente de produção, descobrem que a versão do banco de dados é diferente, as configurações de rede não são as mesmas, e várias dependências estão em versões incompatíveis.

??? warning "Problemas comuns"
    Situações comuns incluem: diferenças nas versões de bibliotecas, configurações de banco de dados distintas, variáveis de ambiente não sincronizadas, permissões de acesso diferentes, ou até mesmo sistemas operacionais com configurações específicas que funcionam em um ambiente mas falham em outro.

Este tipo de problema motivou o surgimento do conceito de **Infrastructure as Code**, uma abordagem que revolucionou a forma como gerenciamos e provisionamos infraestrutura.

## O que é Infrastructure as Code?

**Infrastructure as Code (IaC, Infraestrutura como Código)** é uma prática de **gerenciamento** e **provisionamento de infraestrutura** através de **código**, ao invés de **processos manuais**. Com **IaC**, você define sua infraestrutura usando arquivos de configuração que podem ser **versionados**, testados e reutilizados. 

!!! important "Definição"
    **Infrastructure as Code** é a prática de gerenciar e provisionar recursos de infraestrutura (servidores, redes, bancos de dados, etc.) através de código declarativo ou imperativo, permitindo automação, versionamento e reprodutibilidade da infraestrutura.

Pense na **IaC** como uma receita detalhada para construir sua infraestrutura. Ela especifica exatamente quais recursos criar e como configurá-los.

### Características Fundamentais

A Infrastructure as Code possui algumas características fundamentais que a diferenciam do gerenciamento tradicional de infraestrutura:

**Declarativa vs Imperativa**: A maioria das ferramentas de **IaC** utiliza uma abordagem declarativa, onde você descreve o estado desejado da infraestrutura, e a ferramenta determina como alcançar esse estado.

**Versionamento**: Como a infraestrutura é definida em código, ela pode ser versionada utilizando *git* (por exemplo), permitindo rastreamento de mudanças e rollbacks quando necessário.

**Idempotência**: Executar o mesmo código de infraestrutura múltiplas vezes deve produzir o mesmo resultado, sem efeitos colaterais indesejados.

!!! exercise text long
    Explique com suas palavras o que significa **"idempotência"** no contexto de **Infrastructure as Code** e por que essa característica é importante.

    !!! answer "Resposta"
        Idempotência significa que executar a mesma operação várias vezes produz o mesmo resultado. No contexto de **IaC**, isso significa que aplicar o mesmo arquivo de configuração de infraestrutura múltiplas vezes não criará recursos duplicados ou causará conflitos.
        
        É importante porque garante consistência e previsibilidade, permitindo que você execute o código de infraestrutura com segurança sempre que necessário.

## Por que usar IaC?

A adoção de **Infrastructure as Code** traz benefícios significativos para equipes de desenvolvimento e operações, especialmente em projetos de engenharia de dados onde a infraestrutura tende a ser **complexa** e distribuída.

### Consistência entre Ambientes

Um dos maiores desafios em projetos de dados é garantir que os ambientes de desenvolvimento, homologação e produção sejam idênticos.

Com **IaC**, você utiliza o mesmo código para provisionar todos os ambientes. Isso deve diminuir o uso da frase *"funciona na minha máquina"*!

### Escalabilidade e Automação

À medida que suas necessidades de infraestrutura crescem, modificar manualmente dezenas ou centenas de recursos se torna inviável. A IaC permite escalar sua infraestrutura modificando algumas linhas de código.

### Recuperação de Desastres

Quando sua infraestrutura está definida em código, recriar todo o ambiente após um desastre se torna mais fácil, por envolver menos etapas manuais, que podem demorar dias.

!!! danger "Recuperação de desastres!"
    Recuperação de desastres não é tema principal de estudo deste curso. **IaC** pode facilitar este processo, mas não é uma solução mágica, muito menos completa para este problema.

!!! warning "Realidade dos Ambientes"
    Na prática, é comum encontrar empresas onde os ambientes de desenvolvimento e produção são completamente diferentes, criados manualmente ao longo do tempo.
    
    Isso gera inconsistências que só são descobertas quando algo falha em produção.

### Documentação Viva

O código da infraestrutura serve como documentação sempre atualizada de como o ambiente está configurado. Isso é especialmente valioso para novos membros da equipe ou para auditoria de *compliance*.

!!! exercise choice "Benefícios da IaC"
    Qual dos seguintes benefícios da Infrastructure as Code é mais relevante para equipes que trabalham com múltiplos ambientes (desenvolvimento, teste, produção)?

    - [ ] Redução de custos de hardware
    - [X] Consistência entre ambientes
    - [ ] Aumento da velocidade de processamento
    - [ ] Redução do tamanho do código

    !!! answer "Resposta"
        A **consistência entre ambientes** é o benefício mais relevante, pois permite que a mesma definição de infraestrutura seja aplicada em todos os ambientes, eliminando as diferenças que causam problemas como "funciona no desenvolvimento mas não na produção".

## Tipos de Ferramentas de IaC

As ferramentas de Infrastructure as Code podem ser categorizadas de diferentes formas. Uma classificação útil é entre ferramentas de **configuração** e ferramentas de **orquestração**.

### Ferramentas de Configuração

Focam na configuração de servidores individuais, instalando software, configurando serviços e gerenciando arquivos de configuração.

**Exemplos**: Ansible, Puppet, Chef, SaltStack

Essas ferramentas são ideais para configurar servidores depois que eles são provisionados, garantindo que o software correto esteja instalado e configurado adequadamente.

### Ferramentas de Orquestração

Focam no provisionamento de recursos de infraestrutura como servidores, redes, balanceadores de carga, bancos de dados e outros serviços de nuvem.

**Exemplos**: Terraform, CloudFormation (AWS), Azure Resource Manager

Essas ferramentas definem quais recursos criar, como eles se conectam e suas configurações básicas.

!!! info "Complementaridade"
    Na prática, é comum usar ferramentas de ambas as categorias em conjunto. Por exemplo, usar Terraform para provisionar a infraestrutura base e Ansible para configurar os aplicativos nos servidores criados.

### Abordagens: Declarativa vs Imperativa

**Declarativa**: Você especifica o estado final desejado, e a ferramenta determina como chegar lá.

```yaml
# Exemplo declarativo (genérico)
resources:
  - name: web-server
    type: compute.instance
    count: 3
    size: medium
```

**Imperativa**: Você especifica os passos exatos para criar a infraestrutura.

```bash
# Exemplo imperativo (genérico)
create_instance("web-server-1", "medium")
create_instance("web-server-2", "medium") 
create_instance("web-server-3", "medium")
```

!!! exercise choice "Declarativa vs Imperativa"
    Qual é a principal vantagem da abordagem declarativa em **Infrastructure as Code**?

    - [ ] É mais rápida para executar
    - [ ] Requer menos conhecimento técnico
    - [X] Permite idempotência mais facilmente
    - [ ] Usa menos recursos de sistema

    !!! answer "Resposta"
        A abordagem **declarativa permite idempotência mais facilmente** porque você define o estado desejado e a ferramenta determina automaticamente se alguma alteração é necessária. Se executar novamente com a mesma configuração, nenhuma mudança será feita se o estado já estiver correto.

## IaC no Contexto de Engenharia de Dados

No contexto específico de engenharia de dados, a **IaC** se torna ainda mais crítica devido à complexidade e diversidade dos componentes envolvidos em um *pipeline* de dados moderno.

??? important "Um pipeline de dados típico pode envolver:"
    - Clusters de processamento distribuído (como Spark ou Dask)
    - Bancos de dados relacionais e NoSQL
    - Data lakes e data warehouses
    - Sistemas de mensageria (como RabbitMQ ou Kafka)
    - Ferramentas de orquestração (como Airflow)
    - Serviços de monitoramento e alertas
    - Redes e configurações de segurança específicas

    !!! info "Info"
        Parte destes compomentes já foram explorados no curso e parte será explorada nas próximas aulas!


Gerenciar essa infraestrutura manualmente seria não apenas trabalhoso, mas também propenso a erros que podem comprometer a integridade dos dados.

### Ambientes de Dados

Diferentemente de aplicações web tradicionais, pipelines de dados frequentemente precisam de ambientes com características específicas:

**Desenvolvimento**: Processamento de volumes pequenos com dados sintéticos ou mascarados
**Teste**: Validação com volumes médios e dados representativos
**Produção**: Processamento de volumes completos com alta disponibilidade

Cada ambiente pode ter configurações diferentes de recursos (CPU, memória, armazenamento), mas a estrutura básica deve ser consistente.

!!! exercise text long
    Considerando o pipeline de sensores discutido nas aulas anteriores (sensores → RabbitMQ → processamento → armazenamento), liste pelo menos 5 componentes de infraestrutura que precisariam ser provisionados e configurados.

    !!! answer "Possível resposta"
        1. **Servidor/container para RabbitMQ** com configurações de memória e armazenamento adequadas
        2. **Banco de dados** para armazenar os dados processados dos sensores
        3. **Servidor de aplicação** para executar o código de processamento
        4. **Configurações de rede** para permitir comunicação entre sensores e RabbitMQ
        5. **Sistema de monitoramento** para acompanhar a saúde da fila e do processamento
        6. **Configurações de segurança** como firewalls e controle de acesso
        7. **Armazenamento persistente** para dados históricos e backup

## Desafios e Considerações

Embora a Infrastructure as Code traga muitos benefícios, sua adoção também apresenta alguns desafios que devem ser considerados.

### Curva de Aprendizado

A transição de gerenciamento manual para **IaC** requer aprendizado de novas ferramentas e conceitos. A equipe precisa se familiarizar com sintaxes específicas, conceitos de estado e melhores práticas de cada ferramenta.

### Gerenciamento de Estado

Muitas ferramentas de **IaC** mantêm um "estado" que representa o mapeamento entre a configuração declarada e os recursos reais. O gerenciamento inadequado deste estado pode causar problemas sérios.

### Segurança

Arquivos de **IaC** frequentemente contêm informações sensíveis como credenciais e configurações de acesso. É essencial implementar práticas adequadas de segurança para proteger essas informações.

!!! warning "Cuidado com Credenciais"
    Nunca inclua senhas, chaves de API ou outras credenciais diretamente nos arquivos de **IaC**. Use sempre sistemas de gerenciamento de secrets ou variáveis de ambiente seguras.

### Mudanças Destrutivas

Algumas alterações na configuração podem resultar na destruição e recriação de recursos, causando **perda de dados** ou interrupção de serviços.

!!! warning "Atenção!"
    É fundamental entender as implicações de cada mudança antes de aplicá-la.

!!! exercise text long
    Imagine que você precisa explicar para um colega por que não devemos incluir senhas diretamente nos arquivos de **IaC**.
    
    Quais argumentos você usaria?

    !!! answer "Possível resposta"
        1. **Versionamento**: Os arquivos de **IaC** são versionados em sistemas como Git, o que significaria que as senhas ficariam registradas no histórico do repositório
        2. **Colaboração**: Outros desenvolvedores teriam acesso às credenciais ao visualizar o código
        3. **Segurança**: Se o repositório for comprometido, todas as credenciais seriam expostas
        4. **Auditoria**: Fica difícil rastrear quem teve acesso a quais credenciais
        5. **Rotação**: Mudanças de senhas exigiriam alterações no código e redeployment
        6. **Ambientes diferentes**: Produção e desenvolvimento devem usar credenciais diferentes

## Próximos Passos

Agora que você compreende os conceitos fundamentais de **Infrastructure as Code**, está preparado para explorar ferramentas específicas e aplicá-las na prática.

Nas próximas páginas, vamos explorar o **Terraform** como ferramenta de **IaC**.