# Atelier Lodash (usage Lodex) 

## 📚 Sommaire

1. [Introduction](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/01-introduction.md)
2. [Comprendre les types de données et les opérateurs](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/02-types-et-operateurs.md)
3. [Syntaxe et bonnes pratiques](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/03-syntaxe-et-bonnes-pratiques.md)
4. [Manipuler des chaînes de caractères (strings)](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/04-chaines-de-caracteres.md)
5. [Manipuler des tableaux (arrays)](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/05-tableaux.md)
6. [Manipuler des objets](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/06-objets.md)
7. [Manipuler des collections](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/07-collections.md)
8. [Fonctions sur les types (lang)](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/08-fonctions-sur-les-types-lang.md)
9. [Autres fonctions utiles](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/09-autres-fonctions.md)
10. [Traitements avancés dans un loader](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/10-loader.md)
11. [Scripts avancés et cas d'usage](https://github.com/AnaelKremer/Atelier-Lodash-usage-Lodex/blob/main/11-cas-dusage.md)

---

Bienvenue dans ce dépôt qui porte mal son nom !

Ce projet est né d'ateliers que j'ai montés et animés avec @jj618 en décembre 2023 et mars 2024, à destination de nos collègues de l'Inist-CNRS.

Dans l'éventualité de les reconduire, j'ai continuellement enrichi le contenu : nouvelles fonctions, nouveaux cas d'usages, explications plus poussées... si bien qu'aujourd'hui, ce projet est très différent du premier atelier donné.

L’atelier portait spécifiquement sur l’usage de Lodash dans Lodex, notamment dans le mode enrichissement. Ce dépôt en reprend l’essence, mais va plus loin en explorant aussi l’usage avancé de Lodash et l’intégration d’instructions EZS directement dans les loaders, pour être en mesure d'écrire ses propres scripts de transformation dès l’import d'un dataset.

Cela s'apparente désormais davantage à une collection d'heuristiques qu'à un tutoriel pas-à-pas. J'essaie, tant que faire se peut, de conserver une certaine logique de progression dans le parcours, mais l'idée est surtout de mettre à disposition une large palette de scripts adaptables à une diversité de cas d'usage.

## ✍️ Origine du parcours

À l’origine, je n’avais aucune connaissance en programmation. J’ai découvert Lodash sans savoir coder en JavaScript (ni en quoi que ce soit d'autre d'ailleurs). En réalisant tout ce que Lodash permettait de faire dans Lodex, j'ai décidé d'étudier la chose "sérieusement".

> 💬 Richard Feynman :  
> *« We learn better by teaching. »*

Les traits d’esprit de Feynman m’ont toujours frappé par leur équilibre : simples en apparence, mais d’une pertinence redoutable.
Aussi, j'ai décidé de commenter systématiquement mes scripts comme si je les expliquais à quelqu’un découvrant Lodash pour la première fois. Cela m’obligeait à formuler clairement ce que je croyais avoir compris. Et j'ai rapidement réalisé que j'utilisais des fonctions Lodash sans vraiment comprendre ce qu’elles faisaient. 
Pire encore, j’ai constaté que certaines fonctions qui fonctionnaient parfaitement dans un contexte, échouaient ou produisaient des résultats inattendus dans un autre. À chaque échec, me revenait en tête une autre phrase de Feynman :

> *« If you can't explain something in simple terms, you don't understand it. »*

J’ai donc décidé de multiplier les tests, de varier les contextes, d'épelucher la documentation Lodash, de produire des exemples. Et plus que commenter mes scripts, j’ai commencé à véritablement rédiger une documentation. Petit à petit, ce travail a pris de l’ampleur, jusqu’à devenir la matière des ateliers que j’ai ensuite proposés.

## 🎯 Objectifs

Ce dépôt vous permettra de tirer pleinement parti de Lodex et des vos données grâce à Lodash et d’appliquer directement des traitements de qualité sans passer par des outils externes. 

Voici ce que vous pourrez apprendre en parcourant ce dépôt :

- identifier et comprendre les différents **types de données** manipulés (chaînes de caractères, nombres, tableaux, objets…)

- analyser la structure réelle des champs à transformer, notamment les niveaux d’imbrication (tableaux dans des objets, objets dans des tableaux, etc.)

- connaître les fonctions Lodash les plus utiles dans Lodex

- maîtriser quelques fonctions **JavaScript** utiles en complément de Lodash

- **combiner plusieurs fonctions Lodash** pour créer des enrichissements puissants

- connaître de bonnes pratiques pour anticiper les comportements inattendus et fiabiliser ses transformations

- adapter la logique des enrichissements pour les intégrer directement dans un loader pour améliorer les performances de traitement

- apprendre à rédiger un loader personnalisé pour restructurer un jeu de données 

- définir des transformations réutilisables pour éviter la répétition de code

---

🚧 Ce dépôt n’est pas figé : il est appelé à évoluer, à s’enrichir, à s’affiner. Nouvelles fonctions, nouvelles idées, cas d’usage plus complexes…

Les contributions sont donc les bienvenues ! Remarques, suggestions, corrections ou ajouts. Que ce soit pour signaler un problème, ou même proposer un exercice, n'hésitez pas à ouvrir une issue ou une pull request.

PS : 🙏 Un grand merci à @touv @jj618 & @parmentf 

