---
title: "AV vs EDR vs XDR vs MXDR : Comprendre les Solutions de Cybersécurité Modernes"
summary: "Cet article explore les protocoles SPF, DKIM et DMARC, qui sont essentiels pour authentifier et sécuriser les emails. Il offre un guide complet pour comprendre, mettre en œuvre et tirer parti de ces protocoles afin de prévenir l'usurpation d'identité et le phishing dans les communications par email."
categories: ["Blog","Sécurité"]
tags: ["Cybersécurité", "Protection", "AV", "EDR", "XDR"]
showSummary: true
date: 2023-08-10
draft: false
---


## Introduction

La cybersécurité est un domaine en constante évolution, avec de nouveaux défis et menaces émergents chaque jour. Pour faire face à ces menaces, diverses solutions de sécurité ont été développées, notamment l'Antivirus (AV), la Détection et Réponse sur les Endpoints (EDR), la Détection et Réponse Étendue (XDR) et la Détection et Réponse Étendue Gérée (MXDR). Dans cet article, nous explorerons en détail chacune de ces solutions, comment elles fonctionnent et quelles sont les principales différences entre elles.


## Vue Globale

### Antivirus (AV)

#### Définition
L'antivirus est une solution logicielle conçue pour détecter, prévenir et éliminer les logiciels malveillants. Il se base principalement sur des signatures pour identifier et bloquer les menaces. Bien que l'AV soit essentiel, il est souvent considéré comme une défense de base car il ne peut pas détecter des menaces avancées ou inconnues.


#### Fonctionnement
L'AV utilise des signatures pour identifier les logiciels malveillants. Il scanne les fichiers à la recherche de ces signatures et bloque ou met en quarantaine les fichiers suspects.


### Endpoint detection and response (EDR)

#### Définition
Un **endpoint** est un point d'extrémité ou une entrée dans un réseau. Cela peut être un ordinateur, un smartphone, une tablette ou tout autre appareil connecté à un réseau. 

**EDR** : L'EDR est une solution qui surveille en permanence les activités sur les endpoints pour détecter, enquêter et répondre aux menaces. Contrairement à l'AV qui se concentre sur la prévention, l'EDR offre une visibilité et un contrôle continus sur les menaces potentielles, permettant une réponse rapide.

#### Fonctionnement
L'EDR surveille en continu les activités sur les endpoints, collecte des données, détecte les anomalies et permet aux équipes de sécurité d'enquêter sur les incidents.

### Extended detection and response (XDR)

#### Définition
L'**XDR** est une évolution de l'EDR. Il s'étend au-delà des endpoints pour inclure d'autres sources de données telles que les serveurs, les réseaux, les emails et les applications cloud. L'XDR utilise l'intelligence artificielle et le machine learning pour corréler les données de différentes sources, offrant ainsi une meilleure détection des menaces et une réponse plus rapide.

#### Fonctionnement
L'XDR collecte des données de diverses sources, telles que les endpoints, les charges de travail cloud, les emails et les réseaux. Il utilise l'IA et le machine learning pour analyser ces données, détecter les menaces en temps réel et y répondre automatiquement.


#### Point d'attention
**SOAR** (Security Orchestration & Automated Response) permet aux équipes (matures disposant à minima d'un SOC bien mature) de créer/exécuter des playbook (avec Ansible par exemple) d'automatiser des actions de sécurité.
Une plateforme **SOAR** est complexe, coûteuse et nécessite un SOC très mature pour mettre en œuvre et maintenir les intégrations et les playbooks de partenaires. L’XDR est à considérer comme un « SOAR léger » : une solution simple, intuitive et sans code qui permet d’agir de la plateforme XDR aux outils de sécurité connectés.


### Managed Extended detection and response (MXDR)

#### Définition
Le **MXDR** est une version gérée de l'XDR. Il combine les capacités de l'XDR avec des services gérés par des experts en sécurité. Cela signifie que non seulement la technologie est fournie, mais aussi une équipe d'experts qui surveille en permanence les menaces, enquête sur les incidents et y répond. C'est une solution clé en main pour les organisations qui ne disposent pas de ressources internes suffisantes pour gérer la sécurité.

#### Fonctionnement
Le **MXDR** combine les capacités de l'XDR avec des services gérés, offrant une protection 24/7 avec une expertise dédiée.


## Différences clés

### Portée et Intégration

- **AV** : Se limite à la détection de malwares basée sur des signatures. Fonctionne souvent de manière isolée.
- **EDR** : Se concentre sur les endpoints, offrant une visibilité et un contrôle continus. Peut s'intégrer avec d'autres outils de sécurité.
- **XDR** : Couvre une gamme plus large, intégrant des données de multiples sources pour une meilleure corrélation et détection.
- **MXDR** : Offre tout ce que l'XDR offre, mais avec l'ajout de services gérés, offrant une expertise 24/7.

### Capacités de détection et de réponse

- **AV** : Détecte les menaces basées sur des signatures, donc souvent peut être bypass si l'attaquant est motivé et compétent.
- **EDR** : Détecte les anomalies sur les endpoints, permettant une enquête et une réponse rapides.
- **XDR** : Détecte les menaces en temps réel à partir de diverses sources, offrant une réponse automatisée.
- **MXDR** : Détecte les menaces avec l'aide d'experts en sécurité, offrant une réponse rapide et experte, donc plus pertinente souvent sur les mesures prises.

### Avantages et Limitations

- **AV** : Simple, mais peut ne pas détecter les menaces avancées.
- **EDR** : Offre une meilleure visibilité sur les endpoints, mais peut être limité en termes de portée.
- **XDR** : Offre une détection et une réponse étendues, mais peut nécessiter des ressources pour gérer la solution.
- **MXDR** : Offre une solution complète avec des experts, mais peut être plus coûteux.

### Bonus : Bypass Antivirus/EDR

Il y a différentes sources, l'article n'ayant pas pour objectif de s'intéresser à cette partie, je vais plutôt juste lister différentes sources sur le bypass AV/EDR, il y aura d'autres articles qui se focuseront principalement sur le bypass de ses sécurités :

- https://github.com/wavestone-cdt/EDRSandblast
- https://github.com/hlldz/RefleXXion
- https://github.com/RedTeamOperations/Journey-to-McAfee
- https://github.com/tanc7/EXOCET-AV-Evasion
- https://github.com/naksyn/Pyramid
- https://github.com/Yaxser/Backstab/
- https://github.com/klezVirus/inceptor
- https://github.com/klezVirus/inceptor
-