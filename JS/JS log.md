### Mer. 11 Septembre 2019

#### Types en JS :
1. Types primitifs :
  * undefined
  * null
  * boolean
  * string
  * number
  * object
2. Objets spéciaux :
  * function
  * array
  * regexp
  
__Object__ = key/value pair.

Quand on demande une propriété non-existante d'un objet, un "undefined" est renvoyé.

Quand on définit une fonction, JS crée pour nous un objet qui a 3 propriétés : _name_, _length_ et _prototype_.

à propos du __this__. On peut forcer le __this__ à être dans tel ou tel scope à l'aide de la méthode __call__.

On étudie les [Promesses](quiver:///notes/1B7101EF-46EF-484A-A648-918AB7923C36).


## Mar. 10 Septembre 2019

Voir les slides sur le moodle.

Voir les [nouvelle features de ES6](http://es6-features.org/#Constants)

#### Différences let / var / const

var = variable globale
let = portée fonction
const = même portée, mais pour une constante.
__note__ : on peut faire _muter_ les propriétés d'un objet, malgré une déclaration "const".

#### Templates Strings

On peut désormais utiliser les backticks pour délimiter des __template strings__.
Permet de :
* faire des retours à la ligne directement dans la string
* [interpolation](https://putaindecode.io/articles/es6-es2015-les-template-strings/) : affichage des expressions ${expression}
* fonctions fléchées (ou _lambda_)

```js
myMethod.call(object1)
```



```js
function multiplier2 (nombre) {
	return nombre * 2;
}

const multiplier2 = function (nombre) {
	return nombre * 2;
};

const multiplier2 = (nombre) => nombre * 2;
```

Important : Une arrow function n’a pas de contexte propre et donc ne déclare pas de this.

```js
function saluer() {
	console.log(this);
}

saluer(); // Window {}

saluer.call("Hey"); // "Hey"
// On ne pourrait pas faire ça avec une fonction fléchée. On ne peut pas attribuer un "this" à la volée à une fonction fléchée.
```

```js
const chronometre = {
    compteur : 0,
    demarrer : function() {
        setInterval(function() {
            this.compteur++;
            console.log(this.compteur);
        }, 1000);
    }
};

chronometre.demarrer();

```

* affectation par décomposition

L'affectation par décomposition est une opération qui rend possible l'extraction de variables depuis des tableaux ou des objets. Il s'agit d'une technique très pratique permettant d'extraire rapidement et sans fioritures les données nécessaires d'une structure complexe.

```js
    let numbers = [1, 2, 3];
    let [foo, bar] = numbers;

    console.log(foo); // 1
    console.log(bar); // 2
  
  /////
  
    let numbers = [1, 2, 3];
    let data = {};
    [data.foo, data.bar] = numbers;

    console.log(data); // { foo: 1, bar: 2 }
    
  /////
  
    let numbers = [1, 2, 3];
    let [foo, , baz] = numbers; // Si je n'ai pas besoin du deuxième élément

    console.log(foo); // 1
    console.log(baz); // 3
```

#### DÉCOMPOSITION

ES6 fournit deux nouveaux concepts étroitement liés : le rest et le spread (« décomposition »), pour les fonctions dont le nombre d’arguments varie. Dans cet exercice, nous allons explorer la partie décomposition.

Un exemple classique de fonction variadique (dont le nombre d’arguments varie) est Math.max, qu’on peut appeler avec un nombre quelconque d’arguments : Math.max(1, 2) ou Math.max(1, 2, 3) ou…

En ES6, vous pouvez utiliser la syntaxe ...args pour « décomposer » un tableau en valeurs individuelles, notamment quand vous appelez ce genre de fonction. Par exemple :

```js
    var numbers = [1, 1, 2, 3, 5, 8];
    var max = Math.max(...numbers);
```

