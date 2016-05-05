# Autorização

- [Introdução](#introduction)
- [Definindo Permissões](#defining-abilities)
- [Verificando Permissões](#checking-abilities)
	- [Pela Façade Gate](#via-the-gate-facade)
	- [Pelo Model User](#via-the-user-model)
	- [Dentro dos Templates Blade](#within-blade-templates)
    - [Dentro dos Form Requests](#within-form-requests)
- [Políticas de acesso (Papeis / Grupos)](#policies)
	- [Criando as Políticas de Acesso](#creating-policies)
	- [Escrevendo as Políticas de Acesso](#writing-policies)
	- [Verificando as Políticas de Acesso](#checking-policies)
- [Controller de Autorização](#controller-authorization)

<a name="introduction"></a>
## Introdução

O Laravel disponibiliza uma forma muito simples de organizar o controle de acesso aos recursos da aplicação. Há uma variedade de métodos e helpers para ajudá-lo nesta tarefa, e iremos discriminá-las nesta documentação. Os serviços de autorização são fornecidos nativamente pelo framework em conjunto com os serviços de [authentication](/docs/{{version}}/authentication).

<a name="defining-abilities"></a>
## Definindo as Permissões

O meio mais simples para determinar se um usuário pode executar uma determinada ação é definir uma "permissão" usando a classe `Illuminate\Auth\Access\Gate`. A classe `AuthServiceProvider` fornecida com o Laravel é o local indicado para definir todas as "permissões" da aplicação. Por exemplo, vamos definir uma permissão chamada `update-post` que recebe o `User` logado e um [model](/docs/{{version}}/eloquent) `Post`. Em nossa permissão, iremos determinar se o `id` do `User` é igual ao `user_id` do `Post`:

	<?php

	namespace App\Providers;

	use Illuminate\Contracts\Auth\Access\Gate as GateContract;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
         * Registrar qualquer serviço de autenticação / autorização
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        $this->registerPolicies($gate);

	        $gate->define('update-post', function ($user, $post) {
	        	return $user->id === $post->user_id;
	        });
	    }
	}

Observe que não verificamos se o `$user` é diferente de nulo. O `Gate` retornará `false` automaticamente para todas as **permissões** em caso de usuário não autenticado ou um determinado usuário não for especificado pelo método `forUser`.

#### Classe baseada em permissões

Além de registrar `Closures` como callback de autorização, você pode registrar métodos de classes passando uma string com o nome da classe e o nome do método. Quando necessário, a classe será resolvida via [service container](/docs/{{version}}/container):

    $gate->define('update-post', 'Class@method');

<a name="intercepting-all-checks"></a>
<a name="intercepting-authorization-checks"></a>
#### Interceptando a validação da autorização

Em alguns casos, você pode querer liberar todas as permissões para um determinado usuário. Para esta situação, use o método `before` para definir um callback que execute antes de todas as verificações de autorização.

    $gate->before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Se o callback `before` retornar um resultado diferente de nulo, este será considerado o resultado da verificação.

Você pode usar o método `after` para definir um callback a ser executado depois de cada verificação. Porém, você não pode modificar o resultado da verificação.

    $gate->after(function ($user, $ability, $result, $arguments) {
        //
    });

<a name="checking-abilities"></a>
## Verificando as permissões

<a name="via-the-gate-facade"></a>
### Através da fachada Gate

Com a permissão já definida, podemos verificá-la de várias formas. Primeiro, podemos usar os métodos `check`, `allows` ou `denies` na fachada `Gate`. Todos esses métodos recebem o nome da permissão e os argumentos passados pelo callback da permissão. Você não precisa passar o usuário logado para estes métodos, já que o `Gate` irá automaticamente injetar o usuário logado nos parâmetros passados pelo callback. Assim, ao verificar a permissão `update-post` definida anteriormente, nós simplesmente passamos a instancia `Post` para o método `denies`:

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Atualizar o Post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update-post', $post)) {
        		abort(403);
        	}

        	// Atualizar o Post...
        }
    }

O método `allows` é simplesmente o inverso do método `denies`, e retorna `true` se a ação está autorizada. O método `check` é um apelido para o método `allows`.

#### Verificando as permissões para um Usuário específico

Se você gostaria de usar a fachada `Gate` para verificar se um usuário **diferente do usuário logado** tenha permissão, você pode usar o método `forUser`:

	if (Gate::forUser($user)->allows('update-post', $post)) {
		//
	}

#### Passando múltiplos argumentos

Os callbacks de permissões podem receber múltiplos argumentos, conforme exemplo abaixo:

	Gate::define('delete-comment', function ($user, $post, $comment) {
		//
	});

Se sua permissão precisa de vários argumentos, simplesmente passe-os em um array para os métodos do `Gate`:

	if (Gate::allows('delete-comment', [$post, $comment])) {
		//
	}

<a name="via-the-user-model"></a>
### Através do model User

Você também pode verificar as permissões através da instância do model `User`. Por padrão, o model `App\User` do Laravel usa a trait `Authorizable` que provê dois métodos: `can` e `cannot`. Estes métodos podem ser usados de forma semelhante aos métodos `allows` and `denies` da façade `Gate`. Usando nosso exemplo anterior, vamos modificar o código conforme abaixo:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Atualizando o Post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
        	$post = Post::findOrFail($id);

        	if ($request->user()->cannot('update-post', $post)) {
        		abort(403);
        	}

        	// Atualizar o Post...
        }
    }

O método `@can` é simplesmente o inverso do método `@cannot`:

	if ($request->user()->can('update-post', $post)) {
		// Atualizar o Post...
	}

<a name="within-blade-templates"></a>
### Dentro do template do Blade

Por conveniência, o Laravel provê a diretiva Blade `@can` para verificar se o usuário logado tem permissão. Por exemplo:

	<a href="/post/{{ $post->id }}">Ver Post</a>

	@can('update-post', $post)
		<a href="/post/{{ $post->id }}/edit">Editar Post</a>
	@endcan

Você pode combinar a diretiva `@can` com a diretiva `@else`:

	@can('update-post', $post)
		<!-- O usuário logado pode atualizar o Post -->
	@else
		<!-- O usuário logado não pode atualizar o Post -->
	@endcan

<a name="within-form-requests"></a>
### Dentro dos Form Requests

Você também pode optar por utilizar suas permissões definidas do `Gate` a partir do método `authorize` de um [form request's](/docs/{{version}}/validation#form-request-validation). Por exemplo:

    /**
     * Determinar se o usuário está autorizado a prosseguir com esta requisição.
     *
     * @return bool
     */
    public function authorize()
    {
        $postId = $this->route('post');

        return Gate::allows('update', Post::findOrFail($postId));
    }

<a name="policies"></a>
## Políticas de Acesso

<a name="creating-policies"></a>
### Criando Políticas de Acesso

Definir toda sua lógica de autorização dentro do `AuthServiceProvider` pode se tornar complexo em grandes aplicações, o Laravel permite a você separar sua lógica de autorização em classes de "Políticas de Acesso (Papeis / Grupos)". Uma classe de política de acesso é uma classe PHP simples que agrupa a lógica de autorização.

Primeiro, vamos gerar uma política para gerencia a autorização para seu model `Post`. Você precisa gerar uma política utilizando o [comando artisan](/docs/{{version}}/artisan) `make:policy`. A política gerada será inserida no diretório de políticas `app/Policies`.

	php artisan make:policy PostPolicy

#### Registrando políticas de acesso

Com a política criada, precisamos registrá-la na classe `Gate`. O `AuthServiceProvider` possui uma propriedade chamada `policies` no qual mapeia várias entidades às políticas que os gerenciam. Então, iremos especificar que a política do model `Post` é a classe `PostPolicy`:

	<?php

	namespace App\Providers;

	use App\Post;
	use App\Policies\PostPolicy;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * O mapeamento da política de acesso para a aplicação
	     *
	     * @var array
	     */
	    protected $policies = [
	        Post::class => PostPolicy::class,
	    ];

	    /**
	     * Registrar qualquer serviço de autenticação / autorização.
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        $this->registerPolicies($gate);
	    }
	}

<a name="writing-policies"></a>
### Escrevendo Políticas

Com a política gerada e registrada, podemos adicionar métodos para cada permissão que a autorize. Por exemplo, vamos definir o método `update` em nosso `PostPolicy`, no qual determinaremos se o usuário informado pode atualizar um `Post`:

	<?php

	namespace App\Policies;

	use App\User;
	use App\Post;

	class PostPolicy
	{
		/**
		 * Determina se um post passado pode ser atualizado pelo usuário.
		 *
		 * @param  \App\User  $user
		 * @param  \App\Post  $post
		 * @return bool
		 */
	    public function update(User $user, Post $post)
	    {
	    	return $user->id === $post->user_id;
	    }
	}

Você pode definir quantos métodos forem necessários em sua política. Por exemplo, você pode definir os métodos `show`, `destroy` ou `addComment` para autorizar várias ações de `Post`.

> **Nota:** Todas as políticas são resolvidas pelo [service container](/docs/{{version}}/container) do Laravel, ou seja, você pode digitar quaisquer dependencias no contrutor da política e elas serão injetadas automaticamente.

#### Interceptando todas as verificações

Em alguns casos, você pode querer liberar todas as permissões para um determinado usuário. Para esta situação, defina um método `before` em sua política. Este método será executado antes de todas as verificações de autorização de sua classe:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Se o método `before` retornar um resultado diferente de nulo, este resultado será considerado na verificação.

<a name="checking-policies"></a>
### Verificando as políticas

Os métodos das políticas são chamados da mesma forma que as `Closure` baseadas nos callbacks de autorização. Você pode usar a fachada `Gate`, o model `User`, a diretiva do Blade `@can` ou ainda o helper da `policy`.

#### Através da fachada Gate

O `Gate` determinará automaticamente qual política usar examinando os parâmetros passados para os métodos da classe. Se nós passarmos uma instancia de `Post` para o método `denies`, o `Gate` utilizará a `PostPolicy` correspondente para autorizar a ação:

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Atualizar o Post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update', $post)) {
        		abort(403);
        	}

        	// Atualizar Post...
        }
    }

#### Através do model User

Os métodos `can` and `cannot` do model `User` também utilizará automaticamente as policies quando elas forem passadas nos argumentos. Estes métodos oferecem uma maneira conveniente para autorizar ações de qualquer `User` recuperado pela sua aplicação:

	if ($user->can('update', $post)) {
		//
	}

	if ($user->cannot('update', $post)) {
		//
	}

#### Dentro dos templates Blade

Da mesma forma, a diretiva Blade `@can` utilizará as políticas quando elas forem passadas nos argumentos:

	@can('update', $post)
		<!-- O usuário logado pode atualizar o Post -->
	@endcan

#### Através do Helper da política

A função global do helper da `policy` pode ser usada para capturar a classe `Policy` equivalente da instância de classe fornecida. Por exemplo, nós podemos passar uma instância de `Post` para o helper e pegar uma instância correspondente da classe `PostPolicy`:

	if (policy($post)->update($user, $post)) {
		//
	}

<a name="controller-authorization"></a>
## Controller de Autorização

Por padrão, a classe base `App\Http\Controllers\Controller` incluída com o Laravel, usa a trait `AuthorizesRequests`. Esta trait fornece o método `authorize`, que pode ser usado para autorizar uma ação fornecida e capturar uma `HttpException` se a ação não foi autorizada.

O método `authorize` compartilha a mesma assinatura de vários outros métodos de autorização, bem como `Gate::allows` and `$user->can()`. Vamos usar o método `authorize` para autorizar uma requisição para atualizar um `Post`:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Atualizar um Post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	$this->authorize('update', $post);

        	// Atualizar Post...
        }
    }

Se a ação está autorizada, o controller continuará executando normalmente; caso contrário, se o método `authorized` determinar que a ação não está autorizada, uma `HttpException` será lançada automaticamente que irá gerar uma resposta HTTP com um código de status `403 Not Authorized`. Como pode ver, o método `authorized` é um conveniente e rápido meio de autorizar ações ou capturar exceções com uma simples linha de código.

A trait `AuthorizesRequests` também fornece o método `authorizeForUser` para autorizar uma ação em um usuário que não está autenticado:

	$this->authorizeForUser($user, 'update', $post);

#### Determinando Métodos de Política Automaticamente

Frequentemente, um método de política corresponderá ao método no controller. Por exemplo, no método `update` abaixo, o método do controller e o método da política compartilham o mesmo nome: `update`.

Por esta razão, o Laravel permite a você simplesmente passar a instância como argumento do método `authorize`, e a permissão a ser autorizada será determinada automaticamente com base no nome da função que está chamando. Neste exemplo, `authorize` é chamado pelo método `update` no controller, o método `update` também será chamado no `PostPolicy`:

    /**
     * Atualizar o Post.
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
    	$post = Post::findOrFail($id);

    	$this->authorize($post);

    	// Atualizar Post...
    }
