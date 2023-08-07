---
title: "React: Composants, Cycle de Vie, et Évolution Modernes"
summary: "De l'introduction de React à la transformation avec la version 16, cet article explore l'évolution des composants de classe et fonctionnels, met en évidence les avantages des Hooks et démontre pourquoi React est devenu un choix incontournable dans le développement d'interfaces utilisateur."
categories: ["Blog","Développement",]
tags: ["React", "JavaScript","Front-end"]
showSummary: true
date: 2023-08-06
draft: false
---

## Histoire de React

React est une bibliothèque JavaScript open-source créée par Facebook. Elle a été lancée en 2013 avec l'idée de construire des interfaces utilisateur réactives et efficaces. Le concept principal de React est la création de composants réutilisables qui gèrent leur propre état. Cela a révolutionné la façon dont les développeurs créent des applications web, en introduisant une manière plus modulaire et maintenable de construire des interfaces utilisateur.


## Innovation

React a marqué une innovation significative dans le développement front-end en introduisant le concept de composants réutilisables et en centralisant la gestion de l'état. Avant React, la gestion de l'état dynamique et l'interactivité dans les applications web étaient souvent complexes et dispersées. React a résolu ces problèmes en permettant aux développeurs de construire des interfaces utilisateur modulaires où chaque composant gère son propre état et sa logique. De plus, grâce à son algorithme de réconciliation virtuel, React est capable d'effectuer des mises à jour dans le DOM de manière très efficace. Ces innovations ont transformé la manière de développer des applications web, en rendant le processus plus structuré, maintenable, et performant.


## Différence entre React.js, Vue.js et Angular

### React

**Nature** : React est principalement une bibliothèque pour construire des interfaces utilisateur, et elle se concentre sur la vue dans le modèle MVC (Modèle-Vue-Contrôleur).
**Flexibilité** : React offre plus de flexibilité dans le choix des autres bibliothèques et outils à utiliser avec elle. Vous pouvez choisir vos propres bibliothèques pour la gestion de l'état, le routage, etc.
**Composants** : React utilise JSX, une syntaxe qui ressemble à HTML et qui est intégrée dans JavaScript, facilitant la création de composants.

### Angular

**Nature** : Angular est un cadre complet qui fournit non seulement des fonctionnalités pour la vue, mais aussi pour le modèle et le contrôleur. C'est une solution tout-en-un.
**Complexité** : Angular a une courbe d'apprentissage plus raide en raison de sa complexité et de son ensemble complet de fonctionnalités.
**Langage** : Angular utilise TypeScript, qui apporte des fonctionnalités de typage statique, augmentant la lisibilité et la robustesse du code.

### Vue.js

**Nature** : Vue.js est un cadre progressif, ce qui signifie qu'il peut être adopté de manière incrémentielle. Vous pouvez l'utiliser pour des parties de votre projet ou pour une application complète.
**Simplicité** : Vue.js est souvent loué pour sa simplicité et sa facilité d'apprentissage, en particulier pour les débutants.
**Flexibilité et Conception** : Vue a un équilibre entre la flexibilité de React et les fonctionnalités d'Angular, avec une syntaxe simple et une séparation claire entre le modèle, la vue et la logique.



## Les principales différences avec React v16

Avec la sortie de React **v16** en septembre 2017, plusieurs changements majeurs ont été introduits. Le plus notable est le nouvel algorithme de réconciliation, nommé Fiber, qui a rendu le rendu des composants beaucoup plus efficace. React v16 a également apporté la prise en charge des fragments, la gestion des erreurs avec les limites d'erreur, et l'asynchronisme dans le rendu.

## Class components

Les composants de classe en React offrent une manière structurée de créer des composants ayant un état et un cycle de vie. Le cycle de vie est une série de phases que le composant traverse de sa création à sa destruction. Voici un aperçu plus détaillé :

- **constructor**: Le constructeur est une méthode spéciale pour créer et initialiser un objet créé à partir d'une classe. En React, il est souvent utilisé pour initialiser l'état local et lier les gestionnaires d'événements. Par exemple:

```javascript
    constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
    }
```

- **componentDidMount**: Cette méthode est appelée après que le composant a été inséré dans le DOM (Document Object Model), qui est la représentation structurée de votre document HTML. "Monter" un composant signifie que le composant est créé et inséré dans le DOM. Par exemple, vous pouvez démarrer une requête réseau ici.

```javascript
    componentDidMount() {
    // Appel à une API ou autre logique
    }
```

- **componentDidUpdate**: Appelé après que le composant a été mis à jour dans le DOM. "Mettre à jour" signifie que quelque chose dans le composant a changé, comme l'état ou les props, et le rendu du composant est recalculé.

```javascript
    componentDidUpdate(prevProps, prevState) {
    // Effectuer des opérations de mise à jour
    }
```

- **componentWillUnmount**: Appelé juste avant que le composant ne soit supprimé du DOM. C'est ici que vous effectuerez tout nettoyage nécessaire, comme l'annulation de requêtes réseau ou la suppression d'écouteurs d'événements.

```javascript
    componentWillUnmount() {
    // Nettoyer les ressources
    }
```

![classcomponent](./classcomponent.png "Component lifecycle diagramme")

## Les composants fonctionnels: une meilleure voie

Les composants fonctionnels en React sont une manière plus simple et plus concise de créer des composants. Contrairement aux composants de classe, ils sont écrits comme de simples fonctions JavaScript, ce qui les rend plus lisibles et maintenables.

Avec l'introduction des Hooks dans React v16.8, les composants fonctionnels peuvent maintenant accéder à des fonctionnalités autrefois réservées aux composants de classe, comme l'état et le cycle de vie.

Voici un exemple d'un composant fonctionnel utilisant le Hook useState :

```javascript
    import React from 'react';

    function Counter() {
    const [count, setCount] = React.useState(0);

    return (
        <div>
            <p>Le compteur est à : {count}</p>
            <button onClick={() => setCount(count + 1)}>Incrementer</button>
        </div>
    );
    }
```

Avec cette façon de fonctionner react a ajouté de : 
**Optimalité** :

- **Simplicité** : Les composants fonctionnels sont plus légers et nécessitent moins de code que les composants de classe, ce qui peut améliorer la performance.

- **Réutilisation du code** : Avec les Hooks personnalisés, vous pouvez extraire la logique du composant dans des fonctions réutilisables, évitant ainsi la duplication du code.

**Facilite les tests** :

- **Moins d'Implémentation de Détails** : Les composants fonctionnels encouragent une séparation plus claire entre la logique et la présentation, ce qui rend les tests plus faciles car vous pouvez vous concentrer sur le comportement plutôt que sur la manière dont il est implémenté.
- 
**Compatibilité avec les Outils de Test** : Les bibliothèques de test comme React Testing Library fonctionnent bien avec les composants fonctionnels, facilitant la rédaction de tests qui suivent les meilleures pratiques.
Un article sur les tests automatiques sera posté par la suite pour traiter les différents types de tests et leur utilité.

## Hooks et le cycle de vie d'un composant 

Les Hooks permettent aux composants fonctionnels d'accéder à des fonctionnalités qui étaient auparavant réservées aux composants de classe. Voici comment ils se comparent au cycle de vie des composants de classe:

- `useState` : Remplace `this.state` dans les composants de classe.
- `useEffect` : Peut être utilisé pour gérer les effets secondaires et remplace `componentDidMount`, `componentDidUpdate`, et `componentWillUnmount`.
- `useContext` : Permet d'accéder au contexte sans avoir à utiliser un Consumer.

Les Hooks favorisent une meilleure réutilisation du code et une séparation des préoccupations, rendant votre code plus propre et plus facile à maintenir.

## Bonus : Les Méthodes de Cycle de Vie Obsolètes

Avec l'évolution de React, certaines méthodes de cycle de vie sont devenues obsolètes et ont été remplacées par des alternatives plus modernes et plus efficaces. Comprendre ces changements est essentiel pour maintenir des applications performantes et à jour.

### Méthodes Dépréciées
Voici quelques-unes des méthodes de cycle de vie qui ont été dépréciées dans les versions récentes de React :

- `componentWillMount` : Utilisé avant que le composant ne soit monté dans le DOM. Remplacé par le constructeur dans les composants de classe et le Hook `useEffect` avec un tableau de dépendances vide dans les composants fonctionnels.

- `componentWillReceiveProps` : Appelé avant que le composant ne reçoive de nouvelles props. Remplacé par `static getDerivedStateFromProps`.

- `componentWillUpdate` : Appelé juste avant que le rendu ne se produise lors d'une mise à jour. Remplacé par `componentDidUpdate`.

### Raisons de la Dépréciation
Ces méthodes ont été dépréciées pour plusieurs raisons, notamment :

- **Amélioration des Performances** : Les nouvelles méthodes offrent une meilleure optimisation et un rendu plus efficace.
- **Compatibilité avec les Futures Fonctionnalités** : La dépréciation facilite l'introduction de fonctionnalités asynchrones et d'autres améliorations dans React.
- **Lisibilité et Maintenance** : Les nouvelles méthodes et Hooks favorisent un code plus propre et plus maintenable.