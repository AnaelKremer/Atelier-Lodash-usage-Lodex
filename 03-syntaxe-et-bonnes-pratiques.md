# Les principes de base pour structurer ses transformations

Avant même d’écrire ses premières transformations dans Lodex, il est essentiel de comprendre quelques notions clés.  
Cette section revient sur les concepts fondamentaux qui régissent la logique des enrichissements, et donne quelques bases pour adopter les bons réflexes dès le départ.

## Les fondamentaux de value et self

Les enrichissements ou traitements de données peuvent se faire dans Lodex ou dans un Loader à l’import d’un jeu de données. La syntaxe à employer est légèrement différente selon que l’on écrit des fonctions dans l’un ou dans l’autre.  
Dans un Loader, pour créer une nouvelle colonne, on utilise l’instruction `[assign]`, suivie du nom de la colonne dans `path`, puis de la ou des valeurs que l’on souhaite transformer dans `value`.

```js
[assign]
path = allKeywords
value = get('keywordsAuthor').concat(self.keywordsMesh)
```

Dans Lodex en revanche, c’est le nom de l’enrichissement qui fait office de nom de colonne. On assigne systématiquement `value` à `path`, enfin on préfixera toujours le nom du champ à récupérer par `value.`.

```js
[assign]
path = value
value = get('value.keywordsAuthor')
```

Le terme `value` désigne ici non pas une valeur, mais le dataset complet ! Ainsi *value.keywordsAuthor* désigne le chemin, à savoir le champ *keywordsAuthor* contenu dans le dataset (les points caractérisent une imbrication).

Cela signifie que dans le contexte de notre colonne, nous avons importé l’intégralité du dataset par le biais de `get('value')`, que nous avons écrasé par le seul champ *keywordsAuthor*.
Autrement dit, au moment où Lodex exécute cette instruction, `value` devient égal à *keywordsAuthor*.  

C’est pourquoi lorsque l’on souhaite concaténer un autre champ on ne peut plus écrire simplement `concat('value.keywordsMesh')`, value ne contenant plus que les mots-clés d’auteurs.  

Il faut dans ce cas utiliser `self` (et enlever les quotes par la même occasion).

```js
[assign]
path = value
value = get('value.keywordsAuthor').concat(self.value.keywordsMesh)
```

`self` est ici une référence constante à l’objet initial, `self.value` permet donc de ré-interroger l’intégralité du dataset tel qu’il est avant la création de la nouvelle colonne en cours.

En conclusion
- `value` fait référence à la **valeur courante** que vous êtes en train de transformer.
- `self.value` permet de **revenir à l’objet de départ**, tel qu’il était avant la transformation.

---

## Le nommage des colonnes/enrichissements

Dans Lodex, le nom de l'enrichissement devient le nom de la colonne. Il est donc crucial d'y penser dès le départ.  

Un bon nommage est essentiel pour maintenir des transformations lisibles, compréhensibles et réutilisables.  

### Éviter les caractères spéciaux et les espaces

Tout d'abord, il convient d'éviter les caractères spéciaux ainsi que les espaces. Cela peut rendre plus complexe l'accès aux champs.  

A titre d'exemple, on souhaite concaténer un champ que l'on avait nommé **Titre de l'article**.

`value = get('value.publicationYear').concat(self.value.Titre de l'article)`  

Ici, Lodex n'arrivera pas à récupérer le champ, d'une part à cause des espaces, et d'autre part parce que l’apostrophe est interprétée comme un délimiteur de chaîne.
Essayer de résoudre le problème en tentant d'échapper l'apostrophe par `\` ne suffira pas.  

S'il vous arrive d'avoir des données d'origine avec des champs nommé de la sorte voici deux solutions pour récupérer les champs :
- `.concat(_.get(self, "value.Titre de l\'article"))`
- `.concat(self.value['Titre de l\'article'])`

On peut constater que c'est tout de suite moins lisible que la **notation pointée** habituelle `self.value.KeywordsMesh`.

### Adopter une convention de nommage

Le meilleur moyen de bien nommer ses enrichissements est de s'appuyer sur une convention de nommage.  

On conseillera le **camelCase**, où le premier mot débute par une minuscule, puis chaque mot suivant commence par une majuscule.
 - `authorName`
 - `publicationYear`
 - `keywordsMesh`...

On peut aussi opter pour le **PascalCase** où chaque mot commence par une majuscule, y compris le premier.
 - `AuthorName`
 - `PublicationYear`
 - `KeywordsMesh`...

**Quoi qu’il en soit, évitez de créer vos propres conventions de nommage !**

Même si vous travaillez seul·e sur un projet, il est important d’adopter des règles de nommage cohérentes et standardisées. Cela facilitera la lecture de vos scripts (même plusieurs semaines plus tard), la collaboration avec d’autres personnes (même occasionnelle) le réemploi de vos traitements.  

Un exemple observé : `APILpublicationDate`, `APILsource`, `APILpublisher` etc.

Ici le nommage navigue entre le Pascal et le camelCase et surtout, un préfixe est utilisé dans tous les noms. Il est donc superflu, mais aussi obscur pour qui ne sait ce que signifie ce préfixe.

### Choisir des noms significatifs

Un nom bien choisi et mûrement réfléchi est déjà une documentation en soi !

On empruntera quelques conseils à l'[oncle Bob](https://fr.wikipedia.org/wiki/Robert_C._Martin) pour savoir comment bien nommer ses enrichissements :

- Le nom d’une fonction ou d’une variable (un enrichissement dan notre cas) doit dire ce qu’elle fait.
- Eviter les mots parasites.
- Privilégier les noms longs.
- Eviter la désinformation (ex : pour des valeurs comme *1997* ou *2025*, il est plus exacte de nommer le champ `publicationYear` que `publicationDate`).
- Eviter les préfixes ou suffixes.
- Eviter les associations mentales.
- Donner des noms techniques, un code fortement lié à un domaine doit employer des noms de ce domaine.
- Ne pas ajouter de contexte inutile (comme l'exemple *APIL* vu plus haut).

On doit pouvoir distinguer les données brutes des données transformées : `keywords` VS `keywordsDeburred` (pour le champ nettoyé à l'aide de la fonction `deburr`).  

💡 Un autre avantage à adopter des noms explicites et cohérents : cela vous évite d’avoir à vérifier sans cesse comment vous avez nommé vos colonnes. En faisant un nouvel enrichissement, vous savez instinctivement comment appeler les colonnes dont vous avez besoin, sans avoir à fouiller dans les données ou dans les scripts.

---


## Le nommage des variables utilisées

Vous aurez sans doute recours aux fonctions fléchées dans vos traitements et vous devrez vous-même nommer les variables.  

Dans de nombreux exemples, on rencontre des fonctions anonymes définies à l’aide de la syntaxe =>, dans lesquelles les variables sont souvent nommés arbitrairement :  

```js
.filter(x => ['France'].includes(x))
.map(v => String(v).trim())
.map((item, i) => `${i+1} - ${item}`)
.filter(value=> !value.startsWith("E-mail"))
```

Le dernier exemple en particulier peut prêter à confusion, puisque `value` ne désigne ici pas le dataset comme expliqué précédemment, mais est simplement un nommage arbitraire pour désigner les valeurs ou éléments d’un tableau.  
`v`, `x` et `item` représentent la même chose.

Lorsqu’une fonction a 2 arguments comme `map((item, i)`, le 1er argument est toujours une valeur et le second est sa position dans le tableau. `i` est donc ici la contraction d’`index`.

Ainsi, pour une meilleure lisibilité, il est conseillé de nommer ses variables en fonction de ce qu’elles représentent : 

```js
.filter(country => ['France'].includes(country))
.map(title => String(title).trim())
.map((catégorie, index) => `${index+1} - ${catégorie}`)
.filter(address=> !address.startsWith("E-mail"))
```

Cela facilite nettement la lisibilité du code et *"en rendant un code facile à lire, on le rend plus facile à écrire"*.

---


## Mise en forme et commentaires

### Formatage du code

Soigner la mise en forme de son code est une habitude précieuse, surtout lorsque les transformations deviennent complexes.  

Quand une transformation implique plusieurs opérations chaînées, pensez à sauter des lignes pour chaque étape. Cela rend le script plus lisible, plus maintenable, et facilite les tests ou les corrections futures.  

Exemple en une ligne :
```js
value = get("documentType").thru(arr => arr.length === 1 ? "Non concerné (document unique)" : _.chain(arr).uniq().size().thru(n => n === 1 ? "Types de document identiques" : "Types de document différents").value())
```
  
Exemple aéré et indenté :   
```js
value = get("documentType") \
    .thru(arr => \
        arr.length === 1 \
            ? "Non concerné (document unique)" \
            : _.chain(arr) \
                .uniq() \
                .size() \
                .thru(n => \
                    n === 1 \
                        ? "Types de document identiques" \
                        : "Types de document différents" \
                ) \
                .value() \
    )
```

### Commenter son code

> 💬 Robert C. Martin :  
> *« Le meilleur commentaire est celui que l'on peut éviter d'écrire. »*

Un bon code parle de lui-même, mais ce n’est pas toujours possible — surtout lorsque l’on débute.  

N’hésitez donc pas à ajouter des commentaires chaque fois que cela vous semble utile : ils seront précieux lorsque vous relirez votre code plus tard.  

Il peut notamment être pertinent de commenter pour expliquer les raisons d’une décision, par exemple un choix métier ou une règle spécifique appliquée.

```js
[assign]
path = simplifiedDocumentType
; Choix métier : on regroupe tous les types de documents rares sous l'étiquette 'Autres'
value = get('value.documentType') \
    .thru(type => ["Article", "Chapitre", "Ouvrage"].includes(type) ? type : "Autres")
```

Une transformation peut s'avérer complexe ou inexplicable à première vue en raison de la structure des données (champs absents, formats incohérents etc.). Un commentaire s'avère donc nécessaire dans ce cas. Mais il faut toujurs garder à l'esprit qu'un bon commentaire explique un **pourquoi** et pas juste ce que fait le code.  

👉 [Chapitre suivant](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/04-chaines-de-caracteres.md)