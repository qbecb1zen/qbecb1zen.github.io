---
title: "React Hooks: Introduction, Utilisation et Bonnes Pratiques"
summary: "Des composants fonctionnels aux hooks moins connus, cet article décortique les capacités et les applications des Hooks dans React, permettant un code plus propre, modulable et optimisé."
categories: ["Blog","Développement",]
tags: ["React", "JavaScript","Front-end"]
showSummary: true
date: 2023-08-07
draft: false
---

## Introduction aux Hooks React

Les Hooks sont une fonctionnalité introduite dans React 16.8 qui permet aux développeurs d'utiliser l'état et d'autres fonctionnalités de React sans avoir à écrire une classe. Avant les Hooks, la gestion de l'état et l'accès aux fonctionnalités du cycle de vie étaient confinés aux composants de classe, ce qui pouvait rendre le code plus lourd et moins réutilisable. Les Hooks offrent une manière plus élégante et fonctionnelle d'écrire des composants, favorisant une meilleure réutilisation du code, une plus grande lisibilité, et une meilleure séparation des préoccupations. Ils permettent aux développeurs d'extraire la logique de composant dans des fonctions réutilisables, rendant le code plus propre et plus modulaire.

## Hooks classiques

Pour la présentation des **hooks**, chaque hook sera défini avec son utilité puis 1 ou 2 cas d'usages classiques et un squelette d'exemple pratique.

### `useState`

**Définition** : Le Hook `useState` permet d'ajouter un état local à un composant fonctionnel.

**Pourquoi c'est utile** : Il permet une gestion de l'état dans un composant sans avoir besoin de convertir le composant en classe.

**Cas d'usage** :

- Contrôler un formulaire.
- Afficher/Cacher un élément conditionnellement.

**Exemple pratique** :

```javascript
    const [count, setCount] = useState(0);
    <button onClick={() => setCount(count + 1)}>Incrémenter</button>
```

### `useEffect`

**Définition** : Le Hook useEffect permet d'exécuter des effets secondaires dans les composants fonctionnels.

**Pourquoi c'est utile** : Il remplace les méthodes de cycle de vie componentDidMount, componentDidUpdate, et componentWillUnmount dans les composants de classe.

**Cas d'usage** :

- Récupération de données depuis une API.
- Ajouter/Retirer un écouteur d'événement.

**Exemple pratique** :

```javascript
    useEffect(() => {
    document.title = `Compteur ${count}`;
    }, [count]); // Dépendance sur la variable count
```
### `useContext`

**Définition** : Le Hook useContext permet d'accéder à la valeur actuelle du contexte sans avoir à envelopper le composant dans un Context.Consumer.

**Pourquoi c'est utile** : Simplifie l'accès aux valeurs du contexte dans un composant fonctionnel.

**Cas d'usage** :

- Thématisation d'une application.
- Gestion globale de l'état.

**Exemple pratique** :

```javascript
    const ThemeContext = React.createContext('light');
    const theme = useContext(ThemeContext);
```

### `useReducer`

**Définition** : Le Hook useReducer est similaire à useState mais permet de gérer une logique d'état plus complexe. Il utilise un réducteur pour gérer l'état du composant.

**Pourquoi c'est utile** : Idéal pour gérer l'état qui implique plusieurs valeurs interdépendantes.

**Cas d'usage** :

- Gestion d'un formulaire avec plusieurs champs.
- Manipulation d'un état complexe dans une application.

**Exemple pratique** :

```javascript
    const [state, dispatch] = useReducer(reducer, initialState);
    dispatch({ type: 'INCREMENT' });
```

### `useCallback`

**Définition** : Le Hook useCallback retourne une version mémoïsée d'une fonction callback.

**Pourquoi c'est utile** : Il aide à optimiser les performances en évitant la création de nouvelles instances de la fonction lors de chaque rendu.

**Cas d'usage** :

- Passer des callbacks à des composants enfants optimisés.
- Éviter les re-rendus inutiles.

**Exemple pratique** :

```javascript
    const memoizedCallback = useCallback(() => {
    // Logique de la fonction
    }, [dependencies]);
```

### `useMemo`

**Définition** : Le Hook useMemo retourne une valeur mémoïsée, calculée à l'aide d'une fonction.

**Pourquoi c'est utile** : Permet d'éviter des calculs coûteux lors de chaque rendu.

**Cas d'usage** :

- Calcul de valeurs dérivées complexes.
- Amélioration des performances en cas de calculs intensifs.

**Exemple pratique** :

```javascript
    const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

### `useRef`

**Définition** : Le Hook useRef permet de garder une référence mutable qui persiste à travers les rendus.

**Pourquoi c'est utile** : Accéder aux éléments du DOM ou garder une valeur qui ne déclenche pas de rendu lorsqu'elle change.

**Cas d'usage** :

- Focus sur un champ de formulaire.
- Garder une trace d'une variable sans provoquer de rendu.

**Exemple pratique** :

```javascript
    const inputRef = useRef(null);
    <input ref={inputRef} />
    <button onClick={() => inputRef.current.focus()}>Focus</button>
```

### Conclusion
Les Hooks dans React ont révolutionné la façon dont nous écrivons et gérons les composants. Ils offrent une syntaxe plus claire, une meilleure réutilisation du code, et accèdent aux fonctionnalités avancées de React dans des composants fonctionnels.

## Hooks moins connus

Ces hooks sont beaucoup moins utilités car ça permet de réaliser des optimisations qui peuvent mener à des anti-patterns, ou car ils correspondent à des utilisations très "niche" liée à de l'optimisation qui n'est que dans de rares cas nécessaire.

Exclusivement pour cette section, des explications plus détaillées seront rédigées pour expliquer les hooks dans le détail :

### `useImperativeHandle`

**Définition** : Le Hook useImperativeHandle personnalise l'instance d'un composant exposée aux parents qui utilisent ref.

**Utilité** : Ce Hook est spécifiquement conçu pour permettre à un composant parent d'interagir avec certaines méthodes d'un composant enfant. Il vous permet de personnaliser la valeur exposée à l'aide de la `ref`.

**Pourquoi il est moins utilisé** : Il s'écarte de l'approche fonctionnelle et déclarative typique de React, et peut donc rendre le code plus difficile à suivre. L'utilisation excessive peut conduire à des anti-modèles.

**Cas d'usage** :

- Exposer des méthodes de contrôle pour un composant enfant.
- Cacher certaines implémentations internes d'un composant.

**Exemple pratique** :

```javascript
    function MyComponent(props, ref) {
    useImperativeHandle(ref, () => ({
        focus: () => {
        // Logique de focus
        }
    }));
    }
    const MyComponentWithRef = forwardRef(MyComponent);
```

### `useLayoutEffect`

**Définition** : Le Hook useLayoutEffect a la même signature que useEffect, mais il s'exécute de manière synchrone après que toutes les modifications du DOM aient été effectuées.

**Utilité** : Semblable à `useEffect`, ce Hook est utilisé pour effectuer des lectures ou des écritures synchrones sur le DOM. Il est exécuté avant que le navigateur n'ait eu la chance de "peindre", permettant donc des transitions et des animations fluides.

**Pourquoi il est moins utilisé** : Son utilisation est spécifique aux situations où le timing de l'effet est crucial. Dans la plupart des cas, useEffect est suffisant, et l'utilisation incorrecte de useLayoutEffect peut causer des problèmes de performances.


**Cas d'usage** :

Mesurer la taille ou la position d'un élément du DOM après le rendu.
Animer les transitions.

**Exemple pratique** :

```javascript
    useLayoutEffect(() => {
    // Accéder aux valeurs du DOM ici
    }, [dependencies]);
```

### `useDebugValue`

**Définition** : Le Hook useDebugValue peut être utilisé pour afficher une étiquette pour des Hooks personnalisés dans React DevTools.

**Utilité** : Ce Hook améliore le débogage en permettant d'afficher une étiquette dans React DevTools pour un Hook personnalisé. Il peut rendre le débogage plus clair et plus informatif.

**Pourquoi il est moins utilisé** : Son application est très spécifique à la création et au débogage de Hooks personnalisés. Pour de nombreux développeurs, le besoin de ce Hook peut être rare.

**Cas d'usage** :

Afficher une valeur lisible lors du débogage.
Fournir des informations supplémentaires sur un Hook personnalisé.

**Exemple pratique** :

```javascript
    function useMyHook(value) {
    useDebugValue(value ? 'Actif' : 'Inactif');
    // Logique du Hook
    }
```

### Point d'attention
Ces Hooks plus **niches** offrent des fonctionnalités puissantes mais spécifiques qui ne sont pas nécessaires dans la plupart des situations de développement quotidiennes. Leur utilisation cible des cas d'utilisation plus avancés ou spécialisés.

L'utilisation de ces Hooks nécessite une compréhension plus profonde de leur fonctionnement et du moment où ils sont appropriés. Bien que moins communs, ils peuvent être extrêmement utiles dans les bonnes situations, et comprendre leur fonctionnement peut aider à résoudre certains problèmes complexes ou spécifiques dans une application React.

## Gestion des Effets Secondaires

### Définition
Dans une application React, vous serez souvent confronté à la nécessité de réaliser des opérations qui vont au-delà du rendu du composant, comme les requêtes réseau, les écouteurs d'événements, les souscriptions, et plus encore. Ces opérations sont connues sous le nom d'effets secondaires, et elles nécessitent une gestion soignée.

### Utilisation du Hook `useEffect`

Le Hook `useEffect` est un outil puissant dans React qui vous permet de gérer ces effets secondaires dans les composants fonctionnels. Il combine les fonctionnalités des méthodes `componentDidMount`, `componentDidUpdate`, et `componentWillUnmount` dans les composants de classe.

**Exemple de requête réseau** :

```javascript
    useEffect(() => {
    const fetchData = async () => {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        // Mettre à jour l'état avec les données
    };
    fetchData();
    }, []); // Le tableau vide signifie que l'effet s'exécute une fois après le premier rendu
```

**Exemple d'écouteur d'événements** :

```javascript
    useEffect(() => {
    const handleResize = () => {
        // Logique de gestion du redimensionnement
    };

    window.addEventListener('resize', handleResize);

    // Retourner une fonction de nettoyage pour supprimer l'écouteur
    return () => {
        window.removeEventListener('resize', handleResize);
    };
    }, []); // Encore une fois, s'exécute une fois après le premier rendu
```

### Conclusion
La gestion des effets secondaires est un aspect crucial du développement en React. Le Hook `useEffect` offre une manière élégante et puissante de gérer ces effets secondaires, rendant votre code plus propre et plus maintenable.

## Bonnes Pratiques et Anti-modèles

### Bonnes Pratiques
- **Ne pas placer de Hooks à l'intérieur de boucles, conditions, ou fonctions imbriquées** : Assurez-vous que les Hooks sont toujours appelés dans le même ordre.

- **Utiliser plusieurs appels à `useEffect` pour séparer des logiques différentes** : Cela rend votre code plus lisible et modulable.

- **Utiliser la fonction de nettoyage dans `useEffect`** : Pour éviter les fuites de mémoire, retournez une fonction de nettoyage pour supprimer les écouteurs, les souscriptions, etc.

### Anti-modèles
- **Omission de dépendances dans le tableau de dépendances** : Si vous utilisez une valeur qui change dans l'effet, assurez-vous de l'inclure dans le tableau de dépendances de `useEffect`.

- **Utilisation excessive de l'état global** : Utiliser un état global pour tout peut rendre l'application plus difficile à maintenir. Utilisez l'état local lorsque c'est approprié.

- **Dépendance sur l'ordre d'exécution des Hooks** : Ne dépendez pas de l'ordre d'exécution entre différents Hooks; ils doivent être indépendants.