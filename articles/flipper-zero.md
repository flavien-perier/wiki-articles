---
title: Flipper Zero
type: WIP
categories:
  - system
  - security
description: Configuration d'un équipement Flipper Zero.
author: Flavien PERIER <perier@flavien.io>
date: 2022-01-11 18:00
---

Je tiens avant toute chose à préciser que cet article est rédigé uniquement à but pédagogique. Je n'ai jamais projeté d'attaquer quelque entreprise que ce soit. Si vos intentions en vous rendant sur ce site sont malveillantes je vous invite à consulter [l'article 323-1 du Code pénal](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000030939438/) qui stipule que les intrusions dans un système informatique peuvent vous couter jusqu'a deux ans d'emprisonnement et 60 000 € d'amende. Cela étant dit, je décline toute responsabilité de ce que vous ferez du contenu de cet article.

Le [Flipper Zero](https://flipperzero.one/) est un dispositif d'attaque physique. Il s'agit d'une sorte de couteau suisse de l'attaque sans fil.

En effet le dispositif dispose d'un grand nombre de capteurs :
- Radio
- Infrarouge
- NFC
- RFID
- Bluetooth
- iButton

Par défaut le Flipper Zero ne dispose pas de carte wifi, mais il est possible d'en connecter une externe en utilisant le GPIO.

Il est également possible de connecter l'appareil en USB a un ordinateur afin de l'utiliser comme une [Rubber Ducky](https://www.flavien.io/wiki/rubber-ducky.md) ou comme d'une clé de sécurité (comme une [YubiKey](https://www.yubico.com/)).

![Flipper Zero](https://medias.flavien.io/articles/flipper-zero/flipper-zero.webp)

## Avantage de la conception

Le Flipper Zero est relativement petit et léger. En cas d'intrusion physique, il peut être facilement utilisé afin de copier des badges d'accès, ou des télécommandes. Sa conception lui confère la forme d'un jouet, ce qui devrait rendre moins méfiants la plupart des gens (même si vu son succès beaucoup d'informaticiens savent de quoi il s'agit).

## Vecteurs d'attaque

De très nombreux scénarios peuvent être imaginés avec cet appareil.

Le principal scénario à mon sens concerne les intrusions physiques dans des bâtiments. Le Flipper Zero peut permettre de cloner le badge d'accès à un bâtiment d'un employé, ou encore la télécommande d'accès à un parking de l'entreprise.

Une fois dans le bâtiment il est possible de se servir du Flipper Zero comme d'un moyen d'exécuter rapidement des instructions sur un poste qui n'aurait pas été verrouillé.

## Comment s'en défendre

Pour les zones vraiment sensibles d'une infrastructure (par exemple l'accès à un data center), il peut être bon de privilégier l'utilisation de système d'accès basé sur de la biométrie et non sur des supports physiques tel que des badges.

Dans les grosses structures, demander à des employés de garder en évidence constamment des badges avec leur matricule peut permettre de repérer plus facilement un intrus.

## Comment s'en servir

### Changement du firmware

Le firmware de base de l'appareil peut être assez limité. En effet, il existe des firmwares opens sources incluant par défaut de nombreux outils communautaires et débloquant certains protocoles. Il faut cependant faire très attention, car pour respecter la réglementation, le logiciel de base limite l'accès à certaines bandes de fréquences en fonction des pays, ce que ne font pas les firmwares communautaires. C'est pourquoi même si je conseille (tout comme les créateurs de l'appareil) de ne pas se limiter au logiciel de base, il faut faire très attention de ne pas émettre sur ces fréquences (et de préférence ne pas écouter non plus, même si c'est moins grave).

Le firmware [unleashed-firmware](https://github.com/DarkFlippers/unleashed-firmware) semble assez populaire. Il ne change pas radicalement le fonctionnement de l'appareil, mais se charge d'embarquer de nombreux outils que l'utilisateur n'aura pas à intégrer lui-même.

### 

### Utilisation en tant que clé de sécurité

Dans ce cas précis, l'objectif est d'utiliser le Flipper Zero non pas comme un appareil offensif, mais défensif. Parmi les nombreux cas d'utilisation de l'appareil, il est possible de le connecter à un ordinateur afin de chiffrer ou signer des documents, ou encore l'utiliser comme moyen d'authentification. La clé privée utilisée est stockée sur la mémoire de l'appareil.

### Quelques exemples de scripts

- [Awesome Flipper Zero](https://github.com/djsime1/awesome-flipperzero)
