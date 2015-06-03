# Configuração

- [Introdução](#introducao)
- [Após a instalação](#apos-a-instalacao)
- [Acessando valores de configuração](#acessando-valores-de-configuracao)
- [Configuração de ambiente](#configuracao-de-ambiente)
- [Configurando o cache](#configurando-o-cache)
- [Modo de manutenção](#modo-de-manutencao)
- [URLs amigáveis](#urls-amigaveis)

<a name="introducao"></a>
## Introdução

Todos os arquivos de configuração do framework Laravel estão localizados na pasta `config`. Cada opção esta documentada, então fique a vontade para dar uma olhada nos arquivos e se familiarizar com as opções que estão disponíveis para você.

<a name="apos-a-instalacao"></a>
## Após a instalação

### Nomeando sua aplicação

Depois de instalar o Laravel, você pode colocar um "nome" em sua aplicação. Por padrão, o diretório `app` utiliza o namespace `App`, e é carregado automaticamente pelo Composer usando o [padrão de autoloading PSR-4](http://www.php-fig.org/psr/psr-4/). Porém, você pode mudar o namespace para o nome de sua aplicação, facilmente através do comando Artisan `app:name`.

Por exemplo, se sua aplicação tem o nome "Blog", você pode rodar o seguinte comando na raiz da sua instalação:

	php artisan app:name Blog

Renomear a sua aplicação é totalmente opcional, e você é livre para continuar utilizando o namespace `App` se você desejar.

### Outras configurações

O Laravel precisa de muito pouca configuração após a instalação. Você é livre para começar a desenvolver! Porém, você pode querer revisar o arquivo `config/app.php` e sua documentação. Ele contém várias configurações, tais como `timezone` e `locale` que você pode querer mudar de acordo com sua localidade.

Uma vez que o Laravel está instalado, você também pode querer [configurar seu ambiente local](/docs/5.0/configuration#environment-configuration).

> **Note:** Você nunca deve ter a configuração de `app.debug` com valor `true` para uma aplicação de produção.

<a name="permissoes"></a>
### Permissões

O Laravel pode precisar que você configure as permissões de escrita para a pasta `storage` para seu servidor web.

<a name="acessando-valores-de-configuracao"></a>
## Acessando valores de configuração

Você pode ter acesso fácil aos seus arquivos de configuração a partir do facade `Config`:

	$valor = Config::get('app.timezone');

	Config::set('app.timezone', 'America/Sao_Paulo');

Você também pode utilizar o helper `config`:

	$valor = config('app.timezone');

<a name="configuracao-de-ambiente"></a>
## Configuração de ambiente

É bem útil ter diferentes valores de configuração baseados no ambiente em que a aplicação está rodando. Por exemplo, você pode querer usar um mecanismo de cache diferente em seu ambiente local do utilizado em produção. Isto é fácil usando configurações baseadas no ambiente.

Para facilitar sua vida, o Laravel utiliza a biblioteca PHP [DotEnv](https://github.com/vlucas/phpdotenv) de Vance Lucas. Em uma instalação limpa do Laravel, o diretório raiz de sua aplicação irá conter um arquivo `.env.example`. Se você instalar o Laravel via Composer, este arquivo será automaticamente renomeado para `.env`. Caso contrário, você terá que renomeá-lo manualmente.

Todas as variáveis listadas neste arquivo serão carregadas para a variável super-global `$_ENV` do PHP quando sua aplicação receber uma requisição. Você pode utilizar o helper `env` para retornar os valores destas variáveis. Na realidade, se você revisar os arquivos de configuração do Laravel, você irá perceber que vários opções estarão utilizando este helper!

Fique a vontade para modificar as variáveis de seu ambiente para seu ambiente de desenvolvimento, assim como para seu ambiente de produção. Porém, seu arquivo `.env` não deve ser commitado para seu controle de versão, visto que cada desenvolvedor / servidor usando a sua aplicação podem necessitar de um arquivo de configuração diferente.

Se você está desenvolvendo com um time, você pode continuar incluindo um arquivo `.env.example` na sua aplicação. Colocando os valores de exemplo no arquivo de configuração, assim outros desenvolvedores vernao claramente quais variáveis de ambiente são necessárias para rodar sua aplicação.

#### Acessando qual o seu ambiente atual

Você pode saber qual o seu ambiente atual através do método `environment` de uma instância da classe `Application`:

	$ambiente = $app->environment();

Você também pode passar um valor para o método `environment` para verificar se o ambiente bate com o valor enviado:

	if ($app->environment('local'))
	{
		// O ambiente é local
	}

	if ($app->environment('local', 'staging'))
	{
		// O ambiente é local ou homologação
	}

Para obter uma instância da aplicação, utilize o contrato `Illuminate\Contracts\Foundation\Application` utilizando uma dependência do [service container](/docs/5.0/container). Claro que se você estiver dentro de um [service provider](/docs/5.0/providers), a instância da aplicação é acessada através da váriavel `$this->app`.

Uma instância da aplicação também pode ser acessada pelo helper `app` ou pelo facade `App`:

	$ambiente = app()->environment();

	$ambiente = App::environment();

<a name="configurando-o-cache"></a>
## Configurando o cache

Para dar uma pequena turbinada na sua aplicação, você pode cachear todos os seus arquivos de configuração em um único arquivo através do comando Artisan `config:cache`. Este irá combinar todas as opções de seu arquivo de configurão em um único arquivo que pode ser carregado rapidamente pelo framework.

Você normalmente pode utilizar o comando `config:cache` como parte de sua rotina de deploy.

<a name="modo-de-manutencao"></a>
## Modo de manutenção

Quando sua aplicação está em modo de manutenção, um view customizada será exibida para todas as requisições que sua aplicação receber. Isto torna fácil "disabilitar" sua aplicação enquanto você está atualizando ou quando você está fazendo alguma manutenção. O modo de manutenção está incluso no middeware padrão de sua aplicação. Se sua aplicação estiver no modo de manutenção, uma exceção `HttpException` será executada com um status code 503.

Para habilitar o modo de manutenção, simplesmente rode o comando Artisan `down`:

	php artisan down

Para desabilitar o modo de manutenção, utilize o comando `up`:

	php artisan up

### Template de modo de manutenção

O template padrão de modo de manutenção está localizado em `resources/views/errors/503.blade.php`.

### Modo de manutençnao & Filas

Enquanto sua aplicação está em modo de manutenção, nenhuma [tarefa da fila](/docs/5.0/queues) será gerenciada. As tarefas retornarão ao normal assim que sua aplicação sair do modo de manutenção.

<a name="urls-amigaveis"></a>
## URLs amigáveis

### Apache

O framework já vem com um arquivo `public/.htaccess` que é utilizado para executar as urls sem o `index.php`. Se você usa o Apache para servir sua aplicação Laravel, tenha certeza de que o módulo `mod_rewrite` está habilitado.

Se um arquivo `.htaccess` que vem com o Laravel nnao funcionar com sua instalação do Laravel, tente o seguinte:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

No Nginx, a seguinte diretiva nas configurações de seu site irão habilitar URLs amigáveis:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

Claro que, quando estiver usando o [Homestead](/docs/5.0/homestead), as URLs amigáveis serão configuradas automaticamente.
