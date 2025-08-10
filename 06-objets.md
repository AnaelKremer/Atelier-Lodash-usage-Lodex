# Manipuler les objets

Dans Lodex, de nombreuses colonnes ou valeurs peuvent contenir des objets (paires clé-valeur) : noms d'auteurs, adresses, informations bibliographiques… Lodash offre de nombreuses fonctions pour les parcourir, les transformer ou les enrichir.

Certaines fonctions ci-dessous sont exclusives aux objets, tandis que d'autres s'appliquent plus largement aux collections (tableaux ou objets). Cette feuille se concentre sur les fonctions spécifiques aux objets. Les fonctions applicables aux collections seront abordées [ici.]()

> [!NOTE]  
> Vous pouvez tester chaque *exemple* dans [EZS Playground](http://ezs-playground.daf.intra.inist.fr/). Ecrivez l'input de cette façon : 
> ```json
> [
>   {
>     "value": {
>       "title": "Mouvement historique et histoire suspendue",
>       "year": 2012,
>       "source": "Vingtième Siècle Revue d histoire",
>       "authors": [
>         {
>           "forename": "Hartmut",
>           "surname": "Rosa",
>           "fullname": "Hartmut Rosa"
>         },
>         {
>           "forename": "Johann",
>           "surname": "Chapotot",
>           "fullname": "Johann Chapotot"
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
> path = entree
> value = get("value.title")
>
> [dump]
> indent = true
> ```

## get  

Récupère la valeur d’une clé (y compris dans des objets imbriqués).  

```js
value = get("value.title")
// → Sortie : "Mouvement historique et histoire suspendue"
```

## set  

Modifie ou crée une clé avec une nouvelle valeur.  

```js
value = get("value").set("DOI", null)
// → Sortie : "DOI": null
```

## update  

Met à jour la valeur d’une clé à l’aide d’une fonction.  

```js
value = get("value").update("year", y => y + 1)
// → Sortie : "year": 2013
```

## has  

Vérifie si une clé existe ou non.  

```js
value = get("value").has("source")
// → Sortie : true
```

## omit  

Retourne un nouvel objet sans certaines clés.  

```js
value = get("value").omit(["source", "year"])
// → Sortie : { title: "Mouvement historique et histoire suspendue", authors: [...] }
```

## pick  

Retourne un nouvel objet avec uniquement certaines clés.  

```js
value = get("value").pick(["title", "year"])
// → Sortie : { title: "Mouvement historique et histoire suspendue", year: 2012 }
```

## keys  

Retourne les clés d’un objet.  

```js
value = get("value").keys()
// → Sortie : ["title", "year", "source", "authors"]
```

## values  

Retourne les valeurs d’un objet.  

```js
value = get("value").values()
// → ["Mouvement historique...", 2012, "Vingtième Siècle Revue d histoire", [ ... ]]
```

## entries / toPairs  

Retourne les paires [clé, valeur] d’un objet.  

```js
value = get("value").toPairs()
// → [["title","Mouvement historique..."], ["year", 2012], ...]
```

---

## mapValues  

Applique une fonction à toutes les valeurs d’un objet.  

```js
value = get("value").mapValues(v => _.isString(v) ? v.toUpperCase() : v)
// → toutes les chaînes de caractères en majuscules
```

---

## mapKeys  

Applique une fonction à toutes les clés d’un objet.  

```js
value = get("value").mapKeys((v, k) => k.toUpperCase())
// → { TITLE: "...", YEAR: 2012, ... }
```

---

## invert  

Inverse les clés et valeurs.  

```js
value = _.invert({ a: "x", b: "y" })
// → { x: "a", y: "b" }
```

---

## merge  

Fusionne plusieurs objets.  

```js
value = _.merge({ a: 1 }, { b: 2 }, { a: 3 })
// → { a: 3, b: 2 }
```

---

## transform  

Transforme un objet en un autre en appliquant une fonction à chaque clé/valeur.  

```js
value = get("value").transform((result, val, key) => {
  result[key] = _.isNumber(val) ? val + 10 : val
}, {})
// → year passe de 2012 à 2022
```
