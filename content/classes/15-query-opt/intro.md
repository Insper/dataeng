# Introdução

Nas aulas anterires, exploramos como definir e orquestrar pipelines, abordamos arquiteturas como *data warehouses* e *data lakehouse*.

Considerando que nossos sistemas estão operacionais e os dados fluem de forma confiável, muito provavelmente um novo desafio surgirá: a **performance**!

!!!! info "Importante!"
    Não basta que uma consulta **SQL** retorne os dados corretos; ela precisa fazer isso de forma eficiente, especialmente quando lidamos com volumes de dados em crescimento.

Imagine os seguintes cenários:

1. Um sistema transacional que demora dezenas de segundos para processar o cadastro de um usuário.
2. Um *dashboard* analítico, que antes carregava em poucos segundos, agora necessita de alguns minutos para ser exibido.

A queda de performance pode frustrar clientes, analistas de negócios e atrasar a disponibilização de informações críticas para a tomada de decisão. A funcionalidade continua a mesma, os dados estão corretos, mas a lentidão torna a solução impraticável.

!!! exercise text long
    Com base em seus conhecimentos prévios sobre bancos de dados e desenvolvimento de software, quais você acredita que poderiam ser as causas para essa degradação de performance? Onde você começaria a investigar o problema?

    !!! answer "Resposta"
        A degradação de performance em sistemas de banco de dados geralmente está associada a alguns fatores chave. O mais óbvio é o **aumento do volume de dados**: uma consulta que é eficiente em uma tabela com mil linhas pode se tornar insustentável em uma com milhões.
        
        Outras causas comuns incluem a **ausência de índices** apropriados, forçando o banco a varrer tabelas inteiras (*full table scan*), ou **consultas mal escritas** que realizam junções ineficientes ou subconsultas desnecessárias. A investigação geralmente começa analisando as consultas mais lentas para entender como o banco de dados está, de fato, executando-as.


Nesta aula, nosso objetivo é explorar detalhes de performance no **PostgreSQL**, em especial, o otimizador de consultas.

Siga para a próxima página para começar!
