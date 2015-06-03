# Laravel Cashier

- [Introdução](#introducao)
- [Configuração](#configuracao)
- [Assinaturas: pagamentos recorrentes](#assinaturas)
- [Cobrança única](#cobranca-unica)
- [Período de testes sem cartão inicial](#periodo-de-testes-sem-cartao-inicial)
- [Trocando planos](#trocando-planos)
- [Quantidade de assinaturas](#quantidade-de-assinaturas)
- [Cancelando uma assinatura](#cancelando-uma-assinatura)
- [Reativando uma assinatura](#reativando-uma-assinatura)
- [Verificando o status de uma assinatura](#verificando-o-status-de-uma-assinatura)
- [Gerenciando falhas de pagamento](#gerenciando-falhas-de-pagamento)
- [Gerenciando outros gatilhos do Stripe](#gerenciando-outros-gatilhos-do-stripe)
- [Histórico de Faturas](#historico-de-faturas)

<a name="introducao"></a>
## Introdução

Laravel Cashier provê uma interface expressiva e fluente para os serviços de pagamento do [Stripe](https://stripe.com). Ele gerencia quase todo o processo de meio de pagamento que você teme escrever. Além do gerenciamento de assinaturas básico, o Cashier pode tratar cupons, troca de planos, "quantidade" de assinaturas, periodos de carência de cancelamento, e até gerar faturas em PDF.

<a name="configuracao"></a>
## Configuração

#### Composer

Primeiro, adicione o pacote Cashier a seu `composer.json`:

	"laravel/cashier": "~4.0" (For Stripe APIs on 2015-02-18 version and later)
	"laravel/cashier": "~3.0" (For Stripe APIs up to and including 2015-02-16 version)

#### Service Provider

Depois, registre o `Laravel\Cashier\CashierServiceProvider` em seu arquivo de configuração `app`.

#### Migration

Antes de usar o Cashier, nós precisaremos adicionar várias colunas em seu banco de dados. Não se preocupe, você pode usar o comando `cashier:table` no Artisan para criar o arquivo de migração necessário para adicionar as colunas. Por exemplo, para adicionar as colunas a tabela users use o comando `php artisan cashier:table users`. Uma vez o arquivo de migração é criado, simplesmente rode o comando `migrate`.

#### Configuração do Model

Depois, adicione a trait `Billable` e os modificadores de data apropriados as definições do seu model:

	use Laravel\Cashier\Billable;
	use Laravel\Cashier\Contracts\Billable as BillableContract;

	class User extends Eloquent implements BillableContract {

		use Billable;

		protected $dates = ['trial_ends_at', 'subscription_ends_at'];

	}

#### Stripe Key

Então, configure sua Stripe key no arquivo de configuração `services.php`:

	'stripe' => [
		'model'  => 'User',
		'secret' => env('STRIPE_API_SECRET'),
	],

De forma alternativa, você pode gravar este valor em um de seus arquivos de bootstrap ou em um service provider, como por exemplo em `AppServiceProvider`:

	User::setStripeKey('stripe-key');

<a name="assinaturas"></a>
## Assinaturas: pagamentos recorrentes

Uma vez que você tem uma instância de um model, você pode facilmente fazer este usuário assinar um plano no Stripe:

	$user = User::find(1);

	$user->subscription('monthly')->create($creditCardToken);

Se você gostaria de adicionar um cupom de desconto a assinatura, você pode usar o método `withCoupon`:

	$user->subscription('monthly')
	     ->withCoupon('code')
	     ->create($creditCardToken);

O método `subscription` irá criar automaticamente uma assinatura no Stripe, assim como irá atualizar seu banco de dados com o ID de cliete do Stripe e outras informações relevantes sobre o pagamento. Se o seu plano tem um periodo de trial configurado no Stripe, a data de fim do trial tamém será atualizada será atualizada automaticamente no registro do usuário no banco de dados.

Se você tem um perido de trial que **não** está configurado no Stripe, você deverá configurar a data de fim do trial manualmente depois que for feita a assinatura:

	$user->trial_ends_at = Carbon::now()->addDays(14);

	$user->save();

### Especificando detalhes adicionais do Usuário

Se você gostaria de especificar informações adicionais do seu cliente, você pode fazer isto passando os dados como segundo argumento para o método `create`:

	$user->subscription('monthly')->create($creditCardToken, [
		'email' => $email, 'description' => 'Meu primeiro cliente'
	]);

Para aprender mais sobre campos adicionais suportados pelo stripe, verifique a [documentação de cadastro de clientes](https://stripe.com/docs/api#create_customer) do Stripe.

<a name="cobranca-unica"></a>
## Cobrança única

Se você deseja fazer uma cobrança única em um cartão de crédito de um cliente que fez uma assinatura, você deve usar o método `charge`:

	$user->charge(100);

O método `charge` aceita a quantia que você gostaria de cobrar no **menor denominador da moeda**. Então, por exemplo, no exemplo acima irá cobrar 100 centavos de dollar, ou $1.00, do cartão de crédito de um cliente.

O método `charge` aceita um array como segundo argumento, permitindo que você passe qualquer opção disponivel na criação de pagamentos da API do Stripe:

	$user->charge(100, [
		'source' => $token,
		'receipt_email' => $user->email,
	]);

O método `charge` irá retornar `false` se a cobrança falhar. Isto normalmente indica que a cobrança foi negada:

	if ( ! $user->charge(100))
	{
		// A cobrança foi negada...
	}

Se a cobrança for realizada com sucesso, a resposta completa do Stripe será retornada.

<a name="periodo-de-testes-sem-cartao-inicial"></a>
## Período de testes sem cartão inicial

Se sua aplicação oferece um trial sem a necessidade de um cartão inicial, utilize a propriedade `cardUpFront` no seu model com o valor `false`:

	protected $cardUpFront = false;

Na criação da conta, certifique-se de configurar a data em que o periodo de testes termina no seu model:

	$user->trial_ends_at = Carbon::now()->addDays(14);

	$user->save();

<a name="trocando-planos"></a>
## Trocando planos

Para trocar o plano de um usuário para uma nova assinatura, use o método `swap`:

	$user->subscription('premium')->swap();

Se o usuário estiver em um período de testes, este período será mantido normalmente. Também, se alguma quantia ainda exista para a assinatura atual, ela será mantida.

<a name="quantidade-de-assinaturas"></a>
## Quantidade de assinturas

As vezes as assinaturas são afetadas por "quantidades". Por exemplo, sua aplicação pode cobrar $10 por mês por usuário em uma conta. Para aumenta ou diminuir a quantia de suas assinaturas, use os métodos `increment` e `decrement`:

	$user = User::find(1);

	$user->subscription()->increment();

	// Add five to the subscription's current quantity...
	$user->subscription()->increment(5);

	$user->subscription()->decrement();

	// Subtract five to the subscription's current quantity...
	$user->subscription()->decrement(5);

<a name="cancelando-uma-assinatura"></a>
## Cancelando uma assinatura

Cancelar uma assinatura é como dar uma volta no parque:

	$user->subscription()->cancel();

Quando uma assinatura é cancelada, o Cashier irá popular o valor da coluna `subscription_ends_at` no seu banco de dados. Esta coluna é utilizada para saber quando o método `subscribed` deve iniciar a retornar `false`. Por exemplo, se um cliente cancela a assinatura no dia 1 de Março, mas a assinatura está programada para acabar no dia 5 de Março, o método `subscribed` irá continuar retornando `true` até dia 5 de Março.

<a name="reativando-uma-assinatura"></a>
## Reativando uma assinatura

Se um usuário que cancelou uma assinatura quiser reativá-la, use o método `resume`:

	$user->subscription('monthly')->resume($creditCardToken);

Se um usuário cancela uma assinatura e então reativa um assinatura que ainda não tenha expirado totalmente, ele não será cobrado imediatamente. Sua assinatura será simplesmente reativada, e ele será cobrado no periodo de cobrança inicial.

<a name="verificando-o-status-de-uma-assinatura"></a>
## Verificando o status de uma assinatura

Para verificar se um usuário fez uma assinatura em sua aplicação, use o comando `subscribed`:

	if ($user->subscribed())
	{
		//
	}

O método `subscribed` é um grande candidato a ser um [middleware de rotas](/docs/5.0/middleware):

	public function handle($request, Closure $next)
	{
		if ($request->user() && ! $request->user()->subscribed())
		{
			return redirect('billing');
		}

		return $next($request);
	}

Você também pode verificar se o usuário ainda está no período de testes (se aplicável) usando o método `onTrial`:

	if ($user->onTrial())
	{
		//
	}

Para determinar se o usuário já foi um assinante, mas cancelou sua assinatura, você pode usar o método `cancelled`:

	if ($user->cancelled())
	{
		//
	}

Você também pode verificar se um usuário cancelou sua assinatura, mas ainda está no "período de carência" até que sua assinatura expire. Por exemplo, se um usuário cancela sua assinatura no dia 5 de Março e a assinatura está programada para finalizar no dia 10 de Março, o usuário está no "período de carência" até o dia 10 de Março. Lembre que o método `subscribed` ainda retorna `true` durante este período.

	if ($user->onGracePeriod())
	{
		//
	}

O método `everSubscribed` pode ser usado para verificar se o usuário já assinou algum plano em sua aplicação alguma vez:

	if ($user->everSubscribed())
	{
		//
	}

O método `onPlan` pode ser usado para determinar se o usuário é assinante de um plano com base em um ID:

	if ($user->onPlan('monthly'))
	{
		//
	}

<a name="gerenciando-falhas-de-pagamento"></a>
## Gerenciando falhas de pagamento

O que acontece se o cartão de crédito de seu cliente expira? Não se preocupe - o Cashier inclui um controller de gatilho que pode facilmente cancelar a assinatura de um cliente para vocie. Apenas mapeia uma rota a este controller:

	Route::post('stripe/webhook', 'Laravel\Cashier\WebhookController@handleWebhook');

É isso! Pagamentos que falham serão capturados e gerenciados pelo controller. O controller cuidará de cancelar a assinatura do seu cliente depois de 3 tentativas de cobrança. A URI `stripe/webhook` é apenas um exemplo. Você precisará configurar a URI nas suas configurações do Stripe.

<a name="gerenciando-outros-gatilhos-do-stripe"></a>
## Gerenciando outros gatilhos do Stripe

Se você tem outros eventos de gatilho no Stripe que você gostaria de gerenciá-los, simplesmente extenda o controller gatilho. O nome dos sesus métodos devem seguir a convenção definida pelo Cashier, especificamente, métodos devem iniciar com `handle` e o nome do gatilho do Stripe que você deseja gerenciar. Por exemplo, se você quer gerenciar o gatilho `invoice.payment_succeeded`, você deve adicionar um método `handleInvoicePaymentSucceeded` ao controller.

	class WebhookController extends Laravel\Cashier\WebhookController {

		public function handleInvoicePaymentSucceeded($payload)
		{
			// Gerenciar evento
		}

	}

> **Note:** Ao atualizar as informações de uma assinatura no seu banco de dados, o controller de gatilho também irá cancelar a assinatura usando a API do Stripe.

<a name="historico-de-faturas"></a>
## Histórico de faturas

Você pode simplesmente retornar uma lista de faturas de um usuário usando o método `invoices`:

	$invoices = $user->invoices();

Quando estiver listando as faturas de um cliente, você pode usar estes métodos para exibir informações relevantes sobre as cobranças:

	{{ $invoice->id }}

	{{ $invoice->dateString() }}

	{{ $invoice->dollars() }}

Utilize o método `downloadInvoice` para gerar um PDF para download do histórico. Sim, é fácil assim:

	return $user->downloadInvoice($invoice->id, [
		'vendor'  => 'Sua empresa',
		'product' => 'Seu produto',
	]);
