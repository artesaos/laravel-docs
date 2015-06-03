# Contratos

- [Introdução](#introducao)
- [Por que contratos?](#por-que-contratos)
- [Referência de contratos](#referencia-de-contratos)
- [Como usar os contratos](#como-usar-os-contratos)

<a name="introducao"></a>
## Introdução

Os contratos do Laravel são uma lista de interfaces que define o core dos serviços fornecidos pelo framework. Por exemplo, o contrato `Queue` define os métodos necessários para uma fila de trabalhos, enquanto o contrato `Mailer` define os métodos necessários para o envio de e-mail.

Cada contrato tem uma implementação correspondente fornecida pelo framework. Por exemplo, o Laravel fornece uma implementação de `Queue` com uma variedade de drivers, e uma implementação de `Mailer` que usa o  [SwiftMailer](http://swiftmailer.org/).

Todos os contratos do Laravel estão no [seu próprio repositório Github](https://github.com/illuminate/contracts). Desta forma temos uma referência rápida a todos os contratos disponíveis, assim como um simples, pacote desacoplado que pode ser utilizado por outros desenvolvedores.

<a name="por-que-contratos"></a>
## Por que contratos?

Você deve ter várias questões a respeito do uso de contratos. Por que utilizar interfaces em tudo? Usar interfaces não é mais complicado?

Vamos listar as razões para usar interfaces dentro dos seguintes tópicos: desacoplamento e simplicidade.

### Desacoplamento

Primeiro, vamos revisar o código abaixo que está totalmente ligado a uma implentação de cache. Considere o seguinte:

	<?php namespace App\Orders;

	class Repository {

		/**
		 * The cache.
		 */
		protected $cache;

		/**
		 * Cria uma nova instância do repositório
		 *
		 * @param  \SomePackage\Cache\Memcached  $cache
		 * @return void
		 */
		public function __construct(\SomePackage\Cache\Memcached $cache)
		{
			$this->cache = $cache;
		}

		/**
		 * Retorna um pedido pelo ID.
		 *
		 * @param  int  $id
		 * @return Order
		 */
		public function find($id)
		{
			if ($this->cache->has($id))
			{
				//
			}
		}

	}

Nesta classe, o código está totalmente acoplado a determinada implementação de cache. Isto ocorre porque nós dependemos de uma classe concreta de um tipo de Cache. Se a API deste pacote mudar seu código você terá que modificar tudo também.

Da mesma forma, se nós quisermos trocar a nossa tecnologia de cache (Memcached) por outra tecnologia (Redis), nós novamente termos que modificar nosso repositório. Nosso repositório não deve ter tanto conhecimento sobre quem está provendo dados a ele ou como eles estão provendo isto.

**Ao invés disto, nós podemos melhorar nosso código dependendo de uma simples interface, independente da tecnologia empregada:**

	<?php namespace App\Orders;

	use Illuminate\Contracts\Cache\Repository as Cache;

	class Repository {

		/**
		 * Create a new repository instance.
		 *
		 * @param  Cache  $cache
		 * @return void
		 */
		public function __construct(Cache $cache)
		{
			$this->cache = $cache;
		}

	}

Agora nosso código não está amarrado a uma tecnologia específica, e nem mesmo ao Laravel. Como os contratos não tem implementação e dependências, você pode facilmente codificar uma versão alternativa de qualquer contrato, permitindo que você troque o seu cache sem ter que modificar seu código que o utiliza.

### Simplicidade

Quando todos os serviços do Laravel estão ordenadamente definidos dentro de interfaces simples, é muito fácil determinar a funcionalidade oferecida por determinado serviço. **O contrato serve como uma documentação sucinta as funcionalidades do framework.**

Além disso, quando você depende de interfaces simples, seu código é mais fácil de entender e manter. Ao invés de rastrear os métodos que estão disponíves dentro de classes enormes, e complicadas, você pode consultar uma simples, interface limpa.

<a name="referencia-de-contratos"></a>
## Referência de contratos

Esta é uma referência aos contratos do Laravel, assim como suas implementaçóes pelo Laravel:

Contrato  |  Facade Laravel 4.x
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="como-usar-os-contratos"></a>
## Como usar os contratos

Então, como você chega a uma implementação de um contrato? É bem simples. Muitos tipos de classes no laravel são resolvidas pelo [service container](/docs/5.0/container), incluindo controllers, event listeners, filtros, filas de trabalho, e até Closures de rota. Então, para chegar a uma implementação de um contrato, você pode apenas colocar a assinatura da interface no construtor de sua classe. Por exemplo, de uma olhada neste gerenciador de eventos:

	<?php namespace App\Handlers\Events;

	use App\User;
	use App\Events\NewUserRegistered;
	use Illuminate\Contracts\Redis\Database;

	class CacheUserInformation {

		/**
		 * Implementação do banco de dados Redis.
		 */
		protected $redis;

		/**
		 * Cria uma nova instância do gerenciador de eventos.
		 *
		 * @param  Database  $redis
		 * @return void
		 */
		public function __construct(Database $redis)
		{
			$this->redis = $redis;
		}

		/**
		 * Gerencia um evento.
		 *
		 * @param  NewUserRegistered  $event
		 * @return void
		 */
		public function handle(NewUserRegistered $event)
		{
			//
		}

	}

Quando o event listener é resolvido, o service container lê as assinaturas no construtor da classe, e injeta o valor apropriado. Para saber mais sobre como registrar coisas no service container, de uma olhada [na documentação](/docs/5.0/container).
