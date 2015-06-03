# Autenticação

- [Introdução](#introducao)
- [Autenticando usuários](#autenticando-usuarios)
- [Obtendo o usuário logado](#obtendo-o-usuario-logado)
- [Protegendo rotas](#protecting-routes)
- [HTTP Basic Authentication](#http-basic-authentication)
- [Lembrando & resetando senhas](#lembrando-e-resetando-senhas)
- [Autenticação via redes sociais](#autenticacao-via-redes-sociais)

<a name="introducao"></a>
## Introdução

O Laravel torna a implementação de autenticação muito simples. Na verdade, quase tudo é configurado para você automaticamente. O arquivo de configuração de autenticação esta localizado em `config/auth.php`, que contem muitas opções, bem documentadas, para alterar o comportamento de usuários autenticados.

Por padrão, o Laravel inclui um modelo `App\User` no seu diretório `app`. Este modelo deve ser usado com o driver padrão de autenticação Eloquent.

Lembre-se: quando criar a estrutura de banco de dados para este modelo, crie o campo password com no mínimo 60 caracteres. Atenção, tenha certeza de que a sua tabela `users` (ou equivalente) contenha um campo string de 100 caracteres, que aceita nulo, chamado `remember_token`. Esta coluna vai ser usada para gravar uma chave de sessão "lembre-me" que será mantida por sua aplicação. Este campo pode ser criado com o seguinte código `$table->rememberToken();` em um arquivo de migração. Mas é claro que o Laravel 5 já vem com os arquivos de migração para estas colunas por padrão!

Se sua aplicação não utilizar a ORM Eloquent, você deve utilizar o driver de autenticação `database`, que utiliza o gerador de consultas do Laravel.

<a name="autenticando-usuarios"></a>
## Autenticando Usuários

O Laravel vem com dois controllers de autenticação prontos. O `AuthController` gerencia o cadastro de novos usuários e "login", enquanto o `PasswordController` contém a lógica para ajudar os usuários a resetar suas senhas esquecidas.

Cada um destes controllers usa uma trait para incluir os métodos necessários. Para muitas aplicações, você não precisará modificar estes controllers. As views que estes controllers renderizam estão localizadas no diretório `resources/views/auth`. Você fica livre para customizar estas views da forma que quiser.

### O cadastro de usuários

Para modificar os campos obrigatórios no formulário para cadastro de novos usuários de sua aplicação, você deve modificar a classe `App\Services\Registrar`. Esta classe é responsável por validar e criar novos usuários de sua aplicação.

O método `validator` da classe `Registrar` contém as regras de validação para novos usuários, enquanto o método `create` é responsável por criar um novo registro de `User` no seu banco de dados. Você pode modificar livremente cada um destes métodos, da forma que desejar. O serviço `Registrar` é chamado pelo controller `AuthController` através dos métodos contidos na trait `AuthenticatesAndRegistersUsers`.

#### Manual de Authenticação

Se você escolher não utilizar a implementação de `AuthController`, você precisará gerenciar a autentição de usuários usando as classes de autenticação do Laravel diretamente. Não se preocupe, Don't worry, a coisa ainda continua fácil! Primeiro, vamos verificar o método `attempt`:

	<?php namespace App\Http\Controllers;

	use Auth;
	use Illuminate\Routing\Controller;

	class AuthController extends Controller {

		/**
		 * Handle an authentication attempt.
		 *
		 * @return Response
		 */
		public function authenticate()
		{
			if (Auth::attempt(['email' => $email, 'password' => $password]))
			{
				return redirect()->intended('dashboard');
			}
		}

	}

O método `attempt` aceita um array com pares de chave / valor como primeiro argumento. O valor `password` será criptografado [hashed](/docs/5.0/hashing). Os outros valores no array serão utilizados para buscar o usuário na sua tabela do banco de dados. Então, no exemplo acima, o usuário será retornado pelo valor da coluna `email`. Se o usuário for encontrado, a senha criptografada gravada no banco de dados será comparada com o valor criptografado de `password` recebido pela chave do array. Se as duas senhas conferirem, uma nova sessão para este usuário será iniciada.

O método `attempt` irá retornar `true` se a autenticação for realizada com sucesso. Caso contrário, `false` será retornado.

> **Atenção:** Neste exemplo, `email` não é uma opção obrigatória, e foi utilizado meramente como um exemplo. Você deve usar qualquer valor que corresponda a um "username" no seu banco de dados.

A função de redirecionamento `intended` irá redirecionar o usuário para a URL original, que ele estava tentando acessar antes de ser filtrado e redirecionado para autenticação. Uma URI de fallback deve ser adicionada como parâmetro deste método para o caso de o destino original não existir.

#### Autenticando um usuário com condições

Você também pode adicionar condições adicionais para a consulta de autenticação do usuário:

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1]))
    {
        // O usuário está ativo, não foi suspenso, e existe.
    }

#### Verificando se um usuário está autenticado

Para verificar se um usuário já está logado na sua aplicação, você deve usar o método `check`:

	if (Auth::check())
	{
		// O usuário está logado...
	}

#### Autenticando um usuário e "Lembrando" ele

Se você gostaria de adicionar a funcionalidade "lembre-me" na sua applicação, você deve passar um boolean como segundo parâmetro para o método `attempt`, que irá manter o usuário autenticado por tempo indefinido, ou até que eles desloguem manualmente. É claro, que sua tabela `users` deve incluir o campo string `remember_token`, que irá armazenar a chave para o "lembre-me".

	if (Auth::attempt(['email' => $email, 'password' => $password], $remember))
	{
		// Este usuário está sendo lembrado
	}

Se você esta "lembrando" usuários, você pode usar o método `viaRemember` para verificar se um usuário está logado através do cookie de "lembre-me":

	if (Auth::viaRemember())
	{
		//
	}

#### Autenticando usuários pelo ID

Para logar um usuário da aplicação usando seu ID, utilize o método `loginUsingId`:

	Auth::loginUsingId(1);

#### Validando credenciais de usuário sem fazer login

O método `validate` permite validar as credenciais de usuário sem efetivamente logar o usuário na aplicação:

	if (Auth::validate($credentials))
	{
		//
	}

#### Logar o usuário para uma única requisição

Você também pode utilizar método `once` para logar um usuário na aplicação para uma única requisição. Neste caso não será utilizada nem sessão nem cookie:

	if (Auth::once($credentials))
	{
		//
	}

#### Logar um usuário manualmente

Se você precisa logar uma instância de um usuário da sua aplicação, você pode chamar o método `login`:

	Auth::login($user);

Isto é equivalente a logar um usuário através de credenciais do método `attempt`.

#### Deslogar usuários

	Auth::logout();

Porém, é claro que existe um método de logout pronto se você estiver usando os controllers padrões do Laravel.

#### Eventos de autenticação

Quando o método `attempt` é chamado, o [evento](/docs/5.0/events) `auth.attempt` é disparado. Se a autenticação for realizada com sucesso, o evento `auth.login` também será disparado.

<a name="obtendo-o-usuario-logado"></a>
## Obtendo o usuário logado

Uma vez que um usuário está autenticado, existem várias formas de obter uma instância deste usuário.

Primeiro, você pode acessar o usuário a partir do facade `Auth`:

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile()
		{
			if (Auth::user())
			{
				// Auth::user() returns an instance of the authenticated user...
			}
		}

	}

Segundo, você pode acessar o usuário autenticado a partir de uma instância de `Illuminate\Http\Request`:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile(Request $request)
		{
			if ($request->user())
			{
				// $request->user() returns an instance of the authenticated user...
			}
		}

	}

Terceiro, você pode injetar como dependência o contrato `Illuminate\Contracts\Auth\Authenticatable`. Esta injeção de dependência pode ser adicionada no construtor do controller, em um método do controller, ou qualquer outro construtor de uma classe que possa ser resolvida pelo [service container](/docs/5.0/container):

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use Illuminate\Contracts\Auth\Authenticatable;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile(Authenticatable $user)
		{
			// $user is an instance of the authenticated user...
		}

	}

<a name="protegendo-rotas"></a>
## Protegendo rotas

[Middleware de rotas](/docs/5.0/middleware) pode ser utilizado para permitir somente usuários logados possam acessar determinada rota. O Laravel provê o middleware `auth` por padrão, e ele está definido em `app\Http\Middleware\Authenticate.php`. Tudo o que você precisa fazer é adicioná-lo a definição de uma rota:

	// Com uma Rota Closure...

	Route::get('profile', ['middleware' => 'auth', function()
	{
		// Apenas usuários autenticados podem acessar...
	}]);

	// Com um Controller...

	Route::get('profile', ['middleware' => 'auth', 'uses' => 'ProfileController@show']);

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

HTTP Basic Authentication é uma forma rápida de autenticar usuários de sua aplicação sem precisar configurar uma "página dedicada para o login". Para utilizar, adicione o middleware `auth.basic` a sua rota:

#### Protecting A Route With HTTP Basic

	Route::get('profile', ['middleware' => 'auth.basic', function()
	{
		// Apenas usuários autenticados podem acessar...
	}]);

Por padrão, o middleware `basic` irá utilizar o campo `email` na tabela de usuários como "nome de usuário".

#### Configurando um Stateless HTTP Basic Filter

Você também pode utilizar HTTP Basic Authentication sem configurar um cookie de usuário na sessão, que é util para autenticações de API. Para fazer isto, [defina um middleware](/docs/5.0/middleware) que chame o método `onceBasic`:

	public function handle($request, Closure $next)
	{
		return Auth::onceBasic() ?: $next($request);
	}

Se você está utilizando PHP com FastCGI, o HTTP Basic authentication pode não funcionar corretamente com as configurações padrão. As seguintes linhas devem ser adicionadas a seu arquivo `.htaccess`:

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="lembrando-e-resetando-senhas"></a>
## Lembrando & resetando senhas

### Model & Tabela

Boa parte das aplicações web provêm uma forma para seus usuários resetarem suas senhas quando esquecidas. Ao invés de forçar que você re-implemente esta funcionalidade toda vez que você cria uma aplicação, o Laravel provê métodos convenientes para enviar dicas para lembrar senhas e realizar o reset de uma senha.

Antes de mais nada, verifique que seu model `User` implementa o contrato `Illuminate\Contracts\Auth\CanResetPassword`. É claro que o model `User` incluso no framework já implementa esta interface, e utiliza a trait `Illuminate\Auth\Passwords\CanResetPassword` para incluir os métodos necessários para implementar a interface.

#### Gerando o arquivo de migração para tabela de Lembrar Senhas

Agora, uma tabela deve ser criada para armazenar as chaves de resetar senha. O arquivo de migração para esta tabela está incluso no Laravel por padrão, e fica no diretório `database/migrations`. Então tudo o que você precisa fazer é rodar o comando migrate:

	php artisan migrate

### Controller para Lembrar Senhas

O Laravel também inclui um controller `Auth\PasswordController` que contém a lógica necessária para resetar as senhas do usuário. Nós também provêmos views para que você comece a utilizar! As views estão localizadas no diretório `resources/views/auth`. Você é livre para modificar estas views como quiser, para se adequar ao layout do seu site.

Seu usuário irá receber um e-mail com um link que aponta para o método `getReset` do controller `PasswordController`. Este método irá renderizar um formulário para resetar a senha. Depois que a senha é resetadas, o usuário irá logar automáticamente na aplicação e será redirecionado para `/home`. Você pode alterar o redirecionamento  pós-resetar senha utilizando a propriedade `redirectTo` do `PasswordController`:

	protected $redirectTo = '/dashboard';

> **Atenção:** Por padrão, as chaves para resetar senha expiram após uma hora. Você pode modificar este valor na chave `reminder.expire` no arquivo de configuração `config/auth.php`.

<a name="autenticacao-via-redes-sociais"></a>
## Autenticação via redes sociais

Além da autenticação padrão, baseada em formulário, o Laravel também provê, uma forma simples e conveniente de autenticar os usuários através de provedores Oauth usando o pacote [Laravel Socialite](https://github.com/laravel/socialite). **Socialite atualmente suporta autenticação com Facebook, Twitter, Google, GitHub e Bitbucket.**

Para iniciar com o Socialite, inclua o pacote em seu arquivo `composer.json`:

	"laravel/socialite": "~2.0"

Depois, registre o `Laravel\Socialite\SocialiteServiceProvider` no seu arquivo de configuração `config/app.php`. Você pode também registrar um [facade](/docs/5.0/facades):

	'Socialize' => 'Laravel\Socialite\Facades\Socialite',

Você precisará adicionar as credenciais para os serviços de OAuth que sua aplicação utiliza. Estas credenciais devem ser configuradas no arquivo `config/services.php`, e devem usar as chaves `facebook`, `twitter`, `google`, ou `github`, dependendo dos provedores que sua aplicação necessita. Por exemplo:

	'github' => [
		'client_id' => 'seu-app-id-no-github',
		'client_secret' => 'seu-app-secret-no-github',
		'redirect' => 'http://sua-url-de-callback',
	],

Depois, você está pronto para autenticar os usuários! Você irá precisar de 2 rotas: uma para redirecionar os usuários para o provedor OAuth, e outra para receber o callback do provedor depois da autenticação. Aqui está um exemplo de uso do facade `Socialize`:

	public function redirectToProvider()
	{
		return Socialize::with('github')->redirect();
	}

	public function handleProviderCallback()
	{
		$user = Socialize::with('github')->user();

		// $user->token;
	}

O método `redirect` se encarrega de enviar o usuário ao provedor OAuth, enquanto o método `user` vai ler a requisição de retorno e obter as informações do provedor. Antes de redirecionar o usuário, você pode também definir os "escopos" da requisição:

	return Socialize::with('github')->scopes(['scope1', 'scope2'])->redirect();

Uma vez que você tem a instância do usuário, você pode obter algumas outras informações do usuário:

#### Obtendo informações do usuário

	$user = Socialize::with('github')->user();

	// Provedores OAuth 2 
	$token = $user->token;

	// Provedores OAuth 1
	$token = $user->token;
	$tokenSecret = $user->tokenSecret;

	// Todos os provedores
	$user->getId();
	$user->getNickname();
	$user->getName();
	$user->getEmail();
	$user->getAvatar();
