# Service Container

- [Introdução](#introducao)
- [Como utilizar](#como-utilizar)
- [Mapeando interfaces para implementações](#mapeando-interfaces-para-implementacoes)
- [Mapeamento contextual](#mapeamento-contextual)
- [Tagging](#tagging)
- [Aplicações práticas](#aplicacoes-praticas)
- [Container de eventos](#container-de-eventos)

<a name="introducao"></a>
## Introdução

O service container do Laravel é uma poderosa ferramenta para gerenciar dependências de classes. Injeção de dependência é o nome chique dado para o significado essencial do padrão de projetos: classes colocadas como dependências são "injetadas" dentro de sua classe através de seu construtor ou, em alguns casos, metodos "setter".

Vamos olhar o seguinte exemplo:

	<?php namespace App\Handlers\Commands;

	use App\User;
	use App\Commands\PurchasePodcast;
	use Illuminate\Contracts\Mail\Mailer;

	class PurchasePodcastHandler {

		/**
		 * A implementação de mailer.
		 */
		protected $mailer;

		/**
		 * Cria uma nova instância.
		 *
		 * @param  Mailer  $mailer
		 * @return void
		 */
		public function __construct(Mailer $mailer)
		{
			$this->mailer = $mailer;
		}

		/**
		 * Comprar um podcast.
		 *
		 * @param  PurchasePodcastCommand  $command
		 * @return void
		 */
		public function handle(PurchasePodcastCommand $command)
		{
			//
		}

	}

Neste exemplo, a classe que trata a compra de podcasts (`PurchasePodcast`) precisa enviar e-mails quando um podcast é comprado. Então, nós iremos **injetar** um serviço que é capaz de enviar e-mails. Como o serviço é injetado, nós somos capazes de trocar entre as suas possíveis implementações. Podemos até injetar um "mock", ou criar uma implemtação fake quando estivermos testando nossa aplicação.

Conhecer profundamente o service container do Laravel é essencial para construir aplicações grandes e poderosas, assim como para poder contribuir com o código-fonte do Laravel.

<a name="como-utilizar"></a>
## Como utilizar

### Binding

Quase todos os seus mapeamentos para o service container serão registrados no [service providers](/docs/5.0/providers), então todos os exemplos irão seguir este contexto. Porém, se você precisa de uma instância do container em qualquer outro ponto de sua aplicação, como em um factory, você deve adicionar a assinatura do contrato `Illuminate\Contracts\Container\Container` e uma instância será injetada para você. Alternativamente, você pode usar o facade `App` para acessar o container.

#### Registrando um "resolvedor" base

Dentro do service provider, você sempre terá acesso a class App através da variável de instância `$this->app`.

Existem vários métodos para poder registrar dependências, incluindo Closure callbacks e mapeando interfaces para implementações. Primeiro, vamos explorar Closure callbacks. Um resolvedor Closure é registrado no container com uma chave (tipicamente o nome da classe) e uma Closure que retorna algum valor:

	$this->app->bind('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### Registrando um singleton

As vezes, você quer mapear algo no container que deve ser resolvido apenas uma vez, e a mesma instância deve ser retornada nas chamadas subsequentes ao container:

	$this->app->singleton('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### Mapeando uma instância existente dentro do container

Você também pode mapear um objeto existente ao container usando o método `instance`. Esta instância será sempre chamadas nas chamadas subsequentes ao container:

	$fooBar = new FooBar(new SomethingElse);

	$this->app->instance('FooBar', $fooBar);

### Resolvendo

Existem várias formas de resolver algo a partir do container. Primeiro, você pode usar o método `make`:

	$fooBar = $this->app->make('FooBar');

Segundo, você pode usar "acesso de array" ao container, visto que ele implementa a interface `ArrayAccess` do PHP:

	$fooBar = $this->app['FooBar'];

Por último, mas não menos importante, você pode "assinar" a dependência no construtor de uma classe que será resolvida automaticamente pelo container. Esta forma pode ser utilizada em controllers, event listeners, filas de trabalho, filtros, e mais:

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Users\Repository as UserRepository;

	class UserController extends Controller {

		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

		/**
		 * Show the user with the given ID.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($id)
		{
			//
		}

	}

<a name="mapeando-interfaces-para-implementacoes"></a>
## Mapeando interfaces para implementações

### Injetando dependências concretas

Uma das mais poderosas funcionalidades do service container é sua habilidade de mapear interfaces para implementações. Por exemplo, talvez sua aplicação tenha uma integração com o web service do [Pusher](https://pusher.com), que envia e recebe eventos em tempo real. Se você está utilizando a SDK PHP do Pusher, podemos injetar uma dependência do cliente do Pusher na classe:

	<?php namespace App\Handlers\Commands;

	use App\Commands\CreateOrder;
	use Pusher\Client as PusherClient;

	class CreateOrderHandler {

		/**
		 * The Pusher SDK client instance.
		 */
		protected $pusher;

		/**
		 * Cria uma nova instância do gerenciador de pedidos
		 *
		 * @param  PusherClient  $pusher
		 * @return void
		 */
		public function __construct(PusherClient $pusher)
		{
			$this->pusher = $pusher;
		}

		/**
		 * Executa um dado comando
		 *
		 * @param  CreateOrder  $command
		 * @return void
		 */
		public function execute(CreateOrder $command)
		{
			//
		}

	}

Neste exemplo, o bom é que estamos injetando a dependência na classe; porém, nós estamos totalmente amarrados a SDK do Pusher. Se os métodos da SDK do Pusher SDK mudarem ou nós decidirmos por mudar para um  serviço totalmente novo, nós teremos que mudar o código da nossa classe `CreateOrderHandler`.

### Programando para uma interface

Para "isolar" a nossa classe `CreateOrderHandler` contra mudanças, nós podemos definir uma interface `EventPusher` e uma implementação `PusherEventPusher`:

	<?php namespace App\Contracts;

	interface EventPusher {

		/**
		 * Envia um novo evento a todos os clientes
		 *
		 * @param  string  $event
		 * @param  array  $data
		 * @return void
		 */
		public function push($event, array $data);

	}

Uma vez que nós codificamos a implementação `PusherEventPusher` da interface, nós podemos registrá-la no  service container da seguinte forma:

	$this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

Isto diz ao container que ele deve injetar um `PusherEventPusher` quando uma classe precisa de uma implementação de `EventPusher`. Agoramos podemos adicionar a assinatura da interface `EventPusher` em nosso construtor:

		/**
		 * Cria uma nova instância do gerenciador de pedidos
		 *
		 * @param  EventPusher  $pusher
		 * @return void
		 */
		public function __construct(EventPusher $pusher)
		{
			$this->pusher = $pusher;
		}

<a name="mapeamento-contextual"></a>
## Mapeamento contextual

As vezes você tem duas classes que utilizam a mesma interface, mas você deseja injetar diferentes implementações em cada classe. Por exemplo, quando o seu sistema recebe um novo Pedido, nós queremos enviar um evento através do [PubNub](http://www.pubnub.com/) ao invés do Pusher. O Laravel fornece uma forma simples e fluente para definir este comportamento:

	$this->app->when('App\Handlers\Commands\CreateOrderHandler')
	          ->needs('App\Contracts\EventPusher')
	          ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## Tagging

As vezes, você pode precisar resolver todos os mapeamentos para uma "categoria". Por exemplo, talvez você esteja criando um agregador de relatórios que recebe um array com os diferentes tipos de interfaces de `Report`. Depois de registradas as implementações de `Report`, você pode adicionar uma tag a elas usando o método `tag`:

	$this->app->bind('SpeedReport', function()
	{
		//
	});

	$this->app->bind('MemoryReport', function()
	{
		//
	});

	$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Uma vez que os serviços estão "tagueados", você pode facilmente resolvê-los através do método `tagged`:

	$this->app->bind('ReportAggregator', function($app)
	{
		return new ReportAggregator($app->tagged('reports'));
	});

<a name="aplicacoes-praticas"></a>
## Aplicações práticas

O Laravel fornece inúmeras oportunidade para você usar o service container para aumentar a flexibilidade e a "testabilidade" de sua aplicacão. Um primeiro exemplo é resolvendo controllers. Todos os controllers são resolvidos pelo service container, o que significa que você pode colocar dependências na assinatura do construtor de seus controllers, e elas serão automaticamente resolvidas.

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\OrderRepository;

	class OrdersController extends Controller {

		/**
		 * Uma instância do repositório de pedidos.
		 */
		protected $orders;

		/**
		 * Cria uma nova instância do controller
		 *
		 * @param  OrderRepository  $orders
		 * @return void
		 */
		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		/**
		 * Exibe todos os pedidos
		 *
		 * @return Response
		 */
		public function index()
		{
			$orders = $this->orders->all();

			return view('orders', ['orders' => $orders]);
		}

	}

Neste exemplo, a classe `OrderRepository` é automaticamente injetada no controller. Isto significa que um "mock" de `OrderRepository` pode ser enviada pelo container quando você estiver rodando [testes unitários](/docs/5.0/testing), permitindo que não seja necessário simular a camada de banco de dados durante os testes.

#### Outros exemplos de uso do container

É claro, como mencionado acima, controllers não são as únicas classes que o Laravel resolve através do service container. Você também pode adicionar assinaturas das dependências em Closures de rota, filtros, filas de trabalho, event listeners, e mais. Para exemplos de uso do service container nestes contextos, favor ver as documentações deles.

<a name="container-de-eventos"></a>
## Container de eventos

#### Registrando um Resolving Listener

O container executa um evento cada vez que ele resolve um objeto. Você pode "ouvir" para estes eventos usando o método `resolving`:

	$this->app->resolving(function($object, $app)
	{
		// Chamado quando o container resolve um objeto de qualquer tipo
	});

	$this->app->resolving(function(FooBar $fooBar, $app)
	{
		// Chamado quando o container resolve um objeto do tipo "FooBar"...
	});

O objeto que está sendo resolvido é passado como callback.
