# Manipuler des objets

Dans Lodex, de nombreuses colonnes ou valeurs peuvent contenir des objets (paires clé-valeur) : noms d'auteurs, adresses, informations bibliographiques… Lodash offre de nombreuses fonctions pour les parcourir, les transformer ou les enrichir.

Certaines fonctions ci-dessous sont exclusives aux objets, tandis que d'autres s'appliquent plus largement aux collections (tableaux ou objets). Cette feuille se concentre sur les fonctions spécifiques aux objets. Les fonctions applicables aux collections seront abordées [ici.]()

> [!NOTE]  
> Vous pouvez tester chaque *exemple* dans [EZS Playground](https://ezs-playground.lodex.inist.fr/). Ecrivez l'input de cette façon : 
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
>           "surname": "Chapoutot",
>           "fullname": "Johann Chapoutot"
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

## assign

Ajoute des propriétés à un objet ou remplace la valeur de propriétés existantes.

```js
// Ajouter de nouvelles propriétés
value = get("value").cloneDeep().assign({ \
    type: "article", \
    language: "fr" \
})
// Ajoute les propriétés type et language.

// Remplacer une propriété existante
value = get("value").cloneDeep().assign({ \
    year: 2026 \
})
// La propriété year existe déjà : sa valeur 2012 est remplacée par 2026.
```

`assign` permet donc aussi bien d'enrichir un objet avec de nouvelles propriétés que de modifier la valeur de propriétés déjà présentes.

> [!WARNING]
> `assign` modifie l'objet sur lequel il est appliqué. Cela peut provoquer des effets de bord si le même objet est utilisé ensuite dans le traitement.
>
> Par exemple :
>
> ```js
> value = get("value").assign({ \
>     type: "article", \
>     language: "fr", \
>     year: 2026 \
> })
> ```
>
> Ici, l'objet `value` lui-même est modifié. L'utilisation de `cloneDeep()` avant `assign()` permet de travailler sur une copie et de préserver l'objet d'origine :
>
> ```js
> value = get("value").cloneDeep().assign({ \
>     year: 2026 \
> })
> ```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbiMgMS4gQWpvdXRlciBkZSBub3V2ZWxsZXMgcHJvcHJp6XTpc1xucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZVwiKS5jbG9uZURlZXAoKS5hc3NpZ24oeyBcXFxuICAgIHR5cGU6IFwiYXJ0aWNsZVwiLCBcXFxuICAgIGxhbmd1YWdlOiBcImZyXCIgXFxcbn0pXG5cbiMgMi4gUmVtcGxhY2VyIHVuZSBwcm9wcmnpdOkgZXhpc3RhbnRlXG5wYXRoID0gc29ydGllMlxudmFsdWUgPSBnZXQoXCJ2YWx1ZVwiKS5jbG9uZURlZXAoKS5hc3NpZ24oeyBcXFxuICAgIHllYXI6IDIwMjYgXFxcbn0pXG5cbiMgMy4gTW9udHJlciBxdWUgYXNzaWduIG1vZGlmaWUgbCdvYmpldCBzb3VyY2VcbnBhdGggPSBzb3J0aWUzXG52YWx1ZSA9IGdldChcInZhbHVlXCIpLmFzc2lnbih7IFxcXG4gICAgdHlwZTogXCJhcnRpY2xlXCIsIFxcXG4gICAgbGFuZ3VhZ2U6IFwiZnJcIiwgXFxcbiAgICB5ZWFyOiAyMDI2IFxcXG59KVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## at

Récupère plusieurs valeurs d’un objet à partir de leurs chemins et les retourne dans un tableau.

```js
value = get("value").at(["title", "year", "authors[0].fullname"])
// Entree : {"title":"Mouvement historique et histoire suspendue","year":2012,"source":"Vingtième Siècle Revue d histoire","authors":[...]}
// Sortie : ["Mouvement historique et histoire suspendue",2012,"Hartmut Rosa"]
```

Les chemins peuvent cibler des propriétés imbriquées. Ici, `authors[0].fullname` permet de récupérer le nom complet du premier auteur.

Contrairement à `pick`, qui retourne un objet avec les propriétés sélectionnées, `at` retourne directement un tableau contenant les valeurs demandées, dans l’ordre des chemins indiqués.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikuYXQoW1widGl0bGVcIiwgXCJ5ZWFyXCIsIFwiYXV0aG9yc1swXS5mdWxsbmFtZVwiXSlcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZSJ9)

## defaults

Ajoute des valeurs par défaut aux propriétés absentes d’un objet sans remplacer les propriétés déjà définies.

```js
value = get("value").cloneDeep().defaults({ \
    year: 2026, \
    language: "fr" \
})
// Entree : {"title":"Mouvement historique et histoire suspendue","year":2012,"source":"Vingtième Siècle Revue d histoire","authors":[...]}
// Sortie : {"title":"Mouvement historique et histoire suspendue","year":2012,"source":"Vingtième Siècle Revue d histoire","authors":[...],"language":"fr"}
```

Dans cet exemple, `year` existe déjà et conserve donc sa valeur `2012`. En revanche, `language` est absent et prend la valeur par défaut `"fr"`.

Contrairement à `assign`, `defaults` ne remplace pas la valeur d'une propriété déjà définie. Il permet donc de compléter un objet tout en conservant les données existantes.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbiMgQWpvdXRlIHVuZSBwcm9wcmnpdOkgYWJzZW50ZVxucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZVwiKS5jbG9uZURlZXAoKS5kZWZhdWx0cyh7IFxcXG4gICAgbGFuZ3VhZ2U6IFwiZnJcIiBcXFxufSlcblxuIyBFc3NhaWUgZGUgZm91cm5pciB1bmUgdmFsZXVyIHBhciBk6WZhdXQg4CB1bmUgcHJvcHJp6XTpIGV4aXN0YW50ZVxucGF0aCA9IHNvcnRpZTJcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikuY2xvbmVEZWVwKCkuZGVmYXVsdHMoeyBcXFxuICAgIHllYXI6IDIwMjYsIFxcXG4gICAgbGFuZ3VhZ2U6IFwiZnJcIiBcXFxufSlcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## defaultsDeep

Ajoute des valeurs par défaut, y compris à l'intérieur des objets et tableaux imbriqués, sans remplacer les valeurs déjà définies.

```js
value = get("value").cloneDeep().defaultsDeep({ \
    authors: [{ \
        fullname: "Nom inconnu", \
        orcid: "ORCID inconnu" \
    }] \
})
```

Dans cet exemple, `fullname` existe déjà et conserve donc sa valeur `"Hartmut Rosa"`. En revanche, `orcid` est absent et prend la valeur par défaut `"ORCID inconnu"` :

```json
{
  "forename": "Hartmut",
  "surname": "Rosa",
  "fullname": "Hartmut Rosa",
  "orcid": "ORCID inconnu"
}
```

Contrairement à `defaults`, `defaultsDeep` permet de compléter également les structures imbriquées.

> [!NOTE]
> Comme avec `merge`, les éléments des tableaux sont associés selon leur position. Ici, les valeurs par défaut définies dans le premier élément de `authors` concernent donc l'auteur situé à l'index `0`.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikuY2xvbmVEZWVwKCkuZGVmYXVsdHNEZWVwKHsgXFxcbiAgICBhdXRob3JzOiBbeyBcXFxuICAgICAgICBmdWxsbmFtZTogXCJOb20gaW5jb25udVwiLCBcXFxuICAgICAgICBvcmNpZDogXCJPUkNJRCBpbmNvbm51XCIgXFxcbiAgICB9XSBcXFxufSlcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWUifQ==)

## findKey

Recherche le nom d'une propriété dont la valeur respecte une condition.

```js
value = get("value").findKey(valeur => valeur === 2012)
// Entree : {"title":"Mouvement historique et histoire suspendue","year":2012,"source":"Vingtième Siècle Revue d histoire","authors":[...]}
// Sortie : "year"
```

`findKey` est notamment utile lorsque l'on connaît une valeur mais que l'on souhaite retrouver le nom de la propriété qui la contient.

> [!NOTE]
> `findKey` recherche uniquement parmi les propriétés de l'objet sur lequel il est appliqué. Il ne parcourt pas automatiquement les objets et tableaux imbriqués.
>
> Par exemple, si `2012` se trouve dans `metadata.publication.year`, un `findKey` appliqué à l'objet racine ne retournera pas le chemin `metadata.publication.year`.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikuZmluZEtleSh2YWxldXIgPT4gdmFsZXVyID09PSAyMDEyKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## get  

Récupère la valeur d’une clé (y compris dans des objets imbriqués).  

```js
value = get("value.title")
// → Sortie : "Mouvement historique et histoire suspendue"
```


## has  

Vérifie si une clé existe ou non.  

```js
value = get("value").has("source")
// → Sortie : true
```

## invert  

Inverse les clés et valeurs.  

```js
value = get("value").invert()
// → Sortie : {"2012": "year","Mouvement historique et histoire suspendue": "title"...}
```

## keys  

Retourne les clés d’un objet.  

```js
value = get("value").keys()
// → Sortie : ["title", "year", "source", "authors"]
```

## mapKeys  

Applique une fonction à toutes les clés d’un objet.  

```js
value = get("value").mapKeys((value, key) => key.toUpperCase())
// → Sortie : {"TITLE": "Mouvement historique et histoire suspendue","YEAR": 2012,"SOURCE": "Vingtième Siècle Revue d histoire"...}
```

## mapValues  

Applique une fonction à toutes les valeurs d’un objet.  

```js
value = get("value").mapValues(value => _.isString(value) ? value.toUpperCase() : value)

// → Sortie : {"title": "MOUVEMENT HISTORIQUE ET HISTOIRE SUSPENDUE","year": 2012,"source": "VINGTIÈME SIÈCLE REVUE D HISTOIRE"...}
// → Si une valeur est une chaîne de caractères, on la passe en majuscules.
```

## merge

Fusionne les propriétés de plusieurs objets, y compris à l'intérieur des objets et tableaux imbriqués.

```js
value = get("value").cloneDeep().merge({ \
    authors: [{ \
        orcid: "0000-0000-0000-0001" \
    }] \
})
```

Dans cet exemple, la propriété `orcid` est ajoutée au premier auteur sans supprimer ses propriétés existantes :

```json
{
  "forename": "Hartmut",
  "surname": "Rosa",
  "fullname": "Hartmut Rosa",
  "orcid": "0000-0000-0000-0001"
}
```

Le second auteur reste inchangé.

Contrairement à `assign`, qui remplace une propriété existante, `merge` peut fusionner son contenu avec celui qui existe déjà.

> [!NOTE]
> Pour les tableaux, `merge` effectue la fusion selon la position des éléments. Ici, l'objet placé à l'index `0` dans `authors` est donc fusionné avec Hartmut Rosa, qui se trouve également à l'index `0`.
>
> `merge` ne recherche pas automatiquement un élément à partir de son contenu ou d'un identifiant.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikuY2xvbmVEZWVwKCkubWVyZ2UoeyBcXFxuICAgIGF1dGhvcnM6IFt7IFxcXG4gICAgICAgIG9yY2lkOiBcIjAwMDAtMDAwMC0wMDAwLTAwMDFcIiBcXFxuICAgIH1dIFxcXG59KVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## omit  

Retourne un nouvel objet sans certaines clés.  

```js
value = get("value").omit(["source", "year"])
// → Sortie : { title: "Mouvement historique et histoire suspendue", authors: [...] }
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikub21pdChbXCJzb3VyY2VcIiwgXCJ5ZWFyXCJdKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## omitBy

Supprime les propriétés d’un objet dont la valeur respecte une condition.

```js
value = get("value").omitBy(valeur => typeof valeur === "string")
// Entree : {"title":"Mouvement historique et histoire suspendue","year":2012,"source":"Vingtième Siècle Revue d histoire","authors":[...]}
// Sortie : {"year":2012,"authors":[{"forename":"Hartmut","surname":"Rosa","fullname":"Hartmut Rosa"},{"forename":"Johann","surname":"Chapoutot","fullname":"Johann Chapoutot"}]}
```

Contrairement à `omit`, qui supprime des propriétés à partir de leur nom, `omitBy` les supprime en fonction d’une condition appliquée à leur valeur.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikub21pdEJ5KHZhbGV1ciA9PiB0eXBlb2YgdmFsZXVyID09PSBcInN0cmluZ1wiKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## pick  

Retourne un nouvel objet avec uniquement certaines clés.  

```js
value = get("value").pick(["title", "year"])
// → Sortie : { title: "Mouvement historique et histoire suspendue", year: 2012 }
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikucGljayhbXCJ0aXRsZVwiLCBcInllYXJcIl0pXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuXG5bZHVtcF1cbmluZGVudCA9IHRydWUifQ==)

## pickBy

Conserve les propriétés d’un objet dont la valeur respecte une condition.

```js
value = get("value").pickBy(valeur => typeof valeur === "string")
// Entree : {"title":"Mouvement historique et histoire suspendue","year":2012,"source":"Vingtième Siècle Revue d histoire","authors":[...]}
// Sortie : {"title":"Mouvement historique et histoire suspendue","source":"Vingtième Siècle Revue d histoire"}
```

Contrairement à `pick`, qui sélectionne des propriétés à partir de leur nom, `pickBy` les sélectionne en fonction d’une condition appliquée à leur valeur.

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJ0aXRsZVwiOiBcIk1vdXZlbWVudCBoaXN0b3JpcXVlIGV0IGhpc3RvaXJlIHN1c3BlbmR1ZVwiLFxuICAgICAgXCJ5ZWFyXCI6IDIwMTIsXG4gICAgICBcInNvdXJjZVwiOiBcIlZpbmd0aehtZSBTaehjbGUgUmV2dWUgZCBoaXN0b2lyZVwiLFxuICAgICAgXCJhdXRob3JzXCI6IFtcbiAgICAgICAge1xuICAgICAgICAgIFwiZm9yZW5hbWVcIjogXCJIYXJ0bXV0XCIsXG4gICAgICAgICAgXCJzdXJuYW1lXCI6IFwiUm9zYVwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJIYXJ0bXV0IFJvc2FcIlxuICAgICAgICB9LFxuICAgICAgICB7XG4gICAgICAgICAgXCJmb3JlbmFtZVwiOiBcIkpvaGFublwiLFxuICAgICAgICAgIFwic3VybmFtZVwiOiBcIkNoYXBvdXRvdFwiLFxuICAgICAgICAgIFwiZnVsbG5hbWVcIjogXCJKb2hhbm4gQ2hhcG91dG90XCJcbiAgICAgICAgfVxuICAgICAgXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWVcIikucGlja0J5KHZhbGV1ciA9PiB0eXBlb2YgdmFsZXVyID09PSBcInN0cmluZ1wiKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlIn0=)

## set  

Modifie ou crée une clé avec une nouvelle valeur.  

> [!WARNING]  
> **Fonction mutante** 
> Cette fonction modifie la structure passée en argument. 
> Dans un pipeline, cela peut entraîner des effets de bord si la valeur est réutilisée.

```js
value = get("value.entree").set("DOI", null)
// Entrée : [{"value":{"entree":{"title":"Exemple"}}}]
// Sortie : [{"value":{"entree":{"title":"Exemple","DOI":null}},"sortie":{"title":"Exemple","DOI":null}}]
```

L'objet initial entrée est également modifié.  

✅ Alternative non mutante recommandée : `cloneDeep`

```js
value = get("value.entree").cloneDeep().set("DOI", null)
// Entrée : [{"value":{"entree":{"title":"Exemple"}}}]
// Sortie : [{"value":{"entree":{"title":"Exemple"}},"sortie":{"title":"Exemple","DOI":null}}]
```

## toPairs  

Convertit un objet en un tableau de paires [clé, valeur]. Chaque élément du tableau résultant est un tableau à deux éléments : le premier est la clé (ou le nom de la propriété), et le second est la valeur associée. 

```js
value = get("value").toPairs()
// → Sortie : [["title","Mouvement historique..."], ["year", 2012], ...]
```

## transform  

Transforme un objet en un autre en appliquant une fonction à chaque clé/valeur.  

```js
value = get("value").transform((result, val, key) => {result[key] = _.isNumber(val) ? val + 10 : val}, {})
// → Sortie : "year": 2022
// → Si une clé a comme valeur un nombre, on y ajoute 10.
```

## update  

Met à jour la valeur d’une clé à l’aide d’une fonction.  

```js
value = get("value").update("year", y => y + 1)
// → Sortie : "year": 2013
```

## values  

Retourne les valeurs d’un objet.  

```js
value = get("value").values()
// → Sortie : ["Mouvement historique...", 2012, "Vingtième Siècle Revue d histoire", [ ... ]]
```

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/07-collections.md)