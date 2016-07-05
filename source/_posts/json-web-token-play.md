---
layout: post
title:  "Json Web Token no Play Framework(Java)"
date:   2016-05-24 21:16:01
categories:
- java
tags:
- java
- jwt
---
Trabalhando em um projeto da universidade fiquei responsável por desenvolver a autenticação de usuários por token em uma REST API. Escolhi utilizar o Json Web Token para gerar os tokens quando o usuário se conecta ou se registra no sistema. Esses tokens devem ser passados em todas as requisições feitas pelo usuário ao sistema e devem ter validade de 24h.

## Instalando o Java JWT

Utilizei a biblioteca Java JWT para criar e verificar os JWTs. Para instalar no Play Framework é bem simples, basta adicionar a biblioteca como dependência no `build.sbt`:
{% codeblock lang:java %}
libraryDependencies += "io.jsonwebtoken" % "jjwt" % "0.5"
{% endcodeblock %}
Gerando o token com o Java JWT
Utilizei métodos estáticos da classe `Jwts` do pacote `io.jsonbwetoken` para configurar e gerar o token.
{% codeblock lang:java %}
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.impl.crypto.MacProvider;
import java.security.Key;

Key key = MacProvider.generateKey();

Date dateNow = new Date();
Date expires = new Date(dateNow.getTime() + 86400000);

String email = "user@email.com";

String token = Jwts.builder().setSubject(email)
                             .signWith(SignatureAlgorithm.HS256, key)
                             .setExpiration(expires).compact();
{% endcodeblock %}

Comecei a configuração determinando o `subject`, que é a informação que será codificada, passando o email do usuário em `setSubject(email)`. Determinei a data de validade passando um objeto do tipo `Date` em `setExpiration(date)` e em `signWith(algoritmo, key)` passei qual algoritmo deve ser usado, nesse caso utilizei `HS256`(você pode ver a lista de algoritmos que a biblioteca suporta aqui), e a chave de assinatura do app( que foi gerado por `MacProvider.generateKey()`). Para gerar a String com o valor do token usei o metodo `.compact()` que gera algo parecido com isso:

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4
gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

## Recuperando a informação que foi codificada

Para recuperar a informação que foi codificada também utilizei os métodos estáticos da classe `Jwts`.

{% codeblock lang:java %}
String email = Jwts.parser().setSigningKey(key).parseClaimsJws(token).getBody().getSubject();
{% endcodeblock %}

Em `setSigningKey(key)` passei a mesma key que utilizei para gerar o token e em `parseClaimsJws(token)` passei a String com o token. O método `getBody()` vai retornar um objeto como todos os dados token e utilizei `setSubject()` para receber a informação que foi codificada, nesse caso o email do usuário.

> Se tentar recuperar a informação após a data de validade do token será retornado uma Exception informando que a o token expirou. Certifique-se que está tratando isto.

## Utilizando Cookies para salvar o token gerado no navegador do usuário.

Utilizei Cookies para salvar o token no navegador. Após o cadastro ou login de um usuário o token é enviado para ser armazenado em um cookie no navegador. Sempre que o usuário enviar uma requisição para o servidor o token armazenado é enviado na requisição. Enviei o token atráves da URL(como `/?t=yJhbGciOiJIUzI1NiIsI…` ou `/:token`) e valido a requisição verificando se o token ainda é valido.
Para enviar um token para ser salvo nos cookies do navegador você pode fazer assim:

{% codeblock lang:java %}
response().setCookie(Http.Cookie.builder("authToken", token).build());
return ok(Json.toJson("Token armazenado nos Cookies em chave valor authToken:" + token));
{% endcodeblock %}

Para receber o token de uma requisição enviada pelo usuário, pegando do `HEADER` da requisição:

{% codeblock lang:java %}
request().cookies().get("authToken").value();
{% endcodeblock %}

Se você passou por URL, utilize o método de recuperar informação referente ao método que você utilizou para passar o token.
Para remover um token dos cookies:

{% codeblock lang:java %}
response().discardCookie("authToken");
return ok(Json.toJson("Token removido dos cookies"));
{% endcodeblock %}

Código de exemplo de algumas operações utilizando Token e Cookie:

{% codeblock lang:java %}
public static final String AUTH_TOKEN = "authToken";
Key key = MacProvider.generateKey();

public Result register(String email){
	Date dateNow = new Date();
	Date expires = new Date(dateNow.getTime() + 86400000);
	try {
		String token = Jwts.builder().setSubject(email).signWith(SignatureAlgorithm.HS256, key).setExpiration(expires).compact();
		response().setCookie(Http.Cookie.builder(AUTH_TOKEN, token).build());
		return ok(Json.toJson("Usuario criado com sucesso"));
	} catch (Exception e) {
		Logger.info("400 - " + e.getMessage());
		return status(400, e.getMessage());
	}
}

// Se a validade do token tiver expirado será lançado uma 'exception' e o metodo ira retornar um '401'
public Result loginWithToken(String token){
	try {
		String email = Jwts.parser().setSigningKey(key).parseClaimsJws(token).getBody().getSubject();
		return ok(Json.toJson(email));
	} catch (Exception e){
		response().discardCookie(AUTH_TOKEN);
		return status(401, e.getMessage());
	}
}

public Result logout(){
	response().discardCookie(AUTH_TOKEN);
	return redirect("/");
}

// para url com '?t=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gR...'
public Result getList() {
	String token = request().getQueryString("t");
	try {
		String userEmail = Jwts.parser().setSigningKey(key).parseClaimsJws(token).getBody().getSubject();
		List list = getListByEmail(userEmail);
		return ok(Json.toJson(list));
	} catch (Exception e) {
		Logger.info("400 - " + e.getMessage());
		return status(400, e.getMessage());
	}
}
{% endcodeblock %}
