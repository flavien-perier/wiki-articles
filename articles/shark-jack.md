---
title: Shark Jack
type: WIP
categories:
  - system
  - security
description: Configuration d'un équipement Shark Jack.
author: Flavien PERIER <perier@flavien.io>
date: 2021-11-10 18:00
---

Je tiens avant toute chose à préciser que cet article est rédigé uniquement à but pédagogique. Je n'ai jamais projeté d'attaquer quelque entreprise que ce soit. Si vos intentions en vous rendant sur ce site sont malveillantes je vous invite à consulter [l'article 323-1 du Code pénal](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000030939438/) qui stipule que les intrusions dans un système informatique peuvent vous couter jusqu'a deux ans d'emprisonnement et 60 000 € d'amende. Cela étant dit, je décline toute responsabilité de ce que vous ferez du contenu de cet article.

La [Shark Jack](https://shop.hak5.org/products/shark-jack) est un dispositif d'attaque physique vendu par [Hak5](https://hak5.org/). Elle permet d'effectuer des attaques réseau assez simplement. D'un point de vue physique, elle mesure seulement 6cm de long et possède une prise RJ45 mâle. D'un point de vue plus logiciel elle dispose d'un système d'exploitation Linux fortement alléger, un espace disque de quelques méga-octets et quelques outils tels que [nmap](https://nmap.org/) ou [tcpdump](https://www.tcpdump.org/).

![Shark Jack](https://medias.flavien.io/articles/shark-jack/shark-jack.webp)

## Avantages et inconvénients

La Shark Jack est un dispositif de petite taille et donc facile à dissimuler quand on se rend à un endroit où on ne devrait pas être. Elle est facile à dissimuler et la LED multicolore permet de savoir précisément où le script en est dans son déroulement.

Ses inconvénients sont pour moi liés à sa conception minimaliste, le stockage est très limité, le processeur ridicule et l'autonomie de 10 à 15 minutes en fonction des tâches effectuées. Les limites de stockage rendent impossible lefait de rajouter des paquets. Il pourrait par exemple être intéressant sur ce genre de dispositif de disposer de [Python](https://www.python.org/) avec [Scapy](https://scapy.net/) afin d'analyser les trames qui transitent sur le réseau et de les réécrire à la volée. D'un point de vue processeur, la faible puissance nous dissuade rapidement d'utiliser des outils tel que [arpspoof](https://github.com/alandau/arpspoof) afin de créer une attaque [MITM](https://fr.wikipedia.org/wiki/Attaque_de_l%27homme_du_milieu) (le processeur aurait trop de mal à faire transiter les paquets entre le client et le routeur, du coup la navigation serait fortement ralentie et la victime devrait vite suspicieuse, même si elle n'est pas sensibilisée à la sécurité) et de toute façon, `tcpdump` ne pourrait pas enregistrer des heures de navigations vu les limitations de stockage et de batterie.

Autre point, je pense personnellement qu'utiliser ce dispositif afin de se connecter à un serveur distant afin d'y déposer par exemple le résultat d'un nmap me semble être une mauvaise idée. Déjà parce que la plupart des réseaux d'entreprises ne nous laisseront pas contacter une machine située à l'extérieur du réseau sans passer par un proxy et ensuite dans l'hypothèse où cette connexion aboutisse, cela laisserait une trace facile à remonter... Celle de notre serveur. Cela nous oblige donc à utiliser uniquement les ressources limitées du dispositif afin de faire des analyse légères ou de modifier furtivement la configuration réseau à notre avantage afin d'y accèder plus tard de l'extérieur.

À cela il faut rajouter que la conception en plus d'être minimaliste est très low cost. La batterie de mon dispositif ayant explosé après seulement quelques minutes d'utilisation... Cela étant surement lié à l'absence de limiteur de charge ou simplement que la batterie est d'une référence douteuse... Bref le concept est assez intéressent, mais la concertions du dispositif assez douteux.

## Vecteurs d'attaque

Les attaques effectuées par ce dispositif sont limitées dans le temps. Il n'est donc pas pertinent de l'utiliser lui-même comme backdoor (avec ouverture d'une session, reverse SSH par exemple). Il faut donc concevoir des attaques courtes et ça tombe bien il y a de nombreuses possibilités.

Tous les vecteurs d'attaque avec ce dispositif ont pour point commun que le hacker devra réussir à accéder à une zone où se trouve des prises RJ45 apparentes reliant au réseau cible. Pour ce faire, il va bien souvent devoir user d'ingénierie sociale pour y arriver.

- Le cas d'utilisation le plus évident (et celui qui est déjà préchargé dans le dispositif à l'achat) est de faire un nmap sur le réseau et de sauvegarder le résultat dans la mémoire. De cette façon nous pourrons l'analyser plus tard. Pour gagner du temps sur l'exécution de notre script, on peut éventuellement essayer de chercher uniquement quelques ports (comme le 22 pour le ssh, le 80 et le 443 pour les dispositifs disposant d'une interface de configuration, le 21 pour le telnet sur les vieux appareils...). Le problème de nmap est qu'il laisse une trace assez visible sur le réseau. Mais son avantage est qu'il est rapide et précis. Une variante peut être d'utiliser tcpdump afin d'écouter les paquets émis en brodcast et de dresser une carte de la topologie du réseau en relisant les trames capturées. Le problème de cette seconde approche est qu'elle est moins précise (on peut identifier les machines, mais pas la plupart des protocoles qu'elles utilisent) et plus lente.

- Les équipements de la marque [Cisco](https://www.cisco.com/) (et autre) disposent généralement d'une prise RJ45 spécifique pour la configuration de l'équipement. Il serait alors possible de brancher le dispositif à cet endroit afin de reconfigurer l'équipement et de mettre en place une attaque de l'homme du milieu directement au niveau d'un routeur. Dans ce cas d'usage la Shark Jack ne sert qu'à déployer rapidement la configuration frauduleuse.


## Comment s'en défendre

La seule protection efficace contre ce genre d'attaque est un contrôle strict des accès au réseau autant physique, que numérique. Les équipements doivent être placés dans baies sécurisées dont seuls les employés habilités peuvent s'approcher. Et leur mot de passe d'accès ne doit pas être celui par défaut.

Etant donné qu'un utilisateur qui se branche physiquement à un réseau n'a pas besoin de mot de passe pour y accèder, il faut faire en sorte que les prises RJ45 soient le moins accèssible possible. Donc ne pas en mettre ailleur que dans les bureaux.

## Comment s'en servir

### Se connecter au dispositif

Pour se connecter au dispositif, il suffit de le mettre en mode `ARMING`, de brancher à la prise RJ45 d'un ordinateur et de l'alimenter à l'aide du cordon USB.

Le dispositif va alors créer un réseau virtuel par lequel il sera possible d'accéder en SSH au système embarqué à l'adresse `172.16.24.1`.

```bash
ssh root@172.16.24.1
```

### Quelques exemples de scripts

Il existe plusieurs sites qui proposent des scripts déjà tout faits. Il est intéressent de passer par la afin de s'en inspirer :

- [GitHub de Hak5](https://github.com/hak5/sharkjack-payloads)

#### Quelques conseils

- Eviter de faire des analyses trop complètes qui pouraient bloiquer le dispositif pendant un long laps de temps.

#### Reverse SSH

Comme précisé plus haut ce script ne devrait pas être utile lors d'un véritable scénario d'attaque, cependant il peut permettre de conbfigurer l'appareil de manière plus simple qu'en utilisant le réseau virtuel de l'appareil. En effet ce dernier ne permettant pas d'accéder à internet il ne sera pas possible d'installer de nouveau packets ou de les mettres à jour.

En prérequis il va falloir installer sur le dispositif les certificats nécesssaires à une connection SSH par clé (pour laquelle aucun mot de passe sera demandé).

```bash
#!/bin/bash
LED SETUP

ssh -R 22222:127.0.0.1:22 flavien@192.168.1.11

LED FINISH
```