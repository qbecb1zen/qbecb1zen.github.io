---
title: "Audit de Code : La quête des vulnérabilités cachées dans vos applications."
summary: "Cet article technique dévoile les objectifs, la méthodologie étape par étape, et les vulnérabilités courantes identifiées lors d'un audit de code."
categories: ["Blog","PASSI",]
tags: ["PASSI","Preparation","Audit code"]
showSummary: true
date: 2023-07-30
draft: false
---

## Définition
Un audit de code est un processus d'évaluation technique et approfondie du code source d'une application logicielle afin de détecter les erreurs de programmation et les vulnérabilités de sécurité. L'objectif principal de cet audit est d'identifier les failles potentielles qui pourraient être exploitées par des attaquants pour compromettre la sécurité du système ou provoquer un dysfonctionnement de l'application. L'audit de code est réalisé à l'aide d'outils d'analyse statique et dynamique du code, ainsi que par des examens manuels effectués par des experts en sécurité informatique. Les résultats de l'audit de code sont présentés dans un rapport détaillé, fournissant une liste des vulnérabilités détectées et des recommandations pour les corriger, permettant ainsi d'améliorer la qualité et la sécurité de l'application.


## Objectif de l'audit :
L'audit de code a pour objectif principal de détecter les vulnérabilités de sécurité et les erreurs de programmation dans le code source d'une application. Il vise à identifier les failles potentielles qui pourraient être exploitées par des attaquants pour compromettre la sécurité du système et de proposer un plan d'action pour les corriger.



## Suite à quel type d'audit on fait ce genre d'audit?
L'audit de code est généralement réalisé dans le cadre d'un audit de sécurité plus large, tel qu'un audit de sécurité des applications ou un audit de sécurité des systèmes d'information. Il est également réalisé lors de la phase de développement ou avant le déploiement d'une application pour s'assurer que le code est sûr et conforme aux meilleures pratiques de sécurité.
Il est aussi réalisé lors du processus classique :
Boite noire -> Boîte grise -> Boîte blanche (ICI)

## Méthodologie étape par étape :

Il n'y a pas de méthodologie type, mais globalement, un audit de code se structure sur les axes suivants :

- **Analyse des exigences de sécurité** : Comprendre les exigences de sécurité spécifiques de l'application à auditer pour orienter l'audit.
- **Revue du code source** : Analyser le code source ligne par ligne à l'aide d'outils d'analyse statique pour identifier les vulnérabilités potentielles.
- **Identification des vulnérabilités** : Rechercher des erreurs courantes telles que les injections SQL, les XSS (Cross-Site Scripting), les CSRF (Cross-Site Request Forgery), les erreurs de gestion des sessions, etc.
- **Analyse de la configuration de sécurité** : Examiner la configuration de l'application pour s'assurer que les paramètres de sécurité sont correctement définis.
- **Test des contrôles de sécurité** : Vérifier l'efficacité des contrôles de sécurité implémentés dans l'application pour se prémunir contre les attaques.
- **Rapport d'audit** : Présenter les résultats de l'audit, y compris les vulnérabilités identifiées et les recommandations pour les corriger.


## Déroulement de l'audit :
L'audit de code commence généralement par une analyse statique du code source, qui consiste à utiliser des outils d'analyse automatisée pour identifier les erreurs et les vulnérabilités. Ensuite, des tests manuels sont effectués pour vérifier les contrôles de sécurité et détecter les vulnérabilités plus complexes. Enfin, un rapport détaillé est généré, fournissant une liste complète des vulnérabilités détectées et des recommandations pour les corriger.


## Attendus clients de l'audit :
Les clients qui font réaliser un audit de code attendent de recevoir un rapport complet et précis sur les vulnérabilités détectées dans leur code source. Ils souhaitent également recevoir des recommandations concrètes pour corriger ces vulnérabilités et améliorer la sécurité globale de l'application.
Ainsi le rapport idéalement doit comporter à minima les éléments suivants :
- Synthèse globale (overall view)
- Un plan d'action issu des vulnérabilités remontés par l'audit


## Quelques exemples de vulnérabilités courantes identifiées lors d'un audit de code :

Quelques exemples de vulnérabilités classiques découvertes lors des audit de code :

- **Injection SQL** : L'absence de validation ou d'échappement des entrées utilisateur peut permettre aux attaquants d'injecter des requêtes SQL malveillantes, compromettant ainsi la base de données et pouvant entraîner la divulgation d'informations sensibles (exemple : "SELECT * FROM users WHERE username='$username';").

- **Cross-Site Scripting (XSS)** : Des données non vérifiées insérées dans des pages Web peuvent permettre aux attaquants d'injecter du code malveillant dans les navigateurs des utilisateurs, ce qui peut entraîner des vols de session ou des redirections vers des sites frauduleux (exemple : "<script>code malveillant ici</script>").

- **Cross-Site Request Forgery (CSRF)** : Les attaquants peuvent inciter les utilisateurs authentifiés à envoyer des requêtes non intentionnelles à un site Web, par exemple, en cliquant sur un lien malveillant dans un e-mail, ce qui peut entraîner des actions indésirables sur le compte de l'utilisateur (exemple : envoyer un formulaire avec une action malveillante sans que l'utilisateur en soit conscient).

- **Mauvaise gestion des erreurs** : Les messages d'erreur non gérés peuvent divulguer des informations sensibles sur l'application ou son environnement, ce qui pourrait être exploité par des attaquants pour mieux cibler leurs attaques (exemple : affichage du message d'erreur complet avec la trace du serveur).

- **Mauvaise validation des entrées utilisateur** : Le manque de validation appropriée des entrées utilisateur peut entraîner des vulnérabilités telles que l'injection SQL ou les XSS, en permettant aux attaquants d'insérer des données malveillantes dans l'application (exemple : l'application accepte des valeurs non attendues dans les formulaires).

- **Utilisation de fonctions non sécurisées** : L'utilisation de fonctions non sécurisées ou obsolètes peut rendre l'application vulnérable à des attaques, par exemple, l'utilisation de fonctions de hachage faibles (exemple : MD5) qui peuvent être facilement déchiffrées.

- **Dépassement de tampon (Buffer Overflow)** : Une mauvaise gestion des tampons peut permettre aux attaquants d'écrire des données malveillantes dans des zones mémoires adjacentes, entraînant des exécutions de code arbitraire (exemple : une variable tampon est débordée avec des données malveillantes => AAAAAAAAAAAAAAAAAAAA pour faire de l'écrasement de mémoire).

- **Mauvaise gestion des droits** : Une application qui accorde des privilèges excessifs à des utilisateurs non autorisés peut permettre aux attaquants d'accéder à des fonctionnalités ou à des données sensibles (exemple : un utilisateur non administrateur peut accéder à des pages d'administration).

- **Utilisation de mots de passe en clair** : Stocker les mots de passe des utilisateurs sans cryptage expose les informations sensibles en cas de violation de la base de données (exemple : stocker les mots de passe en texte brut au lieu de les hacher).

- **Exposition d'informations sensibles** : L'exposition non autorisée de données sensibles, tels que les numéros de sécurité sociale, les informations financières, etc., peut entraîner une violation de la confidentialité et des atteintes à la vie privée (exemple : rendre accessible des informations sensibles via des API non sécurisées).

- **Utilisation de composants obsolètes ou vulnérables** : L'incorporation de composants logiciels obsolètes ou ayant des vulnérabilités connues peut ouvrir des portes aux attaquants qui exploitent ces faiblesses pour compromettre l'application (exemple : utilisation d'une version non corrigée d'une bibliothèque tiers).

- **Mauvaise gestion des sessions** : Une mauvaise gestion des sessions peut conduire à des vulnérabilités de vol de session, d'usurpation d'identité et d'accès non autorisé (exemple : sessions sans expiration ou non invalidées après la déconnexion).

- **Inclusion de fichiers non sécurisée (Local File Inclusion)** : L'inclusion de fichiers distants ou non vérifiés peut permettre aux attaquants d'accéder à des fichiers sensibles sur le serveur ou d'exécuter du code malveillant (exemple : inclusion de fichiers basée sur des entrées utilisateur non filtrées).

- **Utilisation de protocoles de communication non sécurisés** : L'utilisation de protocoles non sécurisés, tels que HTTP non chiffré, expose les données de l'application à des interceptions et à des attaques d'écoute passive (exemple : transmission de mots de passe en clair).

- **Absence de contrôles d'accès** : L'absence de contrôles d'accès appropriés peut permettre aux utilisateurs non autorisés d'accéder à des fonctionnalités sensibles ou restreintes (exemple : un utilisateur non authentifié peut accéder à des pages d'administration).

- **Exposition de points d'API non sécurisés** : Des API mal configurées peuvent permettre aux attaquants d'accéder à des données ou des fonctionnalités sensibles sans authentification appropriée (exemple : des API sans clés d'API ou avec des autorisations trop permissives).

- **Erreurs de configuration de sécurité** : La mauvaise configuration des paramètres de sécurité, tels que les fichiers de configuration ou les autorisations de fichiers, peut exposer l'application à des risques inutiles (exemple : accès public à des fichiers de configuration contenant des informations sensibles).

- **Utilisation de bibliothèques et de frameworks non sécurisés** : L'utilisation de versions obsolètes ou vulnérables de bibliothèques et de frameworks peut exposer l'application à des failles de sécurité connues (exemple : utilisation d'une version de bibliothèque qui contient une vulnérabilité corrigée dans les versions ultérieures).

- **Utilisation excessive de commentaires sensibles dans le code source** : L'inclusion de commentaires sensibles dans le code peut exposer des informations confidentielles sur le fonctionnement interne de l'application (exemple : inclusion de mots de passe ou de clés API dans les commentaires).

- **Absence de mécanismes de protection contre les attaques par force brute** : L'absence de mécanismes de protection, tels que des délais de verrouillage des comptes ou des CAPTCHA, peut faciliter les attaques par force brute contre les comptes d'utilisateurs (exemple : pas de limitation du nombre de tentatives de connexion).

## Documentation utile à garder sous la main

![PASSI](./audit_code_passi.png "Extrait référentiel PASSI")