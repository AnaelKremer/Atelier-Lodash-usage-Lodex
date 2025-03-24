# Lodash - Introduction

Lodash est une bibliothèque JavaScript qui fournit des fonctions utilitaires pour les tâches de programmation courantes en utilisant le paradigme de la programmation fonctionnelle.

Programmation qui est donc centrée sur l’utilisation de fonctions (au sens mathématique), de valeurs et qui permet l’association de plusieurs fonctions (composition).

Lodash est un « fork » d’Underscore, c’est-à-dire qu’il est créé à partir du code source d’Underscore mais en devient indépendant afin de bénéficier de ses propres développements. 

A l’origine, Underscore avait été conçu pour faciliter le développement en fournissant des utilitaires permettant de travailler plus facilement avec les tableaux, les nombres, les objets et les chaînes de caractères.

Lodash est créé comme sur-ensemble (superset) d’Underscore dans le sens où il apporte plus de fonctionnalités que ce dernier. Plus lourd qu’Underscore, Lodash est aussi plus rapide, mieux adapté à l’enchaînement de fonctions et surtout, il permet de gérer des objets imbriqués (nested).

Les deux bibliothèques tirent leur nom du caractère _ , qui est la variable globale qui contient toutes les fonctions utilitaires.

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


