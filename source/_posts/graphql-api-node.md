---
layout: post
title:  "Basico de uma API com GraphQL e Node.js"
date:   2016-06-17 00:16:00
categories:
- graphq
tags:
- api
- nodejs
---
GraphQL é uma linguagem de consulta de dados desenvolvida e usado pelo Facebook para realizar requisições e entregar informações para aplicações web e mobile desde 2012. GraphQL facilita o processo de entregar ao client apenas o que foi requisitado pelo mesmo e na ordem em que foi requisitado.

*Exemplo*:

Se temos um objeto `User`:
{% codeblock lang:javascript %}
{
	"name": "Joao",
	"full_name": "Joao Paulo",
	"age": 22,
	"city": "Campina Grande",
	"tag": "ribeirojpn",
	"url": "https://twitter.com/ribeirojpn",
	"knowledge": [
		{
			"language": "Javascript",
			"frameworks": ["express.js", "hapi.js", "AngularJS"]
		},
		{
			"language": "Java",
			"frameworks": ["Play"]
		},
		{
			"language": "Python",
			"frameworks": []
		}
	]
}

{% endcodeblock %}

Podemos solicitar apenas os dados `name`, `age`, `city` e as linguagens que o usuário possui conhecimento(`knowledge.language`) ao fazer a seguinte requisição para o servidor:

{% codeblock lang:javascript %}
query {
	user(name:"Joao") {
		name
		age
		city
		knowledge {
			language
		}
	}
}

// Retornando:
{
  "data": {
    "user": {
      "name": "Joao",
      "age": 22,
      "city": "Campina Grande",
      "knowledge": [
        {
          "language": "Javascript"
        },
        {
          "language": "Java"
        },
        {
          "language": "Python"
        }
      ]
    }
  }
}
{% endcodeblock %}

Se eu também quiser que retorne os frameworks basta adicionar `frameworks` dentro do bloco de `knowledge`, ficando assim:

{% codeblock lang:javascript %}
query {
	user(name:"Joao") {
		name
		age
		city
		knowledge {
			language
			frameworks
		}
	}
}

// Retornando:
{
  "data": {
    "user": {
      "name": "Joao",
      "age": 22,
      "city": "Campina Grande",
      "knowledge": [
        {
          "language": "Javascript",
          "frameworks": ["express.js","hapi.js","AngularJS"]
        },
        {
          "language": "Java",
          "frameworks": ["Play"]
        },
        {
          "language": "Python",
          "frameworks": []
        }
      ]
    }
  }
}
{% endcodeblock %}

Essas requisições podem ser feitas pelo client utilizando o método `GET`, a url da requisição na rota `/graphql` do exemplo acima seria igual a:

{% codeblock lang:javascript %}
/graphql?query={user(name:"user"){name,age,city,knowledge{language,frameworks}}}
{% endcodeblock %}

> **OBS**: GraphQL ainda é uma novidade e não achei muito sobre boas praticas. Assim que ficar sabendo mais a respeito, postarei aqui. ^^

## Usando GraphQL no Node.js

Vamos usar o [graphql-js](https://github.com/graphql/graphql-js) e o [express-graphql](https://github.com/graphql/express-graphql) para criarmos uma 'mini-API'.

 - graphql-js é a implementação do GraphQL no javascript
 - express-graphql é um middleware que costumiza o sevidor HTTP do express para ter suporte ao estilo do GraphQL

``` bash
$ npm init
$ npm install express graphql express-graphql --save
```

Usaremos os dados abaixo em um arquivo chamado `users.json`:
{% codeblock lang:javascript %}
[
	{
		"id":1,
		"name": "Joao",
		"age": 22,
		"knowledge": [
			{
				"language": "Javascript",
				"frameworks": ["express.js", "hapi.js", "AngularJS"]
			},
			{
				"language": "Java",
				"frameworks": ["Play"]
			},
			{
				"language": "Python",
				"frameworks": []
			}
		]
	},
	{
		"id":2,
		"name": "Maria",
		"age": 20,
		"knowledge": [
			{
				"language": "Javascript",
				"frameworks": ["ReactJS", "AngularJS"]
			},
			{
				"language": "Java",
				"frameworks": ["Play", "Spring"]
			},
			{
				"language": "Python",
				"frameworks": ["Django"]
			}
		]
	},
	{
		"id":3,
		"name": "Ana",
		"age": 20,
		"knowledge": [
			{
				"language": "Javascript",
				"frameworks": ["ReactJS", "express.js"]
			},
			{
				"language": "Ruby",
				"frameworks": ["Ruby on Rails"]
			},
			{
				"language": "Python",
				"frameworks": ["Django"]
			}
		]
	}
]
{% endcodeblock %}

Criaremos um arquivo chamado `schema.js` com o seguinte código:

{% codeblock lang:javascript %}
const graphql = require('graphql')
const users = require('./user.json')

let knowledgeType = new graphql.GraphQLObjectType({
  name:'Knowledge',
  fields: {
		language: { type: graphql.GraphQLString },
		frameworks: { type: new graphql.GraphQLList(graphql.GraphQLString ) }
  }
})

let userType = new graphql.GraphQLObjectType({
  name: 'User',
	fields: {
		id: { type: new graphql.GraphQLNonNull(graphql.GraphQLInt) },
		name: { type: new graphql.GraphQLNonNull(graphql.GraphQLString) },
		full_name: { type: graphql.GraphQLString },
		age: { type: graphql.GraphQLInt },
		city: { type: graphql.GraphQLString },
		tag: { type: graphql.GraphQLString },
		url: { type: graphql.GraphQLString },
		knowledge: { type: new graphql.GraphQLList(knowledgeType) }
	}
})

let schema = new graphql.GraphQLSchema({
	query: new graphql.GraphQLObjectType({
    name: 'Query',
    fields: {
		  user: {
				type: userType,
      	args: {
          id:{
            type: graphql.GraphQLInt
          }
        },
        resolve: function (_ , args) {
					let response = users.find(function (user){
						return (user.id === args.id)
					})
          return response
        }
			}
		}
	})
})

module.exports = schema
{% endcodeblock %}

No código acima criamos dois [`GraphQLObjectType`](http://graphql.org/docs/api-reference-type-system/#graphqlobjecttype) chamados de `userType` e `knowledgeType`, eles funcionam como 'models' para o `schema`. Precisamos criar um tipo especifico para 'knowledge' para que possamos solicitar apenas parte dos dados desse campo, como no exemplo anterior.

O `schema` é um objeto do tipo [`GraphQLSchema`](http://graphql.org/docs/api-reference-type-system/#graphqlschema) que recebe um `GraphQLObjectType` com as propriedades `name` e `fields`. Em `fields` ficam os métodos que serão usados na query. No nosso caso, temos `user` que vai usar o `userType` passado na propriedade `type` como tipo de dado a ser manipulado e em `args` passamos os argumentos que este método deve receber para realizar a operação como propriedades que recebem de qual tipo esse argumento será, no caso acima temos o argumento `id` que será do tipo `int`. A propriedade `resolve` de `user` recebe uma função, está função será responsável por buscar e retornar o usuário que possui a `id` passada como argumento.

Agora, criaremos o arquivo `index.js` onde iniciaremos o servidor e determinaremos a rota que usará o `schema` que acabamos de criar. O código de `index.js` será este:

{% codeblock lang:javascript %}
const graphql = require('graphql')
const graphqlHTTP = require('express-graphql')
const express = require('express')

const users = require('./schema')

const app = express()

app.use('/user', graphqlHTTP({schema:users, pretty: true}))

app.listen(3000, function () {
  console.log('Server on.')
})
{% endcodeblock %}

Aqui iniciamos o servidor com express na porta `3000` e criamos uma rota para `/user` passando a função `graphqlHTTP({schema:users, pretty: true})`
para ser chamado sempre que for realizada uma requisição nessa rota. Foi passado um objeto com a propriedade `schema` com o schema que criamos em `schema.js` como valor dessa propriedade. Essa função se responsabiliza por verificar a requisição e executar a `resolve` referente a propriedade passada na requisição.

Inicie o servidor no seu terminal:
``` bash
node index.js
```

E acesse a seguinte url no seu browser:

``` bash
http://localhost:3000/user?query={user(id:2){id,name,age,knowledge{language,frameworks}}}
```

Você deve receber isso:

{% codeblock lang:javascript %}
{
  "data": {
    "user": {
      "id": 2,
      "name": "Maria",
      "age": 20,
      "knowledge": [
        {
          "language": "Javascript",
          "frameworks": [
            "ReactJS",
            "AngularJS"
          ]
        },
        {
          "language": "Java",
          "frameworks": [
            "Play",
            "Spring"
          ]
        },
        {
          "language": "Python",
          "frameworks": [
            "Django"
          ]
        }
      ]
    }
  }
}
{% endcodeblock %}

Experimente variar as propriedades solicitadas e veja o que retorna:
```
http://localhost:3000/user?query={user(id:2){id,name,age,knowledge{language,frameworks}}}
http://localhost:3000/user?query={user(id:3){id,name,age,knowledge{language}}}
http://localhost:3000/user?query={user(id:1){id,knowledge{language,frameworks}}}
http://localhost:3000/user?query={user(id:1){id,name,age}}
http://localhost:3000/user?query={user(id:2){id,name,age,knowledge{}}}
```

A Propriedade `query` em `schema` recebe em `fields` os métodos que serão chamados em requisições `GET`, como no nosso caso `user` que ira retornar o usuário através da função declarada na propriedade de `user` chamada `resolve`.

Mas e pra retornar a lista com todos os usuários? Simples, basta adicionarmos o código abaixo como outra propriedade de `fields`:
{% codeblock lang:javascript %}
users: {
	type: new graphql.GraphQLList(userType),
	resolve: function (_ , args) {
		return users
	}
}
{% endcodeblock %}

Assim passamos a ter dois métodos de query no schema, `user` que retorna o usuário com a id passada como argumento e `users` que retorna a lista com todos os usuários. Experimente:

```
http://localhost:3000/user?query={users{id,name,age,knowledge{language,frameworks}}}
```

Essa url tem uma query no seguinte formato:
{% codeblock lang:javascript %}
query {
	users {
		id
		name
		age
		knowledge {
			language
			frameworks
		}
	}
}
{% endcodeblock %}

Ficou faltando falar de criar, atualizar e deletar dados, certo? não se preocupe, farei outro post falando sobre isso porque este já ficou bem extenso. Até mais!

**Fontes:**

 - [http://graphql.org/](http://graphql.org/)
 - [https://code.facebook.com/posts/1691455094417024/graphql-a-data-query-language/](https://code.facebook.com/posts/1691455094417024/graphql-a-data-query-language/)
