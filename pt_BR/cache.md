# Cache

- [Configuração](#configuracao)
- [Usando cache](#usando-cache)
- [Incrementando & Decrementando](#incrementando-e-decrementando)
- [Tags em cache](#tags-em-cache)
- [Eventos de cache](#eventos-de-cache)
- [Cache em banco de dados](#cache-em-banco-de-dados)

<a name="configuracao"></a>
## Configuração

O Laravel fornece uma API unificada para vários sistemas de cache. As configurações de cache estão localizadas em `config/cache.php`. Neste arquivo você pode especificar qual driver de cache você gostaria de usar por padrão em sua aplicação. O Laravel suporta mecanismos de cache populares como [Memcached](http://memcached.org) e [Redis](http://redis.io).

O arquivo de configuração de cache também contém várias outras opções, que estão documentadas no próprio arquivo, então tenha certeza de que leu estas opções. Por padrão, o Laravel é configurado para usar o driver de cache `file` (arquivo), que salva objetos serializados em arquivos físicos. Para aplicações maiores, é recomendado usar cache em memória utilizando drivers Memcached ou APC por exemplo. Você também pode configurar multiplas configurações de cache para o mesmo driver.

Antes de usar o driver Redis no Laravel, você precisará instalar o pacote `predis/predis` (~1.0) via Composer.

<a name="usando-cache"></a>
## Usando cache

#### Gravando um item no cache

	Cache::put('chave', 'valor', $minutos);

#### Usando objetos Carbon para dizer em que momento o cache expira

	$dataDeExpiracao = Carbon::now()->addMinutes(10);

	Cache::put('chave', 'valor', $dataDeExpiracao);

#### Gravando um item no cache se ele não existir

	Cache::add('chave', 'valor', $minutos);

O método `add` irá retornar `true` se o item for **realmente adicionado** ao cache. Caso contrário, o método irá retornar `false`.

#### Verificando a existência de uma chave no cache

	if (Cache::has('chave'))
	{
		//
	}

#### Obtendo um item do cache

	$valor = Cache::get('chave');

#### Retornando um item ou um valor padrão

	$valor = Cache::get('chave', 'padrão');

	$valor = Cache::get('chave', function() { return 'padrão'; });

#### Gravando um item no cache permanentemente

	Cache::forever('chave', 'valor');

As vezes você precisará obter um item do cache, mas também retornar um valor padrão se a chave não existir. você pode fazer isto usando o método `Cache::remember`:

	$valor = Cache::remember('usuarios', $minutos, function()
	{
		return DB::table('usuarios')->get();
	});

Você também pode combinar os métodos `remember` e `forever`:

	$valor = Cache::rememberForever('usuarios', function()
	{
		return DB::table('usuarios')->get();
	});

Note que todos os itens salvos no cache são serializados, então você é livre para gravar qualquer formato de dados.

#### Puxando um item do cache

Se você precisa obter um item do cache e então deletá-lo, você deve usar o método `pull`:

	$valor = Cache::pull('chave');

#### Removendo um item do cache

	Cache::forget('chave');

#### Acessando um setor de cache específico

Quando estiver usando múltiplos mecanismos de cache, você pode acessá-los através do método `store`:

	$valor = Cache::store('memoria')->get('chave');

<a name="incrementando-e-decrementando"></a>
## Incrementando & Decrementando

Todos os drivers exceto `file` e `database` suportam as operações `increment` e `decrement`:

#### Incrementando um valor

	Cache::increment('chave');

	Cache::increment('chave', $quantidade);

#### Decrementando um valor

	Cache::decrement('chave');

	Cache::decrement('chave', $quantidade);

<a name="tags-em-cache"></a>
## Tags em cache

> **Note:** Tags em cache não são suportadas quando você está usando os mecanismos de cache `file` ou `database`. Por outro lado, quando você utiliza várias tags que são salvos "forever", a performance será a melhor com drivers como `memcached`, que automaticamente remove registros antigos.

#### Acessando um cache com tag

Tags em cache permitem que você relacione items no cache, e então possa apagar todos que contenham o nome dado a esta tag. Para acessar um cache com tag, utilize o método `tags`.

Você pode gravar um cache com tag passando uma lista ordenada de nomes como argumento, ou como um array ordenado:

	Cache::tags('pessoas', 'autores')->put('João', $joao, $minutos);

	Cache::tags(['pessoas', 'artistas'])->put('Ana', $ana, $minutos);

Você pode usar qualquer outro método combinado com o método tags, incluindo os métodos `remember`, `forever`, e `rememberForever`. Você também pode acessar os itens que estão no cache em uma determinada tag, assim como outros métodos como `increment` e `decrement`.

#### Acessando itens do cache que foram "tageados"

Para acessar um item no cache que foi "tageado", passe a mesma ordem da lista de tags usadas para salvá-lo.

	$ana = Cache::tags('pessoas', 'artistas')->get('Ana');

	$joao = Cache::tags(['pessoas', 'autores'])->get('João');

Você também pode apagar todos os itens que estão em uma determinada tag com um nome ou uma lista de nomes. Por exemplo, o código abaixo irá remover todos os caches com as tags `pessoas`, `autores`, ou ambos. Então, ambos "Ana" e "João" seriam removidos do cache:

	Cache::tags('pessoas', 'autores')->flush();

Por outro lado, o seguinte código removeria apenas os caches tageados com nome `autores`, então "João" seria removido, mas "Ana" não.

	Cache::tags('autores')->flush();

<a name="eventos-de-cache"></a>
## Eventos de cache

Para executar algum código para cada operação de cache, você deve "ouvir" pelos eventos disparados pelo cache:

	Event::listen('cache.hit', function($key, $value) {
		//
	});

	Event::listen('cache.missed', function($key) {
		//
	});

	Event::listen('cache.write', function($key, $value, $minutes) {
		//
	});

	Event::listen('cache.delete', function($key) {
		//
	});

<a name="cache-em-banco-de-dados"></a>
## Cache em banco de dados

Quando usado o mecanismo de cache `database`, você precisará configurar uma tabela que irá conter os itens do cache. Você encontrará um exemplo do `Schema` da tabela abaixo:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
