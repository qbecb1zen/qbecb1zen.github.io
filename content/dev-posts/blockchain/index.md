---
title: "Comprendre la Blockchain : De la Cryptographie au Minage"
summary: "Dans cet article, nous plongeons dans le monde fascinant de la blockchain. Découvrez comment cette technologie révolutionnaire fonctionne, pourquoi le minage est essentiel, et les nuances entre différentes cryptomonnaies. Aucune connaissance préalable n'est nécessaire, juste une curiosité pour le monde numérique."
categories: ["Technologie", "Cryptomonnaie"]
tags: ["Blockchain", "Bitcoin", "Ethereum", "dApps", "Minage"]
showSummary: true
date: 2023-08-14
draft: false
---

## Introduction aux blockchains

La blockchain, ou chaîne de blocs, est une technologie de stockage et de transmission d'informations sans organe central. Elle est constituée de blocs, chacun contenant un ensemble de transactions. Ces blocs sont liés de manière chronologique et sécurisée grâce à la cryptographie. L'idée de la blockchain a été conceptualisée pour la première fois en **2008** par une entité pseudonyme, **Satoshi Nakamoto**, avec la publication de la white papper du Bitcoin. Depuis lors, elle a évolué pour soutenir une multitude d'applications au-delà des cryptomonnaies.

La blockchain offre une transparence inégalée. Chaque transaction est enregistrée de manière immuable, ce qui signifie qu'une fois qu'une transaction est ajoutée, elle ne peut être modifiée ou supprimée. De plus, la décentralisation garantit qu'aucune entité unique ne contrôle la blockchain, offrant une résistance à la censure. Elle élimine également le besoin d'intermédiaires, réduisant les coûts et les délais de transaction.

## Comment fonctionne la blockchain ?

### Les blocs et le chaînage :

Chaque bloc de la blockchain contient un ensemble de transactions, une empreinte du bloc précédent (via un "hash"), et un numéro de séquence. Lorsqu'un bloc est complété, un nouveau bloc est généré. Les blocs sont liés de manière linéaire, formant une chaîne. Cette structure garantit que les données ne peuvent être modifiées rétroactivement sans altérer tous les blocs suivants, offrant une sécurité robuste.

Chaque bloc de la blockchain est une structure de données qui contient :

- **L'entête du bloc** :
    - **Hash précédent** : Il s'agit de l'empreinte cryptographique du bloc précédent. Cette empreinte est produite en utilisant une fonction de hachage, généralement SHA-256 pour Bitcoin, qui prend en entrée les informations du bloc précédent et produit une sortie fixe de taille 256 bits.
    - **Timestamp** : La date et l'heure à laquelle le bloc a été créé.
    - **Nonce** : Un nombre arbitraire utilisé une seule fois. Dans le contexte du minage, le nonce est ajusté pour trouver un hash qui satisfait les exigences du réseau (par exemple, un certain nombre de zéros au début du hash).
    - **Root du Merkle Tree** : Le Merkle Tree est une structure de données qui résume toutes les transactions du bloc. Le root, ou racine, est un hash qui représente toutes les transactions du bloc. Si une seule transaction change, le root du Merkle Tree change également.

- **Le corps du bloc** :
    - Liste des transactions : Chaque transaction contient des informations sur l'expéditeur, le destinataire, le montant, ainsi que des signatures numériques pour vérifier l'authenticité.

Lorsqu'un mineur réussit à créer un bloc valide (c'est-à-dire qu'il trouve un hash qui répond aux exigences du réseau), ce bloc est ajouté à la blockchain. Le processus de recherche de ce hash valide est ce qu'on appelle la "preuve de travail" (Proof of Work, PoW). C'est un processus coûteux en termes de temps et d'énergie, ce qui rend la falsification de la blockchain extrêmement difficile.

La nature séquentielle et liée des blocs signifie que pour modifier une transaction dans un bloc, un attaquant devrait recalculer le hash de ce bloc ainsi que de tous les blocs suivants. Étant donné la puissance de calcul du réseau, cela est pratiquement impossible.

De plus, la décentralisation de la blockchain signifie qu'il existe de nombreuses copies de la blockchain sur différents nœuds du réseau. Pour réussir une attaque, il ne suffit pas de modifier une seule copie de la blockchain ; il faudrait obtenir un consensus de plus de 50% du réseau, ce qui est extrêmement coûteux et difficile.


### Les nœuds :

Les nœuds sont des ordinateurs qui participent au réseau blockchain. Chaque nœud détient une copie de la blockchain, garantissant sa disponibilité et sa résilience. La décentralisation signifie qu'aucun nœud ou groupe de nœuds ne détient l'autorité sur le réseau. Cela rend la blockchain résistante à la censure et aux attaques.

Au final, un "node" (nœud en français) dans le contexte de la blockchain est un participant actif du réseau. C'est un ordinateur ou un serveur qui possède une copie complète ou partielle de la blockchain et qui suit les règles du protocole de la cryptomonnaie concernée.

- **Fonctions principales d'un nœud** :
    - **Validation** : Les nœuds valident les transactions et les blocs selon les règles du protocole. Si une transaction ou un bloc ne respecte pas ces règles, il est rejeté.
    - **Transmission** : Les nœuds transmettent les transactions et les blocs valides à d'autres nœuds, assurant ainsi la propagation des informations à travers le réseau.
    - **Conservation** : Les nœuds conservent une copie de la blockchain, garantissant sa disponibilité et sa résilience.

- **Types de nœuds** :
    - **Nœud complet (Full Node)** : C'est un nœud qui détient une copie complète de la blockchain. Il valide toutes les transactions et les blocs, et est considéré comme le type de nœud le plus sécurisé. Les nœuds complets jouent un rôle crucial dans la décentralisation et la sécurité du réseau.
    - **Nœud léger (Light Node ou SPV, Simplified Payment Verification)** : Il ne détient qu'une partie de la blockchain, généralement seulement les en-têtes des blocs. Il s'appuie sur les nœuds complets pour obtenir et valider des informations spécifiques. Ils sont moins gourmands en ressources et sont souvent utilisés dans des applications mobiles.
    - **Nœuds de minage (Mining Nodes)** : Ce sont des nœuds qui participent activement au processus de création de nouveaux blocs, appelé minage. Ils comprennent généralement des nœuds complets associés à du matériel de minage.


En résumé, un nœud est un participant essentiel du réseau blockchain qui valide, transmet et stocke les informations. La santé et la sécurité du réseau dépendent en grande partie du nombre et de la diversité de ses nœuds.


### La cryptographie dans la blockchain :

La blockchain utilise la cryptographie pour assurer la sécurité des données. Chaque transaction est signée numériquement par l'expéditeur, garantissant l'authenticité. Les empreintes cryptographiques, ou "hashes", sont utilisées pour lier les blocs entre eux. Si une seule transaction dans un bloc est modifiée, le hash du bloc change, alertant le réseau de la modification.

#### Signature numérique :
- Chaque participant du réseau possède une paire de clés : une clé publique, qui est partagée avec tout le monde, et une clé privée, qui reste secrète.
- Lorsqu'une transaction est créée, l'expéditeur utilise sa clé privée pour signer la transaction. Cette signature est une preuve que la transaction provient bien de l'expéditeur et qu'elle n'a pas été altérée en cours de route.
- Les autres nœuds du réseau utilisent la clé publique de l'expéditeur pour vérifier la validité de cette signature. Si la signature est valide, cela signifie que la transaction est authentique.

**N.B** : Lors de la création d'un wallet par exemple avec metamask, deux clés sont directements crées :
- **Private Key** : qui est une chaîne secrète qui permet au détenteur de signer les transactions et de prouver la propriété des fonds. (c'est la clé secrète qui permet d'avoir accès à son wallet par exemple et vérifier l'authenticité des personnes)
- **Public key** : qui sert principalement d'adresse ou d'identifiant sur la blockchain.
C'est les mêmes utilisées dans la partie de signature.

#### Hashing :
- Le hachage est le processus de conversion d'une entrée (quelle que soit sa taille) en une sortie fixe de taille déterminée, généralement en utilisant une fonction de hachage comme SHA-256 (utilisée dans Bitcoin).
- Chaque bloc contient le hash du bloc précédent, créant ainsi un lien entre eux. C'est ce qui forme la "chaîne" dans la blockchain.
- Si une transaction dans un bloc est modifiée, même légèrement, le hash du bloc change complètement. C'est une propriété des fonctions de hachage cryptographiques : une petite modification de l'entrée produit un changement radical de la sortie.

### La validation des transactions :

Lorsqu'une transaction est initiée sur une blockchain, elle ne devient pas immédiatement une partie permanente de celle-ci. Elle doit d'abord subir un processus de validation rigoureux. Voici comment cela fonctionne étape par étape :

- **Création de la transaction** :
    - Lorsqu'un utilisateur souhaite effectuer une transaction (par exemple, envoyer des cryptomonnaies à une autre adresse), il crée une transaction en spécifiant les détails pertinents tels que l'adresse du destinataire, le montant et les frais de transaction.
    - Cette transaction est ensuite signée numériquement à l'aide de la clé privée de l'expéditeur. Cette signature garantit que la transaction a bien été initiée par le propriétaire de l'adresse et qu'elle n'a pas été altérée en transit.
- **Propagation de la transaction** :
    - La transaction signée est diffusée sur le réseau blockchain, où elle est reçue par les nœuds. Ces nœuds vérifient d'abord la validité de la transaction (signature, solde suffisant de l'expéditeur, etc.) avant de la transmettre à d'autres nœuds.
- **Processus de minage** :
    - Les mineurs sont des participants spéciaux du réseau équipés de matériel de calcul spécialisé. Leur rôle principal est de regrouper les transactions valides en blocs et de sécuriser ces blocs en résolvant des énigmes cryptographiques complexes.
    - Cette "énigme" est en réalité un problème mathématique qui nécessite une grande puissance de calcul pour être résolu. Le premier mineur à résoudre le problème annonce la solution aux autres nœuds du réseau.

- **Validation du bloc** :
    - Lorsqu'un mineur trouve une solution, il crée un bloc contenant plusieurs transactions et la soumet au réseau pour validation.
    - Les autres nœuds vérifient la solution proposée (ce qui est beaucoup plus rapide que de trouver la solution en premier lieu) et, si elle est correcte, le bloc est considéré comme valide.
- **Ajout à la blockchain** :
    - Une fois le bloc validé, il est ajouté à la blockchain, ce qui rend les transactions qu'il contient irréversibles. L'ajout d'un bloc à la blockchain est souvent accompagné d'une récompense pour le mineur, qui reçoit généralement une certaine quantité de cryptomonnaie pour son travail.

- **Consensus** :
    - Il est important de noter que pour qu'un bloc soit ajouté à la blockchain, un consensus doit être atteint parmi les nœuds du réseau. Ce consensus garantit que même si certains participants agissent de manière malveillante ou si des erreurs se produisent, la majorité honnête du réseau garantira l'intégrité et la sécurité de la blockchain.


## L'importance du minage dans la blockchain

### Qu'est-ce que le minage ? :

Le minage est le processus par lequel les transactions sont vérifiées et ajoutées à la blockchain. Les mineurs utilisent une puissance de calcul pour résoudre des énigmes cryptographiques complexes. En récompense de leur travail, ils reçoivent des cryptomonnaies.

### Pourquoi miner ? L'incitation monétaire et la preuve de travail :

Miner est essentiel pour sécuriser le réseau. La preuve de travail est un mécanisme qui nécessite un effort considérable pour résoudre des énigmes cryptographiques. Cela garantit que les transactions sont validées de manière équitable. Les mineurs sont incités à participer grâce à des récompenses en cryptomonnaie.

### La puissance de calcul et la compétition entre mineurs :

La puissance de calcul fait référence à la capacité des mineurs à résoudre des énigmes cryptographiques. Plus ils ont de puissance, plus ils ont de chances de résoudre l'énigme en premier et de recevoir la récompense. Cela crée une compétition saine, garantissant la sécurité du réseau.

### La récompense du bloc et la création de nouvelles pièces :

Lorsqu'un mineur résout une énigme, il reçoit une récompense sous forme de nouvelles pièces. C'est le mécanisme principal par lequel de nouvelles pièces sont introduites dans le système. Au fil du temps, la récompense du bloc diminue, un événement connu sous le nom de "halving".

## Les applications décentralisées (dApps)

### Définition et avantages des dApps :

Les dApps, ou applications décentralisées, sont des applications qui fonctionnent sur un réseau blockchain décentralisé. Contrairement aux applications traditionnelles, elles ne sont pas contrôlées par une entité unique. Les dApps sont transparentes, immuables et résilientes aux pannes. Elles peuvent fonctionner sans temps d'arrêt et sont résistantes à la censure.
Vous avez absolument raison, et je m'excuse pour cette omission. Les dApps, bien que décentralisées dans leur logique de back-end grâce à la blockchain, ont souvent une interface utilisateur (front-end) qui est hébergée de manière centralisée, tout comme n'importe quel autre site web ou application. Cette interface est généralement construite avec des technologies web standard comme JavaScript, HTML et CSS. Ainsi, bien que le back est décentralisé, le front reste centralisé et donc vulnérable à des attaques type DDoS, il est donc important pour les applications les plus importantes de respecter les best practices en terme cyber pour se protéger contre les attaques de disponibilité.


### Comment les dApps fonctionnent-ils sur la blockchain ? :

Les dApps interagissent avec la blockchain à l'aide de contrats intelligents (smart contracts). Ces contrats sont des programmes autonomes qui exécutent des actions spécifiques lorsque certaines conditions sont remplies. Par exemple, un dApp pourrait automatiquement transférer des fonds d'un compte à un autre lorsque certaines conditions sont remplies.

Un contrat intelligent est un programme auto-exécutable dont les instructions sont écrites dans le code et stockées sur la blockchain. Il est déclenché par des événements ou des actions spécifiques, et une fois déployé, il ne peut pas être modifié.

Les contrats intelligents sont écrits dans des langages de programmation spécifiques à la blockchain, tels que Solidity pour Ethereum. Une fois écrits, ils sont compilés et déployés sur la blockchain.
Une fois déployé, un contrat intelligent ne peut pas être modifié. Cela garantit que les règles définies lors de la création du contrat seront toujours respectées.Ceci reste très **nuancé**, parce qu'il y a toujours moyen de faire : 
- Des contrats évolutifs (**Upgradable Contracts**) : Certains contrats intelligents sont conçus dès le départ pour être évolutifs. Cela signifie qu'ils ont une architecture qui permet de remplacer ou de modifier certaines parties de leur logique tout en conservant l'état ou les données du contrat. Cela est généralement réalisé en utilisant des contrats proxy ou des contrats de stockage de données séparés. 
Ex. Utilisation d'*OpenZeppelin*.
Il y a cependant des contraintes à ce genre de contrats, par exemple : Impossibilité de changer l'ordre/type des variables déclarées ou supprimer une variable existante.
- Fonctions de gestion (**Admin Functions**) : Un contrat peut inclure des fonctions administratives qui permettent à certaines adresses (généralement les propriétaires ou les administrateurs du contrat) de modifier des variables ou des paramètres spécifiques. Cela peut inclure des choses comme changer les taux de frais, ajouter ou retirer des membres autorisés, etc.
- Il faut aussi garder en tête qu'un contrat peut aussi être souvent désigné à bloquer toute transaction en cas de découverte de faille pour permettre une migration vers un contrat où la faille est corrigée. On traitera des vulnérabilités classiques et failles dans les smart contrats dans un article plus tard.


### Exemples d'utilisation des dApps dans la vie réelle :

Les dApps ont une multitude d'applications dans la vie réelle. Ils peuvent être utilisés pour créer des systèmes de vote décentralisés, des plateformes de financement participatif, des marchés décentralisés, et bien plus encore. Un exemple célèbre est CryptoKitties, un jeu basé sur la blockchain où les joueurs peuvent acheter, vendre et élever des chats virtuels.

## La diversité des cryptomonnaies

### Bitcoin vs Ethereum : les différences fondamentales :

Bitcoin, la première cryptomonnaie, a été conçu comme un système de paiement décentralisé. Son principal objectif est de permettre des transactions peer-to-peer sans intermédiaire. Ethereum, en revanche, a été conçu comme une plateforme pour exécuter des contrats intelligents et créer des dApps. Bien qu'il possède sa propre cryptomonnaie, l'Ether, son objectif principal est de servir de plateforme pour d'autres applications.

![Bitcoin vs Ethereum](./btc-vs-eth.webp)

### Pourquoi existe-t-il différentes cryptomonnaies ? :

Il existe des milliers de cryptomonnaies, chacune avec ses propres caractéristiques et objectifs. Certaines, comme Bitcoin, sont conçues pour être des monnaies numériques. D'autres, comme Ethereum, sont conçues pour soutenir des applications décentralisées. Il y a aussi des cryptomonnaies axées sur la confidentialité, comme Monero, qui offrent des transactions entièrement anonymes.

- Bitcoin (BTC) :
    - **Objectif principal** : Bitcoin a été la première cryptomonnaie, conçue comme une monnaie numérique décentralisée sans besoin d'intermédiaires comme les banques.
    - **Technologie** : Utilise la preuve de travail (Proof of Work) pour valider et ajouter des transactions à la blockchain.
    - **Particularités** : En raison de sa première-mover advantage, Bitcoin est souvent considéré comme une "réserve de valeur" dans l'écosystème des cryptomonnaies, similaire à l'or dans le monde financier traditionnel.

- Ethereum (ETH) :
    - **Objectif principal** : Conçu comme une plateforme pour exécuter des contrats intelligents et créer des applications décentralisées (dApps).
    - **Technologie** : Tout comme Bitcoin, Ethereum a initialement utilisé la preuve de travail, mais il y a des plans pour passer à la preuve d'enjeu (Proof of Stake) avec Ethereum 2.0.
    - **Particularités** : Ethereum a introduit le concept de contrats intelligents, permettant des opérations programmables complexes sur la blockchain. Il a également introduit le standard ERC-20, qui a facilité la création de nombreux nouveaux tokens et projets.

- Monero (XMR) :
    - **Objectif principal** : Offrir des transactions privées et non traçables.
    - **Technologie** : Utilise des signatures en anneau (Ring Signatures) et des adresses furtives (Stealth Addresses) pour masquer les détails des transactions.
    - **Particularités** : Contrairement à Bitcoin, où les transactions sont transparentes et peuvent être tracées jusqu'à leur origine, Monero offre une confidentialité totale, rendant les transactions indéchiffrables et non liées à un individu.

- Diversité des objectifs :
    - Monnaies numériques : Bitcoin, Litecoin et d'autres ont été conçus principalement comme des monnaies numériques pour faciliter les transactions en ligne.
    - Plateformes de développement : Ethereum, EOS et Cardano sont des exemples de plateformes conçues pour le développement d'applications décentralisées.
    - Confidentialité : Monero, Zcash et Dash sont axés sur la fourniture de transactions privées.
    - Stablecoins : Des cryptomonnaies comme USDC, Tether et DAI sont conçues pour avoir une valeur stable, généralement liée à une monnaie fiduciaire comme le dollar américain.
    - Autres objectifs : Il existe des cryptomonnaies pour presque tous les cas d'utilisation, de la finance décentralisée (DeFi) à la propriété d'actifs numériques comme les NFTs (tokens non fongibles).


### Les tokens ERC-20 sur Ethereum :

**ERC-20** est un standard ou une interface pour les tokens sur la blockchain Ethereum. "ERC" signifie "Ethereum Request for Comment", et "20" est le numéro attribué à cette proposition spécifique.

Les tokens ERC-20 doivent implémenter un ensemble spécifique de fonctions, ce qui garantit que les différents logiciels, applications et autres tokens peuvent interagir avec eux de manière prévisible. Ces fonctions incluent, entre autres, `totalSupply`, `balanceOf`, `transfer`, `transferFrom`, `approve` et `allowance`.

- **Utilisations courantes** :
    - **Représentation d'actifs** : Les tokens ERC-20 peuvent représenter une variété d'actifs, tels que des actions d'une entreprise, des points de fidélité, ou même des monnaies stables liées à la valeur d'une monnaie fiduciaire.
    - **Moyens d'échange** : Ils peuvent être utilisés comme monnaie au sein d'une application décentralisée ou comme moyen d'échange entre utilisateurs.
    - **Levées de fonds** : Les Initial Coin Offerings (ICO) ont souvent utilisé des tokens ERC-20 pour lever des fonds pour de nouveaux projets.


## La subtilité entre les tokens et les cryptomonnaies

### La différence entre un token et une cryptomonnaie :

Toutes les cryptomonnaies sont des tokens, mais tous les tokens ne sont pas des cryptomonnaies. Une cryptomonnaie est une monnaie numérique utilisée pour les transactions. Un token, en revanche, peut représenter n'importe quel actif ou utilité sur une blockchain. Par exemple, un token pourrait représenter une part dans une entreprise, un droit de vote, ou un accès à un service.

Les différences principales sont les suivantes : 
- **Blockchain indépendante vs Blockchain existante** : Les cryptomonnaies ont généralement leur propre blockchain, tandis que les tokens sont émis sur des blockchains existantes (basée sur du eth par exp).
- **Utilité vs Valeur** : Bien que les cryptomonnaies soient principalement utilisées comme monnaies, les tokens peuvent avoir une variété d'utilités et de cas d'utilisation. Un token peut, par exemple, donner accès à une fonction spécifique d'une application décentralisée.
- **Émission et gestion** : Les cryptomonnaies sont généralement émises dans le cadre du processus de minage (comme le Bitcoin), tandis que les tokens sont souvent émis lors d'ICO (Initial Coin Offerings) ou d'autres formes de levées de fonds, ou tout simplement lors du lancement d'un token random qui a une "utilité".

### Les tokens non fongibles (NFTs) et leur unicité :

Un NFT est un type de token numérique qui représente un objet ou un élément unique sur une blockchain. Contrairement aux tokens fongibles, comme les cryptomonnaies classiques ou les tokens ERC-20, chaque NFT est distinct et ne peut pas être échangé à parité avec un autre NFT.
La principale caractéristique des NFTs est leur unicité. Chaque NFT a des informations ou des métadonnées uniques qui le distinguent des autres. Cette unicité est vérifiée et sécurisée par la blockchain, garantissant la provenance et l'authenticité de l'objet numérique.
Globalement, il s'agit du standard **ERC-721** qui définit un ensemble min de fonctions pour la gestion et l'interrogation de NFTs en donnant un ID unique.

Dans l'ensemble, ça reste assez spéculatif.

