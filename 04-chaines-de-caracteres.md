# Manipuler des chaînes de caractères

De nombreux champs sont constitués de simples chaînes de caractères. Qu’il s’agisse de titres, de résumés ou de DOIs, Lodash offre une panoplie de fonctions pour **nettoyer**, **transformer** ou **tester** ces données.

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

  ```js
  value = get("value.entree").trim()
  // Entree : "    Ceci est une phrase avec trop d'espaces.  " → Sortie : "Ceci est une phrase avec trop d'espaces."
  ```

## deburr

Supprime les accents et diacritiques.  

  ```js
  value = get("value.entree").deburr()
  // Entree : "éèêëàáâäîïìíôöòóùúûüçñãõčšž" → Sortie : "eeeeaaaaiiiioooouuuucnaocsz"
  ```

## escape / unescape

Encode ou décode les caractères HTML : `&`, `<`, `>`...  

  ```js
  value = get("value.entree").escape()
  // Entree : "<script>" → Sortie : "&lt;script&gt;"
  ```

  ```js
  value = get("value.entree").unescape()
  // Entree : "&amp;" → Sortie : "&"
  ```
## capitalize

Met en majuscule la première lettre d’une chaîne.  

  ```js
  value = get("value.entree").capitalize()
  // Entree : "ceci est une chaîne." → Sortie : "Ceci est une chaîne."
  ```

## toLower / lowerCase

`toLower` convertit une chaîne en minuscules.  

  ```js
  value = get("value.entree").toLower()
  // Entree : "John von Neumann" → Sortie : "john von neumann"
  ```

`lowerCase` convertit une chaîne, considérée comme une suite de mots séparés par des espaces, en minuscules. La fonction reformate la chaîne en ajoutant des espaces là où il détecte des mots (basés sur la casse, les tirets ou underscores...)  

  ```js
  value = get("value.entree").lowerCase()
  // Entree : "PascalCase_ou_camelCase_?" → Sortie : "pascal case ou camel case"
  ```

## toUpper / upperCase

`toUpper` convertit une chaîne en majuscules.  

  ```js
  value = get("value.entree").toUpper()
  // Entree : "Richard Feynman" → Sortie : "RICHARD FEYNMAN"
  ```

`.upperCase` convertit une chaîne, considérée comme une suite de mots séparés par des espaces, en majuscules. La fonction reformate la chaîne en ajoutant des espaces là où il détecte des mots (basés sur la casse, les tirets ou underscores...)  

  ```js
  value = get("value.entree").upperCase()
  // Entree : "snakeCase_or_kebabCase_?" → Sortie : "SNAKE CASE OR KEBAB CASE"
  ```

## startCase

Convertit une chaîne en mots séparés par des espaces, avec une majuscule au début de chaque mot.  

  ```js
  value = get("value.entree").startCase()
  // Entree : "MANIAC : mathematical analyzer, numerical integrator, and computer" → Sortie : "MANIAC Mathematical Analyzer Numerical Integrator And Computer"
  ```
## words

Coupe une chaîne en un tableau de mots, en tenant compte des majuscules et séparateurs.  

  ```js
  value = get("value.entree").words()
  // Entree : "Turing complete" → Sortie : ["Turing","complete"]
  ```

## split

Sépare une chaîne en tableau selon un séparateur.  

  ```js
  value = get("value.entree").split(";")
  // Entree : "A;B;C" → Sortie : ["A","B","C"]
  ```

## startsWith

Vérifie si une chaîne commence par un motif donné (renvoie un booléen).  

  ```js
  value = get("value.entree").startsWith("10.")
  // Entree : "10.1093/femsec/fiz063" → Sortie : true
  ```

## endsWith

Vérifie si une chaîne se termine par un motif donné (renvoie un booléen).  

  ```js
  value = get("value.entree").endsWith(".pdf")
  // Entree : "http://drehu.linguist.univ-paris-diderot.fr/ismo-2019/fichiers/abstracts/ISMo_2019_paper_41.pdf" → Sortie : true
  ```

## prepend (EZS)

Ajoute un préfixe dans une chaîne.  

  ```js
  value = get("value.entree").prepend("www.")
  // Entree : "lodex" → Sortie : "www.lodex"
  ```

## append (EZS)

Ajoute un suffixe dans une chaîne.  

  ```js
  value = get("value.entree").append(".fr")
  // Entree : "www.lodex" → Sortie : "www.lodex.fr"
  ```

## replace

Remplace un motif par un autre.

  ```js
  value = get("value.entree").replace("ant","insect")
  // Entree : "ant" → Sortie : "insect"
  ```

> [!WARNING]
> Quand on débute, on a souvent recours à `replace`, qui semble simple et pratique pour remplacer des termes à la volée… mais dont l’usage abusif ou mal maîtrisé peut entraîner des effets de bord difficiles à détecter.

Un de ces effets de bord est la **collision de termes**, que l’on peut rapidement rencontrer dès lors que l’on souhaite effectuer plusieurs remplacements.  
Reprenons le même exemple, on souhaite maintenant ajouter le remplacement  d'*antelope* par *mammal* : 

  ```js
  value = get("value.entree").replace("ant","insect").replace("antelope", "mammal")
  // Entree : "ant" → Sortie : "insect"
  // Entree : "antelope" → Sortie : "insectelope"
  ```

Lorsque vous effectuez plusieurs replace à la suite, les remplacements se font **les uns après les autres**, sur la chaîne modifiée à chaque étape.  
Le second `replace` ne se fait donc pas car *antelope* qui contient *ant* a été remplacé par *insectelope* !  

Pour que les `replace` fonctionnent, il faudrait inverser leur ordre. Cela devient donc très compliqué d'anticiper tous les cas de chevauchement dès lors que l'on a beaucoup de motifs à remplacer. Sans parler du fait que le code va devenir difficile à lire et très lent à éxecuter...

> [!NOTE]
> D'une façon générale, pensez au principe DRY : *Don't Repeat Yourself*  
> Si vous avez au moins 3 `replace` dans votre script, c'est le moment d'utiliser autre chose !


Autre effet de bord, `replace` ne remplace **que la première occurence** d'un motif dans une chaîne de caractères.  
Attention donc si vous manipulez des chaînes qui contiennent toujours un seul mot ou plusieurs. 

  ```js
  value = get("value.entree").replace("ant","insect")
  // Entree : "The ant found another ant" → Sortie : "The insect found another ant"
  ```

Pour remplacer plusieurs fois le même terme il faut utiliser une expression régulière d'une part, et le flag "g" d'autre part pour "global".

  ```js
  value = get("value.entree").replace(/ant/g,"insect")
  // Entree : "The ant found another ant" → Sortie : "The insect found another insect"
  ```

💡 Il existe diverses méthodes pour effectuer de multiples remplacements, qui sont plus explicites, plus rapides, plus flexibles et surtout plus sécurisées. Voir [thru](), [env]() ou...**TODO**

> [!TIP]
> Personnellement, je n’utilise `replace` qu'avec des *regex* et lorsque c’est la seule solution possible, par exemple pour supprimer des espaces superflus.
> `.replace(/\s+/g, " ")`
> Pour tout le reste (corrections, renommages, équivalences), je préfère `thru` ou `env`, qui évitent les effets indésirables et facilitent la maintenance.

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/05-tableaux.md)