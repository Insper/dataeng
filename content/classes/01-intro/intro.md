# Introdução

Nos últimos dez anos, a quantidade de dados gerados cresceu de forma exponencial, ultrapassando 30 mil gigabytes por segundo, e esse ritmo continua a acelerar (DDIA).

!!! info
    As referências de cada aula sempre estarão listadas ao final da última página da aula.

Aposto que você já cursou outras disciplina com a palavra *"Dado"* no título! Então, tente relembrar seus aprendizados e responder o exercício na sequência.

!!! exercise text long
     defina, com suas palavras, o conceito de **"Dado"**.

    !!! answer "Answer"
        Um **dado** é a representação **bruta e não processada** de um fato, evento ou conceito. Ele é a unidade mais básica, isolada, que por si só não carrega um significado completo ou contexto. Pense no dado como a matéria-prima de todo o conhecimento.

        Em um contexto de computação, dados podem ser:

        * **Valores numéricos:** "150", "3.14"
        * **Caracteres:** "a", "Z", "@"
        * **Strings:** "José da Silva", "endereço@email.com"
        * **Símbolos:** como os de um código de barras.

        De forma isolada, um dado como `"150"` não nos diz nada. Ele pode ser uma idade, um valor monetário, uma quantidade ou qualquer outra coisa. Ele é apenas um registro sem um propósito claro.


Esses dados são extremamente diversos, abrangendo desde conteúdos produzidos por usuários — como postagens em blogs, tweets, interações em redes sociais e fotografias — até registros de sistemas e sensores.

??? "A diferença entre **dado** e **informação**"
    A **informação** surge quando os dados são processados, organizados e contextualizados, ganhando um significado que permite a tomada de decisões ou a construção de conhecimento. A informação é o resultado da interpretação dos dados.

    A principal distinção é que a informação responde a perguntas como "quem?", "o quê?", "onde?", "quando?" e "por que?".

    Vamos usar um exemplo prático para ilustrar a diferença:

    * **Dados:** "João", "30", "Rua das Palmeiras, 100", "2024-08-09"
    * **Informação:** "João, que tem 30 anos, mora na Rua das Palmeiras, 100. Essas informações foram registradas em 9 de agosto de 2024."

    Neste caso, a informação foi criada ao combinar e interpretar os dados brutos. O dado "30" se tornou a "idade" de "João", e o conjunto de dados formou um registro coerente e útil.

    Em resumo, a relação entre os dois conceitos pode ser vista como um ciclo:

    1.  **Dados** são coletados.
    2.  **Dados** são processados e organizados.
    3.  O resultado é **informação**, que tem um significado claro.
    4.  Essa **informação**, ao ser utilizada e interpretada, gera **conhecimento**.

Nesse contexto, as empresas **data-driven** buscam ativamente coletar, processar e analisar esses dados brutos para transformá-los em **informações estratégicas**.

O objetivo principal é utilizar essa inteligência para otimizar operações, tomar decisões mais precisas e orientadas por **evidências**, prever tendências de mercado, personalizar a experiência do cliente e, em última análise, impulsionar o crescimento e a inovação.

!!! info
    Todo negócio, seja ele consciente disso ou não, requer **análise de dados**.

É neste contexto, de **prover capacidade de extrair valor dos dados**, que surgem a **engenharia de dados** e **ciência de dados**.

## O Que é Engenharia de Dados?

Embora hoje o termo seja amplamente utilizado, ainda existe confusão sobre o que realmente significa engenharia de dados. Na prática, ela existe desde que empresas começaram a usar dados para análises preditivas, relatórios e estudos descritivos, mas ganhou destaque com a **ascensão da ciência de dados** a partir de **2010**.

!!! important "Definição"
    Vamos utilizar a seguinte definição para **Engenharia de Dados**:

    *Engenharia de Dados é a disciplina técnica e prática que se dedica ao design, construção e manutenção de sistemas e infraestrutura que possibilitam a **coleta**, **processamento**, **análise** e **armazenamento** de grandes volumes de dados. Ela abrange atividades como extração, transformação e carga (**ETL**), além de garantir a **integridade**, **acessibilidade**, **relevância** e **escalabilidade** dos dados. O objetivo é assegurar que os dados estejam limpos, estruturados e prontos para serem utilizados por cientistas de dados e analistas em processos analíticos, preditivos e de tomada de decisão.*

    Ou de forma mais simples:

    *A engenharia de dados é o campo dedicado ao fluxo, processamento e administração de dados, garantindo sua organização e utilização eficiente.*

??? info "Definição conforme a bibliografia oficial"

    Esta é a definição conforme o livro *Fundamentals of Data Engineering* (que iremos referenciar como **FDE**), uma das bibliograficas principais do curso:

    !!! info
        As referências de cada aula sempre estarão listadas ao final da última página da aula.

    *"A engenharia de dados envolve a criação, implementação e manutenção de sistemas e fluxos de trabalho que transformam dados brutos em informações de alta qualidade e consistentes. Essas informações são essenciais para apoiar atividades como análise de dados e aprendizado de máquina. Essa área abrange a integração de segurança, gerenciamento de dados, DataOps, arquitetura de dados, orquestração de processos e engenharia de software. O engenheiro de dados é responsável por gerenciar o ciclo de vida dos dados, desde a captura das fontes de dados até sua disponibilização para os mais diversos casos de uso, como análise e machine learning."*

Sendo assim, o principal papel de um engenheiro de dados é construir e manter os sistemas que coletam, armazenam e preparam grandes volumes de dados para uso.
<!-- Pense em como construímos aplicações hoje: elas geralmente são intensivas em dados, não em processamento. Os maiores desafios não são a força bruta do processador, mas sim a quantidade e a complexidade dos dados, além da velocidade com que eles mudam. -->

## Aplicações Intensivas em Dados

Os sistemas ou aplicações modernos são tipicamente **intensivos em dados**, não em processamento. O poder bruto da CPU raramente é o fator limitante. Os maiores desafios são o volume de dados, sua complexidade e a velocidade com que mudam.

Essas aplicações são construídas com blocos padronizados que fornecem funcionalidades essenciais: armazenar dados para recuperação posterior (bancos de dados), lembrar resultados de operações complexas (caches), permitir buscas e filtros (índices de busca), enviar mensagens entre processos (processamento de *streams*) e processar grandes volumes acumulados (processamento em lote).

## A Complexidade da Escolha

Embora esses sistemas de dados sejam abstrações bem-sucedidas que usamos constantemente, a realidade não é simples. Existem diversos sistemas de banco com características diferentes porque aplicações têm requisitos distintos. Há várias abordagens para cache, múltiplas formas de construir índices de busca. Ao desenvolver uma aplicação, ainda precisamos descobrir quais ferramentas e abordagens são mais apropriadas para cada tarefa, e pode ser desafiador combinar ferramentas quando uma única não resolve tudo sozinha.

## Nosso objetivo

Neste curso, nosso objetivo é desmistificar a engenharia de dados, preparando você para os desafios e oportunidades profissionais da área. Nosso foco é garantir que, ao final do curso, você saiba tanto a teoria quanto aplicar as ferramentas e processos de infraestrutura de dados na prática, implementando soluções escaláveis e eficientes que atendam às necessidades de diferentes organizações.
