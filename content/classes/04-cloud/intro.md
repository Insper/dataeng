# Introdução

## *Cloud Computing* para Engenharia de Dados

Nas aulas anteriores, exploramos o **ciclo de vida da engenharia de dados** (ingestão, transformação e disponibilização), a importância de sistemas desacoplados usando filas de mensagens, e as diferenças fundamentais entre **OLTP** e **OLAP**. 

Hoje vamos dar um passo importante: entender **onde** e **como** implementar nossa **infraestrutura de dados**.

!!! info
    Esta aula é introdutória e focada na formação de conceitos fundamentais sobre computação em nuvem para engenharia de dados.

## O Contexto: Onde Executar Nossa Infraestrutura?

Imagine que você foi contratado como engenheiro de dados em uma empresa que está crescendo rapidamente.

Os dados estão aumentando exponencialmente, e a infraestrutura atual já mostra sinais de sobrecarga. A diretoria questiona:

!!! quote "Diretoria"
    *Devemos investir em mais servidores próprios ou migrar para a nuvem?*

Esta é uma decisão que pode ter **significado existencial** para organizações modernas:

- Mover-se muito devagar pode significar ficar para trás da concorrência mais ágil.

- Por outro lado, uma migração mal planejada pode levar a falhas tecnológicas e custos catastróficos.

!!! exercise text long
    Antes de prosseguir, reflita: onde você acha que as empresas executam seus sistemas de dados atualmente? Quais fatores você considera importantes nessa decisão?

    !!! answer "Resposta"
        As empresas hoje têm múltiplas opções: **on-premises** (servidores próprios), **cloud** (nuvem), **híbrido** (combinação de ambos) ou **multicloud** (múltiplos provedores de nuvem).

        Dentre os fatores importantes, podemos citar custo, escalabilidade, segurança, controle, compliance e agilidade para inovação.

Vamos explorar as principais opções de **localização** (*location*) para executar nossa infraestrutura de dados.

## On-Premises: O Modelo Tradicional

Embora novas *startups* nasçam cada vez mais na nuvem, os sistemas **on-premises** ainda são o padrão para empresas estabelecidas.

!!! important "Definição: **On-Premises**"
    No modelo **on-premises**, as empresas **possuem** seu próprio *hardware*, que pode estar localizado em *data centers* próprios ou em espaços de **colocation** alugados.
    
    As empresas são **operacionalmente responsáveis** por seu hardware e pelo software que roda nele.

    ??? info "Colocation"
        **Colocation** (infraestrutura) é um modelo de hospedagem em que uma empresa aluga espaço físico em um *data center* de terceiros para instalar seus próprios servidores, equipamentos de rede e armazenamento.

        Principais recursos e serviços compartilhados:

        - **Espaço físico**: pode ser apenas um *rack unit* (U), *racks* inteiros ou até áreas dedicadas no data center.

        - **Energia elétrica**: fornecida com redundância (geradores e no-breaks).

        - **Climatização**: controle de temperatura e umidade para manter os equipamentos estáveis.

        - **Conectividade**: acesso a diversos provedores de internet (*carrier-neutral*), possibilitando links redundantes e de alta velocidade.

        - **Segurança física**: controle de acesso, monitoramento 24/7, câmeras, vigilância.

        Exemplos de provedores de *colocation* em São Paulo: [ODATA](https://odatacolocation.com/blog/data-center/dc-sp01/) e [Ascenty](https://ascenty.com/data-centers/localizacao/brasil/sao-paulo-capital/)

!!! exercise text long
    Por que uma empresa iria optar por **colocation** em vez de *data centers* próprios?

    !!! answer "Resposta"
        Construir e gerenciar a infraestrutura de TI internamente pode ser caro e demorado:

        - Será que a distribuidora de energia elétrica irá nos fornecer a energia necessária para suportar a demanda?
        - Quais serão as alterações necessárias na infraestrutura do prédio (climatização, elétrica, rede, etc.)?
        - Temos espaço físico suficiente para instalar novos servidores?
        - Temos equipe interna capacitada para gerenciar a infraestrutura?

        O **colocation** permite que as empresas se beneficiem de infraestrutura de *data center* de alta qualidade sem os custos e responsabilidades de gerenciar fisicamente o *hardware*.

### Características do On-Premises

#### Responsabilidades Operacionais
- **Manutenção de hardware**: Se o hardware falha, a empresa precisa reparar ou substituir
- **Ciclos de atualização**: Gestão de upgrades a cada poucos anos
- **Capacidade de pico**: Necessidade de hospedar capacidade suficiente para picos de demanda

!!! example "Exemplo Prático"
    Para um varejista online, isso significa ter capacidade suficiente para lidar com os picos de carga da Black Friday.
    
    Para engenheiros de dados, significa **comprar sistemas grandes** o suficiente para permitir boa performance em cargas de pico sem gastar excessivamente.

#### Vantagens do On-Premises
- **Práticas estabelecidas**: Empresas estabelecidas têm práticas operacionais que funcionam bem
- **Controle total**: Controle completo sobre hardware, software e dados
- **Compliance**: Pode ser necessário para certas regulamentações
- **Custos previsíveis**: Investimento inicial alto, mas custos operacionais mais previsíveis

#### Desafios do On-Premises
- **Falta de agilidade**: Tempo longo para provisionar novos recursos
- **Subutilização**: Hardware parado durante períodos de baixa demanda
- **Expertise técnica**: Necessidade de equipe especializada em infraestrutura
- **Escalabilidade limitada**: Difícil escalar rapidamente para demandas imprevistas

!!! exercise choice "Question"
    Uma empresa de e-commerce precisa dobrar sua capacidade de processamento de dados para a Black Friday, que acontece em 2 meses.

    Qual seria o principal desafio usando infraestrutura **on-premises**?

    - [X] Tempo longo para comprar, instalar e configurar novos servidores
    - [ ] Custo alto dos servidores
    - [ ] Falta de expertise técnica
    - [ ] Problemas de segurança

    !!! answer "Resposta"
        O principal desafio seria o **tempo**.
        
        Comprar, instalar e configurar novos servidores pode levar semanas ou meses, tornando impossível atender à demanda da Black Friday a tempo.

## *Cloud*: A Revolução!

A **computação em nuvem** inverte completamente o modelo **on-premises**.

!!! important "Definição: Cloud Computing"
    Na **computação em nuvem**, ao invés de comprar hardware, você simplesmente **aluga** hardware e serviços gerenciados de um provedor de nuvem (como AWS, Azure ou Google Cloud).
    
    Esses recursos podem ser reservados em bases extremamente curtas: VMs são criadas em menos de um minuto, e o uso subsequente é cobrado em incrementos (por tempo de uso).

### Por Que a Nuvem é Revolucionária?

#### Escalabilidade Dinâmica
A nuvem permite aos usuários **escalar dinamicamente** recursos que eram inconcebíveis com servidores **on-premises**.

!!! example "Exemplo de Escalabilidade"
    Um pipeline de dados que processa 100GB por dia durante a semana pode precisar processar 1TB no final do mês. Na nuvem, você pode:
    
    1. Executar com 2 máquinas durante a semana
    2. Automaticamente escalar para 20 máquinas no final do mês
    3. Retornar para 2 máquinas depois do processamento
    4. Pagar apenas pelos recursos utilizados

#### Agilidade para Experimentação
Engenheiros podem **rapidamente** lançar projetos e experimentar sem se preocupar com planejamento de *hardware* de longo prazo. Podem começar a rodar servidores assim que seu código está pronto para *deploy*.

!!! exercise text long
    Pense em um cenário onde você precisa testar uma nova arquitetura de pipeline de dados. Como a agilidade da nuvem poderia acelerar esse processo comparado ao **on-premises**?

    !!! answer "Resposta"
        Na nuvem, você pode instantaneamente criar o ambiente de teste completo (bancos de dados, processamento, storage), testar sua arquitetura, e depois deletar tudo pagando apenas pelas horas utilizadas.
        
        No modelo **on-premises** isso poderia levar semanas apenas para conseguir *hardware* disponível para teste.

## Evolução dos Serviços em Nuvem

A era inicial da nuvem era dominada por ofertas de **Infrastructure as a Service (IaaS)**. No modelo **IaaS**, a provedora fornece produtos como *VMs* e discos virtuais, que são essencialmente **fatias alugadas** de *hardware*.

### Três Modelos de Serviço

#### Infrastructure as a Service (<span style="color:green">IaaS</span>)
- **O que é**: Recursos básicos de computação (VMs, storage, rede)
- **Responsabilidade**: Você gerencia o sistema operacional e aplicações
- **Exemplos**: Amazon EC2, Google Compute Engine, Azure Virtual Machines

#### Platform as a Service (<span style="color:green">PaaS</span>)
- **O que é**: Inclui IaaS + serviços gerenciados mais sofisticados
- **Responsabilidade**: Provedor gerencia infraestrutura e plataforma
- **Exemplos para Dados**: 
  - Bancos gerenciados: Amazon RDS, Google Cloud SQL
  - Streaming: Amazon Kinesis, Azure Event Hubs
  - Containers: Google Kubernetes Engine (GKE), Azure Kubernetes Service (AKS)

#### Software as a Service (<span style="color:green">SaaS</span>)
- **O que é**: Plataforma de software empresarial totalmente funcional
- **Responsabilidade**: Provedor gerencia tudo, você apenas usa
- **Exemplos**: Salesforce, Google Workspace, Fivetran (para engenharia de dados)

!!! info "Serverless: O Futuro?"
    **Serverless** está se tornando cada vez mais importante em ofertas PaaS e SaaS.
    
    Produtos **serverless** geralmente oferecem escalonamento automatizado de zero a taxas de uso extremamente altas, são cobrados por uso, e permitem que engenheiros operem sem conhecimento operacional dos servidores subjacentes.

!!! exercise choice "Question"
    Você precisa de um banco de dados **PostgreSQL** para seu pipeline de dados, mas não quer se preocupar com backups, atualizações de segurança ou monitoramento.
    
    Qual modelo de serviço seria mais adequado?

    - [ ] IaaS - Criar uma VM e instalar PostgreSQL
    - [X] PaaS - Usar um serviço gerenciado como Amazon RDS
    - [ ] SaaS - Usar uma ferramenta de BI
    - [ ] Serverless - Usar funções Lambda

    !!! answer "Resposta"
        **PaaS** seria o modelo ideal.
        
        Serviços como **Amazon RDS**, **Google Cloud SQL** ou **Azure Database** fornecem **PostgreSQL** totalmente gerenciado, onde o provedor cuida de backups, patches de segurança, monitoramento e manutenção.

## Por Que Cloud é Importante para Engenharia de Dados?

A nuvem oferece uma série de vantagens que a tornam importante para engenharia de dados:

### 1. Natureza dos Workloads de Dados

Os *workloads* de engenharia de dados têm características únicas:

- **Variabilidade**: Processamento **batch** pode precisar de 10x mais recursos durante execução
- **Experimentação**: Necessidade constante de testar novas arquiteturas e ferramentas  
- **Sazonalidade**: E-commerce tem picos na Black Friday, varejo tem sazonalidade
- **Crescimento imprevisível**: Volume de dados pode crescer exponencialmente

!!! example "COVID-19"
    A ocorrência do **COVID-19** em 2020 foi um grande *driver* da adoção de nuvem.
    
    Empresas reconheceram o valor de rapidamente escalar processos de dados para obter *insights* em um clima de negócios altamente incerto.
    
    Também tiveram que lidar com carga substancialmente aumentada devido ao pico de compras *online* e trabalho remoto.

### 2. Diversidade de Ferramentas

Engenharia de dados moderna requer uma ampla gama de ferramentas especializadas:

- **Armazenamento**: Data lakes, data warehouses, object storage
- **Processamento**: Batch processing, stream processing, ETL/ELT
- **Orquestração**: Workflow management, job scheduling
- **Monitoramento**: Observability, data quality, lineage

!!! info "Managed Services"
    Provedores de nuvem oferecem versões gerenciadas dessas ferramentas, permitindo que engenheiros **ignorem detalhes operacionais** de gerenciar máquinas individuais e implantar *frameworks* em sistemas distribuídos.

### 3. Modelo de *Pricing* Flexível

#### Problemas do Modelo Traditional
- **Overprovisioning**: Comprar para pico de capacidade
- **Underutilization**: Hardware parado maior parte do tempo
- **Capex vs Opex**: Grande investimento inicial *vs* custos operacionais flexíveis

#### Vantagens do Modelo Cloud
- **Pay-as-you-go**: Pague apenas pelos recursos utilizados
- **Spot instances**: Instâncias com desconto para *workloads* tolerantes a falhas
- **Reserved instances**: Descontos para comprometimentos de longo prazo

!!! exercise text long
    Uma empresa processa dados em *batch* uma vez por mês, usando 100 servidores por 8 horas. Como os custos se comparariam entre **on-premises** e cloud?

    !!! answer "Resposta"
        **On-premises**: Precisa manter 100 servidores 24/7, pagando por 720 horas por mês mesmo usando apenas 8 horas (mesmo que sejam desligados, foi necessário comprá-los).

        **Cloud**: Paga apenas pelas 8 horas × 100 servidores = 800 server-hours por mês. Potencial economia de custos de infraestrutura, energia, refrigeração, espaço físico e pessoal operacional.

        !!! warning "Não se engane!"
            A nuvem não é uma solução mágica e a praticidade tem seu custo!
            
            Decisões erradas podem levar a gastos inesperados.

### 4. Escalabilidade e Performance

#### Limitações Físicas
Estamos chegando aos **limites físicos** de quão rápido uma única CPU pode ser. Para escalar para mais dados, você deve ser capaz de **paralelizar** sua computação.

!!! info "*Horizontal* *vs* *Vertical Scaling*"
    - **Vertical scaling**: Comprar uma máquina melhor (limitado fisicamente)
    - **Horizontal scaling**: Adicionar mais máquinas (virtualmente ilimitado)

#### Algoritmos Paralelos
Isso levou ao surgimento de algoritmos paralelos *shared-nothing* e seus sistemas correspondentes, como **MapReduce**. A nuvem facilita enormemente esse tipo de processamento distribuído.

!!! exercise choice "Question"
    Seu pipeline de dados precisa processar 10TB de dados em 20 minutos para um relatório urgente. Qual abordagem seria mais eficaz?

    - [ ] Usar uma única máquina muito potente (*vertical scaling*)
    - [X] Distribuir o processamento entre 100 máquinas menores (*horizontal scaling*)
    - [ ] Processar sequencialmente durante a madrugada
    - [ ] Reduzir o volume de dados processados

    !!! answer "Resposta"
        **Horizontal scaling** seria mais eficaz.
        
        Distribuir entre 100 máquinas permite paralelização massiva, completando o *job* muito mais rápido que uma única máquina, independentemente de quão poderosa seja.

## Impacto na Engenharia de Dados Moderna

### Transformação dos Data Warehouses e Data Lakes

Três fatores transformaram completamente a paisagem de **analytics** e **data warehousing** nos últimos 10 anos:

1. **Facilidade de construção**: Deploy de **pipelines**, **data lakes**, **warehouses** e processamento analítico na nuvem sem larga dependência de departamentos de TI
2. **Queda contínua nos custos**: Armazenamento em nuvem ficou extremamente barato (pelo menos inicialmente!)
3. **Databases colunares escaláveis**: Amazon Redshift, Snowflake, Google BigQuery

!!! important "Relembrando definições Importantes"
    **Data Warehouse**: *Database* onde dados de diferentes sistemas são armazenados e **modelados para suportar análise**. Dados estruturados e otimizados para *queries* de relatório.

    **Data Lake**: Onde dados são armazenados sem a estrutura ou otimização de *query* de um **data warehouse**. Alto volume e variedade de tipos de dados.

??? info "Novas Arquiteturas"

    A nuvem permite arquiteturas que eram impraticáveis on-premises:

    - **Serverless data processing**: Processamento em escala, sob demanda, sem necessidade de gerenciar servidores ou infraestrutura
    - **Multi-region deployments**: Replicação global com baixa latência
    - **Event-driven architectures**: Processamento reativo baseado em eventos
    - **Elastic clusters**: Clusters que crescem e encolhem baseado na demanda

!!! exercise text long
    Como você descreveria, com suas palavras, a principal diferença entre um **data warehouse** e um **data lake**?
    
    Em que cenários cada um seria mais adequado?

    !!! answer "Resposta"
        **Data Warehouse** é como uma biblioteca organizada: dados limpos, catalogados e otimizados para consultas específicas. Ideal para relatórios regulares e análises de BI.

        **Data Lake** é como um depósito: armazena todos os tipos de dados (estruturados e não estruturados) sem organização prévia. Ideal para exploração de dados, *machine learning* e quando você não sabe ainda que análises precisará fazer.

## Considerações e Desafios

### Mudança de Mentalidade

Migrar para a nuvem requer uma **mudança dramática na forma de pensar**, especialmente sobre **pricing**. Empresas frequentemente cometem erros graves de *deployment* por não adaptar adequadamente suas práticas ao modelo de *pricing* da nuvem.

### FinOps: Financial Operations

!!! info "FinOps"
    **FinOps** é uma prática cultural e disciplina operacional que une equipes de tecnologia, negócios e finanças para acelerar a criação de valor de negócio através de tomada de decisão informada sobre dados financeiros na nuvem.

Principais práticas de FinOps para engenharia de dados:

- **Tagging e categorização** de recursos
- **Monitoring de custos** em tempo real
- **Right-sizing** de instâncias
- **Scheduling** de recursos não-críticos
- **Reserved instances** para workloads previsíveis

### Desafios Comuns

1. **Vendor lock-in**: Dependência de serviços específicos de um provedor
2. **Complexidade de pricing**: Centenas de serviços com modelos diferentes
3. **Segurança e compliance**: Novos desafios em ambiente compartilhado
4. **Skills gap**: Necessidade de novas competências técnicas

!!! exercise choice "Question"
    Uma empresa está preocupada com os custos crescentes de sua infraestrutura de dados na nuvem. Qual seria a primeira ação recomendada?

    - [ ] Migrar de volta para on-premises
    - [ ] Trocar de provedor de nuvem
    - [X] Implementar práticas de FinOps e monitoramento de custos
    - [ ] Reduzir a quantidade de dados processados

    !!! answer "Resposta"
        A primeira ação deve ser **implementar práticas de FinOps**.
        
        Muitas vezes os custos altos são resultado de recursos mal dimensionados, não tagueados ou executando desnecessariamente.
        
        Monitoramento e otimização são fundamentais antes de considerar mudanças drásticas.

## Conclusão

A computação em nuvem não é apenas uma questão de **onde** executar sua infraestrutura, é uma mudança fundamental em **como** pensamos sobre recursos computacionais, escalabilidade e operações.

Para engenheiros de dados, a nuvem oferece:

- **Agilidade** para experimentar e inovar rapidamente
- **Escalabilidade** para lidar com volumes crescentes de dados  
- **Diversidade** de ferramentas especializadas e gerenciadas
- **Flexibilidade** financeira com modelos pay-as-you-go

!!! warning "Próximos Passos"
    Nas próximas aulas, exploraremos na prática como utilizar serviços de nuvem da **AWS** para implementar *pipelines* de dados.

!!! exercise text long
    Reflita sobre sua futura carreira: em que contextos você escolheria **on-premises** *vs* **cloud**?
    
    Quais fatores seriam decisivos na sua recomendação para uma empresa?

    !!! answer "Resposta"
        **On-premises** pode ser adequado para:

        - Empresas com regulamentações rígidas de dados
        - Workloads extremamente previsíveis e estáveis  
        - Organizações com expertise técnica profunda
        - Casos onde a economia de escala justifica o investimento
        
        **Cloud** é geralmente preferível para:

        - Startups e empresas em crescimento
        - Workloads variáveis ou experimentais
        - Necessidade de agilidade e inovação
        - Falta de expertise em infraestrutura
        - Requisitos de escalabilidade global
        
        A decisão deve considerar: **custo total de propriedade**, **requisitos de compliance**, **expertise disponível**, **velocidade de inovação necessária** e **características dos workloads**.

## Leituras interessantes

Leitura opcional sobre empresas deixando a nuvem:

- [Dropbox 1](https://www.wired.com/2016/03/epic-story-dropboxs-exodus-amazon-cloud-empire/) e [Dropbox 2](https://www.datacenterknowledge.com/cloud/dropbox-s-reverse-migration-from-cloud-to-own-data-centers-five-years-on)
- [Why Companies Are Ditching the Cloud](https://thenewstack.io/why-companies-are-ditching-the-cloud-the-rise-of-cloud-repatriation/)