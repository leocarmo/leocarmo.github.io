+++
title = "Hyperf - PHP Coroutine Framework baseado em Swoole"
author = "leocarmo"
date = "2021-07-15T19:30:00-03:00"
tags = ["php", "swoole", "hyperf"]
description = "O Hyperf é um framework PHP de alto desempenho e altamente flexível baseado em Swoole 4.5+ com PHP CLI. Ele possui um servidor de corrotina integrado."
cover = "/hyperf-php-coroutine-framework-baseado-em-swoole/images/programming.jpeg"
+++

Durante muito tempo que estudei sobre [Swoole](https://www.swoole.co.uk/), eu sempre fazia as coisas "do zero" para tentar entender a fundo sobre ele. Cheguei a pesquisar alguns frameworks baseados nele, mas não encontrei nenhum que tenha me chamado a atenção. Então acabei seguindo meus estudos por conta mesmo, até agora.

Quando conheci o [Hyperf](https://github.com/hyperf/hyperf) não imaginei o quanto ele era incrível até começar a descobrir tudo que ele poderia me entregar. Praticamente todos os desafios que eu estava tendo para implementar do zero com Swoole, ele resolveu.

Pode até parecer ótimo ter tudo pronto, mas do que adianta ter um carro de fórmula 1 se você não souber dirigir? Então não é tão simples assim. É preciso ter uma base de Swoole, programação concorrente, aplicação stateful, etc., para conseguir extrair todo o poder que ele pode te dar.

A ideia aqui é te ajudar com os primeiros passos e ter um projeto mínimo rodando, pois, só de utilizar o Hyperf como framework, você já conseguirá sentir uma diferença absurda em performance.

A documentação é bacana, aborda bastante coisa e tem vários exemplos, então vou passar por alguns tópicos e ir referenciando tudo aqui.  Alguns conteúdos da documentação ainda estão em chinês, mas estão sendo traduzidos, enquanto isso, é só traduzir que da tudo certo!

Minha missão aqui é:

> **Te incentivar a construir as suas próximas aplicações utilizando Hyperf e Swoole.**

Então, bora lá!

Tópicos:
1. [O que é?](#1-o-que-e)
2. [Instalação](#2-instalacao)
3. [Hot Reload](#3-hot-reload)
4. [Estrutura](#4-estrutura)
5. [Melhorando o ambiente](#5-melhorando-o-ambiente)
6. [Database](#6-database)
7. [Rotas e Controllers](#7-rotas-e-controllers)
8. [Parallel](#8-parallel)
9. [Benchmark](#9-benchmark)

---

## 1. O que é?

O Hyperf é um framework de alto desempenho e altamente flexível baseado em Swoole 4.5+ com `PHP CLI`. Ele possui um servidor de corrotina integrado com diversos componentes comumente usados.

Durante cerca de meio ano, o Hyperf foi testado em diversas companhias pelo mundo, após ótimos resultados, ele foi liberado para a comunidade em 20/06/2019.

> Os dois trechos acima foram retirados e traduzidos da [documentação oficial](https://hyperf.wiki/2.1/#/en/README)

Caso você não conheça o Swoole PHP, recomendo dar uma olhada antes, pois vou assumir que você já conhece a forma com que ele funciona, já que o Hyperf é baseado nele.

> Algumas recomendações de leitura para que você conheça e aprenda mais sobre Swoole:
- [O que é Swoole](https://www.swoole.co.uk/docs/)
- [Como Swoole funciona](https://www.swoole.co.uk/how-it-works)
- [Websocket em PHP com Swoole](https://www.linkedin.com/pulse/websocket-em-php-sim-%C3%A9-poss%C3%ADvel-ronie-neubauer/)
- [Livro Mastering Swoole PHP](https://www.amazon.com.br/Mastering-Swoole-PHP-performance-concurrent-ebook/dp/B0881B227S)
- [Série de resumo do Livro](https://diegoborgs.com.br/blog/mastering-swoole-php-parte-i-introdu%C3%A7%C3%A3o)

Uma coisa bem interessante, é que o Hyperf é muito similar ao [Laravel](https://laravel.com/), então se você já trabalhou com Laravel, vai se adaptar muito bem ao Hyperf.

Por ser baseado em Swoole, ele roda em cima do `PHP CLI`, sendo stateful, e é extremamente performático comparado ao "PHP tradicional", utilizado via `php-fpm`, e também comparado a outras linguagens. :eyes:

A proposta do Hyperf foi trazer todas as features do Swoole de forma simples para a comunidade, fazendo com que não fosse preciso implementar tudo do zero. Grandes exemplos são as pools de conexões, servidor HTTP, TCP, gRPC, [entre outros](https://hyperf.wiki/2.1/#/en/README).

Para exemplificar como funciona, ao utilizar o client pronto de Redis do Hyperf, você já tem uma pool criada por baixo com o client de Redis do Swoole, o que já ajuda muito na hora de fazer um projeto. A mesma coisa para o Eloquent ORM, ao utiliza-lo no Hyperf, por baixo já está tudo resolvido.

> Ou seja, não tem motivos para não achar isso incrível <3

Para entender o [Lifecycle do Hyperf](https://hyperf.wiki/2.1/#/en/lifecycle), é muito importante entender o [Lifecycle do Swoole](https://www.swoole.co.uk/docs/modules/swoole-http-server-doc). Como o Swoole é executado via `PHP CLI`, o Hyperf se inicia também via `PHP CLI`, utilizando o [symfony/console](https://github.com/symfony/console) como `command`.

---

## 2. Instalação

A maneira mais fácil de iniciar é utilizando Docker, como em tudo rs. Com isso, você não precisa se preocupar com a versão do PHP, da extensão do Swoole na sua máquina, etc.

Primeiro, crie uma pasta para iniciar o projeto:

```sh
mkdir hyperf-skeleton && cd hyperf-skeleton
```

Inicie o container mapeando o diretório que criamos como um volume para receber o projeto:

```sh
docker run -v ${PWD}:/app -p 9501:9501 -it --entrypoint /bin/sh hyperf/hyperf:7.4-alpine-v3.11-swoole
```

Já dentro do container, instale o Hyperf na pasta `app` que mapeamos anteriormente. O framework é bastante modular, então ele vai perguntar se você quer utilizar algumas coisas, inicialmente, escolha todas as opções padrão que já vai nos atender.

```sh
composer create-project hyperf/hyperf-skeleton app
```

Caso você queira iniciar esse projeto novamente de maneira mais fácil e já cair na raiz do projeto, crie o arquivo do `docker-compose.yaml` com o seguinte conteúdo:

```docker
version: '3.9'

services:

  hyperf-skeleton:
    container_name: hyperf-skeleton
    image: hyperf/hyperf:7.4-alpine-v3.11-swoole
    working_dir: /app
    entrypoint: sh
    volumes:
      - ./:/app
    ports:
      - 9501:9501
```

Sobre o `entrypoint` ser direto o `sh` e não o start do projeto, veremos mais tarde o motivo disso. Depois disso, para iniciar o container basta usar o seguinte comando:

```sh
docker compose run --rm --service-ports hyperf-skeleton
```

Agora é só iniciar o webserver:

```sh
php bin/hyperf.php start
```

Sua aplicação está pronta para responder em: [http://0.0.0.0:9501](http://0.0.0.0:9501)

```json
{
    "method": "GET",
    "message": "Hello Hyperf."
}
```

> Mais detalhes na [documentação oficial](https://hyperf.wiki/2.1/#/en/quick-start/install)

---

## 3. Hot Reload

Como já vimos que o Swoole é executado via `PHP CLI`, em ambiente de desenvolvimento isso pode ser ruim, pois podemos perder muito tempo parando e iniciando o servidor. Bom, felizmente o Hyperf resolveu isso de forma fácil via o componente [Watcher](https://hyperf.wiki/2.1/#/en/watcher).

> Importante: não utilize esse componente em ambiente produtivo, ele é destinado ao ambiente de desenvolvimento!

Primeiro precisamos instalar esse modulo no nosso projeto:

```sh
composer require hyperf/watcher --dev
```

Publicar o arquivo de configurações:

```sh
php bin/hyperf.php vendor:publish hyperf/watcher
```

Um arquivo foi criado em: `config/autoload/watcher.php`, nele você pode definir as configurações que achar melhor, aqui vamos seguir com as que já estão lá mesmo.

Agora, para rodar nosso projeto em **ambiente de desenvolvimento**, nosso comando passa a ser:

```sh
php bin/hyperf.php server:watch
```

Assim, sempre que fizermos alguma alteração no projeto, esse componente vai recarregar nosso código automaticamente sem ser preciso reiniciar manualmente.

![magic.gif](images/magic.gif)

---

## 4. Estrutura

A estrutura é bem simples, como comentei é bem similar ao Laravel, então não tem muita complexidade. As principais pastas são:

- Arquivo de start: `bin/hyperf.php`
- Centro de configurações: `config/*`
- Rotas: `config/routes.php`
- Server: `config/autoload/server.php`
- Injeção de dependências: `config/autoload/dependencies.php`
- Design do projeto: `config/autoload/devtool.php`
- Aplicação: `app/*`

> Para conhecer mais sobre o centro de configurações, acesse [este link](https://hyperf.wiki/2.1/#/en/config-center).

Eu queria destacar um dos itens acima que achei super interessante, o arquivo `devtool.php`. Com ele é possível montar todo o design da sua arquitetura da forma que você achar melhor. Ou seja, em um único arquivo você consegue definir sua estrutura de pastas para utilizar [DDD](https://pt.stackoverflow.com/questions/19548/o-que-realmente-%C3%A9-ddd-e-quando-ele-se-aplica) com [clean architecture](https://medium.com/luizalabs/descomplicando-a-clean-architecture-cf4dfc4a1ac6), [hexagonal architecture](https://netflixtechblog.com/ready-for-changes-with-hexagonal-architecture-b315ec967749), ou qualquer outra que desejar, te dando muito poder de trabalhar da forma que preferir.


--- 

## 5. Melhorando o ambiente

Na próxima seção, vamos iniciar com uma ideia básica de CRUD. Mas antes, vamos fazer uma melhoria no nosso `docker-compose-yaml` para ajudar no nosso desenvolvimento, adicionando o nosso banco de dados e o Redis (que vamos utilizar mais adiante).

Atualize o arquivo `docker-compose-yaml` para:

```docker
version: '3.9'

services:

  hyperf-skeleton:
    container_name: hyperf-skeleton
    image: hyperf/hyperf:7.4-alpine-v3.11-swoole
    working_dir: /app
    entrypoint: ["php", "bin/hyperf.php", "server:watch"]
    volumes:
      - ./:/app
    ports:
      - 9501:9501
    depends_on:
      - hyperf-skeleton-mariadb
      - hyperf-skeleton-redis

  hyperf-skeleton-mariadb:
    container_name: hyperf-skeleton-mariadb
    image: mariadb:latest
    volumes:
      - ./.docker/mariadb:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: "secret"
      MYSQL_DATABASE: "hyperf-skeleton"

  hyperf-skeleton-redis:
    container_name: hyperf-skeleton-redis
    image: redis:latest
    ports:
      - 6379:6379
```

Agora para iniciar nosso projeto completo basta rodar o seguinte comando:

```sh
docker-compose up -d
```

E claro, para remover tudo:

```sh
docker-compose down
```

---

## 6. Database

> Caso queira entender um pouco mais sobre o que vamos utilizar agora no Hyperf, acesse a documentação oficial e veja sobre: [Database](https://hyperf.wiki/2.1/#/en/db/quick-start), [Model](https://hyperf.wiki/2.1/#/en/db/model) e [Migration](https://hyperf.wiki/2.1/#/en/db/migration).

> Dica: evite copiar e colar o código, tente escrever tudo, assim seu aprendizado será muito maior (:

Com nosso banco de dados rodando, vamos criar uma migration para a tabela de usuários:

```sh
docker container exec -it hyperf-skeleton php bin/hyperf.php gen:migration create_users_table
```

Com o nosso arquivo gerado em `migrations/*_create_users_table.php`, vamos inserir os campos básicos para serem criados:

```php
Schema::create('users', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```

> Sim, vamos usar `uuid` como chave primaria, espero que em todos os seus projetos você também esteja usando :eyes:, se ainda não estiver, da uma olhada [aqui](https://mareks-082.medium.com/auto-increment-keys-vs-uuid-a74d81f7476a).

O próximo passo é criar o nosso modelo, então na pasta `App/Model` vamos criar o nosso `User`:

```php
<?php declare(strict_types=1);

namespace App\Model;

use Hyperf\DbConnection\Model\Model;

/**
 * @property $id
 * @property $name
 * @property $email
 * @property $created_at
 * @property $updated_at
 */
class User extends Model
{
    /**
     * @var string
     */
    public $keyType = 'string';

    /**
     * @var bool
     */
    public $incrementing = false;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['id', 'name', 'email', 'created_at', 'updated_at'];
}
```

Antes de rodar a nossa migration, normalmente é preciso verificar se os parâmetros de conexão do banco estão devidamente configurados. Porém, no nosso caso vamos manter tudo como está, mas vale a pena dar uma olhada no conteúdo do arquivo, ele fica em `config/autoload/databases.php`.

O que vamos precisar fazer é trocar as `envs` no arquivo `.env` para realizar a conexão ao nosso container. Só vamos precisar alterar as seguintes `envs`:

```
DB_HOST=hyperf-skeleton-mariadb
DB_DATABASE=hyperf-skeleton
DB_USERNAME=root
DB_PASSWORD=secret
```

Agora é só rodar a migration no container da aplicação:

```sh
docker container exec -it hyperf-skeleton php bin/hyperf.php migrate
```

Resultado de sucesso:

```
[INFO] Migration table created successfully.
Migrating: *_create_users_table
Migrated:  *_create_users_table
```

> Importante, o hot reload não funciona para recarregar variaveis de ambiente, então é preciso dar um restart manual no container.

---

## 7. Rotas e Controllers

> Caso queira entender um pouco mais sobre o que vamos utilizar agora no Hyperf, acesse a documentação oficial e veja sobre: [Controller](https://hyperf.wiki/2.1/#/en/controller) e [Router](https://hyperf.wiki/2.1/#/en/router).

Agora vamos criar o nosso `UserController`, na pasta `app/Controller` inicialmente vazio:

```php
<?php declare(strict_types=1);

namespace App\Controller;

class UserController extends AbstractController
{

}
```

Depois, precisamos definir nossas rotas. Então faremos algo assim:

```php
use App\Controller\UserController;

Router::addGroup('/users', function () {
    Router::get('', [UserController::class, 'index']);
    Router::get('/{id}', [UserController::class, 'show']);
    Router::post('', [UserController::class, 'store']);
    Router::delete('/{id}', [UserController::class, 'delete']);
});
```

Voltando para o nosso `Controller`, vamos criar os métodos conforme nossas rotas já utilizando a classe `User` e fazendo as devidas operações com o ORM:

```php
<?php declare(strict_types=1);

namespace App\Controller;

use App\Model\User;
use Hyperf\HttpServer\Contract\RequestInterface;

class UserController extends AbstractController
{
    public function index(RequestInterface $request)
    {
        return User::get();
    }

    public function show(string $id)
    {
        return User::find($id);
    }

    public function store(RequestInterface $request)
    {
        return User::create($request->all());
    }

    public function delete(string $id)
    {
        return User::destroy($id);
    }
}
```

Essa é uma estrutura extremamente básica, e com alguns problemas... Contudo, a ideia não é ensinar tudo aqui, e sim te direcionar para explorar e construir coisas interessantes. Para isso, vou deixar alguns links aqui para você melhorar este código:

- [Paginator](https://hyperf.wiki/2.1/#/en/db/paginator)
- [Validation](https://hyperf.wiki/2.1/#/en/validation)
- [Cache](https://hyperf.wiki/2.1/#/en/cache)
- [Response](https://hyperf.wiki/2.1/#/en/response)
- [Event Dispatcher](https://hyperf.wiki/2.1/#/en/event)

Todos os itens acima tem uma boa documentação e com bons exemplos. Você consegue implementar tudo isso nesse simples CRUD para exercitar os recursos do Hyperf.

Ainda tenho uma ideia bônus: crie uma interface e uma implementação para a Repository Pattern refatorando este código e injete via [Dependency Injection](https://hyperf.wiki/2.1/#/en/di), tenho certeza que será bacana entender este fluxo.

E claro, não esqueça de criar testes. O Hyperf criou o `co-phpunit`, que é basicamente o `phpunit` com corrotinas. Maneiro de mais né? Para conhecer mais acesse a [documentação](https://hyperf.wiki/2.1/#/en/testing).

---

## 8. Parallel

Outra coisa que achei muito bacana, foi a implementação que o Hyperf fez do gerenciamento via channels de corrotinas usando wait groups e handle de exceptions. Basicamente ele encapsulou toda essa implementação na classe `Hyperf\Utils\Parallel`.

Usando o exemplo do que fizemos acima, para criar vários usuários em paralelo e só continuar depois que todos finalizarem, podemos fazer assim no nosso `UserController@store`:

```php
public function store(RequestInterface $request)
{
    $parallel = new Parallel();

    foreach ($request->input('users') as $user) {
        $parallel->add(
            function () use ($user) {
                User::create($user);
            },
            $user['id']
        );
    }

    return $parallel->wait();
}
```

> Claro que não fariamos esta implementação no mundo real, é apenas um exemplo rsrs.

É importante destacar que ao disparar uma corrotina, tudo que for utilizado de I/O dentro do seu contexto precisa ser compatível com corrotinas, caso contrário seu código será bloqueante.

Todos os componentes do Hyperf já são compatíveis, então você pode utilizar sem problemas, tenha atenção apenas com bibliotecas como a do `mongodb`, que, até o momento não tem suporte dos [Hooks do Swoole](https://www.swoole.co.uk/docs/modules/swoole-runtime-flags).

---

## 9. Benchmark

Falamos muito sobre performance, diferenças entre o Swoole e o "PHP Tradicional", mas agora vamos aos números!

Realizei um teste na minha própria máquina, então os resultados podem ser diferentes de um servidor em produção ou da sua máquina. Não vamos focar nos números absolutos, vamos focar nas referências entre um e outro, pois mesmo mudando de ambiente, provavelmente serão mantidas similares.

> Para realizar os testes utilizei o [wrk](https://github.com/wg/wrk) na rota de retorno base dos frameworks.

**Laravel 8 com PHP 7.4 (93 rq/s)**

```
Running 10s test @ http://0.0.0.0:8000
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   550.01ms  335.24ms   1.12s    63.01%
    Req/Sec    12.38      8.80    70.00     84.72%
  938 requests in 10.10s, 16.64MB read
  Socket errors: connect 155, read 1214, write 12, timeout 0
Requests/sec:     92.83
Transfer/sec:      1.65MB
```

**Lumen 8 com PHP 7.4 (279 rq/s)**

```
Running 10s test @ http://0.0.0.0:8000
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   108.93ms   41.13ms 193.70ms   76.31%
    Req/Sec    73.96     40.36   184.00     69.49%
  2820 requests in 10.10s, 751.82KB read
  Socket errors: connect 157, read 3987, write 4, timeout 0
Requests/sec:    279.10
Transfer/sec:     74.41KB
```

**Laravel Octane com PHP 8.0 e Swoole (1.273 rq/s)**
```
Running 10s test @ http://0.0.0.0:8000
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   110.06ms  190.13ms   1.65s    84.78%
    Req/Sec   118.57     96.64   550.00     69.97%
  12796 requests in 10.05s, 13.09MB read
  Socket errors: connect 157, read 101, write 4, timeout 0
Requests/sec:   1273.75
Transfer/sec:      1.30MB
```

**Hyperf 2.1 com PHP 7.4 (90.799 rq/s)**

```
Running 10s test @ http://0.0.0.0:9502
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   628.86us    2.28ms 127.72ms   97.39%
    Req/Sec    20.96k    12.14k   62.60k    69.07%
  914415 requests in 10.07s, 165.69MB read
  Socket errors: connect 155, read 84, write 0, timeout 2
Requests/sec:  90799.66
Transfer/sec:     16.45MB
```

**Swoole puro com PHP 8.0 (159.581 rq/s)**

```
Running 10s test @ http://0.0.0.0:9501
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.47ms  431.57us  29.87ms   97.10%
    Req/Sec    13.37k     5.49k   25.36k    59.74%
  1612224 requests in 10.10s, 267.53MB read
  Socket errors: connect 157, read 107, write 0, timeout 0
Requests/sec: 159581.95
Transfer/sec:     26.48MB
```

Comparações usando o mais lento como referência:

| Framework | rq/s | ref. |
|---|---|---|
| Laravel | 92 | x1 |
| Lumen | 279 | +3x |
| Laravel Octane | 1.273 | +13x |
| Hyperf | 90.799 | +986x |
| Swoole puro | 159.581 | +1.734x |

Ou seja, o Hyperf é mais de 986 vezes mais rápido do que o Laravel e o Swoole puro 1.734, isso sem considerar a latência, que também teve uma diferença enorme. Incrível né? Claro, este é um benchmark realizado localmente em uma rota extremamente simples, mas a diferença realmente é insana, mesmo em outros ambientes.

---

O resultado deste projeto está no meu github:

[https://github.com/leocarmo/hyperf-skeleton](https://github.com/leocarmo/hyperf-skeleton)

Bom, acredito que passamos pelo básico do Hyperf, agora você já consegue explorar tudo que ele pode fornecer com este ambiente que iniciamos.

Pretendo escrever novos artigos aprofundando mais em assuntos específicos, principalmente sobre os diversos clients que ele implementa e sobre as [connections pools](https://hyperf.wiki/2.1/#/en/pool).

Espero que você tenha gostado e que esse guia possa ter te ajudado. Apesar de algumas partes da documentação ainda não ter sido traduzida, ela é bem completa de exemplos, então da para implementar praticamente tudo que está disponível.

E aí, já conhecia o Hyperf? Me conta o que achou dele nos comentários. (:

--- 

> Agradecimento especial ao [@ronieneubauer](https://www.linkedin.com/in/ronieneubauer/) pela revisão.