# Collections

- [Introdução](#introducao)
- [Como Utilizar](#como-utilizar)

<a name="introducao"></a>
## Introdução

A classe `Illuminate\Support\Collection` fornece uma forma fluente e conveniente para trabalhar com arrays. Por exemplo, dê uma olhada no código abaixo. Vamos utilizar o helper `collect` para criar uma nova coleção a partir de um array:

	$colecao = collect(['taylor', 'abigail', null])->map(function($nome)
	{
		return strtoupper($nome);
	})
	->reject(function($nome)
	{
		return empty($nome);
	});


Como você pode ver, a classe `Collection` permite que você encadeie os métodos para que o código fique fluente. No geral, cada método de `Collection` retorna uma nova instância de `Collection`. Para conhecer melhor, continue lendo!

<a name="como-utilizar"></a>
## Como Utilizar

#### Criando Collections

Como mencionado acima, o helper `collect` retornará uma nova instância de `Illuminate\Support\Collection` para um dado array. Você também pode utilizar o comando `make` da classe `Collection`:

	$colecao = collect([1, 2, 3]);

	$colecao = Collection::make([1, 2, 3]);

Claro, que as coleções de objetos [Eloquent](/docs/5.0/eloquent) são sempre retornadas como instâncias de `Collection`; de qualquer forma, você pode ficar a vontade para utilizar a classe `Collection` sempre que achar conveniente em sua aplicação.

#### Explorando um Collection

Ao invés de listar todos os métodos (que são vários) disponíveis na classe Collection, confira a [API de documentação da classe](http://laravel.com/api/master/Illuminate/Support/Collection.html)!
