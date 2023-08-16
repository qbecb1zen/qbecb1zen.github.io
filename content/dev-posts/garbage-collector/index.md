---
title: "Garbage Collection en JavaScript : Une plongée profonde dans la gestion de la mémoire"
summary: "Un examen technique approfondi du Garbage Collection en JavaScript, en comparant les mécanismes de gestion de la mémoire avec des langages de bas niveau comme le C, et en explorant les détails internes du ramasse-miettes."
categories: ["Blog", "Développement Web"]
tags: ["JavaScript", "Garbage Collection", "Mémoire", "Optimisation", "TypeScript"]
showSummary: true
date: 2023-08-16
draft: false
---

# Garbage Collection en JavaScript

## Une brève introduction à la gestion de la mémoire

### Définition 
Le Garbage Collector en JavaScript est un mécanisme qui libère automatiquement la mémoire qui n'est plus utilisée par le programme. Lorsqu'un programme crée des objets ou des variables, il réserve de la mémoire dans la RAM de l'ordinateur pour les stocker. Avec le temps, la mémoire utilisée par le programme peut augmenter, ce qui peut causer des problèmes de performance ou même faire planter le programme. Le Garbage Collector résout ce problème en identifiant et en supprimant les objets et les variables qui ne sont plus utilisés.

### Pourquoi le Garbage Collection ?

La gestion manuelle de la mémoire est source d'erreurs. Les fuites de mémoire (lorsque la mémoire n'est pas libérée) et les accès à des zones de mémoire déjà libérées sont des erreurs courantes qui peuvent être difficiles à détecter et à corriger. Le GC aide à prévenir ces erreurs en automatisant la gestion de la mémoire.



### Gestion de la mémoire en C vs JavaScript
En C, la gestion de la mémoire est manuelle. Lorsque vous allouez de la mémoire avec malloc(), vous devez explicitement la libérer avec free(). Si vous oubliez, cela entraîne une fuite de mémoire. En revanche, JavaScript automatise ce processus avec le Garbage Collection (GC).

```C
typedef struct {
    char key[10];
    char value[10];
} Object;

Object* createObject() {
    Object* obj = (Object*)malloc(sizeof(Object));
    strcpy(obj->key, "key");
    strcpy(obj->value, "value");
    return obj;
}

int main() {
    Object* obj = createObject();
    // ... utilisation de l'objet ...
    free(obj); // Libération manuelle de la mémoire
    return 0;
}
```

Alors qu'en JavaScript, la mémoire est automatiquement libérée lorsque les objets ne sont plus référencés.

```javascript
function createObject() {
    return { key: "key", value: "value" };
}

let obj = createObject();
// ... utilisation de l'objet ...
obj = null; // Indication pour le Garbage Collector que l'objet n'est plus nécessaire
```

Dans cet exemple, en **C**, nous définissons une structure `Object` et allouons de la mémoire pour elle avec `malloc()`. Après avoir utilisé l'objet, nous devons explicitement libérer la mémoire avec `free()`. En revanche, en JavaScript, après avoir utilisé l'objet, nous pouvons simplement déréférencer l'objet en affectant `null` à la variable, et le Garbage Collector se chargera de libérer la mémoire pour nous.


### Stack vs Heap

La mémoire en informatique est divisée en deux zones principales : la stack (pile) et le heap (tas). La stack est une zone de mémoire organisée de manière LIFO (Last In, First Out) où les données sont stockées et récupérées en suivant l'ordre d'arrivée. Elle est utilisée pour stocker des variables primitives et des pointeurs vers des objets. Le heap, en revanche, est une zone de mémoire moins organisée où les objets sont stockés.

En JavaScript, lorsque vous déclarez une variable primitive (comme un `int` ou un `bool`), elle est stockée dans la stack. Mais quand vous créez un objet, il est stocké dans le heap, et seul un pointeur vers cet objet est stocké dans la stack.

**N.B** :
La différence entre variable primitive et Objet est qu'une variable primitive sont de type natif au langage, par exemple en `Javascript` il s'agit de `boolean`, `number`, `string`, `null`, `bigint`, `symbol` ou `undefined`, alors qu'un objet c'est tout ce qui n'est pas un type primitif est considéré comme un objet en JavaScript. Cela inclut les fonctions, les tableaux, les dates, les expressions régulières et, bien sûr, les objets littéraux.

## Comment fonctionne le Garbage collection en JavaScript ?

### Algorithme de base : Référence comptage

L'une des approches les plus simples pour le GC est le "comptage de référence". L'idée est simple : chaque objet a un compteur qui garde une trace du nombre de références à cet objet. Lorsque cet objet n'est plus référencé, le compteur atteint zéro et la mémoire peut être libérée.

Cependant, cette approche a un inconvénient majeur : elle ne peut pas gérer les références circulaires. Par exemple, si deux objets se référencent mutuellement, leur compteur ne sera jamais zéro, même s'ils ne sont plus utilisés ailleurs dans le programme.

![](./counting-gc.jpg "Référence comptage")

### Algorithme de marquage et de balayage (Actuel)
JavaScript utilise principalement cet algorithme pour le GC. L'idée est de "marquer" tous les objets vivants, puis de "balayer" et de libérer la mémoire des objets non marqués.

![](./initial-gc.svg "Mark-Sweep initial")

Lors de la phase de marquage, le GC commence par les "racines", qui sont généralement des objets globaux accessibles directement (comme window en navigateur). Il marque ces objets, puis parcourt leurs références, marquant chaque objet rencontré. Ce processus se répète récursivement jusqu'à ce que tous les objets accessibles soient marqués.

Ensuite, lors de la phase de balayage, le GC parcourt tous les objets et libère la mémoire des objets non marqués. (non accessible)

![](./final-gc.svg "Mark-Sweep suppression")

Cet algorithme est appelé **Mark-Sweep** où le GC commence par un ensemble de "racines" (généralement l'objet global et tout objet ou variable actuellement utilisé). L'algorithme parcourt ensuite récursivement le graphe d'objets, marquant tous les objets et variables encore utilisés. Une fois tous les objets vivants marqués, le Garbage Collector supprime tous les objets non marqués.

### Cycle de collecte
Le GC ne s'exécute pas en continu. Il fonctionne par cycles, en fonction de l'allocation de la mémoire, des besoins du programme et d'autres facteurs. Pendant un cycle de collecte, le GC marquera et balayera la mémoire, libérant ainsi l'espace inutilisé.


## Fuites de mémoire et références circulaires
Même avec le GC, les fuites de mémoire peuvent encore se produire en JavaScript, en particulier à cause des références circulaires. Considérez l'exemple suivant :

```javascript
function fuiteDeMemoire() {
  let obj1 = {};
  let obj2 = {};
  
  obj1.a = obj2; // obj1 référence obj2
  obj2.a = obj1; // obj2 référence obj1
}

fuiteDeMemoire();
```

Ici, `obj1` et `obj2` se référencent mutuellement, créant une référence circulaire. Même si ces objets ne sont plus accessibles après l'exécution de la fonction, ils ne sont pas collectés par le GC car ils se référencent toujours mutuellement.


## Optimisation de la gestion de la mémoire en JavaScript

### Utilisation explicite de `null`
L'une des meilleures pratiques pour aider le GC est de déréférencer explicitement les objets lorsque vous n'en avez plus besoin.

```javascript
let grosObjet = { /* ... des données volumineuses ... */ };
// ... utilisation de grosObjet ...
grosObjet = null;
```

### Éviter les fermetures globales
Les fermetures (ou closures en anglais) sont une caractéristique puissante de JavaScript qui permet à une fonction d'avoir accès à son environnement lexical externe. Cependant, si elles sont utilisées de manière excessive ou inappropriée, en particulier au niveau global, elles peuvent entraîner des fuites de mémoire car elles retiennent des références à des objets qui ne sont plus nécessaires.

```javascript
let data = fetchData(); // Supposons que cette fonction récupère des données

window.processData = function() {
    // Utilise les données
    console.log(data);
}
```
Dans cet exemple, la fonction `processData` est une fermeture globale qui retient une référence à `data`. Même si vous n'avez plus besoin de `data` ailleurs, elle ne sera pas récupérée par le garbage collection tant que `processData` existe.


### Utiliser des outils de profilage
Les outils de profilage, comme ceux disponibles dans les DevTools des navigateurs modernes, peuvent aider à identifier les fuites de mémoire, les zones de code gourmandes en ressources et d'autres problèmes de performance. Ils offrent des visualisations et des analyses détaillées de l'utilisation de la mémoire, des appels de fonctions, et plus encore.

Par exemple on peut utiliser les DevTools de Chrome : il suffit d'aller dans l'onglet "Memory" et prendre un instantané de la mémoire. on peut ensuite inspecter les objets, voir leur taille et identifier d'éventuelles fuites de mémoire.


### Utiliser des variables locales
Les variables locales sont détruites une fois que la fonction qui les contient a terminé son exécution. Cela aide à libérer de la mémoire plus rapidement. En revanche, les variables globales persistent jusqu'à ce que la page soit fermée ou rechargée, ce qui peut entraîner une consommation inutile de mémoire.

```javascript
function processData() {
    let localData = fetchData(); // Variable locale
    console.log(localData);
}
```

Dans cet exemple, une fois que `processData` a terminé son exécution, `localData` est éligible pour le garbage collection.

### Nettoyer les écouteurs d'événements

Si vous ajoutez des écouteurs d'événements à des éléments DOM, il est essentiel de les supprimer lorsque vous n'en avez plus besoin, en particulier pour les éléments qui sont supprimés du DOM. Si vous ne le faites pas, cela peut entraîner des fuites de mémoire car les éléments DOM et les fonctions associées ne peuvent pas être récupérés par le garbage collection.

```javascript
let button = document.getElementById('myButton');

button.addEventListener('click', handleButtonClick);

function removeButton() {
    button.removeEventListener('click', handleButtonClick);
    button.parentNode.removeChild(button);
}
```

Dans cet exemple, avant de supprimer le bouton du DOM avec `removeButton`, nous supprimons d'abord l'écouteur d'événements associé pour éviter une fuite de mémoire.