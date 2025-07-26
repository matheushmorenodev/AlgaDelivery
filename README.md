# 🚀 AlgaDelivery - Microsserviços

Este projeto é uma demonstração de uma arquitetura de microsserviços para um sistema de entregas, desenvolvido como parte de um estudo aprofundado em tecnologias de backend e sistemas distribuídos. O sistema é composto por serviços independentes que se comunicam de forma síncrona (REST) e assíncrona (eventos com Kafka).

## 🏛️ Arquitetura do Sistema

A arquitetura é dividida nos seguintes componentes:

* **`delivery-tracking`**: Microsserviço principal responsável por gerenciar todo o ciclo de vida de uma entrega, desde o rascunho até a finalização.
* **`courier-management`**: Microsserviço que gerencia o cadastro de entregadores e atribui entregas a eles com base em eventos.
* **`eureka-server`**: Serviço de *Service Discovery* que permite que os microsserviços se encontrem dinamicamente na rede.
* **`PostgreSQL`**: Sistema de banco de dados utilizado para a persistência dos dados, com um banco de dados dedicado para cada microsserviço.
* **`Apache Kafka`**: Plataforma de streaming de eventos utilizada para a comunicação assíncrona entre os serviços.

### Fluxo de Comunicação

1.  **Criação da Entrega (REST)**: Ao criar uma nova entrega, o serviço `delivery-tracking` faz uma chamada síncrona via REST para o `courier-management` a fim de calcular o valor do repasse (payout) ao entregador.
2.  **Atribuição da Entrega (Kafka)**: Quando o pedido da entrega é efetivado (`placed`), o `delivery-tracking` publica um evento no Kafka. O `courier-management` consome este evento e atribui a entrega a um entregador disponível.

![Fluxo](https://i.imgur.com/K1nUaWk.png)

## 🛠️ Tecnologias Utilizadas

* **Linguagem**: Java 17
* **Framework**: Spring Boot
* **Comunicação**: Spring Web, REST Assured
* **Mensageria**: Spring for Apache Kafka
* **Banco de Dados**: Spring Data JPA, PostgreSQL
* **Service Discovery**: Spring Cloud Netflix Eureka
* **Resiliência**: Resilience4j (Circuit Breaker, Retry)
* **Containerização**: Docker e Docker Compose

## 🔧 Pré-requisitos

Antes de começar, garanta que você tenha as seguintes ferramentas instaladas em seu ambiente:

* [Git](https://git-scm.com/)
* [Java Development Kit (JDK) 17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)
* [Docker](https://www.docker.com/get-started/) e [Docker Compose](https://docs.docker.com/compose/install/)
* Um cliente de API como [Postman](https://www.postman.com/) ou [Insomnia](https://insomnia.rest/)

## ⚙️ Como Executar o Projeto

Siga os passos abaixo para clonar, configurar e executar a aplicação completa.

### 1. Clonar o Repositório

```bash
git clone [https://github.com/seu-usuario/seu-repositorio.git](https://github.com/seu-usuario/seu-repositorio.git)
cd seu-repositorio
2. Iniciar a Infraestrutura com Docker
Toda a infraestrutura necessária (PostgreSQL, Kafka, pgAdmin, Kafka-UI e Eureka) é gerenciada pelo Docker Compose. O serviço do Eureka é construído localmente para garantir a estabilidade.

Bash

# O --build é importante na primeira vez para construir a imagem do Eureka
docker-compose up -d --build
Após a execução, os seguintes serviços estarão disponíveis:

Eureka Server: http://localhost:8761

PostgreSQL: localhost:5432

pgAdmin: http://localhost:8083

Kafka UI: http://localhost:8084

3. Criar os Bancos de Dados
A criação dos bancos de dados precisa ser feita manualmente através do pgAdmin.

Acesse o pgAdmin em http://localhost:8083.

Faça login com as credenciais: dba@algadelivery.com / algadelivery.

Crie uma nova conexão de servidor com o host postgres e usuário/senha postgres.

Clique com o botão direito em "Databases" -> "Create" -> "Database..." e crie os dois bancos:

deliverydb

courierdb

4. Iniciar os Microsserviços
Abra dois terminais separados, um para cada microsserviço.

Terminal 1: Iniciar courier-management

Bash

cd courier-management
./mvnw spring-boot:run
Terminal 2: Iniciar delivery-tracking

Bash

cd delivery-tracking
./mvnw spring-boot:run
Aguarde as aplicações iniciarem. Verifique o painel do Eureka em http://localhost:8761 para confirmar que COURIER-MANAGEMENT e DELIVERY-TRACKING estão com o status UP.

✅ Testando a Aplicação
Siga este fluxo para testar a comunicação de ponta a ponta.

1. Cadastrar um Entregador
POST http://localhost:8081/api/v1/couriers

Body (JSON):

JSON

{
    "name": "Maria Souza",
    "phone": "35999887766"
}
2. Criar um Rascunho de Entrega
POST http://localhost:8080/api/v1/deliveries

Body (JSON):

JSON

{
  "sender": {
    "zipCode": "37701-001", "street": "Rua Principal", "number": "100",
    "complement": "Apto 10", "name": "Loja A", "phone": "3537221122"
  },
  "recipient": {
    "zipCode": "37704-002", "street": "Rua Secundária", "number": "200",
    "complement": "Casa", "name": "João Cliente", "phone": "35988776655"
  },
  "items": [
    { "name": "Livro de Java", "quantity": 1 },
    { "name": "Mouse sem fio", "quantity": 1 }
  ]
}
Guarde o id retornado na resposta.

3. Efetivar o Pedido (Disparar Evento)
POST http://localhost:8080/api/v1/deliveries/{deliveryId}/placement

Substitua {deliveryId} pelo ID que você guardou no passo anterior.

Verificação:

Acesse a Kafka UI em http://localhost:8084, navegue até o tópico deliveries.v1.events e observe a nova mensagem DeliveryPlacedEvent.

Acesse o pgAdmin e verifique se uma assigned_delivery foi criada na tabela correspondente do banco courierdb.

📄 Licença
Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

Feito com ❤️ por Matheus Moreno
