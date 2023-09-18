---
title: Serverless
author: Bruno Russo
date: 2023-09-18 22:00 0300
categories: [Cloud, AWS, Serverless]
tags: [Arquitetura, aws, Serverless, desenvolvimento]
layout: post
type: post
---

## Introdução

When you are designing your architecture, don't focus on this question: ”What’s the data that I’m storing and what operations do I need to perform against that?" Instead, ask yourself this: “What are the events that should trigger an action in my system?"


## O que Serverless realmente significa?

Serverless possui 4 características principais:

1. **Ser servidor para gerenciar** - não existe patch para ser aplicado.
2. **Escalabilidade flexível** - define-se uma capacidade mínima, como: memória, CPU, quantidade de leitura/escrita ou volume de dados que será enviado, e o ambiente pode escalar de forma automática essa capacidade de acordo com o a necessidade que o ambiente necessitar
3. **Alta disponibilidade automática e tolerância a falhas** - serviços serverless possuem essas características por padrão e isso não pode ser desligado
4. **Sem capacidade ociosa** – o custo é com base na utilização.


## Do monolito ao Serverless

Existem muitos paradigmas que precisam ser atualizados quando estamos tratando de uma migração para um ambiente serverless. Se olharmos os serviços oferecidos pela AWS, com certeza os mais famosos são S3, Lambda, DynamoDB e API Gateway. Entretanto, existem vários outros serviços que devem ser utilizados para obter o máximo de aproveitamento de uma arquitetura serverless.

Quando olhamos uma aplicação com arquitetura monolítica, toda a gestão pelos componentes é de sua responsabilidade. Na grande maioria das vezes a gestão desse tipo de ambiente é complexa e cara. Além disso, desperdiça o tempo dos engenheiros em rotinas de manutenção ao invés de usá-los nos produtos core da empresa.

Ao migrar uma aplicação para um ambiente serverless, todas as preocupações com gestão do ambiente deixam de existir. A grande chave para conseguir êxito em uma mudança como essa é se preocupar em construir serviços orientados a eventos. É uma grande mudança de paradigma, pois cada solicitação ou alteração se torna um evento que ocorreu e desta forma os componentes deixam de estarem unidos e dependentes para se tornarem desacoplados.


## Lambda

AWS Lambda é um serviço de computação, onde você executa o código de sua aplicação sem provisionar e gerenciar servidores.

5 benefícios ao usar Lambda:

1. executar um código sem provisionar e gerenciar servidores
2. a Lambda é iniciada através de um evento
3. o scaling é automático
4. fornece monitoração integrada com Amazon CloudWatch
5. Integração com diversos serviços da AWS

Ao usar uma Lambda, você está automaticamente se beneficiando de uma arquitetura baseada em eventos. Muitos serviços da AWS criam eventos e estes podem ser a origem para iniciar uma Lambda.


### Como a Lambda funciona

Para que a Lambda seja executada, é necessário que um evento ocorra. Este evento pode executar a Lambda de três maneiras:

1. Síncrona
    1. Invocação - neste caso a Lambda é invocada e executa código, realizando o processamento, em seguida e como saída um retorno ocorre. Toda a estratégia de código é definida por você.
    2. Serviço AWS - a Lambda é iniciada após ser invocada por algum serviço da AWS, que pode ser: API Gateway, Cognito, CloudFormation, Alexa, Lex, CloudFront, etc
    3. Tipo de erro a ser considerado: sem possibilidade de retry
2. Assíncrona
    4. Invocação - neste modelo de execução, a Lambda é invocada e o requisito não aguarda nenhum tipo de resposta. Esse tipo de modelo pode ser muito útil em cenários onde após a execução da Lambda, um outro determinado serviço é invocado.
    5. Serviço AWS - Assim como a forma síncrona, vários serviços da AWS podem ser usados neste cenário, como por exemplo SNS, S3, Event Bridge, etc
    6. Tipo de erro a ser considerado: integrado, com duas tentativas de retry
3. Polling
    7. Polling - neste cenário existe uma integração com serviços baseados em streaming ou fila. A lambda faz um poll nos serviços e uma vez identificado um evento a função Lambda é invocada. Para este cenário, estão integrados Kinesis, SQS e DynamoDB Streams.
    8. Serviço AWS - os seguintes serviços podem ser integrados: DynamoDB, Kinesis, MQ, MSK, SQS, etc
    9. Tipo de erro a ser considerado: depende da origem do evento


### Ambiente de execução de uma Lambda

O ambiente de execução da Lambda consiste em três etapas:

1. Fase Inicial - ocorre a configuração inicial do ambiente onde a função Lambda será executada, consiste em três fases
    1. Extension init - inicia todas as extensões necessárias
    2. Runtime init - realiza o bootstrap do runtime
    3. Function init - executa o código estático da função
2. Fase de Invocação - nesta etapa ocorre a invocação da função handler
3. Fase de Desligamento - nesta etapa ocorre o encerramento da execução da função Lambda, assim como uma limpeza em todo o ambiente criado e inicializado para sua execução.

Performance é um item importante e a possibilidade de otimizar a performance é algo possível de ser realizado.

Quando a Lambda é inicializada, ocorre o que chamamos de Cold Start, ou seja todo o ambiente é inicializado para a execução da função, o que leva um tempo (em muitas vezes é de milissegundos)  para ocorrer. 

Visando melhorar a performance, existe a possibilidade de executar a função em um ambiente denominado Warm Start, ou seja, é um ambiente pré inicializado. Neste cenário a Lambda não exclui o ambiente preparado para executar a função, e isso reduz o tempo de inicialização do ambiente.


### Segurança

Existem duas etapas relacionadas a segurança para que uma função seja executada.

1. É necessário ter permissão para invocar uma função
2. A função precisa ter permissão para acessar os recursos necessários durante a sua execução

Essas permissões são controladas via IAM.


### Método handler

O handler é o método da função que irá processar os eventos, é composto de:

* Event
    * é obrigatório
    * vai diferir em estrutura e conteúdo, com base no evento que iniciou a invocação da Lambda
* Context object (opcional)
    * possibilita que a função interaja com o ambiente em execução, como por exemplo:
        * AWS RequestID - usado para fazer um track da invocação específica
        * Runtime - tempo em milissegundos antes do timeout
        * Logging - informar qual stream de log d CloudWatch deve ser usado para o registro de logs


### Boas práticas

Abaixo estão listadas algumas boas práticas ao codificar a função Lambda. Algumas linguagens de programação possuem particularidades específicas e por isso é recomendável ler atentamente a documentação oficial.

* Uma recomendação é separar o código de negócio do método handler, isso facilita a portabilidade do código e também na escrita de testes unitários.
* Outra recomendação é escrever funções modulares. Por exemplo, imagine o cenário onde uma função insere uma marca dˋágua em uma imagem e gera miniaturas desta imagem. O ideal é ter pelo menos duas funções, uma para criar a marca dˋágua e outra função para gerar as miniaturas da imagem.
* Não salva nenhuma informação no contexto de uma função. As funções são stateless.
* Inclua apenas as dependências que for realmente utilizar na função. Isso diminui o tamanho e o tempo de inicialização de cada uma.
* Inclua declarações de log em seu código. Os logs por padrão, são enviados para o CloudWatch
* Use o código de retorno ao término da função. Cada linguagem possui a sua particularidade.
* Faça uso de variáveis de ambiente. As variáveis de ambiente podem ser alteradas, sem alterar qualquer código na sua função. Além disso, informações sensíveis podem ser declaradas em variáveis garantindo que a informação não fique exposta diretamente no código.
* Evite a recursividade. Criar uma função que durante sua execução faz uma invocação para si mesma, pode causar um escalonamento de invocações, ocasionando perda de controle.


### Simultaneidade (concorrência)

A simultaneidade é um parâmetro muito importante que precisa ser definido na configuração da Lambda. Isso garante a quantidade de invocações que uma função pode  executar ao mesmo tempo.

Existem três tipos:

1. Simultaneidade não reservada - é a quantidade de simultaneidade que não é alocada para nenhum conjunto específico de funções. O valor mínimo é de 100 reservas. Isso significa que você terá 100 reservas que podem ser utilizadas por qualquer função, minimizando qualquer tipo de impacto.
2. Simultaneidade reservada - garante o número máximo de instâncias reservadas para a função. Quando um valor está  definido para uma função, nenhuma outra pode usar essa mesma simultaneidade. Não há custo pela reserva.
3. Simultaneidade provisionada - configura um número de ambientes inicializados, para que  atendam às solicitações quando a função for invocada. Isso garante alto desempenho e baixa latência. Há um custo que precisa ser levado em consideração para este tipo de necessidade.


### Monitorando

A função Lambda é integrada com o CloudWatch e isso permite analisar de forma gráfica diversas métricas. Alguns itens que podem ser observados são:

* Invocações - número de vezes que a função foi invocada e o seu status da execução.
* Duração - a quantidade de tempo que a função utilizou para processar um evento.
* Erros - o número de invocações que terminaram com erro
* Limitações (throttles) - a quantidade de vezes que houve uma falha no processo devido a limites de simultaneidade
* IteratorAge - refere-se ao mapeamento de origem de eventos que leem fluxos e o tempo do último evento registrado.
* DeadLetterErros - quantidade de vezes que a Lambda tenta enviar um evento para uma fila de mensagens mortas, mas falha.
* Execuções concorrentes - quantidade de instâncias da função que estão processando algum evento.