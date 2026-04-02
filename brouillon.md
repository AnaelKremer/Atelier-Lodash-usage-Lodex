# Transformation de données en objets LODEX (MongoDB)

## Introduction

Cette section présente comment un fichier de données structurées est lu, transformé, puis intégré dans LODEX.  

Quel que soit le format d’origine (CSV, JSON, etc.), les données suivent un processus de transformation qui permet de passer d’un ensemble de lignes ou d’objets à une structure adaptée au modèle de données de LODEX.  

Ce processus repose sur plusieurs étapes :  

- la lecture des données source
- leur transformation en objets manipulables
- leur éventuelle restructuration dans le loader
- puis leur insertion dans LODEX sous forme de documents stockés dans MongoDB.  

On obtient ainsi des objets homogènes, composés de paires clé-valeur, qui peuvent être exploités efficacement dans l’interface LODEX.  

## Du CSV à l’objet LODEX

Prenons comme exemple un fichier CSV :  

```csv
doi,title,authors,publication-year
https://doi.org/10.2307/2005041,Theory of Self-Reproducing Automata,Jacob T. Schwartz|John von Neumann|Arthur W. Burks,1967
https://doi.org/10.2307/1232672,The Theory of Games and Economic Behavior,Robert W. Harrison|John von Neumann|Oskar Morgenstern,1945
```

### 1. Interprétation du format CSV (parsing)

Lors de l’import, les données d’un fichier CSV doivent d’abord être interprétées afin de passer d’un format tabulaire à une structure manipulable.  

Cette étape est appelée **parsing**.  

Parser consiste à parcourir le contenu d’un fichier pour en analyser la structure (lignes, colonnes, séparateurs) et en extraire les informations sous une forme exploitable par un programme.

Dans le cas d’un CSV, le parsing permet de transformer un tableau de données en une série d’objets structurés.  

Pour ce faire on utilisera l'instruction EZS `[CSVParse]`

```ini
[use]
plugin = basics

[CSVParse]

[dump]
indent = true
```

Résultat :  

```JSON
[
  [
    "doi",
    "title",
    "authors",
    "publication-year"
  ],
  [
    "https://doi.org/10.2307/2005041",
    "Theory of Self-Reproducing Automata",
    "Jacob T. Schwartz|John von Neumann|Arthur W. Burks",
    "1967"
  ],
  [
    "https://doi.org/10.2307/1232672",
    "The Theory of Games and Economic Behavior",
    "Robert W. Harrison|John von Neumann|Oskar Morgenstern",
    "1945"
  ]
]
```

Le parsing transforme le fichier CSV en un tableau de tableaux :  

- la première ligne (1er tableau ici) contient les noms des colonnes
- chaque ligne suivante correspond à une ligne du fichier
- chaque cellule est représentée sous forme de chaîne de caractères.  

À ce stade :

- les données sont lues correctement
- mais elles ne sont pas encore structurées en objets clé-valeur.

### 2. Transformation en objets JSON

Pour transformer les données issues du parsing en objets structurés on utilisera `[CSVObject]` :

```ini
[use]
plugin = basics

[CSVParse]

[CSVObject]

[dump]
indent = true
```

Résultat :  

```JSON
[{
    "doi": "https://doi.org/10.2307/2005041",
    "title": "Theory of Self-Reproducing Automata",
    "authors": "Jacob T. Schwartz|John von Neumann|Arthur W. Burks",
    "publication-year": "1967"
},
{
    "doi": "https://doi.org/10.2307/1232672",
    "title": "The Theory of Games and Economic Behavior",
    "authors": "Robert W. Harrison|John von Neumann|Oskar Morgenstern",
    "publication-year": "1945"
}]
```

Le bloc `[CSVObject]` utilise la première ligne du tableau (les en-têtes) pour associer chaque valeur à une clé.

On passe ainsi :

- d’un tableau de tableaux (structure issue du parsing)
- à un tableau d’objets JSON.  

Chaque ligne du fichier devient ainsi un ensemble de paires clé-valeur, plus facilement manipulable dans le loader.

## Manipulation des données

À partir de cette étape, on peut commencer à manipuler les données sous forme d’objets.  

### 1. Accéder à une valeur

Il est possible d’accéder directement aux valeurs des champs à l’aide de fonctions comme `get()`.  

Par exemple :  

```ini
[use]
plugin = basics

[CSVParse]

[CSVObject]

[assign]
path = authorsArray
value = get("authors").split("|")

[dump]
indent = true
```

On crée ici avec `[assign]` une nouvelle clé intitulée `authorsArray`, dans laquelle on souhaite comme valeur un tableau d'auteurs constitué d'après le séparateur `|`.  

Pour accéder à la chaîne d'auteurs concaténés un simple `get("authors")` suffit.

```JSON
[{
    "doi": "https://doi.org/10.2307/2005041",
    "title": "Theory of Self-Reproducing Automata",
    "authors": "Jacob T. Schwartz|John von Neumann|Arthur W. Burks",
    "publication-year": "1967",
    "Auteurs": [
        "Jacob T. Schwartz",
        "John von Neumann",
        "Arthur W. Burks"
    ]
},
{
    "doi": "https://doi.org/10.2307/1232672",
    "title": "The Theory of Games and Economic Behavior",
    "authors": "Robert W. Harrison|John von Neumann|Oskar Morgenstern",
    "publication-year": "1945",
    "Auteurs": [
        "Robert W. Harrison",
        "John von Neumann",
        "Oskar Morgenstern"
    ]
}]
```


### 2. Accéder à l’objet complet

Si l’on souhaite manipuler l’ensemble de l’objet courant, on utilisera `self`.  

À titre d’illustration, on va modifier notre jeu de données en ajoutant une nouvelle clé contenant une copie complète de chaque objet.  

Chaque ligne sera ainsi enrichie avec un champ supplémentaire `document`, qui contiendra l’ensemble des informations de l’entrée courante. 

```ini
[use]
plugin = basics

[CSVParse]

[CSVObject]

[replace]
path = document
value = self()

[dump]
indent = true
```

👇

```JSON
[{
    "document": {
        "doi": "https://doi.org/10.2307/2005041",
        "title": "Theory of Self-Reproducing Automata",
        "authors": "Jacob T. Schwartz|John von Neumann|Arthur W. Burks",
        "publication-year": "1967"
    }
},
{
    "document": {
        "doi": "https://doi.org/10.2307/1232672",
        "title": "The Theory of Games and Economic Behavior",
        "authors": "Robert W. Harrison|John von Neumann|Oskar Morgenstern",
        "publication-year": "1945"
    }
}]
```



## Structuration pour LODEX

Une fois les données transformées et enrichies dans le loader, elles doivent être adaptées au modèle de données de LODEX afin de pouvoir être stockées et exploitées.

#### Ajout d’un identifiant logique (`uri`)

Chaque objet doit contenir une clé `uri`, qui permet d’identifier la ressource dans LODEX.

```ini
[assign]
path = uri
value = get("doi")
```

### Dans LODEX / MongoDB

Lors de l’insertion dans la base MongoDB utilisée par LODEX, un identifiant technique `_id` est ajouté automatiquement.  
Les données importées restent stockées dans la clé `value`, tandis que la clé `uri` permet d’identifier logiquement la ressource dans LODEX.

```json
{
  "_id": "69c4489a3049a2e28870097f",
  "value": {
    "uri": "uid:/cnFTQVcxp",
    "doi": "https://doi.org/10.2307/2005041",
    "title": "Theory of Self-Reproducing Automata",
    "authors": "Jacob T. Schwartz|John von Neumann|Arthur W. Burks",
    "publication-year": "1967"
  }
}
```

### Résumé

Le passage du CSV à LODEX peut être résumé ainsi :

1. le fichier CSV est lu ligne par ligne ;
2. chaque ligne est transformée en objet JSON ;
3. le loader réorganise éventuellement les données ;
4. LODEX insère l’objet dans MongoDB en ajoutant un identifiant technique `_id`.