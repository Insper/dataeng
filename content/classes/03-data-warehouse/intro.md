# Introdução

Nas aulas anteriores, exploramos o ciclo de vida de engenharia de dados (**ingestão**, **transformação** e **disponibilização**) e a criação de *softwares* desacoplados que interagem entre si. Na aula de hoje, vamos explorar:

- Data warehouse
- Contexto histórico de criação das solução.

## O problema

Como engenheiro de dados, você terá que trabalhar lado a lado, interagir e levantar requisitos de profissionais como [cientistas de dados e analistas de dados](../01-intro/profissionais.md).

Suponha que você foi designado para um projeto que envolve a criação de um *Dashboard* para analisar os dados de vendas.

Para levantar os requisitos, você marca uma reunião com os *stakeholders* do projeto. Ao ser perguntado sobre os objetivos do projeto, o analista de dados menciona:

!!! quote
    A diretoria do departamento de vendas, que atua como cliente neste projeto, reclamou da falta de informações consolidadas sobre as vendas.

    Os gestores desejam ter uma visão clara e integrada das vendas, permitindo identificar tendências, oportunidades e áreas que precisam de atenção.

O time de engenharia logo comenta:

!!! quote
    Já levantamos que os dados de vendas estão armazenados em um **RDBMS** (ou **SGBD-R**, *Sistema de Gerenciamento de Banco de Dados Relacional*) [**PostgreSQL**](https://www.postgresql.org/), que pode atuar como fonte dos dados.

O analista de dados, por sua vez, complementa:

!!! quote
    Nossa conversa inicial indicou que a principal necessidade, neste momento, seria uma visão integrada e histórica da evolução das vendas.

    Como os gestores desejam visualizar as vendas agregadas por **UF** e realizar filtros por *período*, a proposta se encaminha para a criação de um **Dashboard**.

Sabendo que os dados estão disponíveis no **PostgreSQL**, um analista de dados comenta.

!!! quote
    Nossa solução de *Dashboard* integra com o **PostgreSQL**.
    
    Posso criar uma *query* **SQL** e consumir direto deste banco de dados.

!!! exercise text long
    O que você acha da proposta do analista?

    ??? "Caso não tenha entendido muito a pergunta..."
        O que você acha da proposta do analista de dados de criar um *Dashboard* que se conecta e consome os dados diretamente do **PostgreSQL**?

    !!! answer "Resposta?"
        A proposta do analista de dados de criar um *Dashboard* que se conecta e consome os dados diretamente do **PostgreSQL** é geralmente problemática.

        Apesar de ser uma abordagem utilizada especialmente por empresas menores, ela pode levar a desafios significativos em termos de escalabilidade, manutenção e governança dos dados.

        Possíveis problemas:

        - O que vai acontecer quando um **processamento pesado** for realizado diretamente no **PostgreSQL**?
            - As vendas vão parar para que o **Dashboard** possa ser atualizado?
        - O que vai acontecer quanto à **segurança** e **governança dos dados**?
            - Quem terá acesso a esses dados?
            - Como garantir que os dados sensíveis estejam protegidos?
        - O que vai acontecer quanto à **evolução** do **software** *versus* a evolução do **Dashboard**?
            - Como garantir que as mudanças no sistema gerencial (**novas features**) não quebrem o **Dashboard**?
            - Como garantir que o analista não realize modificações que quebrem o sistema gerencial?

Podemos perceber que o servidor utilizado pelo sistema de vendas da empresa possui um propósito específico, que é o processamento das vendas, garantindo que os dados estejam íntegros ao longo de todo o ciclo de vida das transações. Não é objetivo deste servidor propiciar análises complexas ou relatórios detalhados (*data analytics*).

Com isto, vemos que precisamos entender melhor os diferentes tipos de sistemas de banco de dados e suas finalidades específicas.

## OLTP

O **OLTP** (*Online Transaction Processing*) evoluiu dos primeiros sistemas de processamento de dados comerciais, onde cada escrita no banco correspondia a uma transação comercial real: efetuar uma venda, fazer um pedido, pagar um salário. Hoje, o termo "transação" se expandiu para qualquer grupo de leituras e escritas que formam uma unidade lógica.

Sistemas **OLTP** são projetados para permitir leituras e escritas de **baixa latência** por milhares de usuários simultâneos, contrastando com processamento em lote que roda periodicamente.

### Características principais do OLTP:

- **Padrão de acesso interativo**: A aplicação busca poucos registros por chave usando índices
- **Baixa latência e alta concorrência**: Seleção ou atualização de registros em menos de um milissegundo
- **Transações ACID**: Garantem *Atomicity*, *Consistency*, *Isolation* e *Durability*
- **Dados normalizados**: Estrutura otimizada para minimizar redundância e garantir integridade
- **Operações pontuais**: Inserções e atualizações baseadas em input do usuário

!!! info "OLTP e ACID"
    Nem todo sistema OLTP precisa ter propriedades ACID completas.
    
    Alguns sistemas relaxam essas restrições para melhor performance e escala, usando modelos como **eventual consistency**.

!!! example "Exemplos de sistemas OLTP"
    - **Sistema bancário**: Transferências entre contas (transação atômica)
    - **E-commerce**: Processamento de pedidos e atualizações de estoque
    - **Reserva de voos**: Milhares de usuários consultando e reservando assentos simultaneamente

!!! exercise choice "Características do OLTP"
    Qual das seguintes características **NÃO** é típica de um sistema OLTP?

    - [ ] Processamento rápido de transações pequenas
    - [ ] Garantia de consistência ACID
    - [X] Otimização para consultas analíticas complexas
    - [ ] Suporte a múltiplos usuários concorrentes

    !!! answer "Resposta"
        Sistemas **OLTP** são otimizados para transações rápidas e operações *CRUD*, não para **consultas analíticas complexas**.
        
        Consultas analíticas são características de sistemas **OLAP** (Online Analytical Processing).

!!! example "Exemplos de soluções OLTP"
    - MySQL
    - PostgreSQL
    - Oracle Database

## OLAP

O **OLAP** (*Online Analytical Processing*) surgiu quando bancos de dados começaram a ser usados para **análise de dados** (*data analytics*), com padrões de acesso muito diferentes do processamento transacional.

Enquanto **OLTP** busca poucos registros, uma consulta analítica típica precisa **escanear milhares ou milhões de registros**, lendo apenas **algumas colunas** e calculando estatísticas agregadas (*count*, *sum*, *average*) ao invés de retornar dados brutos.

### Características principais do OLAP:

- **Consultas analíticas de grande escala**: Escaneiam grandes volumes, frequentemente 100MB+ por consulta
- **Foco em agregações**: Cálculos como totais, médias, contagens ao invés de registros individuais
- **Otimizado para *scan***: Bancos **colunares** reduzem a dependência de índices para melhor performance de varredura
- **Processamento em lote**: Atualizações periódicas, não em tempo real
- **Ineficiente para *lookups***: Não adequado para busca de registros individuais

!!! example "Exemplos típicos de consultas OLAP"
    - *"Qual foi a receita total de cada loja em janeiro?"*
    - *"Quantos produtos a mais vendemos durante a promoção?"*
    - *"Qual marca é mais comprada junto com fraldas da marca X?"*

### OLTP vs OLAP: Padrões de Uso

Vamos resumir, de forma geral, os **padrões de uso** de **OLTP** e **OLAP**:

| Aspecto | OLTP | OLAP |
|---------|------|------|
| **Padrão de consulta** | Poucos registros por chave | Scan de milhões de registros |
| **Latência** | Milissegundos | Segundos a minutos |
| **Volume por operação** | Registros individuais | Grandes agregações |
| **Usuários típicos** | Aplicações e usuários finais | Analistas de negócio |
| **Finalidade** | Estado da aplicação | Business Intelligence |
| **Exemplo** | Consultar saldo da conta | Receita total por região/mês |

!!! warning "OLTP para Analytics: Problema Comum"
    Empresas pequenas frequentemente rodam analytics diretamente no OLTP. Isso funciona no curto prazo, mas não é escalável devido a:
    
    - **Limitações estruturais** do OLTP para consultas analíticas
    - **Contenção de recursos** entre cargas transacionais e analíticas
    
    Engenheiros devem configurar integrações adequadas sem degradar a performance dos sistemas produtivos.

!!! exercise text long
    Considerando o cenário apresentado no início da aula, explique por que conectar o **Dashboard** diretamente ao **PostgreSQL** (sistema OLTP) pode causar problemas de performance.

    !!! answer "Resposta"
        Conectar o Dashboard diretamente ao PostgreSQL (OLTP) pode causar sérios problemas de performance porque:

        1. **Consultas analíticas pesadas**: O Dashboard precisará executar queries com agregações complexas (vendas por mês, por UF) que são computacionalmente caras
        
        2. **Bloqueio de operações**: Enquanto o PostgreSQL processa essas consultas pesadas, as operações de venda podem ficar lentas ou bloqueadas
        
        3. **Estrutura não otimizada**: Bancos OLTP são normalizados para eficiência transacional, não para consultas analíticas

        4. **Concorrência**: O sistema precisa atender tanto às vendas em tempo real quanto às consultas do Dashboard, competindo pelos mesmos recursos
        
        Por isso, a solução adequada seria extrair os dados do OLTP, transformá-los e carregá-los em um sistema OLAP otimizado para análises.

Agora que você entendeu a importância de **separar as cargas de trabalho** transacionais e analíticas, vamos explorar os diferentes **ambientes** utilizados por empresas. Não é algo que iremos aprofundar muito, mas é importante ter uma visão geral.

## Ambientes

No desenvolvimento de soluções, é fundamental trabalhar com diferentes ambientes para garantir a qualidade, segurança e confiabilidade dos sistemas. Cada ambiente serve a um propósito específico no ciclo de desenvolvimento e operação.

### Tipos de Ambientes

#### Desenvolvimento (DEV)
Ambiente onde os engenheiros desenvolvem e testam inicialmente suas soluções.

- **Propósito**: Desenvolvimento e testes iniciais
- **Dados**: Dados sintéticos ou subconjuntos pequenos dos dados reais
- **Acesso**: Restrito à equipe de desenvolvimento
- **Estabilidade**: Esperada instabilidade durante desenvolvimento

#### Homologação/Staging (STG)
Ambiente que replica o ambiente de produção para testes finais.

- **Propósito**: Testes de integração, validação de performance e homologação
- **Dados**: Dados similares aos de produção, mas mascarados ou anonimizados
- **Acesso**: Equipe de desenvolvimento, QA e stakeholders para homologação
- **Estabilidade**: Deve ser estável para testes confiáveis

#### Produção (PROD)
Ambiente onde o sistema opera para usuários finais.

- **Propósito**: Operação real do sistema
- **Dados**: Dados reais da empresa
- **Acesso**: Restrito e auditado
- **Estabilidade**: Máxima estabilidade e disponibilidade

!!! warning "Segurança nos Ambientes"
    É fundamental que os dados sensíveis de produção **nunca** sejam copiados diretamente para ambientes de desenvolvimento. Utilize técnicas de:
    
    - **Mascaramento de dados**: Substituição de dados sensíveis por valores falsos
    - **Anonimização**: Remoção de identificadores pessoais
    - **Dados sintéticos**: Geração de dados artificiais que mantêm as características estatísticas

!!! exercise choice "Ambientes de Desenvolvimento"
    Um engenheiro de dados precisa testar um novo pipeline ETL que processará dados de clientes contendo CPF, nomes e informações financeiras. Em qual ambiente ele deve realizar os primeiros testes?

    - [X] Desenvolvimento (DEV) com dados mascarados
    - [ ] Produção (PROD) com dados reais
    - [ ] Diretamente em Homologação (STG)
    - [ ] Produção (PROD) em horário de baixo uso

    !!! answer "Resposta"
        O engenheiro deve usar o ambiente de Desenvolvimento (DEV) com dados mascarados. Nunca se deve testar diretamente em produção, e os primeiros testes devem ocorrer em DEV antes de ir para homologação.

        Ainda, é adequado que dados sensíveis sejam mascarados em ambientes não-produtivos.

Sabendo que necessitaremos de um serviço separado para *analytics*, é importante considerar a arquitetura do sistema e como os dados serão processados e armazenados. Na sequência, exploraremos as soluções de arquitetura propostas para atender a essas necessidades, de um ponto de vista histórico.