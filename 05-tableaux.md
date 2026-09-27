# Manipuler des tableaux  

Dans Lodex, beaucoup de colonnes contiennent des tableaux : listes d’auteurs, mots-clés, affiliations… Lodash propose une riche panoplie de fonctions pour filtrer, ordonner, fusionner, ou transformer ces données.

Certaines fonctions sont dédiées exclusivement aux tableaux, tandis que d'autres s'appliquent plus largement aux collections (tableaux ou objets). Cette feuille se concentre sur les fonctions spécifiques aux tableaux. Les fonctions applicables aux collections seront abordées [ici.]()

> [!NOTE]
> Vous pouvez tester chaque *exemple* dans [EZS Playground](https://ezs-playground.lodex.inist.fr/). Ecrivez l'input de cette façon :  
> ```json
> [
>   {
>     "value": {
>       "entree": ["A", "B"],
>       "entree2": ["A", "C"]
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
> value = get("value.entree")
>
> path = entree2
> value = get("value.entree2")
>
> path = sortie
> value = get("value.entree").intersection(self.value.entree2)
>
> [dump]
> indent = true
> ```

## concat

Concatène des tableaux ou des valeurs au tableau existant.

```js
value = get("value.entree").concat(self.value.entree2)
// Entree : ["A", "B"] Entree2 : ["A", "C"] → Sortie : ["A", "B", "A", "C"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgXCJCXCJdLFxuICAgICAgIFwiZW50cmVlMlwiOiBbXCJBXCIsIFwiQ1wiXVxuICAgICB9XG4gICB9XG4gXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bYXNzaWduXVxucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikuY29uY2F0KHNlbGYudmFsdWUuZW50cmVlMilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgIn0=)

## compact

Supprime toutes les valeurs falsy (`false`, `null`, `0`, `""`, `undefined`, `NaN`) d’un tableau.

```js
value = get("value.entree").compact()
// Entree : ["A", null, "", "B", false] → Sortie : ["A", "B"]
```  

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgbnVsbCwgXCJcIiwgXCJCXCIsIGZhbHNlXVxuICAgICB9XG4gICB9XG4gXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bYXNzaWduXVxucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikuY29tcGFjdCgpXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlXG4gICJ9)

## drop

Supprime *n* éléments d’un tableau depuis le début.  

```js
value = get("value.entree").drop()
// Entree : [1, 2, 3] → Sortie : [2, 3]

value = get("value.entree").drop(2)
// Entree : [1, 2, 3] → Sortie : [3]
```  

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFsxLDIsM11cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllMVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikuZHJvcCgpXG5cblxucGF0aCA9IHNvcnRpZTJcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLmRyb3AoMilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgIn0=)

## dropRight

Supprime *n* éléments d’un tableau depuis la fin.

```js
value = get("value.entree").dropRight()
// Entree : [1, 2, 3] → Sortie : [1, 2]

value = get("value.entree").dropRight(2)
// Entree : [1, 2, 3] → Sortie : [1]
```  

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFsxLDIsM11cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllMVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikuZHJvcFJpZ2h0KClcblxuXG5wYXRoID0gc29ydGllMlxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikuZHJvcFJpZ2h0KDIpXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlXG4gICJ9)

## uniq

Supprime les doublons dans un tableau.

```js
value = get("value.entree").uniq()
// Entree : ["A", "B", "A", "C"] → Sortie : ["A", "B", "C"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFsxLDIsM11cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllMVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikuZHJvcFJpZ2h0KClcblxuXG5wYXRoID0gc29ydGllMlxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikuZHJvcFJpZ2h0KDIpXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlXG4gICJ9)

## uniqBy

Supprime les doublons dans un tableau d’objets selon une propriété donnée.

```js
value = get("value.entree").uniqBy("ppn")
// Entree : [{"nom":"École A","ppn":"123"},{"nom":"École B","ppn":"123"},{"nom":"École C","ppn":"456"}] → Sortie : [{"nom":"École A","ppn":"123"},{"nom":"École C","ppn":"456"}]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1xuICAgICAgICB7XCJub21cIjpcIsljb2xlIEFcIixcInBwblwiOlwiMTIzXCJ9LFxuICAgICAgICB7XCJub21cIjpcIsljb2xlIEJcIixcInBwblwiOlwiMTIzXCJ9LFxuICAgICAgICB7XCJub21cIjpcIsljb2xlIENcIixcInBwblwiOlwiNDU2XCJ9XG4gICAgICBdXG4gICAgfVxuICB9XG5dIiwic2NyaXB0IjoiIyBFWlMgc2NyaXB0XG5bdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cblthc3NpZ25dXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS51bmlxQnkoXCJwcG5cIilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5baWRlbnRpZnldXG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAifQ==)

## first / head

Renvoie le premier élément d’un tableau.

```js
value = get("value.entree").head()
// Entree : ["A", "B", "C"] → Sortie : ["A"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgXCJCXCIsIFwiQ1wiXVxuICAgICB9XG4gICB9XG4gXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLmhlYWQoKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAifQ==)

## last

Renvoie le dernier élément d’un tableau.

```js
value = get("value.entree").last()
// Entree : ["A", "B", "C"] → Sortie : ["C"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgXCJCXCIsIFwiQ1wiXVxuICAgICB9XG4gICB9XG4gXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLmxhc3QoKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAifQ==)

## flatten

Aplati un tableau **d’un seul** niveau. Utile lorsqu'on a un tableau de tableaux ou matrice.

```js
value = get("value.entree").flatten()
// Entree : [[1, 2], [3]] → Sortie : [1, 2, 3]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtbMSwgMl0sIFszXV1cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS5mbGF0dGVuKClcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgIn0=)

## flattenDeep

Aplati récursivement un tableau **à tous** les niveaux.

```js
value = get("value.entree").flattenDeep()
// Entree : [1, [2, [3, [4]]]] → Sortie : [1, 2, 3, 4]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFsxLCBbMiwgWzMsIFs0XV1dXVxuICAgICB9XG4gICB9XG4gXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLmZsYXR0ZW5EZWVwKClcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgIn0=)

## nth

Renvoie l’élément à la position `n`.

```js
value = get("value.entree").nth(1)
// Entree : ["A", "B", "C"] → Sortie : ["B"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgXCJCXCIsIFwiQ1wiXVxuICAgICB9XG4gICB9XG4gXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLm50aCgxKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAifQ==)

## pull

Supprime une valeur d'un tableau.  

> [!WARNING]  
> **Fonction mutante** 
> Cette fonction modifie la structure passée en argument. 
> Dans un pipeline, cela peut entraîner des effets de bord si la valeur est réutilisée.

```js
value = get("value.entree").pull("A")
// Entree : [{"value":{"entree":["A","B"]}}]
// Sortie : [{"value":{"entree":["B"]},"sortie":["B"]}]
```

✅ Alternative non mutante recommandée : `without`

```js
value = get("value.entree").without("A")
// Entree : [{"value":{"entree":["A","B"]}}]
// Sortie : [{"value":{"entree":["A","B"]},"sortie":["B"]}]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgXCJCXCJdXG4gICAgIH1cbiAgIH1cbiBdIiwic2NyaXB0IjoiIyBFWlMgc2NyaXB0XG5bdXNlXVxucGx1Z2luID0gYmFzaWNzXG5cbltKU09OUGFyc2VdXG5zZXBhcmF0b3IgPSAqXG5cbltyZXBsYWNlXVxucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikud2l0aG91dChcIkFcIilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgIn0=)

## pullAll

Supprime plusieurs valeurs du tableau.  

> [!WARNING]  
> **Fonction mutante** 
> Cette fonction modifie la structure passée en argument. 
> Dans un pipeline, cela peut entraîner des effets de bord si la valeur est réutilisée.

```js
value = get("value.entree").pullAll(["B","C"])
// Entree : ["A", "B", "C", "D"]
// Sortie : [{"value":{"entree":["A","D"]},"sortie":["A","D"]}]
```

✅ Alternative non mutante recommandée : `without`  

```js
value = get("value.entree").without("B","C")
// Entree : ["A", "B", "C", "D"]
// Sortie : [{"value":{"entree":["A","B","C","D"]},"sortie":["A","D"]}]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgXCJCXCIsIFwiQ1wiLCBcIkRcIl1cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS53aXRob3V0KFwiQlwiLFwiQ1wiKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAifQ==)

## remove

Cette fonction est assez contre-intuitive, elle supprime tous les éléments qui satisfont une condition mais renvoie le tableau des éléments supprimés.   

> [!WARNING]  
> **Fonction mutante** 
> Cette fonction modifie la structure passée en argument. 
> Dans un pipeline, cela peut entraîner des effets de bord si la valeur est réutilisée.

```js
value = get("value.entree").remove(item=>item.startsWith("B"))
// Entree : ["ABCD", "BCDE", "CDEF", "DEFG"] → Sortie : ["BCDE"]
```

⚠️ Après l'appel à `remove`, le tableau d'origine devient :  
`["ABCD", "CDEF", "DEFG"]`  

✅ Alternative non mutante recommandée : `filter`  

```js
value = get("value.entree").filter(item => item.startsWith("B"))
// Entree : [{"value":{"entree":["ABCD","BCDE","CDEF","DEFG"]}}]
// Sortie : [{"value":{"entree":["ABCD","BCDE","CDEF","DEFG"]},"sortie":["BCDE"]}]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFCQ0RcIixcbiAgICAgICAgICAgICAgICAgICAgICAgXCJCQ0RFXCIsXG4gICAgICAgICAgICAgICAgICAgICAgIFwiQ0RFRlwiLFxuICAgICAgICAgICAgICAgICAgICAgICBcIkRFRkdcIl1cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS5maWx0ZXIoaXRlbSA9PiBpdGVtLnN0YXJ0c1dpdGgoXCJCXCIpKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAgICJ9)

## reverse

Inverse l'ordre des éléments d'un tableau.

> [!WARNING]  
> **Fonction mutante** 
> Cette fonction modifie la structure passée en argument. 
> Dans un pipeline, cela peut entraîner des effets de bord si la valeur est réutilisée.  

```js
value = get("value.entree").reverse()
// Entrée : ["A", "B", "C"] → Sortie : ["C", "B", "A"]
```

⚠️ Après l'appel à `reverse`, le tableau d'origine est lui aussi inversé.  

✅ Alternative non mutante recommandée : `slice` puis `reverse`  

```js
value = get("value.entree").slice().reverse()
// Entrée : [{"value":{"entree":["A","B","C"]}}]
// Sortie : [{"value":{"entree":["A","B","C"]},"sortie":["C","B","A"]}]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIixcIkJcIixcIkNcIl1cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW2Fzc2lnbl1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLnNsaWNlKCkucmV2ZXJzZSgpXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlXG4gICAgIn0=)

## sort

Trie un tableau (attention : tri alphabétique ou par code Unicode). Pour trier des nombres voir les fonctions avancées.

```js
value = get("value.entree").sort()
// Entree : ["C", "Q", "F", "D"] → Sortie : ["C", "D", "F", "Q"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkNcIiwgXCJRXCIsIFwiRlwiLCBcIkRcIl1cbiAgICAgfVxuICAgfVxuIF0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW2Fzc2lnbl1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLnNvcnQoKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAgICJ9)

## without

Renvoie un nouveau tableau en excluant une ou plusieurs valeurs données.  
Contrairement à `pull` ou `pullAll`, `without` **ne modifie pas le tableau d’origine.**

```js
value = get("value.entree").without("B")
// Entrée : ["A", "B", "C"] → Sortie : ["A", "C"]
```

Pour exclure plusieurs valeurs : 

```js
value = get("value.entree").without("B", "C")
// Entrée : ["A", "B", "C", "D"] → Sortie : ["A", "D"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IiBbXG4gICB7XG4gICAgIFwidmFsdWVcIjoge1xuICAgICAgIFwiZW50cmVlXCI6IFtcIkFcIiwgXCJCXCIsIFwiQ1wiXVxuICAgICB9XG4gICB9XG4gXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bYXNzaWduXVxucGF0aCA9IHNvcnRpZVxudmFsdWUgPSBnZXQoXCJ2YWx1ZS5lbnRyZWVcIikud2l0aG91dChcIkFcIilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgICAifQ==)

## zip / unzip

Associe les éléments de plusieurs tableaux entre eux (zip), ou inverse l’opération (unzip).

```js
value = zip(self.value.entree,self.value.entree2)
// Peut aussi s'écrire value = get("value.entree").zip(self.value.entree2)
// Entree : ["Niels Bohr", "Albert Einstein"] Entree2 : ["Danemark", "Allemagne"] → Sortie : [["Niels Bohr","Danemark"],["Albert Einstein","Allemagne"]]

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1wiTmllbHMgQm9oclwiLCBcIkFsYmVydCBFaW5zdGVpblwiXSxcbiAgICAgIFwiZW50cmVlMlwiOiBbXCJEYW5lbWFya1wiLCBcIkFsbGVtYWduZVwiXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gemlwKHNlbGYudmFsdWUuZW50cmVlLHNlbGYudmFsdWUuZW50cmVlMilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgICAifQ==)

value = get("value.entree").unzip()
// Entree : [["Niels Bohr","Danemark"],["Albert Einstein","Allemagne"]] → Sortie : [["Niels Bohr","Albert Einstein"],["Danemark","Allemagne"]]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1tcIk5pZWxzIEJvaHJcIixcIkRhbmVtYXJrXCJdLFtcIkFsYmVydCBFaW5zdGVpblwiLFwiQWxsZW1hZ25lXCJdXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLnVuemlwKClcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgICAifQ==)

## fromPairs

Transforme un tableau de paires en un objet clé-valeur. ! Attention à toujours bien avoir des tableaux des deux éléments.

```js
value = get("value.entree").fromPairs()
// Entree : [["Niels Bohr","Danemark"],["Albert Einstein","Allemagne"]] → Sortie : {"Niels Bohr":"Danemark","Albert Einstein":"Allemagne"}
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1tcIk5pZWxzIEJvaHJcIixcIkRhbmVtYXJrXCJdLFtcIkFsYmVydCBFaW5zdGVpblwiLFwiQWxsZW1hZ25lXCJdXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLmZyb21QYWlycygpXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlXG4gICAgIn0=)

## difference

Renvoie les éléments présents dans le premier tableau mais pas dans les suivants.

```js
value = get("value.entree").difference(self.value.entree2)
// Entree : ["A", "B"] Entree2 : ["A", "C"] → Sortie : ["B"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1wiQVwiLCBcIkJcIl0sXG4gICAgICBcImVudHJlZTJcIjogW1wiQVwiLCBcIkNcIl1cbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS5kaWZmZXJlbmNlKHNlbGYudmFsdWUuZW50cmVlMilcblxuW2RlYnVnXVxudGV4dCA9IGJlZm9yZSBnZW5lcmF0aW5nIGFuIGlkZW50aWZpZXIgcGVyIG9iamVjdFxuXG5bZHVtcF1cbmluZGVudCA9IHRydWVcbiAgICAifQ==)

## intersection

Renvoie les éléments communs à plusieurs tableaux.

```js
value = get("value.entree").intersection(self.value.entree2)
// Entree : ["A", "B"] Entree2 : ["A", "C"] → Sortie : ["A"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1wiQVwiLCBcIkJcIl0sXG4gICAgICBcImVudHJlZTJcIjogW1wiQVwiLCBcIkNcIl1cbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS5pbnRlcnNlY3Rpb24oc2VsZi52YWx1ZS5lbnRyZWUyKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAgICJ9)

## xor

Renvoie les éléments **exclusifs** à chaque tableau (présents dans un seul).

```js
value = get("value.entree").xor(self.value.entree2)
// Entree : ["A", "B"] Entree2 : ["A", "C"] → Sortie : ["B", "C"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1wiQVwiLCBcIkJcIl0sXG4gICAgICBcImVudHJlZTJcIjogW1wiQVwiLCBcIkNcIl1cbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS54b3Ioc2VsZi52YWx1ZS5lbnRyZWUyKVxuXG5bZGVidWddXG50ZXh0ID0gYmVmb3JlIGdlbmVyYXRpbmcgYW4gaWRlbnRpZmllciBwZXIgb2JqZWN0XG5cbltkdW1wXVxuaW5kZW50ID0gdHJ1ZVxuICAgICJ9)

## union

Crée un tableau contenant des valeurs uniques à partir de plusieurs tableaux.

```js
value = get("value.entree").union(self.value.entree2)
// Entree : ["A", "B"] Entree2 : ["A", "C"] → Sortie : ["A", "B", "C"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1wiQVwiLCBcIkJcIl0sXG4gICAgICBcImVudHJlZTJcIjogW1wiQVwiLCBcIkNcIl1cbiAgICB9XG4gIH1cbl0iLCJzY3JpcHQiOiIjIEVaUyBzY3JpcHRcblt1c2VdXG5wbHVnaW4gPSBiYXNpY3NcblxuW0pTT05QYXJzZV1cbnNlcGFyYXRvciA9ICpcblxuW3JlcGxhY2VdXG5wYXRoID0gc29ydGllXG52YWx1ZSA9IGdldChcInZhbHVlLmVudHJlZVwiKS51bmlvbihzZWxmLnZhbHVlLmVudHJlZTIpXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlXG4gICAgIn0=)

## slice

Extrait une partie d’un tableau à partir d’un index de début jusqu’à (mais sans inclure) un index de fin.

```js
value = get("value.entree").slice(2)
// Entree : ["A", "B", "C", "D"] → Sortie : ["C","D"]
```

[Tester cet exemple dans EZS Playground](https://ezs-playground.lodex.inist.fr/?x=eyJpbnB1dCI6IltcbiAge1xuICAgIFwidmFsdWVcIjoge1xuICAgICAgXCJlbnRyZWVcIjogW1wiQVwiLCBcIkJcIiwgXCJDXCIsIFwiRFwiXVxuICAgIH1cbiAgfVxuXSIsInNjcmlwdCI6IiMgRVpTIHNjcmlwdFxuW3VzZV1cbnBsdWdpbiA9IGJhc2ljc1xuXG5bSlNPTlBhcnNlXVxuc2VwYXJhdG9yID0gKlxuXG5bcmVwbGFjZV1cbnBhdGggPSBzb3J0aWVcbnZhbHVlID0gZ2V0KFwidmFsdWUuZW50cmVlXCIpLnNsaWNlKDIpXG5cbltkZWJ1Z11cbnRleHQgPSBiZWZvcmUgZ2VuZXJhdGluZyBhbiBpZGVudGlmaWVyIHBlciBvYmplY3RcblxuW2R1bXBdXG5pbmRlbnQgPSB0cnVlXG4gICAgIn0=)

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/06-objets.md)