# Relembrar

Em nossa primeira aula, exploramos a definição de engenharia de dados e os principais profissionais técnicos da área

Um dos principais conceitos discutidos foi o ciclo de vida de engenharia de dados, você se lembra dele?!

!!! exercise text long
    Quais eram os principais componentes do ciclo de vida de engenharia de dados?

    !!! answer "Resposta"
        Ingestão, Transformação e Disponibilização.

        ```mermaid
        flowchart LR

        %% Plataforma principal
        subgraph PD[Plataforma de Dados]
            direction LR

            G[Geração]

            %% Linha horizontal de entrada + pipeline
            subgraph LINE[Armazenamento]
            direction LR
            I[Ingestão]
            T[Transformação]
            S[Disponibilização]
            I --> T --> S
            end

            %% Saídas diretas (à direita)
            ML[Aprendizado de Máquina]
            AN[Análises]
            REP[Dashboards]

            G --> I
            S --> ML
            S --> AN
            S --> REP
        end

        %% Estilos adaptados para light e dark mode
        classDef gen fill:#64748b,stroke:#475569,color:#ffffff,stroke-width:2px;
        classDef stage fill:#0891b2,stroke:#0e7490,color:#ffffff,stroke-width:2px;
        classDef trans fill:#8b5cf6,stroke:#7c3aed,color:#ffffff,stroke-width:2px;
        classDef serve fill:#10b981,stroke:#059669,color:#ffffff,stroke-width:2px;
        classDef out fill:#f59e0b,stroke:#d97706,color:#ffffff,stroke-width:2px;

        class G gen;
        class I stage;
        class T trans;
        class S serve;
        class ML,AN,REP out;
        ```

No final da aula, fizemos um *warm up*, onde fizemos a leitura de dados do serviço **S3** (*Amazon Simple Storage Service*) da **AWS** (**Amazon Web Services**). Os dados passaram por uma definição de *schema* e posterior escrita no próprio **S3**.

!!! exercise text long
    Como o *script* final do seu *warm up* se relaciona com o ciclo de vida de engenharia de dados? Existiam seções de código que realizaram tarefas relativas a cada etapa do ciclo?

    !!! answer "Resposta"
        O script percorre as três etapas:

        1. **Ingestão**: realizada ao **ler dados** do bucket S3.  

        2. **Transformação**: ao definir um **schema** (tipos de dados, nomes de colunas). Mesmo que não haja cálculos complexos, normalizações ou junções, a simples conversão e padronização de tipos é parte da transformação.

        3. **Disponibilização (Serving)**: ao **escrever no S3 em formato Parquet**, disponibilizamos os dados em um formato otimizado para consulta e processamento posterior (exemplo: ferramentas analíticas).

Nosso [*warm up*](../01-intro/warmup.md) da aula passada Serviu como um exemplo prático de como os dados podem ser ingeridos, transformados e disponibilizados em um fluxo de trabalho simples de engenharia de dados.

Entretanto, é importante notar que começamos com uma representação bem simplista.

Por exemplo, a etapa de **ingestão** foi representada apenas pela leitura de dados do S3, mas na prática, ela pode envolver uma variedade de fontes e métodos de coleta de dados, como APIs, bancos de dados, arquivos CSV, entre outros. Raramente teremos apenas poucos arquivos e raramente eles serão estáticos.

Em cenários de *Big Data* :material-information-outline:{title="Big data é um conjunto de práticas, tecnologias e conceitos voltados para o tratamento de volumes massivos de dados, gerados em alta velocidade e em formatos variados, que ultrapassam a capacidade das ferramentas tradicionais de processamento. Seu objetivo é extrair valor a partir dessas informações, garantindo qualidade, confiabilidade e utilidade para apoiar a tomada de decisões estratégicas. O conceito é frequentemente caracterizado pelos cinco Vs: Volume, Velocidade, Variedade, Veracidade e Valor."}, os dados tendem a ser gerados continuamente, em grandes volumes e a partir de múltiplas origens, como sensores IoT, registros de transações, redes sociais, sistemas corporativos e fluxos de *streaming*. Além disso, a natureza dinâmica e heterogênea dessas fontes exige que a ingestão seja capaz de lidar com dados estruturados, semiestruturados e não estruturados, frequentemente em tempo real.

!!! note "Os cinco **Vs** do *Big Data*:"
    1. **Volume** – Quantidade massiva de dados gerados.
    2. **Velocidade** – Rapidez com que os dados são gerados, processados e analisados.
    3. **Variedade** – Diversidade de formatos e fontes dos dados.
    4. **Veracidade** – Confiabilidade e qualidade das informações.
    5. **Valor** – Utilidade e relevância dos dados para gerar insights.

Ao final do curso, nosso objetivo é capacitar você a lidar com esses desafios e a construir soluções de engenharia de dados que sejam escaláveis, eficientes e capazes de extrair valor real dos dados. Começaremos com cenários mais simples e controlados, mas gradualmente avançaremos para situações mais complexas e realistas.

!!! exercise choice "Question"
    Então a **ingestão** de dados envolverá, necessariamente, que algum *script* seja ativado e faça a leitura de dados de alguma fonte (como o **S3**)?

    - [ ] Sim
    - [X] Não

    !!! answer "Answer"
        Não, veremos na sequência!

Em **engenharia de dados**, a ingestão pode funcionar seguindo um padrão:

- **Push** (a ingestão é chamada): a fonte **envia** dados para você.
- **Pull** (a ingestão chama a fonte): ocorre a **busca** de dados na fonte.
- **Híbrido**: mistura os dois.

## Ingestão **Pull-based**

Neste formato de ingestão, a fonte **empurra** eventos/dados para um *endpoint* seu. Este modelo é particularmente bom para cenários de **tempo real**.

**Analogia**: notificações no celular (você **não pede**, elas chegam).

!!! example "Exemplo"
    Lembra do **Webhook** configurado para as correções das atividades de *SisHard*?
    
    As criações de *tag* batem no endpoint do servidor de correção como um *JSON*.

## Ingestão **Push-based**

Segundo este formato, o pipeline ou script **consulta**/**baixa** dados periodicamente. Este é um formato adequado para processamento em lotes (iremos abordar isso mais adiante) e integrações com sistemas legados.

**Analogia**: checar e-mail manualmente, uma vez que você **vai lá e busca**.

!!! example "Exemplo"
    Um script diário que lê um banco de dados e grava no **S3**
