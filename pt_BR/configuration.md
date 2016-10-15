# Configuração

- [Introdução](#introduction)
- [Acessando Valores de Configuração](#accessing-configuration-values)
- [Configuração de Ambiente](#environment-configuration)
    - [Determinando o Ambiente Atual](#determining-the-current-environment)
- [Cache de configuração](#configuration-caching)
- [Modo de Manutenção](#maintenance-mode)

<a name="introduction"></a>
## Introducão

Todos os arquivos de configuração do framework Laravel são armazenados no diretório `config`. Cada opção é documentada, sinta-se livre para ver todos os arquivos e se familiarizar com as opções disponíveis para você.

<a name="accessing-configuration-values"></a>
## Acessando Valores de Configuração

Você pode facilmente acessar seus valores de configuração usando a função helper `config`, de qualquer lugar da sua aplicação. Os valores de configuração podem ser acessados usando a sintaxe "ponto", que inclui o nome do arquivo e a opção que você deseja acessar. Um valor padrão também pode ser especificado e será retornado se a opção de configuração não existir:

    $value = config('app.timezone');

Para definir valores de configuração em tempo de execução, passe um array para o helper 'config':

    config(['app.timezone' => 'America/Chicago']);

<a name="environment-configuration"></a>
## Configuração de Ambiente

Muitas vezes, é útil ter diferentes valores de configuração com base no ambiente da aplicação que está sendo executado. Por exemplo, você pode querer usar um driver de cache local diferente do no seu servidor de produção. É fácil utilizar a configuração baseada no ambiente.

Para tornar esta uma atividade fácil, o Laravel utiliza a biblioteca PHP [DotEnv](https://github.com/vlucas/phpdotenv) desenvolvida por Vance Lucas. Em uma nova instalação Laravel, o diretório raiz da sua aplicação conterá um arquivo `.env.example`. Se você instalar Laravel via Composer, este arquivo será automaticamente renomeado para `.env`. Caso contrário, você deve renomear o arquivo manualmente.

Todas as variáveis listadas neste arquivo são carregadas na variável super-global `$_ENV` do PHP quando sua aplicação recebe uma requisição. No entanto, você pode usar o helper `env` para recuperar valores dessas variáveis em seus arquivos de configuração. Na verdade, se você analisar os arquivos de configuração do Laravel, você vai notar várias opções que já utilizam esse helper:

    'debug' => env('APP_DEBUG', false),

O segundo valor passado para a função `env` é o "valor padrão". Esse valor será usado se não existir nenhuma variável de ambiente para a chave dada.

Você não deve enviar seu arquivo `.env` para o servidor de código fonte, uma vez que cada desenvolvedor / servidor usando sua aplicatição poderá exigir uma configuração de ambiente diferente.

Se você estiver desenvolvendo com uma equipe, você pode querer manter um arquivo `.env.example` com sua aplicação. Ao colocar valores no arquivo de configuração de exemplo, outros desenvolvedores de sua equipe podem ver claramente quais variáveis de ambiente são necessárias para executar a aplicação.

<a name="determining-the-current-environment"></a>
### Determinando o Ambiente Atual

O ambiente da aplicação atual é determinado através da variável `APP_ENV` de seu arquivo `.env`. Você pode acessar esse valor através do método ambiente na [facade](https://github.com/artesaos/laravel-docs/blob/master/pt_BR/facades.md) `App`:

    $environment = App::environment();

Você também pode passar argumentos para o método `environment` para verificar se o ambiente corresponde a um determinado valor. Se necessário, você pode até passar vários valores para o método `environment`. Se o ambiente corresponde a qualquer um dos valores indicados, o método irá retornar `true`:

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment('local', 'staging')) {
        // The environment is either local OR staging...
    }

Uma instancia da aplicação pode também ser acessada pelo método do helper `app`:

    $environment = app()->environment();

<a name="configuration-caching"></a>
## Cache de configuração

Para dar a sua aplicação um aumento de velocidade, você deve armazenar em cache todos os seus arquivos de configuração em um único arquivo usando o comando Artisan `config: cache`. Isto irá combinar todas as opções de configuração para sua aplicação em um único arquivo que será carregado rapidamente pelo framework.

Você normalmente deve executar o comando `php artisan config:cache` como parte de sua rotina de deploy para o ambiente de produção. O comando não deve ser executado durante o desenvolvimento local onde as opções de configuração freqüentemente precisam ser alteradas durante o curso do desenvolvimento da sua aplicação.

<a name="maintenance-mode"></a>
## Modo de Manutenção

Quando sua aplicação estiver no modo de manutenção, uma view personalizada será exibida para todos as requisições de sua aplicação. Isto facilita "desativar" o seu aplicativo enquanto ele está atualizando ou quando você está realizando uma manutenção. Uma verificação de modo de manutenção é incluída na pilha do middleware padrão para a sua aplicação. Se o aplicativo estiver no modo de manutenção, um `MaintenanceModeException` será lançada com um código de status 503.

Para ativar o modo de manutenção, simplesmente execute o comando `down` do Artisan:

    php artisan down

Você também pode fornecer `message` e `retry` como opções ao comando `down`. O valor de `message` pode ser usado para exibir ou registrar uma mensagem personalizada, enquanto o valor de `retry` será definido no valor do HTTP header `Retry-After`:

    php artisan down --message='Upgrading Database' --retry=60

Para desativar o modo de manutenção, use o comando `up`:

    php artisan up

#### Template de Resposta para o Modo de Manutenção

O template padrão de resposta para o modo de manutenção está localizado em resources/views/errors/503.blade.php. Você pode modificá-lo como necessitar para sua aplicação.

#### Modo de Manutenção & Filas

Enquanto a aplicação está em modo de manutenção, nenhum [queued jobs](https://github.com/artesaos/laravel-docs/blob/master/queues.md) será lançado. Os jobs vão continuar a serem tratados normalmente assim que a aplicação estiver fora do modo de manutenção.

#### Alternativas para modo de manutenção

Alterar para o modo de manutenção requer alguns segundos para finalizar o processo, você pode considerar alternativas como [Envoyer](https://envoyer.io) para zerar esse tempo de deploy com o Laravel.