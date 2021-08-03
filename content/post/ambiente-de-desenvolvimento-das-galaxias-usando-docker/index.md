+++
title = "Ambiente de desenvolvimento das galáxias usando Docker"
slug = "ambiente-de-desenvolvimento-das-galaxias-usando-docker"
author = "leocarmo"
date = "2021-08-03T10:00:00-03:00"
tags = ["docker", "kafka", "redis", "mysql", "mariadb", "postgres", "mongodb"]
description = "Tenha um ambiente de desenvolvimento completo com as principais ferramentas usando Docker."
cover = "/ambiente-de-desenvolvimento-das-galaxias-usando-docker/images/cover.jpeg"
+++

Criar um ambiente de desenvolvimento local pode parecer um desafio. Quem nunca sofreu *~ pelo menos um pouco ~* para conseguir ter as dependências rodando na sua maquina para criar um projeto?

Pois é, normalmente passamos por isso quase sempre. Antes de conhecer o [Docker](https://www.docker.com/), isso era muito mais difícil... Ter que subir um banco de dados localmente sempre dava algum tipo de dor de cabeça, ainda mais quando era preciso ter uma versão específica por conta de algum projeto já existente que precisava de manutenção.

A ideia é te ajudar a montar um ambiente de desenvolvimento local de forma fácil e rápida usando tudo com Docker. Além disso, vou te mostrar algumas ferramentas de apoio que podem ajudar muito na hora de debugar algumas coisas.

É importante que você tenha pelo menos uma base sobre Docker e [Docker Compose](https://docs.docker.com/compose/) e claro, tenha ele [instalado](https://www.docker.com/products/docker-desktop) na sua máquina.

Para facilitar, sugiro que você clone o projeto que deixei preparado para este artigo com todos os arquivos prontos:

[https://github.com/leocarmo/docker-local-stack](https://github.com/leocarmo/docker-local-stack)

```sh
git clone git@github.com:leocarmo/docker-local-stack.git
```

Antes de iniciar as ferramentas, criaremos uma [rede para nossos containers](https://docs.docker.com/network/), pois teremos vários arquivos diferentes para nossas stacks, assim você pode escolher de forma mais fácil qual utilizar e quando.

```sh
docker network create global-default
```

---

Tópicos:
1. [Visibilidade de containers](#1-visibilidade-de-containers)
2. [Redis](#2-redis)
3. [Kafka](#3-kafka)
4. [MongoDB](#4-mongodb)
5. [ElasticSearch](#5-elasticsearch)
6. [Databases](#6-databases)
7. [Mock](#7-mock)
8. [AWS](#8-aws)

---

## 1. Visibilidade de containers

A primeira ferramenta que gosto de usa é o [DockStation](https://dockstation.io/), infelizmente ele precisa ser instalado na sua máquina, prometo que será o único.

Ele tem uma infinidade de ferramentas para te ajudar a gerenciar melhor seus containers, o que precisaremos em breve, e uma interface bem amigável.

Aqui tem um preview dele:

![DockStation](https://cdn.hashnode.com/res/hashnode/image/upload/v1626463235111/WL8MPPAnBw.png)

De qualquer forma, caso você não queira instalar nada na sua máquina, existe uma alternativa que é o [Portainer](https://www.portainer.io/), ele pode ser iniciado via container, e também tem diversas ferramentas disponíveis para te ajudar.

![Portainer](https://cdn.hashnode.com/res/hashnode/image/upload/v1626464059771/lyRrNLO6X.png)

O exemplo para rodar via `compose` está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/container-visibility/docker-compose-portainer.yaml).

Para iniciar o Portainer:

```sh
docker compose -f container-visibility/docker-compose-portainer.yaml up -d
```

---

## 2. Redis

Falando sobre [Redis](https://redis.io/), normalmente usamos ele de duas formas: single node ou cluster. Ainda temos outra ferramenta que traz uma visibilidade incrível, evitando que seja preciso criar código para debugar chaves, consumo de recursos e ainda consegue sugerir melhorias nas buscas, por exemplo. Falaremos dela em breve.

Começando pelo Redis single node, [esse aqui](https://github.com/leocarmo/docker-local-stack/blob/main/redis/docker-compose-single.yaml) é um exemplo do `compose`. Para iniciar:

```sh
docker compose -f redis/docker-compose-single.yaml up -d
```

Sim, bem simples. Agora montaremos um cluster. Para isso, utilizaremos as imagens da [Bitnami](https://bitnami.com/), eles possuem diversas outras ferramentas já prontas para utilizar também, vale a pena dar uma olhada.

> Vale pontuar que não é uma imagem oficial do Redis como a do single node. É possível criar um cluster utilizando a imagem oficial, mas nosso foco aqui é para desenvolvimento local, então utilizaremos o que é mais fácil e rápido para ter rodando.

O repositório com o `docker-compose` original da Bitnami está no [github](https://github.com/bitnami/bitnami-docker-redis-cluster) deles, porém, realizei algumas pequenas modificações para que ele possa ter contato com todos os outros containers que utilizaremos aqui.

O `compose` do Redis cluster com algumas pequenas mudanças que fiz está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/redis/docker-compose-cluster.yaml). Para iniciar:

```sh
docker compose -f redis/docker-compose-cluster.yaml up -d
```

Ótimo, agora temos um single node e um cluster de Redis funcionando!

Vou te mostrar uma ferramenta que gosto muito, o [RedisInsight](https://redislabs.com/redis-enterprise/redis-insight/), que é uma GUI para interagir com o Redis (por isso modifiquei tudo para ficar na `network global-default`).

![RedisInsight](https://cdn.hashnode.com/res/hashnode/image/upload/v1626467588436/4D_s2nTu-.png)

Então nosso `compose` do RedisInsight fica [assim](https://github.com/leocarmo/docker-local-stack/blob/main/redis/docker-compose-insight.yaml). Para iniciar:

```sh
docker compose -f redis/docker-compose-insight.yaml up -d
```

Para conectar o RedisInsight ao nosso cluster ou ao single node é bem simples, ele fica neste endereço: [http://0.0.0.0:8001/](http://0.0.0.0:8001/). Depois, basta selecionar que você já possui um database e colocar as informações para conectar. Um exemplo para conectar no cluster:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1626467726998/oKqgGWqRTc.png)

> Não esqueça da senha: *redis*

---

## 3. Kafka

Para ter um cluster de [Apache Kafka](https://www.confluent.io/what-is-apache-kafka/) rodando, precisamos do [ZooKeeper](https://zookeeper.apache.org/). É bem simples também, [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/kafka/docker-compose-cluster.yaml) tem o `compose` com tudo isso, sendo um cluster de Kafka com 3 brokers. Para rodar:

```sh
docker compose -f kafka/docker-compose-cluster.yaml up -d
```

O destaque aqui vai para a ferramenta [Kowl](https://github.com/cloudhut/kowl), que é uma UI para o Kafka, gostei bastante dela comparada a outras.

![Kowl Kafka](https://github.com/cloudhut/kowl/raw/master/docs/assets/preview.gif)

O Kowl ficará disponível em [http://0.0.0.0:8080/](http://0.0.0.0:8080/).

Ele tem a versão open-source, mas também tem uma versão Business, enquanto ela está na versão beta, você pode conseguir uma licença limitada, porém gratuita e eterna [aqui](https://license.cloudhut.dev/).

> Até o dia de hoje, ainda está liberado, então aproveita!

---

## 4. MongoDB

Para o Mongo, foi bem simples montar algo que já resolva localmente nossa necessidade. Coloquei também o [mongo-express](https://github.com/mongo-express/mongo-express) no `compose` para ajudar no debug.

![mongo-express](https://cdn.hashnode.com/res/hashnode/image/upload/v1626477968205/cU65kS0bc.png)

O `compose` final está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/mongodb/docker-compose-single.yaml). Caso queira ver mais parâmetros disponíveis na imagem oficial, você pode acessar [aqui](https://hub.docker.com/_/mongo). Para rodar:

```sh
docker compose -f mongodb/docker-compose-single.yaml up -d
```

O mongo-express ficará disponível em [http://0.0.0.0:8081/](http://0.0.0.0:8081/).

Caso você precise de algo mais robusto, a Bitnami também tem um `compose` pronto com múltiplos shards, vale a pena conferir [aqui](https://github.com/bitnami/bitnami-docker-mongodb-sharded/blob/master/docker-compose-multiple-shards.yml).

---

## 5. ElasticSearch

Este também foi fácil, a própria [documentação oficial](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-docker.html) disponibiliza um `compose` praticamente pronto, eu apenas reduzi um node, acredito que dois já são suficientes para desenvolvimento local, o link está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/elasticsearch/docker-compose-cluster.yaml). Para rodar:

```sh
docker compose -f elasticsearch/docker-compose-cluster.yaml up -d
```

O ElasticSearch ficará disponível em [http://0.0.0.0:9200/](http://0.0.0.0:9200/) e o Kibana em [http://0.0.0.0:5601/](http://0.0.0.0:5601/).

O destaque aqui vai para o [Cerebro](https://github.com/lmenezes/cerebro), uma UI que ajuda bastante na hora de entender como está a distribuição dos shards, consumo de recurso, entre outras informações importantes do cluster.

![Cerebro](https://cdn.hashnode.com/res/hashnode/image/upload/v1626479198955/m7NkaWdQr.png)

O `compose` do Cerebro está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/elasticsearch/docker-compose-cerebro.yaml) e ficará disponível em [http://0.0.0.0:9000/](http://0.0.0.0:9000/). Para rodar:

```sh
docker compose -f elasticsearch/docker-compose-cerebro.yaml up -d
```

Para começar usar é simples, basta colocar o endereço do ElasticSearch, no nosso caso o nome do container e a porta: `http://es01:9200`.

---

## 6. Databases

Nesta seção, deixei os principais bancos preparados: [MariaDB](https://hub.docker.com/_/mariadb), [MySQL](https://hub.docker.com/_/mysql) e o [Postgres](https://hub.docker.com/_/postgres) com [pgAdmin4](https://www.pgadmin.org/).

Também são bem simples, o `compose` do MariaDB está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/databases/docker-compose-mariadb.yaml), do MySQL [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/databases/docker-compose-mysql.yaml) e do Postgres [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/databases/docker-compose-postgres.yaml). Para rodar cada um:

```sh
docker compose -f databases/docker-compose-mariadb.yaml up -d
docker compose -f databases/docker-compose-mysql.yaml up -d
docker compose -f databases/docker-compose-postgres.yaml up -d
```

Ao subir os containers, os volumes serão criados em `databases/var`, no mesmo diretório dos arquivos do `compose`.

Para gerenciar o Postgres, o pgAdmin estará disponível em [http://0.0.0.0:5050/](http://0.0.0.0:5050/):

![pgAdmin](https://cdn.hashnode.com/res/hashnode/image/upload/v1626483179054/OYLK7Q5QT.png)

Para gerenciar o MariaDB e MySQL, sugiro utilizar o [MySQL Workbench](https://dev.mysql.com/downloads/workbench/), que infelizmente não tem sua versão em container... Mas vale a pena, ele é muito completo e vai te ajudar bastante.

![MySQL Workbench](https://cdn.hashnode.com/res/hashnode/image/upload/v1626483359586/3AK3FMQhK.png)

---

## 7. Mock

Sempre que estamos desenvolvendo microsserviços, em algum momento vamos nos deparar com alguma dependência externa, e isso pode ser uma grande dor de cabeça.

Para ajudar nessa questão, gosto muito de utilizar um [serviço de mock](https://www.mock-server.com/#what-is-mockserver) para desenvolvimento. Além de ajudar muito da hora de codar, pode ser muito útil também para realização de testes, por exemplo. [Aqui](https://www.mock-server.com/#why-use-mockserver) tem mais alguns benefícios de se utilizar.

Venho utilizando o [MockServer](https://www.mock-server.com/), ele é bem customizável, possui inúmeras funcionalidades, é bem fácil de configurar, possui uma [UI](https://www.mock-server.com/mock_server/mockserver_ui.html) para debug bem bacana e a documentação é bem completa.

![MockServer](https://cdn.hashnode.com/res/hashnode/image/upload/v1626709516128/cHznYZYh-.png)

O `compose` dele está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/mock/docker-compose.yaml), e dentro da mesma pasta existe um arquivo de configurações, caso você queira mudar essas configurações, basta seguir [esta documentação](https://www.mock-server.com/mock_server/getting_started.html). Para rodar:

```sh
docker compose -f mock/docker-compose.yaml up -d
```

Após subir os containers, a UI estará disponível em: [http://0.0.0.0:1080/mockserver/dashboard](http://0.0.0.0:1080/mockserver/dashboard), e caso você não tenha alterado o arquivo de configurações, a rota de testes será:
[http://0.0.0.0:1080/hello](http://0.0.0.0:1080/hello)

---

## 8. AWS

Com certeza não poderia faltar o [AWS LocalStack](https://github.com/localstack/localstack). Então se você não conhece esse projeto e usa AWS, recomendo dar uma olhada.

Apesar de muito útil, pode dar algum trabalho configurar alguns recursos nele, mas após configurado corretamente, pode salvar muito tempo tentando resolver algo direto pela AWS, além de economizar na conta rsrs.

A ideia aqui é deixar um [SQS](https://aws.amazon.com/pt/sqs/) configurado e rodando. Para configurar outros recursos basta dar uma olhada na [documentação oficial](https://localstack.cloud/docs/).

O `compose` do SQS está [aqui](https://github.com/leocarmo/docker-local-stack/blob/main/databases/docker-compose-postgres.yaml). Para iniciar:

```sh
docker compose -f aws/docker-compose-sqs.yaml up
```

Se tudo der certo, você verá a seguinte mensagem no console de que o mock do SQS subiu com sucesso na porta 4566:

![SQS LocalStack](https://cdn.hashnode.com/res/hashnode/image/upload/v1626712220281/JAlS_k_VV.png)

> Repare na segunda linha a mensagem: Starting mock SQS service on http port 4566 ...

O LocalStack também sobre uma rota de heath check, que fica disponível em [https://0.0.0.0:4566/health](https://0.0.0.0:4566/health), lá você deveria ver o seguinte json:

```json
{
    "services": {
        "sqs": "running"
    },
    "features": {
        "persistence": "disabled",
        "initScripts": "initialized"
    }
}
```

Ótimo, com tudo isso funcionando, utilizaremos esse serviço. Primeiro, precisamos criar uma fila, pois só subimos o serviço. Para isso, utilizamos o [aws-cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).

Criando uma fila no SQS:

```sh
aws --endpoint-url=http://0.0.0.0:4566 sqs create-queue --queue-name my-queue
```

Para verificar se a fila foi criada com sucesso utilize o comando:

```sh
aws --endpoint-url=http://0.0.0.0:4566 sqs list-queues
```

Agora enviaremos uma mensagem para a fila que criamos:

```sh
aws --endpoint-url=http://0.0.0.0:4566 sqs send-message --queue-url http://0.0.0.0:4566/000000000000/my-queue --message-body 'Hello SQS World'
```

Para ler a mensagem que acabamos de criar:

```sh
aws --endpoint-url=http://0.0.0.0:4566 sqs receive-message --queue-url http://0.0.0.0:4566/000000000000/my-queue
```

Para acessar a lista completa de comandos do SQS via aws-cli acesse a [documentação oficial](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sqs/index.html#cli-aws-sqs).

---

> Dica: caso voce já tenha uma network criada para suas aplicações, basta [conectar](https://docs.docker.com/engine/reference/commandline/network_connect/) os containers criados aqui com a sua network usando este comando:

```sh
docker network connect my-network redis-single
```

> Importante: para as stacks em cluster, é preciso conectar cada container na sua network.

---

Basicamente com essa stack muitos dos problemas do dia a dia podem ser resolvidos. Espero que te ajude de alguma forma no seu fluxo de desenvolvimento local.

Alguma sugestão de alguma ferramenta que faltou aqui? Deixa nos comentários!
