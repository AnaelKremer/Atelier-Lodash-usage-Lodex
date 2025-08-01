# Manipuler des chaînes de caractères

De nombreux champs sont constitués de simples chaînes de caractères. Qu’il s’agisse de titres, de résumés ou de DOIs Lodash offre une panoplie de fonctions pour **nettoyer**, **transformer** ou **tester** ces données.

Cette section présente les principales fonctions Lodash utiles pour manipuler les chaînes de caractères.

> [!NOTE]
> Vous pouvez tester chaque *exemple* dans [EZS Playground](http://ezs-playground.daf.intra.inist.fr/). Ecrivez l'input de cette façon `[{ "value":{"entree":"Turing complete"}}]` (et changez la valeur de *entree*) puis collez ce script que vous adapterez.
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
> path = sortie
> value = get("value.entree").words()
> 
> [dump]
> indent = true
> ```

## trim

Supprime les espaces (et caractères invisibles) en début et fin de chaîne.  
  *Exemple :*

  ```js
  value = get("value.entree").trim()
  // Entrée : "    Ceci est une phrase avec trop d'espaces.  " → Sortie : "Ceci est une phrase avec trop d'espaces."
  ```

## deburr

Supprime les accents et diacritiques.  
  *Exemple :*

  ```js
  value = get("value.entree").deburr()
  // Entrée : "éèêëàáâäîïìíôöòóùúûüçñãõčšž" → Sortie : "eeeeaaaaiiiioooouuuucnaocsz"
  ```

## escape / unescape

Encode ou décode les caractères HTML : `&`, `<`, `>`...  
  *Exemple :*

    ```js
  value = get("value.entree").escape()
  // Entrée : "<script>" → Sortie : "&lt;script&gt;"
  ```
  
  *Exemple :*

    ```js
  value = get("value.entree").unescape()
  // Entrée : "&amp;" → Sortie : "&"
  ```
## capitalize

Met en majuscule la première lettre d’une chaîne.
  *Exemple :*

  ```js
  value = get("value.entree").capitalize()
  // Entrée : "ceci est une chaîne." → Sortie : "Ceci est une chaîne."
  ```

## toLower / lowerCase

`toLower` convertit une chaîne en minuscules.
  *Exemple :*

  ```js
  value = get("value.entree").toLower()
  // Entrée : "John von Neumann" → Sortie : "john von neumann"
  ```

`lowerCase` convertit une chaîne, considérée comme une suite de mots séparés par des espaces, en minuscules. La fonction reformate la chaîne en ajoutant des espaces là où il détecte des mots (basés sur la casse, les tirets ou underscores...)
  *Exemple :*

  ```js
  value = get("value.entree").lowerCase()
  // Entrée : "PascalCase_ou_camelCase_?" → Sortie : "pascal case ou camel case"
  ```

## toUpper / upperCase

`toUpper` convertit une chaîne en majuscules.
  *Exemple :*

  ```js
  value = get("value.entree").toUpper()
  // Entrée : "Richard Feynman" → Sortie : "RICHARD FEYNMAN"
  ```

`.upperCase` convertit une chaîne, considérée comme une suite de mots séparés par des espaces, en majuscules. La fonction reformate la chaîne en ajoutant des espaces là où il détecte des mots (basés sur la casse, les tirets ou underscores...)
  *Exemple :*

  ```js
  value = get("value.entree").upperCase()
  // Entrée : "snakeCase_or_kebabCase_?" → Sortie : "SNAKE CASE OR KEBAB CASE"
  ```

## startCase

Convertit une chaîne en mots séparés par des espaces, avec une majuscule au début de chaque mot.
  *Exemple :*

  ```js
  value = get("value.entree").startCase()
  // Entrée : "MANIAC : mathematical analyzer, numerical integrator, and computer" → Sortie : "MANIAC Mathematical Analyzer Numerical Integrator And Computer"
  ```
## words

Coupe une chaîne en un tableau de mots, en tenant compte des majuscules et séparateurs.
  *Exemple :*

  ```js
  value = get("value.entree").words()
  // Entrée : "Turing complete" → Sortie : ["Turing","complete"]
  ```

## split

Sépare une chaîne en tableau selon un séparateur.
  *Exemple :*

  ```js
  value = get("value.entree").split(";")
  // Entrée : "A;B;C" → Sortie : ["A","B","C"]
  ```

## startsWith

Vérifie si une chaîne commence par un motif donné (renvoie un booléen).
  *Exemple :*

  ```js
    value = get("value.entree").startsWith("10.")
  // Entrée : "10.1093/femsec/fiz063" → Sortie : true
  ```

## endsWith

Vérifie si une chaîne se termine par un motif donné (renvoie un booléen).
  *Exemple :*

  ```js
    value = get("value.entree").endsWith(".pdf")
  // Entrée : "http://drehu.linguist.univ-paris-diderot.fr/ismo-2019/fichiers/abstracts/ISMo_2019_paper_41.pdf" → Sortie : true
  ```

## prepend (EZS)

Ajoute un préfixe dans une chaîne.
  *Exemple :*

  ```js
    value = get("value.entree").prepend("www.")
  // Entrée : "lodex" → Sortie : "www.lodex"
  ```

## append (EZS)

Ajoute un suffixe dans une chaîne.
  *Exemple :*

  ```js
    value = get("value.entree").append(".fr")
  // Entrée : "www.lodex" → Sortie : "www.lodex.fr"
  ```