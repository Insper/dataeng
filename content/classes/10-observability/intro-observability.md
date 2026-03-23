# Introdução

## Observabilidade em Engenharia de Dados

Ao longo das últimas aulas, construímos um repertório sobre o ciclo de vida dos dados, explorando desde a ingestão e orquestração com `DAGs` até o armazenamento em `Data Lakes` e `Lakehouses`.

Vimos como sistemas de dados modernos são complexos, distribuídos e compostos por muitas partes. Agora, imagine que um pipeline de dados crucial para a empresa começa a entregar resultados incorretos silenciosamente, ou que a latência de uma consulta dispara sem um motivo aparente.

!!! exercise text long
    Pense em um pipeline de dados que você já tenha construído. Quais são os possíveis pontos de falha? O que poderia dar errado sem que você percebesse imediatamente?

Quando os sistemas falham, não basta saber *que* algo deu errado. Precisamos ter a capacidade de investigar, entender a causa raiz e, idealmente, antecipar problemas antes que eles impactem os negócios.

!!! info "Importante!"
    É neste ponto que a **observabilidade** se torna importante na Engenharia de Dados, atuando para sustentar a confiabilidade e a qualidade de todo o ecossistema de dados.

## O que é Observabilidade?

A **observabilidade** é a capacidade de medir e entender o estado interno de um sistema a partir dos dados que ele gera externamente, como **logs**, **métricas** e **traces**.

Em vez de apenas reagir a falhas, a **observabilidade** nos permite fazer perguntas abertas sobre o comportamento do sistema, como *"por que a ingestão de dados está mais lenta hoje do que na semana passada?"* ou *"qual alteração no código causou essa inconsistência nos dados?"*.

!!! important "Definição"
    **Observabilidade** é a prática que nos permite inferir e compreender o estado e o comportamento de um sistema de dados, fornecendo visibilidade sobre suas operações.

    !!! warning ""
        Ela vai além do simples monitoramento, permitindo a depuração, a análise de causa raiz e a prevenção proativa de problemas em ambientes complexos e distribuídos.

Muitas vezes, os termos **monitoramento** e **observabilidade** são usados de forma intercambiável, mas eles representam conceitos distintos, embora relacionados.

??? "Observabilidade *vs.* Monitoramento"
    O **monitoramento** tradicional é focado em observar e alertar sobre problemas conhecidos. Ele responde a perguntas predefinidas, como *"o uso de CPU está acima de 90%?"* ou *"o pipeline falhou?"*. Geralmente, isso é feito por meio de *dashboards*.

    A **observabilidade**, por outro lado, é focada em **explorar e entender** problemas desconhecidos. Ela fornece dados contextuais que permitem investigar o **"porquê"** por trás de um sintoma.

    !!! warning ""
        Enquanto o monitoramento lhe diz *que* algo está errado, a observabilidade lhe dá as ferramentas para descobrir *por que* está errado.

## Os Pilares da Observabilidade

Para alcançar um estado de observabilidade, precisamos coletar e analisar três tipos principais de dados:

1.  **Logs**: São registros imutáveis e com carimbo de data/hora de eventos discretos que ocorreram ao longo do tempo. Um **log** pode registrar um **erro**, uma requisição de API ou a conclusão de uma tarefa. Eles fornecem um **contexto detalhado** e **granular**, essencial para a depuração, permitindo que você veja o que estava acontecendo em um ponto específico no tempo.

2.  **Métricas**: São **medições numéricas agregadas** ao longo de um intervalo de tempo. Métricas como **latência**, **taxa de transferência** (*throughput*) e **utilização de recursos** (CPU, memória) são fundamentais para entender o desempenho e a saúde do sistema em larga escala. Elas são eficientes para armazenar e consultar, tornando-as ideais para **dashboards** e **alertas**.

3.  **Traces (Rastreamento Distribuído)**: Em sistemas modernos, uma única requisição pode passar por múltiplos serviços. O **tracing** permite **seguir o caminho completo** dessa requisição, mostrando quanto tempo ela gastou em cada componente. Isso é indispensável para identificar gargalos de desempenho e entender as dependências entre serviços em uma arquitetura de microsserviços ou em um pipeline de dados complexo.

!!! exercise text short
    Considere um pipeline de dados que extrai dados de uma API, os transforma e os carrega em um Data Warehouse. Para cada um dos três pilares (Logs, Métricas, Traces), dê um exemplo de uma informação que seria útil coletar.

    !!! answer "Resposta"
        - **Logs**: Registro de cada execução do pipeline, incluindo erros específicos encontrados durante a transformação dos dados.
        - **Métricas**: Tempo total de execução do pipeline, número de registros processados por minuto e taxa de sucesso/falha das execuções.
        - **Traces**: Tempo gasto em cada etapa do pipeline (extração, transformação, carregamento) e a latência da API durante a extração dos dados.

## Ciclo de Vida dos Dados

A observabilidade não é uma fase isolada, mas uma prática contínua que se aplica a todas as etapas do ciclo de vida da engenharia de dados, desde a origem até o consumo.

-   **Sistemas Fonte e Ingestão**: Aqui, a observabilidade ajuda a monitorar a disponibilidade das fontes, a conformidade dos esquemas e a qualidade dos dados brutos. Durante a ingestão, medimos a taxa de transferência, a latência e a ocorrência de erros para garantir que os dados cheguem de forma confiável ao nosso repositório.

-   **Armazenamento e Transformação**: Em um Data Lake ou Data Warehouse, a observabilidade foca na saúde dos dados armazenados. Isso inclui validar esquemas, verificar estatísticas (como contagens de nulos, valores mínimos/máximos) e garantir a correção das transformações. Testes de qualidade de dados nos datasets de entrada e saída são essenciais para evitar a propagação de dados lixo (`garbage in, garbage out`).

-   **Serviço e Consumo**: Na camada final, onde os dados são consumidos por dashboards, modelos de machine learning ou outras aplicações, a observabilidade garante que os Acordos de Nível de Serviço (**SLOs**) sejam cumpridos. Monitoramos a latência das consultas, a atualização dos dados (`freshness`) e a disponibilidade dos sistemas de serviço.

!!! quote
    A aplicação da observabilidade em todo o ciclo de vida dos dados deu origem a uma nova abordagem, o **Desenvolvimento Orientado à Observabilidade de Dados (DODD)**.

    Semelhante ao Desenvolvimento Orientado a Testes (TDD), o DODD torna a observabilidade uma consideração de primeira classe desde o início do desenvolvimento, garantindo que a visibilidade seja construída em cada etapa do pipeline, e não adicionada como um pensamento tardio.

Ao integrar a observabilidade em nossa cultura e processos, buscamos nos tornar capazes de entregar valor de forma consistente e segura.
