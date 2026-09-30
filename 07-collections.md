# Manipuler des collections

Dans Lodex on peut utiliser des fonctions dédiées aux **collections** sur :
- Des **tableaux** (`arrays`),
- Des **objets** (`objects`),
- Des **chaînes de caractères** (`strings`).

Les fonctions Lodash de cette catégorie permettent d’itérer, filtrer, transformer ou analyser ces collections, quelle que soit leur forme.


> [!NOTE]
> Vous pouvez tester chaque *exemple* dans [EZS Playground](https://ezs-playground.lodex.inist.fr/).  
> Utilisez l'input suivant :  
> ```json
> [
>   {
>     "value": {
>       "title": "Istex: A Database of Twenty Million Scientific Papers with a Mining Tool Which Uses Named Entities",
>       "year": 2019,
>       "source": "Information",
>       "publisher": "MDPI",
>       "authors": [
>         {
>           "fullname": "Denis Maurel",
>           "forename": "Denis",  
>           "surname": "Maurel", 
>           "rnsr": ["201220254T"],
>           "affiliations": [
>             {
>               "address": "Laboratoire d’Informatique Fondamentale et Appliquée de Tours LIFAT, Bases de données et traitement des langues naturelles BDTLN, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Enza Morale",
>           "forename": "Enza",
>           "surname": "Morale",  
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Nicolas Thouvenin",
>           "forename": "Nicolas", 
>           "surname": "Thouvenin",
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Patrice Ringot",
>           "forename": "Patrice",
>           "surname": "Ringot",
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         },
>         {
>           "fullname": "Angel Turri",
>           "forename": "Angel",
>           "surname": "Turri",
>           "rnsr": ["198822446E"],
>           "affiliations": [
>             {
>               "address": "UAR76 / UPS76, Centre National de la Recherche Scientifique CNRS, Institut de l’information scientifique et technique INIST, 2, Allée du Parc de Brabois, Rue Jean Zay, 54500 Vandœuvre-lès-Nancy, FR"
>             }
>           ]
>         }
>       ]
>     }
>   }
> ]
> ```
>
> puis collez ce script que vous adapterez.
> 
> ```js
> [use]
> plugin = basics
>
> [JSONParse]
> separator = *
>
> [replace]
> path = sortie
> value = get("value")
>
> [dump]
> indent = true
> ```

## map

Renvoie un tableau de valeurs en parcourant chaque élément de la collection.  
Cela permet donc d'extraire des propriétés d'une collection :

```js
value = get("value.authors").map("fullname")
// → Sortie : ["Denis Maurel","Enza Morale","Nicolas Thouvenin","Patrice Ringot","Angel Turri"]
```

ou d'appliquer une transformation à ces éléments :  

```js
value = get("value.authors").map(auth => auth.fullname.toUpperCase())
// → Sortie : ["DENIS MAUREL","ENZA MORALE","NICOLAS THOUVENIN","PATRICE RINGOT","ANGEL TURRI"]
```
## flatMap

Combine les rôles de `map` et `flatten`, très pratique pour récupérer en une seule étape des valeurs situées dans des *tableaux imbriqués*.  
Dans cet exemple, *authors* est un tableau et chaque auteur possède une propriété *affiliations* qui est elle-même un tableau contenant les adresses.  
Si l'on souhaite extraie les adresses, il faut donc d'abord aplanir *affiliations* :

```js
value = get("value.authors").map("affiliations").flatten().map("address")
// → Sortie : ["Laboratoire d’Informatique Fondamentale...", "UAR76 / UPS76, Centre National de la Recherche Scientifique C...",...]
```

`flatMap` permet de faire ceci en une seule fois.
```js
value = get("value.authors").flatMap("affiliations").map("address")
```

## invokeMap

Exécute une méthode donnée sur chaque élément d’une collection et renvoie les résultats.  

```js
value = get("value.authors").map("fullname").invokeMap("toUpperCase")
// Entrée : ["Denis Maurel", "Enza Morale", "Nicolas Thouvenin"]
// Sortie : ["DENIS MAUREL", "ENZA MORALE", "NICOLAS THOUVENIN"]
```

Cela permer d'écrire plus facilement :  

```js
value = get("value.authors").map("fullname").map(author => author.toUpperCase())
```

> [!WARNING]
> `invokeMap` ne peut être utilisé qu’avec des méthodes natives : ce sont les fonctions déjà présentes en **JavaScript**.  
> Par exemple, `toUpperCase` fonctionne car c’est une méthode de base des chaînes en **JavaScript** (et donc reconnue par **Lodash**).  
> En revanche, `startCase` ne fonctionnera pas : c’est une fonction spécifique à **Lodash**, absente de **JavaScript** natif.  

## includes

Vérifie si un élément est présent dans la collection.
Dans cet exemple, on cherche si dans les adresses de chaque auteur figure le motif *CNRS*

```js
value = get("value.authors").flatMap("affiliations").map(author => author.address.includes("CNRS"))
// → Sortie : :[false,true,true,true,true]
```

## filter

Renvoie les éléments qui satisfont une condition.
Ici on ne veut conserver dans notre collection que les auteurs ayant comme *rnsr* *198822446E*. On utilise donc `filter` qui va exécuter pour chaque auteur la fonction `includes`.  
Cette dernière renvoyant un booléen, `filter` ne gardera que les auteurs dont `includes` renvoie *true*.

```js
value = get("value.authors").filter(author => author.rnsr.includes("198822446E"))
// → Sortie : le tableau des auteurs ne contient désormais plus que les auteurs ayant comme valeur "198822446E" pour la clé `rnsr`.
```

## reject

Le contraire de `filter`, la fonction retire les éléments qui satisfont une condition.  

On peut retirer des éléments en inversant filter comme ceci :

```js
value = get("value.authors").filter(author => !author.rnsr.includes("198822446E"))
// Renvoie tous les auteurs sauf ceux dont le RNSR est "198822446E"
```

Mais `reject` est peut être plus lisible :

```js
value = get("value.authors").reject(author => author.rnsr.includes("198822446E"))
```

## every

Vérifie si **tous** les éléments satisfont une condition.

```js
value = get("value.authors").every(author => author.rnsr.includes("198822446E"))
// → Sortie : false
```

## some

Vérifie si **au moins un** élément satisfait une condition.

```js
value = get("value.authors").some(author => author.rnsr.includes("198822446E"))
// → Sortie : true
```

## find

Renvoie le **premier** élément correspondant à une condition.

```js
value = get("value.authors").find(author => author.rnsr.includes("198822446E"))
// → Sortie : {"fullname":"Enza Morale","forename":"Enza","surname":"Morale","rnsr":["198822446E"]...}
```

## groupBy

Regroupe les éléments selon une clé.
Ici on va regrouper les auteurs ayant la mention *CNRS* dans leur adresse sous la clé `Auteurs CNRS` et ceux qui ne l'ont pas sous la clé `Autres`.

```js
value = get("value.authors").groupBy(author => \
  author.affiliations.some(aff => aff.address.includes("CNRS")) \
    ? "Auteurs CNRS" \
    : "Autres" \
)
// → Sortie : {"Autres":[{"fullname":"Denis Maurel"...}],"Auteurs CNRS":[{"fullname":"Enza Morale"...}]
```

## sortBy

Trie les éléments selon une fonction.
Ici on voudra trier le tableau d'auteurs par ordre alphabétique d'après leur nom de famille.

```js
value = get("value.authors").sortBy(author => author.surname)
// → Entrée : [{"fullname":"Denis Maurel"...},{"fullname":"Enza Morale"...},{"fullname":"Nicolas Thouvenin"...},{"fullname":"Patrice Ringot"...},{"fullname":"Angel Turri"...}
// → Sortie : [{"fullname":"Denis Maurel"...},{"fullname":"Enza Morale"...},{"fullname":"Patrice Ringot"...},{"fullname":"Nicolas Thouvenin"...},{"fullname":"Angel Turri"...}
```

## size

Renvoie la taille de la collection.

```js
value = get("value.authors").size()
// → Sortie : 5 (il y a bien 5 auteurs)
```

## reduce

Parcourt une collection élément par élément et accumule un résultat au fil des itérations.  
À chaque étape, la fonction reçoit la valeur accumulée jusqu’ici (accumulator) et l’élément courant, puis retourne une nouvelle valeur d’accumulation.  
NB : si aucun accumulateur initial n’est fourni, le premier élément de la collection est utilisé comme valeur de départ.

```js
value = get("value.entrée").reduce((accumulator, current) => accumulator + current, 0)
// → Entrée : [1, 2, 3, 4]
// → Sortie : 10
```

Ici, l’accumulateur initial est *0*.
La fonction reduce parcourt le tableau élément par élément :
* à chaque étape, elle additionne l’accumulateur et la valeur courante,
* puis renvoie ce nouveau total comme nouvel accumulateur pour l’itération suivante.

Ce qui donne :  
0 + 1 → 1  
1 + 2 → 3  
3 + 3 → 6  
6 + 4 → 10  

## keyBy

Transforme un tableau d’objets en un objet en utilisant la valeur d’une propriété comme clé.

```js
value = get("value.authors").keyBy("surname")
// Entree : [{"fullname":"Denis Maurel","surname":"Maurel"},{"fullname":"Enza Morale","surname":"Morale"}, ...]
// Sortie : {"Maurel":{"fullname":"Denis Maurel","surname":"Maurel", ...},"Morale":{"fullname":"Enza Morale","surname":"Morale", ...}, ...}
```

Les valeurs de la propriété `surname` deviennent ici les clés du nouvel objet.

> [!WARNING]
> La propriété utilisée comme clé doit idéalement contenir des valeurs uniques. Si plusieurs éléments possèdent la même valeur, `keyBy` ne conserve que le dernier élément rencontré pour cette clé.
>
> Par exemple :
>
> ```js
> value = get("value.authors").keyBy("rnsr")
> ```
>
> Dans cet exemple, Enza Morale, Nicolas Thouvenin, Patrice Ringot et Angel Turri possèdent tous le RNSR `198822446E`. Seul le dernier auteur rencontré, Angel Turri, est conservé sous cette clé.
>
> ```json
> {
>   "198822446E": {
>     "fullname": "Angel Turri",
>     "forename": "Angel",
>     "surname": "Turri",
>     "rnsr": ["198822446E"]
>   }
> }
> ```
>
> [Tester ce cas dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIklzdGV4OiBBIERhdGFiYXNlIG9mIFR3ZW50eSBNaWxsaW9uIFNjaWVudGlmaWMgUGFwZXJzXCIsXG4gICAgICBcInllYXJcIjogMjAxOSxcbiAgICAgIFwic291cmNlXCI6IFwiSW5mb3JtYXRpb25cIixcbiAgICAgIFwicHVibGlzaGVyXCI6IFwiTURQSVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJEZW5pcyBNYXVyZWxcIixcbiAgICAgICAgICBcImZvcmVuYW1lXCI6IFwiRGVuaXNcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJNYXVyZWxcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMjAxMjIwMjU0VFwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIkVuemEgTW9yYWxlXCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkVuemFcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJNb3JhbGVcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIk5pY29sYXMgVGhvdXZlbmluXCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIk5pY29sYXNcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJUaG91dmVuaW5cIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIlBhdHJpY2UgUmluZ290XCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIlBhdHJpY2VcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJSaW5nb3RcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIkFuZ2VsIFR1cnJpXCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkFuZ2VsXCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiVHVycmlcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9XG4gICAgICBdXG4gICAgfVxuICB9XG5dIiwic2NyaXB0IjoiIyBFWlMgc2NyaXB0XG5bdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cbltyZXBsYWNlXVxucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5hdXRob3JzXCIpLmtleUJ5KFwicm5zclwiKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## partition

Sépare une collection en deux groupes selon une condition. Le premier groupe contient les éléments qui respectent la condition, le second ceux qui ne la respectent pas.

```js
value = get("value.authors").partition(auteur => auteur.rnsr.includes("198822446E"))
// Entree : tableau des auteurs
// Sortie : [[auteurs ayant le RNSR "198822446E"], [autres auteurs]]
```

Dans cet exemple, le premier tableau contient Enza Morale, Nicolas Thouvenin, Patrice Ringot et Angel Turri. Le second contient Denis Maurel.

Contrairement à `filter`, qui ne conserve que les éléments correspondant à la condition, `partition` conserve tous les éléments et les répartit en deux groupes.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIklzdGV4OiBBIERhdGFiYXNlIG9mIFR3ZW50eSBNaWxsaW9uIFNjaWVudGlmaWMgUGFwZXJzXCIsXG4gICAgICBcInllYXJcIjogMjAxOSxcbiAgICAgIFwic291cmNlXCI6IFwiSW5mb3JtYXRpb25cIixcbiAgICAgIFwicHVibGlzaGVyXCI6IFwiTURQSVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJEZW5pcyBNYXVyZWxcIixcbiAgICAgICAgICBcImZvcmVuYW1lXCI6IFwiRGVuaXNcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJNYXVyZWxcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMjAxMjIwMjU0VFwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIkVuemEgTW9yYWxlXCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkVuemFcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJNb3JhbGVcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIk5pY29sYXMgVGhvdXZlbmluXCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIk5pY29sYXNcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJUaG91dmVuaW5cIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIlBhdHJpY2UgUmluZ290XCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIlBhdHJpY2VcIixcbiAgICAgICAgICBcInN1cm5hbWVcIjogXCJSaW5nb3RcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmdWxsbmFtZVwiOiBcIkFuZ2VsIFR1cnJpXCIsXG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkFuZ2VsXCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiVHVycmlcIixcbiAgICAgICAgICBcInJuc3JcIjogW1wiMTk4ODIyNDQ2RVwiXVxuICAgICAgICB9XG4gICAgICBdXG4gICAgfVxuICB9XG5dIiwic2NyaXB0IjoiIyBFWlMgc2NyaXB0XG5bdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cbltyZXBsYWNlXVxucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5hdXRob3JzXCIpLnBhcnRpdGlvbihhdXRldXIgPT4gYXV0ZXVyLnJuc3IuaW5jbHVkZXMoXCIxOTg4MjI0NDZFXCIpKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/08-fonctions-sur-les-types-lang.md)