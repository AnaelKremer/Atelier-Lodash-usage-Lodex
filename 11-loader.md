# Traitements avancés dans un loader

## Fonctionnement d'un loader

## Les instructions

## ...

### Définir des fonctions réutilisables

On a déjà vu qu'avec l'instruction `[ENV]` il était possible de stocker un dictionnaire que l'on pouvait réutiliser.  

De la même manière, `[ENV]` peut contenir des **fonctions utilitaires**, que l'on définit une fois et que l'on réutilise ensuite à plusieurs endroits du pipeline.  

Par exemple, on a un jeu de données qui va contenir beaucoup de *booléens* (is openacces, has repository, is retracted etc). Dans Lodex on souhaiterait afficher des "Oui" ou "Non" plutôt que les *booléens*.  

On va d'abord écire notre fonction dans `[ENV]` et la nommer correctement (c'est souvent la partie la plus compliquée).

```js
[env]
path = labelizeBoolean
value = fix((bool)=> bool === true ? "Oui" : bool === false ? "Non" : "Inconnu")
```

On utilise `fix` pour déclarer notre fonction, puis on écrit une simple *ternaire*.  

Il suffit ensuite dans un `[assign]` de déclarer le champ que l'on veut transformer, puis d'appeler la *variable d'environnement* dans un `thru`.

```js
[assign]
path = value
value = get("isRetracted").thru(env("labelizeBoolean"))
// Entrée : true → Sortie : "Oui"
```

Et si notre champ ne contient pas un seul, mais plusieurs *booléens* dans un tableau, il suffit de remplacer `thru` par `map` !

```js
[assign]
path = value
value = get("isCnrs").map(env("labelizeBoolean"))
// Entrée :  [false, true, true, false] → Sortie : ["Non","Oui","Oui","Non"]
```

Une fois cette logique acquise, on peut aller encore plus loin et se lancer dans **la composition de fonctions**.  

Plutôt que d’écrire de longs traitements redondants, on crée de petites briques spécialisées (nettoyer une chaîne, nettoyer un tableau), puis on les enchaîne et les combine pour construire des transformations plus complexes.  
