# Validation

- [Introdução](#introduction)
- [Validação Começo Rápido](#validation-quickstart)
    - [Definindo As Rotas](#quick-defining-the-routes)
    - [Criando O Controller](#quick-creating-the-controller)
    - [Escrevendo A Lógica de Validação](#quick-writing-the-validation-logic)
    - [Mostrando Os Erros De Validação](#quick-displaying-the-validation-errors)
    - [Requisições AJAX & Validações](#quick-ajax-requests-and-validation)
- [Outras Abordagens Sobre Validações](#other-validation-approaches)
    - [Criando Validadores Manualmente](#manually-creating-validators)
    - [Formulário de Requisições De Validações](#form-request-validation)
- [Trabalhando Com Menssagens De Erro](#working-with-error-messages)
    - [Customizando Menssagens De Erro](#custom-error-messages) 
- [Regras De Validação Disponíveis](#available-validation-rules)
- [Adicionando Regras Condicionalmente](#conditionally-adding-rules)
- [Regras De Validação Personalizadas](#custom-validation-rules)

<a name="introduction"></a>
## Introdução

Laravel fornece várias abordagens diferentes para validar dados de entrada do seu aplicativo. Por padrão, a classe controlador de base de Laravel usa um ` ValidatesRequests` traço que fornece um método conveniente para validar solicitação HTTP de entrada com uma variedade de regras poderosos de validação.


<a name="validation-quickstart"></a>
## Validação Começo Rápido

Para aprender sobre as poderosas caracteristicas de validação do Laravel, vamos ver um exemplo completo de validar um formulário e mostrar as menssagens de erros para o usuário.

<a name="quick-defining-the-routes"></a>
### Definindo As Rotas

Primeiro, vamos assumir que temos as seguintes rotas definidas em nosso `app/Http/routes.php` arquivo:

    // Mostrando um formulário para criar uma postagem no blog...
    Route::get('post/create', 'PostController@create');

    // Salvando uma nova postagem do blog...
    Route::post('post', 'PostController@store');

A rota `GET` irá mostrar o formulário para o usuário criar uma nova postagem no blog, enquanto a rota `POST` irá salvar a nova postagem no banco de dados

<a name="quick-creating-the-controller"></a>
### Criando A Controller

Agora vamos ver um simples exemplo de controller que lida com essas rotas. Iremos deixar o método `store` vazio por hora:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Mostra o fomulário para criar uma nova postagem do blog.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Salva uma nova postagem.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Valida e salva uma nova postagem no blog...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### Escrevendo A Lógica De Validação

Agora estamos prontos para preencher o nosso método `store` com a lógica para validar a nova postagem do blog. Se você examinar a classe controller base da sua aplicação (`App\Http\Controllers\Controller`), verá que essa classe usa a `ValidatesRequests`. Essa classe fornece uma validação conveniente em todos os métodos de suas controllers.

O método `validate` aceita uma requisição HTTP e um conjunto de regras de validação. Se a regra de validação passa, seu código continuará normalmente; contudo, se a validação falha, uma exceção será lançada e a resposta de erro apropriada será automaticamente enviada de volta para o usuário. No caso de uma requisição HTTP tradicional, uma resposta de redirecionamento será gerada, enquanto um arquivo JSON será enviado via requisição AJAX.

Para um melhor entendimento do método de validação, vamos voltar ao método `store`:


    /**
     * Salva uma nova postagem.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // A postagem é válida, salva no banco de dados...
    }

Como você pode ver, nos simplesmente passamos uma requisição HTTP e as regras de validação desejadas no método `validate`. De novo, se a validação falha, a resposta apropriada irá ser gerada automaticamente. Se a validação passa, nosso controller irá continuar a execução normalmente.

#### Uma Nota Sobre Atributos Vazios

Se nossa requisição HTTP contém parámetros vazios, você pode especificá-lo em nossa regra de validação usando a seguinte sintaxe:

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Mostrando Os Erros De Validação

Então, e se os parâmetros de solicitação de entrada não passarem as regras de validação dadas? Como mencionado anteriormente, Laravel irá redirecionar automaticamente o usuário de volta à sua posição anterior. Além disso, todos os erros de validação será automaticamente [guardados na sessão](/docs/{{version}}/session#flash-data).

Observe que nos não temos que explicitamente ligar as menssagens de erros para a view em nossa rota `GET`. Isso acontece porque o Laravel sempre irá sempre checar se há erros nos dados da sessão, e automaticamente vinculá-las à view, se eles estiverem disponíveis. **Assim, é importante notar que uma variável `$erros` estará sempre disponível em todas as views em todas as requisições**, o que lhe permite assumir convenientemente que a variável `$erros` estará sempre definida e poderá ser seguramente usada. A variável `$erros` será uma instancia de `Illuminate\Support\MessageBag`. Para mais informações sobre como trabalhar com esse objeto, [verifique essa documentação] (#working-with-error-messages).

Assim, no nosso exemplo, o usuário será redirecionado para o método `create` do nosso controller quando a validação falhar, o que nos permite exibir as mensagens de erro na view:

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Formulário -->

<a name="quick-customizing-the-flashed-error-format"></a>
#### Customizing The Flashed Error Format

If you wish to customize the format of the validation errors that are flashed to the session when validation fails, override the `formatValidationErrors` on your base controller. Don't forget to import the `Illuminate\Contracts\Validation\Validator` class at the top of the file:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Bus\DispatchesJobs;
    use Illuminate\Contracts\Validation\Validator;
    use Illuminate\Routing\Controller as BaseController;
    use Illuminate\Foundation\Validation\ValidatesRequests;

    abstract class Controller extends BaseController
    {
        use DispatchesJobs, ValidatesRequests;

        /**
         * {@inheritdoc}
         */
        protected function formatValidationErrors(Validator $validator)
        {
            return $validator->errors()->all();
        }
    }

<a name="quick-ajax-requests-and-validation"></a>
### AJAX Requests & Validation

In this example, we used a traditional form to send data to the application. However, many applications use AJAX requests. When using the `validate` method during an AJAX request, Laravel will not generate a redirect response. Instead, Laravel generates a JSON response containing all of the validation errors. This JSON response will be sent with a 422 HTTP status code.

<a name="other-validation-approaches"></a>
## Other Validation Approaches

<a name="manually-creating-validators"></a>
### Manually Creating Validators

If you do not want to use the `ValidatesRequests` trait's `validate` method, you may create a validator instance manually using the `Validator` [facade](/docs/{{version}}/facades). The `make` method on the facade generates a new validator instance:

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

The first argument passed to the `make` method is the data under validation. The second argument is the validation rules that should be applied to the data.

After checking if the request failed to pass validation, you may use the `withErrors` method to flash the error messages to the session. When using this method, the `$errors` variable will automatically be shared with your views after redirection, allowing you to easily display them back to the user. The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.

#### Named Error Bags

If you have multiple forms on a single page, you may wish to name the `MessageBag` of errors, allowing you to retrieve the error messages for a specific form. Simply pass a name as the second argument to `withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');

You may then access the named `MessageBag` instance from the `$errors` variable:

    {{ $errors->login->first('email') }}

#### After Validation Hook

The validator also allows you to attach callbacks to be run after validation is completed. This allows you to easily perform further validation and even add more error messages to the message collection. To get started, use the `after` method on a validator instance:

    $validator = Validator::make(...);

    $validator->after(function($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="form-request-validation"></a>
### Form Request Validation

For more complex validation scenarios, you may wish to create a "form request". Form requests are custom request classes that contain validation logic. To create a form request class, use the `make:request` Artisan CLI command:

    php artisan make:request StoreBlogPostRequest

The generated class will be placed in the `app/Http/Requests` directory. Let's add a few validation rules to the `rules` method:

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

So, how are the validation rules evaluated? All you need to do is type-hint the request on your controller method. The incoming form request is validated before the controller method is called, meaning you do not need to clutter your controller with any validation logic:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPostRequest  $request
     * @return Response
     */
    public function store(StoreBlogPostRequest $request)
    {
        // The incoming request is valid...
    }

If validation fails, a redirect response will be generated to send the user back to their previous location. The errors will also be flashed to the session so they are available for display. If the request was an AJAX request, a HTTP response with a 422 status code will be returned to the user including a JSON representation of the validation errors.

#### Authorizing Form Requests

The form request class also contains an `authorize` method. Within this method, you may check if the authenticated user actually has the authority to update a given resource. For example, if a user is attempting to update a blog post comment, do they actually own that comment? For example:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $commentId = $this->route('comment');

        return Comment::where('id', $commentId)
                      ->where('user_id', Auth::id())->exists();
    }

Note the call to the `route` method in the example above. This method grants you access to the URI parameters defined on the route being called, such as the `{comment}` parameter in the example below:

    Route::post('comment/{comment}');

If the `authorize` method returns `false`, a HTTP response with a 403 status code will automatically be returned and your controller method will not execute.

If you plan to have authorization logic in another part of your application, simply return `true` from the `authorize` method:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

#### Customizing The Flashed Error Format

If you wish to customize the format of the validation errors that are flashed to the session when validation fails, override the `formatErrors` on your base request (`App\Http\Requests\Request`). Don't forget to import the `Illuminate\Contracts\Validation\Validator` class at the top of the file:

    /**
     * {@inheritdoc}
     */
    protected function formatErrors(Validator $validator)
    {
        return $validator->errors()->all();
    }

#### Customizing The Error Messages

You may customize the error messages used by the form request by overriding the `messages` method. This method should return an array of attribute / rule pairs and their corresponding error messages:

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="working-with-error-messages"></a>
## Working With Error Messages

After calling the `errors` method on a `Validator` instance, you will receive an `Illuminate\Support\MessageBag` instance, which has a variety of convenient methods for working with error messages.

#### Retrieving The First Error Message For A Field

To retrieve the first error message for a given field, use the `first` method:

    $messages = $validator->errors();

    echo $messages->first('email');

#### Retrieving All Error Messages For A Field

If you wish to simply retrieve an array of all of the messages for a given field, use the `get` method:

    foreach ($messages->get('email') as $message) {
        //
    }

#### Retrieving All Error Messages For All Fields

To retrieve an array of all messages for all fields, use the `all` method:

    foreach ($messages->all() as $message) {
        //
    }

#### Determining If Messages Exist For A Field

    if ($messages->has('email')) {
        //
    }

#### Retrieving An Error Message With A Format

    echo $messages->first('email', '<p>:message</p>');

#### Retrieving All Error Messages With A Format

    foreach ($messages->all('<li>:message</li>') as $message) {
        //
    }

<a name="custom-error-messages"></a>
### Custom Error Messages

If needed, you may use custom error messages for validation instead of the defaults. There are several ways to specify custom messages. First, you may pass the custom messages as the third argument to the `Validator::make` method:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

In this example, the `:attribute` place-holder will be replaced by the actual name of the field under validation. You may also utilize other place-holders in validation messages. For example:

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### Specifying A Custom Message For A Given Attribute

Sometimes you may wish to specify a custom error messages only for a specific field. You may do so using "dot" notation. Specify the attribute's name first, followed by the rule:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### Specifying Custom Messages In Language Files

In many cases, you may wish to specify your attribute specific custom messages in a language file instead of passing them directly to the `Validator`. To do so, add your messages to `custom` array in the `resources/lang/xx/validation.php` language file.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

<a name="available-validation-rules"></a>
## Available Validation Rules

Below is a list of all available validation rules and their function:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">
[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Before (Date)](#rule-before)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[Image (File)](#rule-image)
[In](#rule-in)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Max](#rule-max)
[MIME Types (File)](#rule-mimes)
[Min](#rule-min)
[Not In](#rule-not-in)
[Numeric](#rule-numeric)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)
</div>

<a name="rule-accepted"></a>
#### accepted

The field under validation must be _yes_, _on_, _1_, or _true_. This is useful for validating "Terms of Service" acceptance.

<a name="rule-active-url"></a>
#### active_url

The field under validation must be a valid URL according to the `checkdnsrr` PHP function.

<a name="rule-after"></a>
#### after:_date_

The field under validation must be a value after a given date. The dates will be passed into the `strtotime` PHP function:

    'start_date' => 'required|date|after:tomorrow'

Instead of passing a date string to be evaluated by `strtotime`, you may specify another field to compare against the date:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-alpha"></a>
#### alpha

The field under validation must be entirely alphabetic characters.

<a name="rule-alpha-dash"></a>
#### alpha_dash

The field under validation may have alpha-numeric characters, as well as dashes and underscores.

<a name="rule-alpha-num"></a>
#### alpha_num

The field under validation must be entirely alpha-numeric characters.

<a name="rule-array"></a>
#### array

The field under validation must be a PHP `array`.

<a name="rule-before"></a>
#### before:_date_

The field under validation must be a value preceding the given date. The dates will be passed into the PHP `strtotime` function.

<a name="rule-between"></a>
#### between:_min_,_max_

The field under validation must have a size between the given _min_ and _max_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-boolean"></a>
#### boolean

The field under validation must be able to be cast as a boolean. Accepted input are `true`, `false`, `1`, `0`, `"1"`, and `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

The field under validation must have a matching field of `foo_confirmation`. For example, if the field under validation is `password`, a matching `password_confirmation` field must be present in the input.

<a name="rule-date"></a>
#### date

The field under validation must be a valid date according to the `strtotime` PHP function.

<a name="rule-date-format"></a>
#### date_format:_format_

The field under validation must match the given _format_. The format will be evaluated using the PHP `date_parse_from_format` function. You should use **either** `date` or `date_format` when validating a field, not both.

<a name="rule-different"></a>
#### different:_field_

The field under validation must have a different value than _field_.

<a name="rule-digits"></a>
#### digits:_value_

The field under validation must be _numeric_ and must have an exact length of _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

The field under validation must have a length between the given _min_ and _max_.

<a name="rule-email"></a>
#### email

The field under validation must be formatted as an e-mail address.

<a name="rule-exists"></a>
#### exists:_table_,_column_

The field under validation must exist on a given database table.

#### Basic Usage Of Exists Rule

    'state' => 'exists:states'

#### Specifying A Custom Column Name

    'state' => 'exists:states,abbreviation'

You may also specify more conditions that will be added as "where" clauses to the query:

    'email' => 'exists:staff,email,account_id,1'

You may also pass `NULL` or `NOT_NULL` to the "where" clause:

    'email' => 'exists:staff,email,deleted_at,NULL'

    'email' => 'exists:staff,email,deleted_at,NOT_NULL'

<a name="rule-image"></a>
#### image

The file under validation must be an image (jpeg, png, bmp, gif, or svg)

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

The field under validation must be included in the given list of values.

<a name="rule-integer"></a>
#### integer

The field under validation must be an integer.

<a name="rule-ip"></a>
#### ip

The field under validation must be an IP address.

<a name="rule-json"></a>
#### json

The field under validation must be a valid JSON string.

<a name="rule-max"></a>
#### max:_value_

The field under validation must be less than or equal to a maximum _value_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

The file under validation must have a MIME type corresponding to one of the listed extensions.

#### Basic Usage Of MIME Rule

    'photo' => 'mimes:jpeg,bmp,png'

Even though you only need to specify the extensions, this rule actually validates against the MIME type of the file by reading the file's contents and guessing its MIME type.

A full listing of MIME types and their corresponding extensions may be found at the following location: [http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

The field under validation must have a minimum _value_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

The field under validation must not be included in the given list of values.

<a name="rule-numeric"></a>
#### numeric

The field under validation must be numeric.

<a name="rule-regex"></a>
#### regex:_pattern_

The field under validation must match the given regular expression.

**Note:** When using the `regex` pattern, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character.

<a name="rule-required"></a>
#### required

The field under validation must be present in the input data and not empty. A field is considered "empty" is one of the following conditions are true:

- The value is `null`.
- The value is an empty string.
- The value is an empty array or empty `Countable` object.
- The value is an uploaded file with no path.

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

The field under validation must be present if the _anotherfield_ field is equal to any _value_.

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

The field under validation must be present unless the _anotherfield_ field is equal to any _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

The field under validation must be present _only if_ any of the other specified fields are present.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

The field under validation must be present _only if_ all of the other specified fields are present.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

The field under validation must be present _only when_ any of the other specified fields are not present.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

The field under validation must be present _only when_ all of the other specified fields are not present.

<a name="rule-same"></a>
#### same:_field_

The given _field_ must match the field under validation.

<a name="rule-size"></a>
#### size:_value_

The field under validation must have a size matching the given _value_. For string data, _value_ corresponds to the number of characters. For numeric data, _value_ corresponds to a given integer value. For files, _size_ corresponds to the file size in kilobytes.

<a name="rule-string"></a>
#### string

The field under validation must be a string.

<a name="rule-timezone"></a>
#### timezone

The field under validation must be a valid timezone identifier according to the `timezone_identifiers_list` PHP function.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

The field under validation must be unique on a given database table. If the `column` option is not specified, the field name will be used.

**Specifying A Custom Column Name:**

    'email' => 'unique:users,email_address'

**Custom Database Connection**

Occasionally, you may need to set a custom connection for database queries made by the Validator. As seen above, setting `unique:users` as a validation rule will use the default database connection to query the database. To override this, specify the connection followed by the table name using "dot" syntax:

    'email' => 'unique:connection.users,email_address'

**Forcing A Unique Rule To Ignore A Given ID:**

Sometimes, you may wish to ignore a given ID during the unique check. For example, consider an "update profile" screen that includes the user's name, e-mail address, and location. Of course, you will want to verify that the e-mail address is unique. However, if the user only changes the name field and not the e-mail field, you do not want a validation error to be thrown because the user is already the owner of the e-mail address. You only want to throw a validation error if the user provides an e-mail address that is already used by a different user. To tell the unique rule to ignore the user's ID, you may pass the ID as the third parameter:

    'email' => 'unique:users,email_address,'.$user->id

If your table uses a primary key column name other than `id`, you may specify it as the fourth parameter:

    'email' => 'unique:users,email_address,'.$user->id.',user_id'

**Adding Additional Where Clauses:**

You may also specify more conditions that will be added as "where" clauses to the query:

    'email' => 'unique:users,email_address,NULL,id,account_id,1'

In the rule above, only rows with an `account_id` of `1` would be included in the unique check.

<a name="rule-url"></a>
#### url

The field under validation must be a valid URL according to PHP's `filter_var` function.

<a name="conditionally-adding-rules"></a>
## Conditionally Adding Rules

In some situations, you may wish to run validation checks against a field **only** if that field is present in the input array. To quickly accomplish this, add the `sometimes` rule to your rule list:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

In the example above, the `email` field will only be validated if it is present in the `$data` array.

#### Complex Conditional Validation

Sometimes you may wish to add validation rules based on more complex conditional logic. For example, you may wish to require a given field only if another field has a greater value than 100. Or, you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your _static rules_ that never change:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game re-sell shop, or maybe they just enjoy collecting. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

    $v->sometimes('reason', 'required|max:500', function($input) {
        return $input->games >= 100;
    });

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is the rules we want to add. If the `Closure` passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

    $v->sometimes(['reason', 'cost'], 'required', function($input) {
        return $input->games >= 100;
    });

> **Note:** The `$input` parameter passed to your `Closure` will be an instance of `Illuminate\Support\Fluent` and may be used to access your input and files.

<a name="custom-validation-rules"></a>
## Custom Validation Rules

Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using the `extend` method on the `Validator` [facade](/docs/{{version}}/facades). Let's use this method within a [service provider](/docs/{{version}}/providers) to register a custom validation rule:

    <?php

    namespace App\Providers;

    use Validator;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

The custom validator Closure receives four arguments: the name of the `$attribute` being validated, the `$value` of the attribute, an array of `$parameters` passed to the rule, and the `Validator` instance.

You may also pass a class and method to the `extend` method instead of a Closure:

    Validator::extend('foo', 'FooValidator@validate');

#### Defining The Error Message

You will also need to define an error message for your custom rule. You can do so either using an inline custom message array or by adding an entry in the validation language file. This message should be placed in the first level of the array, not within the `custom` array, which is only for attribute-specific error messages:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

When creating a custom validation rule, you may sometimes need to define custom place-holder replacements for error messages. You may do so by creating a custom Validator as described above then making a call to the `replacer` method on the `Validator` facade. You may do this within the `boot` method of a [service provider](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### Implicit Extensions

By default, when an attribute being validated is not present or contains an empty value as defined by the [`required`](#rule-required) rule, normal validation rules, including custom extensions, are not run. For example, the [`integer`](#rule-integer) rule will not be run against a `null` value:

    $rules = ['count' => 'integer'];

    $input = ['count' => null];

    Validator::make($input, $rules)->passes(); // true

For a rule to run even when an attribute is empty, the rule must imply that the attribute is required. To create such an "implicit" extension, use the `Validator::extendImplicit()` method:

    Validator::extendImplicit('foo', function($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> **Note:** An "implicit" extension only _implies_ that the attribute is required. Whether it actually invalidates a missing or empty attribute is up to you.
