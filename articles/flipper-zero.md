---
title: Flipper Zero
type: WIKI
categories:
  - system
  - security
description: Configuration d'un équipement Flipper Zero.
author: Flavien PERIER <perier@flavien.io>
date: 2023-01-11 18:00
---

Je tiens avant toute chose à préciser que cet article est rédigé uniquement dans un but pédagogique. Je n'ai jamais projeté d'attaquer quelque entreprise que ce soit. Si vos intentions en vous rendant sur ce site sont malveillantes, je vous invite à consulter [l'article 323-1 du Code pénal](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000030939438/) qui stipule que les intrusions dans un système informatique peuvent vous couter jusqu'à deux ans d'emprisonnement et 60 000 € d'amende. Cela étant dit, je décline toute responsabilité de ce que vous ferez du contenu de cet article.

Le [Flipper Zero](https://flipperzero.one/) est un dispositif d'attaque physique. Il s'agit d'une sorte de couteau suisse de l'attaque sans fil.

En effet, le dispositif dispose d'un grand nombre de capteurs :
- Radio
- Infrarouge
- NFC
- RFID
- Bluetooth
- iButton

Par défaut le Flipper Zero ne dispose pas de carte wifi, mais il est possible d'en connecter une externe en utilisant le GPIO.

Il est également possible de connecter l'appareil en USB à un ordinateur afin de l'utiliser comme une [Rubber Ducky](https://www.flavien.io/wiki/rubber-ducky.md) ou comme une clé de sécurité (comme une [YubiKey](https://www.yubico.com/)).

![Flipper Zero](https://medias.flavien.io/articles/flipper-zero/flipper-zero.webp)

## Avantage de la conception

Le Flipper Zero est relativement petit et léger. En cas d'intrusion physique, il peut être facilement utilisé afin de copier des badges d'accès, ou des télécommandes. Sa conception lui confère la forme d'un jouet, ce qui devrait rendre moins méfiants la plupart des gens (même si vu son succès beaucoup d'informaticiens savent de quoi il s'agit).

## Vecteurs d'attaque

De très nombreux scénarios peuvent être imaginés avec cet appareil.

Le principal scénario à mon sens concerne les intrusions physiques dans des bâtiments. Le Flipper Zero peut permettre de cloner le badge d'accès à un bâtiment d'un employé, ou encore la télécommande d'accès à un parking de l'entreprise.

Une fois dans le bâtiment, il est possible de se servir du Flipper Zero comme d'un moyen d'exécuter rapidement des instructions sur un poste qui n'aurait pas été verrouillé avec les fonctionnalités de BadUSB.

## Comment s'en défendre

Pour les zones vraiment sensibles d'une infrastructure (par exemple l'accès à un data center), il peut être bon de privilégier l'utilisation de système d'accès basé sur de la biométrie et non sur des supports physiques tels que des badges.

Dans les grosses structures, demander à des employés de garder en évidence constamment des badges avec leur matricule peut permettre de repérer plus facilement un intrus.

Il est possible de protéger ses cartes sans contact grâce à des étuis spécialisés. Ces derniers contiennent de fins filaments de cuivre qui font cage de faraday et empêchent donc la lecture de la carte en sans contact, tant qu'elle est dans leur étui.

Il est malheureux de noter que cet appareil peu également permette de s'introduire dans des immeubles d'habitations. Il est possible de répliquer une télécommande radio ou un badge d'accès. Ça, cumulé au fait, qu'il est aujourd'hui possible de se procurer facilement et légalement un jeu de clés PTT (utilisées pour la distribution du courrier afin de rentrer dans les bâtiments et ouvrir les boites aux lettres). Une personne mal intentionnée et bien préparée n'aura donc aucune difficulté à rentrer dans un immeuble dit "sécurisé". La seule sécurité restante est la serrure de son appartement. C'est pourquoi investir dans une serrure 3 points ou posséder un ou plusieurs verrous peut être une bonne solution. L'idéale étant d'avoir une serrure conçue pour ne pas être crochetée facilement, comme celles proposées par la marque française [Point Fort Fichet](https://www.fichet-pointfort.com/fr/fr/products/serrure-de-securite).

## Comment s'en servir

La première étape est d'installer [l'application Android](https://apkpure.com/fr/flipper-mobile-app/com.flipperdevices.app) sur son téléphone et d'y connecter le Flipper Zero. À travers cette interface, il sera notamment possible de mettre à jour l'appareil, ou de changer le Firmware.

![Android application](https://medias.flavien.io/articles/flipper-zero/android-app.webp)

### Changement du firmware

Le firmware de base de l'appareil peut être assez limité. En effet, il existe des firmwares opens sources incluant par défaut de nombreux outils communautaires et débloquant certains protocoles. Il faut cependant faire très attention, car pour respecter la réglementation, le logiciel de base limite l'accès à certaines bandes de fréquences en fonction des pays, ce que ne font pas les firmwares communautaires. C'est pourquoi, même si je conseille de ne pas se limiter au logiciel de base, il faut faire très attention de ne pas émettre sur ces fréquences (et de préférence ne pas écouter non plus, même si c'est moins grave dans la mesure où les communications sont normalement chiffrées).

Le firmware [unleashed-firmware](https://github.com/DarkFlippers/unleashed-firmware) est assez populaire. Il ne change pas radicalement le fonctionnement de l'appareil, mais se charge d'embarquer de nombreux outils que l'utilisateur n'aura pas à intégrer lui-même.

Au moment du téléchargement, plusieurs versions du firmware sont proposées. La `n` ne contient que le firmware modifié, tandis que la `e` contient le firmware et [la plupart des outils communautaires](https://github.com/xMasterX/all-the-plugins). Il est donc bien plus intéressant de télécharger cette seconde version.

### Utilisation en tant que clé de sécurité

Dans ce cas précis, l'objectif est d'utiliser le Flipper Zero non pas comme un appareil offensif, mais défensif. Parmi les nombreux cas d'utilisation de l'appareil, il est possible de le connecter à un ordinateur afin de chiffrer ou signer des documents, ou encore l'utiliser comme moyen d'authentification. La clé privée utilisée est stockée sur la carte micro-sd de l'appareil et peut donc facilement être sauvegardée.

L'exemple suivant permet d'installer le dispositif sur une distribution [Manjaro Linux](https://manjaro.org/). Il faut avant tout brancher le Flipper Zero sur la machine et activer l'U2F.

```bash
sudo pacman -S libfido2 pam-u2f
mkdir -p ~/.config/Yubico

pamu2fcfg >> ~/.config/Yubico/u2f_keys
```

Il est maintenant possible d'utiliser le Flipper Zero afin de se connecter à des sites. Cependant, il n'est pas conseillé d'aller plus loin avec ce dispositif ni de protéger des comptes trop importants avec (comme une banque par exemple), étant donné que contrairement à d'autres dispositifs [FIDO](https://www.yubico.com/authentication-standards/fido2/), il n'est pas certifié pour cet usage.

### Quelques exemples de scripts

Il existe plusieurs repos git assez généralistes qui listent des ressources pour le Flipper Zero. En voici quelques-uns :

- [Awesome Flipper Zero](https://github.com/djsime1/awesome-flipperzero)
- [Frogg Master FlipperZero](https://github.com/FroggMaster/FlipperZero)

Voici quelques ressources qui peuvent être intéressantes à installer sur l'appareil :

- [Flipper-IRDB](https://github.com/logickworkshop/Flipper-IRDB): Un catalogue de payload pour simuler de nombreuses télécommandes en infra rouge.
- [flipperzero-bruteforce](https://github.com/tobiabocchi/flipperzero-bruteforce): Des scripts de bruteforce pour quelques protocoles radio.
- [FlipperMusicRTTTL](https://github.com/neverfa11ing/FlipperMusicRTTTL.git): Une liste de musiques à jouer avec le synthé du Flipper Zero.


Voici un script à jouer à la racine de la carte microSD afin d'avoir accès a quelques catalogues de ressources.

```bash
#!/bin/bash

# Controls the structure of the SD card
mkdir -p ./infrared
mkdir -p ./subghz
mkdir -p ./music_player

# Install Flipper-IRDB
rm -Rf ./infrared/Flipper-IRDB
git clone https://github.com/logickworkshop/Flipper-IRDB.git ./infrared/Flipper-IRDB

# Install flipperzero-bruteforce
rm -Rf ./subghz/flipperzero-bruteforce
git clone https://github.com/tobiabocchi/flipperzero-bruteforce.git ./subghz/flipperzero-bruteforce
mv ./subghz/flipperzero-bruteforce/sub_files/* ./subghz/flipperzero-bruteforce
rmdir ./subghz/flipperzero-bruteforce/sub_files

# Install FlipperMusicRTTTL
rm -Rf ./music_player/FlipperMusicRTTTL
git clone https://github.com/neverfa11ing/FlipperMusicRTTTL.git ./music_player/FlipperMusicRTTTL
mkdir -p ./music_player/FlipperMusicRTTTL/Other
unzip ./music_player/FlipperMusicRTTTL/Unsorted\ 10k\ Song\ Archive.zip -d ./music_player/FlipperMusicRTTTL/Other
rm -Rf ./music_player/FlipperMusicRTTTL/Unsorted\ 10k\ Song\ Archive.zip

# Delete unused files
find . -type f -name "*.md" -delete
find . -type f -name ".git*" -delete
find . -type f -name "*.exe" -delete

find . -type d -name ".git" -exec rm -Rf {} \;
find . -type d -name ".github" -exec rm -Rf {} \;
```

## Sources

- [Lockpicking France](https://lockpickingfrance.org/#victime)
- [USB-Dongle Authentication](https://www.dongleauth.com/)
- [Easy Setup Yubikey on Manjaro Linux](https://credibledev.com/easy-setup-yubikey-on-manjaro-linux/)
