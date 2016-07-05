---
layout: post
title:  "Variáveis, Hoisting, Closures e IIFE em Javascript"
date:   2015-11-15 15:56:12
categories:
 - javascript
tags:
 - javascript
 - learn

---
Nesse artigo irei abordar rapidamente alguns conceitos básicos mas essenciais da linguagem Javascript. Vou apresentar 5 conceitos muito importantes sobre como criar e instanciar variáveis e funções, são eles: Hoisting, Closure, Variável Global, Variável por parâmetro e IIFE.

## Hoisting

O Javascript possui o comportamento de mover a declaração de uma variável ou função para o topo do escopo, o que significa que você pode usar uma função ou variável antes dela ter sido declarada. O que acontece é que as declarações de variáveis e funções são processadas antes que qualquer outro código seja executado, transformando este código:

{% codeblock lang:javascript %}

run();

function run() {
  console.log("Run, Forrest, run!");
}

// vai imprimir -> Run, Forrest, run!
{% endcodeblock %}


neste:

{% codeblock lang:javascript %}
function run() {
  console.log("Run, Forrest, run!");
}

run(); // imprime 'Run, Forrest, run!'
{% endcodeblock %}

Isto possibilita que você utilize funções dentro de outras funções mantendo uma organização que mais lhe agrade. Um exemplo que costumo fazer é colocar o `sets` e `gets` sempre como as ultimas funções no arquivo (me acostumei a fazer assim programando em `Java` ^^ ):

{% codeblock lang:javascript %}
var carro = {};

function meuCarro(marca, portas) {
  setMarca(marca);
  setNumeroDePortas(portas);
}

function setMarca(nomeDaMarca) {
  carro.marca = nomeDaMarca;
}

function setNumeroDePortas(numeroDePortas) {
  carro.portas = numeroDePortas;
}

meuCarro("fiat", 4);
console.log(carro.marca); // imprime 'fiat'
{% endcodeblock %}

No caso de variáveis, se você inicializar a mesma após ter sido utilizada, o hoisting acontecerá porem a variável terá valor `undefined` pois no momento da execução a variável já foi declarada, mas ainda não foi inicializada.

{% codeblock lang:javascript %}
console.log(vader); // imprime 'undefined'
vader = "I am your father"
var vader;
{% endcodeblock %}

esse código se transforma neste:

{% codeblock lang:javascript %}
var vader; // variavel declarada mas não inicializada
console.log(vader); // imprime 'undefined'
vader = "I am your father" // inicialização da variável após ter sido usada no log.
{% endcodeblock %}

## Closures

Closures são funções dentro de outras funções que possuem acesso a variaveis da função a qual está inserido. Elas possuem três escopos acessiveis: o proprio escopo, escopo da função em que está inserido e o escopo global. Isto significa que, alem de ter acesso as variaveis declaradas nestes escopos, os Closures tambem possuem acesso aos argumentos passados na função a qual está inserido. Um exemplo pra ficar mais fácil de compreender:

{% codeblock lang:javascript %}
var carro = {};

function meuCarro(nomeDaMarca, numeroDePortas) {
  function setMarca(){
    carro.marca = nomeDaMarca;
  };

  function setNumeroDePortas(){
    carro.portas = numeroDePortas;
  };

  function mostrarCarro(){
    console.log("Carro da " + nomeDaMarca + " de " + numeroDePortas + " portas.");
  }

  setMarca();
  setNumeroDePortas();
  mostrarCarro();
}
meuCarro("fiat",2); // imprime 'Carro da fiat de 2 portas.'
{% endcodeblock %}

Como Javascript não oferece formas de declarar metodos privados como outras linguagens como Java(utilizando `private` na declaração de um metodo), você pode utilizar closures para emular metodos privados. Veja um exemplo utilizando, mais uma vez, o exemplo do 'meuCarro':

{% codeblock lang:javascript %}
var meuCarro = (function() {
  var carro = { velocidade: 0 };

  function vel(valor){
    carro.velocidade += valor;
  };

  return {
    acelerar: function() {
      vel(10);
    },
    reduzir: function() {
      vel(-10);
    },
    velocidadeAtual: function() {
      return carro.velocidade;
    }
  };
})();

console.log(meuCarro.velocidadeAtual()); // imprime '0'
meuCarro.acelerar();
console.log(meuCarro.velocidadeAtual()); // imprime '10'
meuCarro.acelerar();
meuCarro.acelerar();
console.log(meuCarro.velocidadeAtual()); // imprime '30'
meuCarro.reduzir();
console.log(meuCarro.velocidadeAtual()); // imprime '20'
meuCarro.vel(40); // 'TypeError: meuCarro.vel is not a function'
console.log(meuCarro.velocidadeAtual()); // imprime '20'
{% endcodeblock %}

O método `vel(valor)` só pode ser utilizado dentro do escopo de `meuCarro` tornando assim um método privado.

## Variável Global

Variáveis Globais são variáveis que são declaradas fora de uma função. Essas variáveis podem ser acessadas e alteradas dentro de qualquer função, ficando disponível para todo o documento.

{% codeblock lang:javascript %}
var vader = "I am your father" // variavel global
function change(str){
  vader = str;
}

function speak(){
  console.log(vader);
}

speak(); // imprie 'I am your father'
change("Come to the dark side");
speak(); // imprime 'Come to the dark side'
{% endcodeblock %}

Quando iniciamos uma variável dentro de uma função sem declará-la, essa variável se torna uma Variável Global.

{% codeblock lang:javascript %}
function change(str){
  vader = str;
}

function speak(){
  console.log(vader);
}

speak(); // 'ReferenceError: vader is not defined'
change("Come to the dark side");
speak(); // imprime 'Come to the dark side'
{% endcodeblock %}

A ultima chamada da função `speak()` imprime 'Come to the dark side' porque a variável `vader` inicializada em `change(str)` se torna global por passar pelo processo de [Hoisting](#hoisting).

## Variável por Parâmetro

Parâmetros são valores passados como recurso para uma função, são atribuídos a variáveis locais no escopo da função. Quando uma variável global é passada como parâmetro, apenas o valor da variável é recebida pela função.

{% codeblock lang:javascript %}
var x = 10;

function change(num){
  num = 2;
  return num;
};

console.log(change(x)); // imprime 2 retornado pelo função change(x)
console.log(x)/ // imprime 10, valor de x no escopo global

function changeX(x){
  x = 2;
  return x;
}

console.log(change(x)); // passando variavel global como parametro, imprime 2 retornado pela função change(x)
console.log(x)/ // imprime 10, valor de x no escopo global
{% endcodeblock %}

Quando você passa uma Variável Global como parâmetro de uma função o seu valor é atribuído a uma variável local. No segundo exemplo é criada uma variável "x" no escopo local que recebe o valor da variável global "x", quando altero o valor de x dentro de `change(x)` estou alterando apenas o valor da variável local. Para alterar a variável global preciso que não seja criada uma variável com mesmo nome dentro do escopo local:

{% codeblock lang:javascript %}
var x = 10;
var newValue = 5;

function change(num){
  x = num;
  return x;
};

change(newValue); // retorna 5
console.log(x)/ // imprime 5, novo valor de x no escopo global
{% endcodeblock %}

## Instanciação usando uma IIFE

IIFE é uma sigla para "Immediately-Invoked Function Expression" que em português
pode ser chamado de "Definição de Função Imediatamente Executável". São funções que são executadas no mesmo momento em que são definidas. Utilizar IIFE pode ser muito útil quando não queremos que códigos javascript de outras pessoas acessem ou alterem as variáveis que definimos.

Exemplo:
{% codeblock lang:javascript %}
(function (str) {
  var vader = str;
  console.log(vader);
}('I need a friend...')); // imprime 'I need a friend...'

console.log(vader); // ReferenceError: vader is not defined
{% endcodeblock %}

Uma IIFE é declarada inserindo uma função, variável ou operação dentro de parenteses como parâmetros para uma função, que automaticamente retornara o que está entre os parenteses.

{% codeblock lang:javascript %}
(function () {}) // retorna a function() dentro dos parenteses
(function () {})() // executa a function() dentro dos parenteses
(4*5) // retorna 20
(true) // retorna true
{% endcodeblock %}

Em um dos exemplos de [Closures](#closures) utilizei uma IIFE para atribuir uma função autoexecutável a uma variável.
