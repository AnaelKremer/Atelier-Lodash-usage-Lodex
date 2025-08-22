# Autres fonctions utiles

Certaines fonctions Lodash ne sont pas spécifiques aux **chaînes**, **tableaux**, **objets** ou **collections**, mais sont utiles pour tester, convertir ou manipuler des données de manière générale.  

## thru  

Permet de passer une valeur dans une fonction intermédiaire, puis de continuer la chaîne Lodash.  
Utile pour faire des transformations conditionnelles.  

```js
value = get("value.year").thru(y => y < 2000 ? "ancien" : "récent")
// Entrée : 2019 → Sortie : "récent"
```

---

Il existe plusieurs fonctions qui permettent de tester le type d'une valeur (*.isX*) et qui ont souvent leur pendant pour convertir une valeur dans un type donnée (*.toX*).  
Malheureusement il n'existe pas de fonction unique pour renvoyer le type de la valeur testée. Pour ce faire il faut ituliser une fonction **JavaScript**, `typeof` en l'occurence.

```js
value = get("value.entree").thru(valeur => typeof(valeur))
// Entrée : "texte" → Sortie : "string"
// Entrée : true → Sortie : "boolean"
// Entrée : 42 → Sortie : "number"
// Entrée : { "name": "Einstein", "field": "Physics" } → Sortie : "object"
// Entrée : [1,2,3] → Sortie : "object"
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
> On pourrait également objecter (😄) qu'***undefined*** n'a pas été testé. C'est parce qu'un JSON ne peut pas contenir directement ***undefined***, c'est un concept **JavaScript**.
> Pour ce faire il faut provoquer une erreur en accédant à une clé qui n'existe pas.
> ```js
> value = get("value.entreeQuiNexistePas").thru(valeur => typeof(valeur))
> // Entrée : 42 → Sortie :  "undefined"
> ```

## isString  

Teste si la valeur est une chaîne de caractères.  

```js
value = get("value.entree").isString()
// Entrée : ["Ceci n'est pas un tableau"] → Sortie :  false
// Entrée : "Ceci n'est pas un tableau" → Sortie :  true
```

## toString  

Convertit une valeur en chaîne de caractères.  

```js
value = _.toString(get("value.year"))
// 2019 → "2019"
```



## isBoolean  

Teste si la valeur est un booléen.  

```js
value = get("value.entree").isBoolean()
// Entrée : true → Sortie :  true
// Entrée : "true" → Sortie :  false
```

## isArray  

Vérifie si la valeur est un tableau.  

```js
value = get("value.entree").isArray()
// Entrée : ["Ceci n'est pas un tableau"] → Sortie :  true
// Entrée : "Ceci n'est pas un tableau" → Sortie :  false
```
## castArray  

Convertit une valeur en tableau (si ce n’en est pas déjà un).  

```js
value = _.castArray(get("value.title"))
// Entrée : "une chaîne de caractères" → Sortie : ["une chaîne de caractères"]
// Entrée : [0, 1, 2] → Sortie : [0, 1, 2]
```

## toArray  

Transforme une valeur en tableau :  
- objets → tableau de valeurs,  
- string → tableau de caractères.  

```js
value = get("value.entree").toArray()
// Entrée : "une chaîne de caractères" → Sortie : ["u","n","e"," ","c","h","a","î","n","e"," ","d","e"," ","c","a","r","a","c","t","è","r","e","s"]
```




