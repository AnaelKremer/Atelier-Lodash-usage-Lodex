# Lodash - Introduction

Lodash est une bibliothèque JavaScript qui fournit des fonctions utilitaires pour les tâches de programmation courantes en utilisant le paradigme de la programmation fonctionnelle.

Programmation qui est donc centrée sur l’utilisation de fonctions (au sens mathématique), de valeurs et qui permet l’association de plusieurs fonctions (composition).

Lodash est un « fork » d’Underscore, c’est-à-dire qu’il est créé à partir du code source d’Underscore mais en devient indépendant afin de bénéficier de ses propres développements. 

A l’origine, Underscore avait été conçu pour faciliter le développement en fournissant des utilitaires permettant de travailler plus facilement avec les tableaux, les nombres, les objets et les chaînes de caractères.

Lodash est créé comme sur-ensemble (superset) d’Underscore dans le sens où il apporte plus de fonctionnalités que ce dernier. Plus lourd qu’Underscore, Lodash est aussi plus rapide, mieux adapté à l’enchaînement de fonctions et surtout, il permet de gérer des objets imbriqués (nested).

Les deux bibliothèques tirent leur nom du caractère ```_``` , qui est la variable globale qui contient toutes les fonctions utilitaires.

## Une fonction Lodash classique

Une fonction (ou méthode) Lodash doit toujours déclarer des arguments dans un ordre spécifique.

```
_.replace("abc","abc","hello world")
```
* Le 1er argument déclaré ici est la chaîne de caractères à modifier, en l'occurrence "abc".
* Le 2ème argument est le motif à remplacer dans la chaîne de caractères, c'est aussi "abc".
* Le 3ème argument, "hello world", correspond à la valeur de remplacement du motif trouvé dans la chaîne de caractères.

Ainsi cette fonction replace, remplace la chaîne de caractère "abc" par "hello world".

## Un enchaînement de fonctions Lodash

Afin d'effectuer plusieurs opérations sur des données, il convient de créer une chaîne de méthodes.

```
_.chain("abc")
 .replace("abc","hello world")
 .capitalize()
 .split(" ")
 .push("!")
 .join(" ")
 .value()
```

* Pour ce faire, il faut débuter par la méthode ```_.chain()``` et y déclarer les données à modifier (ici la chaîne "abc").
* On peut ensuite ajouter les différentes méthodes souhaitées.
* Puis on parachève le tout avec ```_.value()```. Cette fonction est **obligatoire** lorsque l'on utilise ```_.chain()```, elle sert à exécuter la chaîne Lodash et récupérer le résultat.

## Ecrire du Lodash dans Lodex

Dans Lodex, les scripts peuvent s'écrire dans le mode enrichissement d'une instance ou dans un Loader au moment de charger ses données (nous verrons les différeces entre ces 2 usages plus tard).
Dans ces cas-là, plusieurs petits programmes entrent en jeu afin de nous faciliter la tâche.
En effet, le chaînage des méthodes y est rendu possible par défaut. Les méthodes ```_.chain``` et ```_.value``` sont actives mais pas visibles. Ce qui permet d'éccire l'enchaînnement de méthodes vu précédement de cette façon :
```
[assign]
path = value
value = get("value.colonneA").replace("abc","hello world").capitalize().split(" ").push("!").join(" ")
```
Autre différence notable, les données à traiter sont déclarées avec **path**, **value** et **get** ici.
Ce qui explique pourquoi il n'y a que 2 arguments déclarés dans ```.replace()``` et non plus 3 comme nous l'avons vu précédemment.
