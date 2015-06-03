# Hashing

- [Introdução](#introducao)
- [Como utilizar](#como-utilizar)

<a name="introducao"></a>
## Introdução

O facade `Hash` do Laravel fornece criptografia segura Bcrypt para gravar as senhas dos usuários. Se você estiver usando o controller `AuthController` que vem incluso na sua aplicação Laravel, ele irá cuidar de verificar a criptografia da senha Bcrypt com uma senha não criptografada fornecida pelo usuário.

Da mesma forma, o serviço `Registrar` que vêm no Laravel faz a chamada `bcrypt` apropriada para criptografar senhas para serem salvas.

<a name="como-utilizar"></a>
## Como utilizar

#### Criptografando uma senha usando Bcrypt

	$senhaCriptografada = Hash::make('segredo');

Você também pode usar o helper `bcrypt`:

	$senhaCriptografada = bcrypt('segredo');

#### Comparando uma senha descriptografada 

	if (Hash::check('segredo', $senhaCriptografada))
	{
		// As senhas batem...
	}

#### Verificando se uma senha precisa ser re-criptografada

	if (Hash::needsRehash($senha))
	{
		$senha = Hash::make($senha);
	}
