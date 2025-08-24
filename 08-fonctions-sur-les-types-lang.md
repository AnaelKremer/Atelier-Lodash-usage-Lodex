# Fonctions sur les types (lang)

La section Lang de **Lodash** regroupe les fonctions dites utilitaires de langage.  
Elles ne transforment pas directement des chaînes, des tableaux ou des objets, mais permettent de tester, comparer et convertir des types de données.
C’est un socle indispensable pour écrire des transformations robustes et éviter les quelques pièges qui peuvent exister.  

---

Parmi celles-ci il existe plusieurs fonctions qui permettent de tester le type d'une valeur (*.isX*) et qui ont souvent leur pendant pour convertir une valeur dans un type donnée (*.toX*).  
Malheureusement il n'existe pas de fonction unique pour renvoyer le type de la valeur testée. Pour ce faire il faut utiliser une fonction **JavaScript**, `typeof` en l'occurence.

```js
value = get("value.entree").thru(valeur => typeof(valeur))
// Entrée : "texte" → Sortie : "string"
// Entrée : true → Sortie : "boolean"
// Entrée : 42 → Sortie : "number"
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : "object"
// Entrée : [1, 2, 3] → Sortie : "object"
// Entrée : null → Sortie : "object"
```

> [!WARNING]
> On s'aperçoit que les trois dernières sorties sont *"object"* alors que l'on a bien un **objet**, mais aussi un **tableau** ainsi qu'un ***null***...  
> Ceci s'explique par le fait qu'**en JavaScript, un tableau est en réalité un objet particulier** dont les clés sont des nombres (0, 1, 2, …) servant d’index.  
>  
> En JavaScript, ***null*** est une **valeur primitive** représentant l’absence volontaire de valeur mais dont `typeof` renvoie pourtant *"object"*.  
>
> Il s’agit en fait d’un **bug historique** : dans les premières versions du langage, les valeurs étaient codées avec des tags binaires en mémoire. 
> Le tag 000 identifiait les objets, et la valeur ***null***, stockée comme 0x00 (adresse nulle, pour représenter l’absence de valeur), a été interprétée à tort comme un objet. 
> Ce bug n'a jamais été corrigé afin de ne pas casser tout le code existant. Pour en savoir plus, vous pouvez aller [ici](https://2ality.com/2013/10/typeof-null.html), [là](https://262.ecma-international.org/5.1/#sec-11.4.3) ou [là](https://medium.com/%40AlexanderObregon/the-real-reason-javascript-typeof-null-returns-object-f41d39c9fe5b).  

> [!NOTE]  
> On pourrait également objecter (😁) qu'***undefined*** n'a pas été testé. C'est parce qu'un JSON ne peut pas contenir directement ***undefined***, c'est un concept **JavaScript**.
> Pour ce faire il faut provoquer une erreur en accédant à une clé qui n'existe pas.
> ```js
> value = get("value.entreeQuiNexistePas").thru(valeur => typeof(valeur))
> // Entrée : 42 → Sortie : "undefined"
> ```

💡 **Cependant** il est tout de même possible de tester n'importe quelle valeur et d'en connaître le type "exact", mais il convient d'utiliser une fonction particulière :

```js
value = get("value.entree").thru(valeur => Object.prototype.toString.call(valeur))
// Entrée : 42 → Sortie : "[object Number]"
```

Le résultat de la fonction sera toujours écrit `[object ...]`, on peut encore affiner cela pour n'obtenir que le type en enlevant les 8 premiers caractères "[object " et le dernier "]" : 

```js
value = get("value.entree").thru(valeur => Object.prototype.toString.call(valeur).slice(8, -1))
// Entrée : "texte → Sortie : "String"
// Entrée : true → Sortie : "Boolean"
// Entrée : 42 → Sortie : "Number"
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : "Object"
// Entrée : [1, 2, 3] → Sortie : "Array"
// Entrée : null → Sortie : "Null"
// Entrée qui n'existe pas → Sortie : "Undefined"
```

📌 Ce script peut être très utile pour s'assurer de l'homogénéité des données que l'on traite.

---

## isString  

Teste si la valeur est une chaîne de caractères.  

```js
value = get("value.entree").isString()
// Entrée : ["texte"] → Sortie : false
// Entrée : "texte" → Sortie : true
```

## toString  

Convertit une valeur en chaîne de caractères.

```js
value = get("value.entree").toString()
// Entrée : 42 → Sortie : "42"
```

> [!NOTE]  
> Toutes les fonctions de type *.toX* **prennent l'ensemble de la valeur donnée** (nombre, objet, tableau, tableau de tableaux…) et tentent de la convertir en **une seule valeur cible** 
> (string, number...), en utilisant les règles de coercition de **JavaScript**.

Ce qui donne :  

```js
value = get("value.entree").toString()
// Entrée : [1, 2, 3] → Sortie : "1,2,3"
// Entrée : [[1, 2], [3, 4]] → Sortie : "1,2,3,4"
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : "[object Object]"
```

---

## isNumber  

Vérifie si la valeur est un nombre.  

```js
value = get("value.entree").isNumber()
// Entrée : 42 → Sortie : true
// Entrée : "42" → Sortie : false
```

## toNumber  

Convertit une valeur en nombre.  

```js
value = get("value.entree").toNumber()
// Entrée : "42" → Sortie : 42
// Entrée : "42, la réponse à tout, ou presque." → Sortie : null (null dans Lodex, NaN pour not a number en JavaScript)
// Entrée : false → Sortie : 0
// Entrée : true → Sortie : 1
// Entrée : [42] → Sortie : 42
// Entrée : [1, 2] → Sortie : null
```

---

## isBoolean  

Teste si la valeur est un booléen.  

```js
value = get("value.entree").isBoolean()
// Entrée : true → Sortie : true
// Entrée : "true" → Sortie : false
```

> [!TIP] Il n'existe pas en **Lodash** de fonction qui transforme une valeur en un booléen. Pour faire ceci il faut utiliser la fonction **JavaScript** `Boolean`.
> Et pour pouvoir l'utiliser, il faut la passer dans une fonction **Lodash** comme `thru` ou `map` par exemple.
> ```js
> value = get("value.entree").thru(valeur => Boolean(valeur))
> // Entrée : 1 → Sortie : true
> // Entrée : 0 → Sortie : false
> // Entrée : "0" → Sortie : true
> // Entrée : "texte" → Sortie : true
> // Entrée : "" → Sortie : false
> // Entrée : [1, 2] → Sortie : true
> // Entrée : [] → Sortie : false ⚠️ (un tableau, même vide, est truthy)
> // Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : true
> // Entrée : {} → Sortie : false ⚠️ (un objet, même vide, est truthy)
> // Entrée : null → Sortie : false
> ```

---

## isArray  

Vérifie si la valeur est un tableau.  

```js
value = get("value.entree").isArray()
// Entrée : [1, 2, 3] → Sortie : true
// Entrée : [] → Sortie : true (un tableau, même vide, est bien un tableau)
// Entrée : [[1, 2], [3, 4]] → Sortie : true 
// Entrée : "texte" → Sortie : false
// Entrée : true → Sortie : false
// Entrée : 42 → Sortie : false
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : false
// Entrée : null → Sortie : false
```
## toArray  

Essaie de convertir la valeur en tableau, en fonction de son type :

- Une chaîne devient un tableau de caractères
- Un objet devient un tableau de ses valeurs
- Pour tout ce qui n’est pas tableau ou objet "itérable", on obtient [].

```js
value = get("value.entree").toArray()
// Entrée : "texte" → Sortie : ["t", "e", "x", "t", "e"]
// Entrée : true → Sortie : []
// Entrée : 42 → Sortie : []
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : ["Einstein", "Physics"]
// Entrée : [1, 2, 3] → Sortie : [1, 2, 3]
// Entrée : [] → Sortie : []
// Entrée : null → Sortie : []
```

## castArray  

A la différence de `toArray`, `castArray` encapsule une valeur dans un tableau, sans la transformer. 

```js
value = get("value.entree").castArray()
// Entrée : "texte" → Sortie : ["texte"]
// Entrée : true → Sortie : [true]
// Entrée : 42 → Sortie : [42]
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : [{ "name": "Einstein", "field": "Physics" }]
// Entrée : [1, 2, 3] → Sortie : [1, 2, 3]
// Entrée : [] → Sortie : []
// Entrée : null → Sortie : [null]
```

👉 En résumé :  

`toArray` découpe ou extrait selon le type.
`castArray` range tel quel dans un tableau.

---

## isObject  

Teste si la valeur est un objet.  

```js
value = get("value.entree").isObject()
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : true
// Entrée : [1, 2, 3] → Sortie : true (un tableau est un objet, comme vu dans `typeof`)
// Entrée : "texte" → Sortie : false
// Entrée : 42 → Sortie : false
// Entrée : true → Sortie : false
// Entrée : null → Sortie : false (ici par contre, ***null*** n'est pas considéré comme objet ! `isObject` est une fonction **Lodash** et pas **JavaScript** )
```

---

## isEmpty  

Vérifie si **le contenu** d'une valeur est vide.  

```js
value = get("value.entree").isEmpty()
// Entrée : [] → Sortie : true
// Entrée : [1, 2, 3] → Sortie : false
// Entrée : {} → Sortie : true
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : false
// Entrée : "" → Sortie : true
// Entrée : "texte" → Sortie : false
// Entrée : null → Sortie : true
// Entrée : undefined → Sortie : true
// Entrée : 42 → Sortie : true ⚠️
// Entrée : 0 → Sortie : true ⚠️
```

> [!WARNING]
> On remarque que les deux dernières entrées, qui sont des nombres, renvoient *true*.
> Comme expliqué, `isEmpty` teste le contenu de ce qui lui est soumis.
> Dans un tableau, il vérifie qu'il contient bien des éléments.
> Dans un objet, il vérifie la présence de propriétés.
> Dans une chaîne, il regarde s'il y a des caractères.
> Pour tout le reste, booléens, ***null***, ***undefined*** et nombres donc, il renvoie **true** car on considère que ce ne sont pas des collections inspectables. Ces valeurs ne contiennent rien, elles sont donc considérées comme vides.


> [!TIP]
> Si `isEmpty` permet de vérifier si le contenu d'une valeur est vide, il n'existe pas de fonction qui teste le contraire.
> Il faut alors inverser la fonction.
> ```js
> value = get("value.entree").thru(arr => !_.isEmpty(arr))
> // Entrée : [1, 2, 3] → Sortie : true
> // Entrée : [] → Sortie : false
> ```
> Attention à bien interpéter le résultat qui signifie littéralement pour false "n'est pas non vide" lorsqu'on lit le code.
> On peut également rendre plus lisible l'inversion du booléen qui ne se voit que par **!** en utilisant la fonction `negate` :
> ```js
> value = get("value.entree").thru(_.negate(_.isEmpty))
> // Entrée : [1, 2, 3] → Sortie : true
> // Entrée : [] → Sortie : false
> ```

Nous verrons dans les [cas d'usage](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/10-cas-dusage.md) comment créer une fonction que l'on nommerait `isFully` pour plus de clarté.

---

## isEqual  

Vérifie si deux valeurs sont **profondément égales**.  

On a vu dans la partie sur les [opérateurs](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/02-types-et-operateurs.md#les-op%C3%A9rateurs-javascript) que `===` servait à tester une égalité stricte. Par exemple :

```js
value = get("value.entree").thru(valeur=>valeur === 42 ? "Oui" : "Non")
// Entrée : 42 → Sortie : "Oui"
```

**42** est un nombre, soit une valeur primitive.  
Pour les structures plus complexes comme les tableaux et objets, `===` ne fonctionnera plus.

```js
value = get("value.entree").thru(valeur=>valeur === [1, 2, 3] ? "Oui" : "Non")
// Entrée : [1, 2, 3] → Sortie : false
```

Il faut utiliser `isEqual` qui compare récursivement les tableaux et objets.

```js
value = get("value.entree").isEqual([1, 2, 3])
// Entrée : [1, 2, 3] → Sortie : true
```

> [!TIP]
> Dans cet exemple, on a directement mis la valeur à comparer en argument de la fonction ([1, 2, 3]).
> Dans Lodex, `isEqual` est particulièrement utile pour comparer les valeurs de 2 colonnes.
> Prennons des colonnes "entree" et "entree2", voici comment les comparer : 
> ```js
>  value = get("value.entree").isEqual(self.value.entree2)
> // "entree" : [1, 2, 3]
> // "entree2" : [1, 2, 3]
> // → Sortie : true
> ``` 

---

## isEqualWith  

Comme `isEqual`, mais permet d’ajouter un comparateur personnalisé. Peut être utile afin de définir sa propre logique de comparaison ou de comparer avec une certaine tolérance.  

A titre d'exemple, on voudrait comparer si deux colonnes contiennent les mêmes chaînes, mais sans tenir compte de la casse (pour simplifier on se limite aux majuscules et minuscules).

```js
value = get("value.entree").isEqualWith( \
  self.value.entree2, \
  (a, b) => _.isString(a) && _.isString(b) && a.toLowerCase() === b.toLowerCase() \
)
// "entree" : "JavaScript"
// "entree2" : "JAVASCRIPT"
// → Sortie : true
```

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/09-autres-fonctions.md)
