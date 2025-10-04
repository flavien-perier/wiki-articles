---
title: Environnement Android
type: WIKI
categories:
  - system
description: Environnement Android orienté vie privée.
author: Flavien PERIER <perier@flavien.io>
date: 2020-11-10 19:00
---

Avant toute chose, cet article traite de manipulations techniques qui pourraient, dans certains cas, causer un certain nombre de dysfonctionnements voire des pertes de données.

Dans cet article, nous allons nous intéresser à la mise en place d'un environnement orienté vie privée et sécurité sur un téléphone portable.

## Le téléphone

Au niveau du téléphone, j'ai personnellement fait le choix de m'orienter sur un [Fairphone 6](https://www.fairphone.com/). Il s'agit d'un téléphone modulaire présentant un grand nombre d'avantages, tant au niveau éthique, que d'un point de vue purement technique.

Tout d'abord, il est important de souligner que l'entreprise essaye de s'orienter le plus possible vers des matériaux issus du commerce équitable. Pour en savoir plus sur cet aspect, je recommande vivement d'aller consulter le [blog](https://www.fairphone.com/fr/blog/) de l'entreprise.

L'autre aspect du Fairphone qui va plus nous intéresser dans cet article, c'est son aspect modulaire. Ce téléphone étant conçu dans une optique de durabilité, il est possible de changer en quelques poignées de minutes n'importe quel composant. Cela peut se faire tant au niveau hardware (changement de l'écran, caméra, carte mère...) qu'au niveau software. En effet, contrairement à beaucoup d'autres constructeurs, Fairphone fait en sorte qu'il soit facile de remplacer le système d'exploitation par défaut de son téléphone.

L'inconvénient souvent répété de ce téléphone est sa puissance jugée trop faible pour un téléphone de ce prix (plus ou moins 450 €). Mais soyons réalistes, si vous ne jouez pas à des jeux 3D, son processeur [Snapdragon 632](https://www.qualcomm.com/products/snapdragon-632-mobile-platform) et ses 4 Go de RAM devraient être largement suffisants pour tous vos usages. Quant au niveau du prix, il faut bien mettre dans la balance le fait que le téléphone étant fait pour durer, le support des mises à jour sera poussé tant que possible (5 ans minimum), contrairement à la plupart des constructeurs (hors Apple) qui partent sur une base de 18 mois. Pour ceux qui se disent qu'ils peuvent continuer à utiliser leur téléphone sans mise à jour de sécurité, je tiens simplement à rappeler qu'Android est aujourd'hui l'un des systèmes d'exploitation les plus attaqués et que, par conséquent, ne pas le mettre à jour revient à mettre en péril les données qu'on y stocke, notamment quand on sait que beaucoup d'utilisateurs n'hésitent pas à installer leurs applications bancaires favorites sur leur téléphone.

## Le système d'exploitation

Comme je l'ai dit précédemment, le Fairphone est compatible avec d'autres variantes d'Android que celle qui nous est généreusement fournie par Google.

La variante d'Android la plus connue est [Lineage OS](https://lineageos.org/) (anciennement [Cyanogen Mod](https://fr.wikipedia.org/wiki/CyanogenMod)) qui nous propose un Android sans couche Google installée par défaut. Comme le projet est open source, il existe d'autres distributions qui s'appuient sur cette dernière et notamment celle qui va nous intéresser, [/e/](https://e.foundation/).

Le slogan de /e/ est "Your data is YOUR data!" (Vos données sont vos données). L'objectif est ici de mettre en place un certain nombre de mécaniques qui vont permettre de garantir au mieux la vie privée de l'utilisateur. L'autre avantage de cette distribution est qu'elle se trouve en [partenariat](https://www.fairphone.com/fr/2020/04/30/keeping-your-data-safe-with-e-os/) direct avec l'entreprise Fairphone et que par conséquent les développeurs s'appliquent à ce que le système soit le plus stable possible sur cet équipement. Il est donc possible d'[installer](https://doc.e.foundation/devices/FP3/install) facilement ce système sur un téléphone, ou tout simplement en [acheter](https://esolutions.shop/fr/shop/murena-fairphone-3-plus-fr/) un déjà préinstallé.

## Faire une sauvegarde

Si le téléphone est actuellement utilisé, il est possible de sauvegarder une grande partie des données déjà en place grâce à des commandes ADB. Pour ce faire, je conseille cette page [GitHub d'AnatomicJC](https://gist.github.com/AnatomicJC/e773dd55ae60ab0b2d6dd2351eb977c1).

Tout d'abord, il faut vérifier qu'[ADB](https://www.flavien.io/wiki/manjaro.md#android-adb) est bien installé sur son PC afin de pouvoir manipuler son mobile.

Ensuite, il faut activer le mode développeur sur le téléphone (aller dans les paramètres, puis dans "À propos du téléphone" et cliquer 7 fois sur le numéro du build). Une fois le menu développeur visible s'y rendre et activer le "débogage USB".

Pour la suite, il est possible d'écrire un script `android-backup` grâce aux informations trouvées sur le précédent repos GitHub :

```bash
#!/bin/bash

TMP_LOCATION=/tmp/android-backup

mkdir -p $TMP_LOCATION/apk

adb usb
if [ $? -ne 0 ]
then
    exit 1
fi

sleep 2

# Backup app data
cd $TMP_LOCATION
adb backup -f data.backup -all -apk -nosystem

# Backup internal storage
cd $TMP_LOCATION
adb pull /sdcard .

# Backup apk files
cd $TMP_LOCATION/apk
for APP in `adb shell pm list packages -3 -f`
do
  adb pull `echo $APP | sed "s/^package://" | sed "s/base.apk=/base.apk /"`.apk
done

cd /tmp
7z a -mx=9 "$HOME/android [`date +%F`].7z" android-backup
rm -Rf $TMP_LOCATION
```

## Installation

La première étape va consister à débloquer le bootloader du Fairphone. Pour ce faire, il suffit de se rendre [sur le site du fabricant](https://www.fairphone.com/en/bootloader-unlocking-code-for-fairphone/) et de suivre les indications.

Une fois cette opération effectuée, il faut télécharger les différents composants pour l'installation :

- [/e/ canal stable](https://images.ecloud.global/stable/FP6/) ou [/e/ canal dev](https://images.ecloud.global/dev/FP6/)
- L'APK [Magisk 29.0](https://github.com/topjohnwu/Magisk/releases/tag/v29.0)

Ensuite, il faut accéder au mode recovery du téléphone (l'éteindre et le rallumer en maintenant les boutons "marche/arrêt" et "volume +" enfoncés) afin d'installer le système (toutes les données utilisateurs vont être perdues).

```bash
unzip IMG-e-*-official-FP6.zip

fastboot flashing unlock_critical
fastboot flashing unlock

fastboot -w

./flash_FP6_factory.sh
```

Une fois le système de base installé, il est possible de rooter le téléphone :

- Dans un premier temps, il va falloir le démarrer et y installer l'application Magisk (grâce à l'APK précédemment récupérée).
- Temps que le téléphone est branché à un ordinateur, nous allons en profiter pour déposer sur le stockage de l'appareil le fichier `boot.img`.
- Depuis l'application Magisk, il faut sélectionner le bouton `installer` depuis l'onglet `Magisk`. Par la suite, on sélectionne le fichier `boot.img`. L'application va alors appliquer ses correctifs root sur le fichier.
- On récupère le fichier généré dans le dossier Téléchargement et on le dépose sur l'ordinateur.
- On redémarre le téléphone sur le bootloader.
- On tape les commandes suivantes sur un terminal de son ordinateur :

```bash
fastboot flash boot_a magisk_patched-*.img
fastboot flash boot_b magisk_patched-*.img
```

Si on ne souhaite pas rooter le téléphone, il est préférable de locker le bootloader pour améliorer la sécurité.

```bash
fastboot flashing lock_critical
```

Et après le reboot :

```bash
fastboot flashing lock
```

Remarque : Avec le root le bootloader ne devra jamais être reverrouillé. Visiblement une vérification d'intégrité est effectuée à ce moment-là et le root est considéré comme une anomalie.

## Configuration de l'APN Bouygues Telecom

L'[APN](https://fr.wikipedia.org/wiki/Access_Point_Name) est une configuration permettant de se connecter au réseau mobile de l'opérateur afin de pouvoir envoyer et recevoir des MMS ainsi que d'accéder à internet au travers de la 3G/4G/5G.

Cependant, dans leur infinie sagesse, les opérateurs peuvent nous passer en IPv6. Le futur d'internet, plus sécurisé, sur lequel on pourra brancher plus d'appareils... Et sur lequel de nombreux services ne fonctionnent pas encore : [Discord](https://discord.com/), [ProtonVPN](https://protonvpn.com/) et bien d'autres.

Si on utilise ces services, il faut repasser en IPv4 pour que ça fonctionne et pour cela, il faut la configuration APN adéquate :

- Nom : `Bouygues`
- APN : `mms.bouygtel.com`
- Proxy :
- Port :
- User :
- Mot de passe :
- Serveur :
- MMSC : `mms.bouyguestelecom.fr/mms/wapenc`
- Proxy mms : `62.201.129.226`
- Port MMS : `8080`
- MCC : `208`
- MNC : `20`
- Authentification : `PAP`
- Type d'APN : `default,supl,mms`
- Protocole : `IPv4`
- protocole itinérance : `IPv4`
- Porteur :
- Type MVNO : `SPN`
- Valeur MVNO : `Bouygues Telecom`

### Applications par défaut

Par défaut le système d'exploitation contient un minimum d'applications pour être utilisable (Dialer, SMS, Launcher, Calendar...). Bien évidemment, je ne vais pas parler de toutes ces applications, déjà parce que ça prendrait beaucoup de temps, mais surtout parce que ça n'apporterait pas grand-chose. Je vais donc me contenter de parler de quelques-unes d'entre elles :

#### [MicroG](https://microg.org/download.html)

Comme dit plus tôt, ce système d'exploitation est conçu pour communiquer le moins possible avec Google. Seul problème, Android et son Framework sont développés par Google. Les interactions avec les serveurs de la société sont donc nombreuses et les couper causerait de graves dysfonctionnements pour de très nombreuses applications.

Heureusement, des développeurs se sont penchés sur le sujet et ont conçu MicroG. Ce dernier simule le comportement des composants Google sur un téléphone utilisant un système Android natif en envoyant le moins d'informations possible sur internet. Cette application est donc un très bon équilibre entre téléphone fonctionnel et vie privée.

#### Le store

/e/ ayant pour objectif de préserver la vie privée de ses utilisateurs, l'implémentation du Google Play Store aurait été un très mauvais choix... C'est pour cette raison que le système d'exploitation possède son propre métastore nommé App Lounge.

Pourquoi un métastore et pas un store ? Parce que l'entreprise ne stocke pas elle-même les applications, mais nous les fournit en allant les chercher directement sur le [Google Play Store](https://play.google.com/) ou sur [F-Droid](https://f-droid.org/).

Cela présente l'avantage d'avoir un catalogue très fourni (puisque constitué de la fusion de deux très grands stores Android), sans risque d'installer une application non certifiée (puisque les applications sont certifiées sur ces deux stores) et de ne pas compromettre la vie privée des utilisateurs puisque les serveurs de /e/ servent de proxy pour aller récupérer les applications sur les stores tiers.

Parmi les fonctionnalités amusantes de ce store, il y a la possibilité d'afficher une note de confidentialité calculée sur le nombre de traqueurs implémentés et la quantité d'autorisations demandées par une application. Tous ces résultats sont obtenus grâce au service [Exodus](https://exodus-privacy.eu.org/).

#### Le cloud

Par défaut /e/ propose la possibilité de se connecter à une instance [NextCloud](https://nextcloud.com/) afin d'y synchroniser ses données. Il est donc possible d'y sauvegarder des photos, agenda, contacts... sans passer par un GAFAM, voire sauvegarder toutes ses données sur une instance auto-hébergée. Heureusement, pour les utilisateurs ne souhaitant pas consacrer du temps à la mise en place d'une instance, la fondation propose son propre cloud (ce dernier n'offre malheureusement que quelques mégaoctets). Suffisant néanmoins pour synchroniser des contacts et agendas (il faudra laisser de côté la sauvegarde des photos).

En cas de problème de synchronisation, il est possible d'utiliser la commande suivante depuis un ordinateur afin de la forcer.

```bash
adb shell am broadcast -a foundation.e.drive.action.FORCE_SYNC --receiver-include-background
```

#### Le navigateur

Forcément pas de Google Chrome en vue, mais du [Chromium](https://www.chromium.org/) (la version open source du navigateur). Quant au moteur de recherche par défaut, là encore, pas de Google, mais un métamoteur aux couleurs de /e/ localisé à l'URL [spot.ecloud.global](https://spot.ecloud.global/). Ce dernier nous restitue une agrégation des résultats trouvés sur d'autres moteurs de recherche tels que [Qwant](https://www.qwant.com/), [DuckDuckGo](https://duckduckgo.com/), [Wikipédia](https://fr.wikipedia.org/)... Toutes les requêtes envoyées par ce métamoteur à des tiers sont routées à travers le réseau [Tor](https://www.torproject.org/). L'architecture de ce moteur semble donc très confidentielle et même si je ne l'utilise pas personnellement, je trouve sa conception très intéressante.

#### Le launcher

Appelé Bliss Launcher, le lanceur d'applications de la distribution est une copie de l'interface Apple, avec des fonctionnalités en moins et des bugs en plus. Pour avoir testé beaucoup d'autres launchers tels que [NovaLauncher](https://novalauncher.com/) ou [Lawnchair](https://lawnchair.app/), je peux affirmer qu'il manque beaucoup de fonctionnalités comme la gestion des gestes, un tiroir et des paramètres de manière générale. Cependant, contrairement à d'autres lanceurs d'applications tels que [Microsoft Launcher](https://apkpure.com/fr/microsoft-launcher/com.microsoft.launcher), celui-ci a l'immense avantage de respecter notre vie privée, de ne pas passer son temps à interagir avec le réseau, d'être open source et économe en batterie. Il ne s'agit donc clairement pas de l'application de l'année, mais elle fait néanmoins bien ce qu'on lui demande.

#### [Magic Earth](https://www.magicearth.com/)

Magic Earth est un simple GPS basé sur [OpenStreetMap](https://www.openstreetmap.org/) avec une interface très propre et intuitive qui propose entre autres choses de télécharger les cartes en local. Grâce à cette application, il est possible d'utiliser le téléphone en tant que GPS sans accéder aux données. Cela permet de réduire sa chauffe, lui permettre de rester plus longtemps allumé et de ne pas être épié par un quelconque [GAFAM](https://fr.wikipedia.org/wiki/G%C3%A9ants_du_Web) durant un voyage.

#### [OpenKeychain](https://www.openkeychain.org/)

Simple application pour gérer vos [clés de chiffrement asymétrique](https://fr.wikipedia.org/wiki/Cryptographie_asym%C3%A9trique). Il est possible d'utiliser cette application pour signer/chiffrer des SMS, textes ou tout autre fichier.

## Les applications à installer

### [f-droid](https://f-droid.org/)

F-Droid est un store d'applications Android open source. Son dépôt par défaut est déjà nativement disponible sur le store de /e/, mais il peut être intéressant d'installer quand même l'application. En effet, cette dernière intègre de nombreuses fonctionnalités telles que le partage d'applications à d'autres appareils via le Bluetooth ou le Wi-Fi. Ce qui peut s'avérer utile dans des contextes extrêmes où la connexion internet serait coupée. L'application propose également de rajouter d'autres dépôts d'applications en plus de celui par défaut. En voici quelques exemples :

- [Nethunter](https://store.nethunter.com/): `https://store.nethunter.com/repo?fingerprint=fe7a23dfc003a1cf2d2add2469b9c0c49b206ba5dc9edd6563b3b7eb6a8f5fab`
- [Guardian project](https://guardianproject.info/fdroid/): `https://guardianproject.info/fdroid/repo?fingerprint=b7c2eefd8dac7806af67dfcd92eb18126bc08312a7f2d6f3862e46013c7a6135`
- [Second Wind](https://secondwind.guardianproject.info/): `https://guardianproject-wind.s3.amazonaws.com/fdroid/repo?fingerprint=182cf464d219d340da443c62155198e399fec1bc4379309b775dd9fc97ed97e1`
- [Collabora Office](https://www.collaboraonline.com/collabora-office/): `https://www.collaboraoffice.com/downloads/fdroid/repo?fingerprint=573258c84e149b5f4d9299e7434b2b69a8410372921d4ae586ba91ec767892cc`

### L'antivirus

Android étant un système d'exploitation particulièrement vulnérable, il est donc important de posséder un antivirus digne de ce nom.

J'ai personnellement utilisé l'antivirus [Kaspersky Internet Security](https://www.kaspersky.fr/internet-security) pendant plusieurs années. Cependant, ce dernier étant conçu en Russie par d’anciens membres des services de renseignement, plusieurs entreprises et organismes conseillent d'éviter son utilisation dans le contexte de [guerre entre la Russie et l'Ukraine](https://fr.wikipedia.org/wiki/Guerre_du_Donbass). Un [article de Zataz](https://www.zataz.com/kaspersky-dangereux-ou-pas/) détaille la situation en pesant les arguments pour et contre l'utilisation de Kaspersky.

Même si je n'ai aucune preuve tangible que cet antivirus est néfaste (et au demeurant, je ne pense personnellement pas qu'il le soit), il vaut mieux s'appuyer sur une solution sur laquelle on peut avoir toute confiance.

De manière générale les performances des antivirus sont changeantes d'une année sur l'autre. Il peut donc être pertinent de consulter fréquemment le site [AV-Test](https://www.av-test.org/fr/antivirus/portables/) afin de voir les derniers comparatifs de ces solutions de sécurité.

### [SpamBlocker](https://f-droid.org/en/packages/spam.blocker/)

Le démarchage téléphonique est devenu insupportable pour beaucoup de personnes, avec parfois plusieurs appels par jour. Une première chose à faire est bien évidemment de s'inscrire sur [Bloctel](https://www.bloctel.gouv.fr/), l'initiative du gouvernement. Cependant, il semblerait que certaines entreprises peu scrupuleuses ne prennent pas réellement cette liste en compte. Il existe de nombreuses applications de blocage de numéros indésirables en ligne. Ces dernières sont souvent très intrusives, car des requêtes sont généralement envoyées à des serveurs avec les numéros qui vous appellent sans qu'on puisse vraiment savoir comment elles sont traitées (quand c'est Orange, je pense qu'on peut avoir confiance, mais quand ce sont des applications américaines qui font leur business autour de quelques applications, je pense qu'il est raisonnable de se méfier).

L'application SpamBlocker n'a rien d'incroyable dans son fonctionnement. Elle permet simplement de bloquer des numéros en fonction de patterns, et ce, sans communiquer avec internet. D'autres applications permettent très certainement de faire la même chose. Celle-là a l'avantage d'être open source et raisonnablement maintenue.

En France, les numéros de démarchage sont tenus de nous appeler avec des plages de numéros précises, donc faciles à identifier (donc à bloquer avec cette application). [Le plan de numérotation de l'ARCEP](https://www.arcep.fr/actualites/actualites-et-communiques/detail/n/plan-de-numerotation-050922.html) référence la liste de ces préfixes. Il ne reste donc plus qu'à écrire les regex et le tour est joué :

```regex
33162.*
33163.*
33270.*
33271.*
33377.*
33378.*
33424.*
33425.*
33568.*
33569.*
33948.*
33949.*
339475.*
339476.*
339477.*
339478.*
339479.*
```

Cette méthode n'est sûrement pas aussi efficace qu'un service en ligne, mais a le mérite de garantir le respect de ses données tout en limitant la nuisance provoquée par ces appels.

### Les réseaux sociaux

Concernant les réseaux sociaux, il est à la fois évident que disposer de ces derniers sur son téléphone pose un problème pour la vie privée et que ne pas les avoir peut être parfois embarrassant dans la vie réelle. Sans être exhaustif, voici quelques conseils pour une meilleure gestion de ces réseaux sur smartphone :

- Le premier, est de bien définir les réseaux qui nous sont utiles et ceux qui ne nous servent pas.

- Il est préférable d'utiliser les réseaux sociaux dans un contexte isolé du reste du téléphone. C'est ce que proposent des applications telles que [Shelter](https://f-droid.org/packages/net.typeblog.shelter/) ou [Island](https://github.com/oasisfeng/island) (la seconde ne fonctionnant pas sur /e/). Ces programmes vont créer des contextes dans lesquels les applications ne pourront pas accéder aux fichiers, contacts... Ces environnements seront également désactivables de sorte qu'aucune application installée dans ce contexte ne puisse tourner en arrière-plan.

- Enfin, quand c'est possible, il est préférable d'utiliser la version light des applications de réseaux sociaux. Ces versions sont développées pour de vieux appareils et font donc moins de choses. Après c'est à chacun de voir si le minimum est suffisant et si l'application est bien proposée pour un réseau social donné (typiquement la version lite de LinkedIn a été abandonnée).
    - [Facebook lite](https://apkpure.com/fr/facebook-lite/com.facebook.lite)
    - [Messenger lite](https://apkpure.com/fr/messenger-lite/com.facebook.mlite)
    - [Instagram lite](https://apkpure.com/fr/instagram-lite/com.instagram.lite)

### [NewPipe](https://newpipe.net/)

Une application alternative au viewer officiel YouTube. Elle permet de bloquer par défaut les publicités, de jouer les vidéos en arrière-plan, de les télécharger, de jouer le son uniquement, de créer des playlists de chaînes sans compte... Et cerise sur le gâteau, elle permet également d'accéder à d'autres sources de vidéos telles que [PeerTube](https://joinpeertube.org/).

### [Nextcloud News](https://f-droid.org/en/packages/co.appreactor.news/)

Un lecteur de flux RSS qu'il est possible de synchroniser avec une instance NextCloud.

### [Collabora Office](https://www.collaboraoffice.com/)

Collabora Office est un pack office basé sur LibreOffice et optimisé pour un usage mobile. L'application est assez lourde, mais très pratique pour lire et éditer des documents. Il est à noter que LibreOffice dispose depuis peu d'une [visionneuse sur mobile](https://f-droid.org/fr/packages/org.documentfoundation.libreoffice/). Cette dernière est plus légère, mais ne permet pas l'édition.

### [KeePassDX](https://www.keepassdx.com/)

KeePassDX est comme son nom l'indique une application permettant de gérer des fichiers KeePass. Elle est légère et, comme les autres outils de cette sélection, open source.

### [Proton](https://proton.me/)

La suite Proton fournit de nombreux outils tels qu'un service e-mail, un calendrier, une solution cloud, un gestionnaire de mots de passe, un VPN... Le tout très bien intégré avec Android et avec une philosophie tournée vers l'open source et la vie privée.

### [PhyPhox](https://phyphox.org/)

Une application permettant d'exploiter les différents capteurs présents dans un téléphone portable afin de faire des mesures physiques. Il est par exemple possible de mesurer l'accélération d'un déplacement, la Transformée de Fourier d'un son, la luminosité... Ces mesures sont bien souvent limitées par l'imprécision des capteurs d'un téléphone, mais peuvent néanmoins donner des ordres de grandeur.

### [TrackerControl](https://trackercontrol.org/)

Cette application permet de faire une boucle VPN en local afin d'analyser tous les flux sortants. Cela permet de mettre en évidence quelles sont les applications qui envoient des informations vers de tierces entreprises et de bloquer ces flux non désirés.

### [Termux](https://termux.com/)

Termux est une application permettant d'utiliser différents outils Linux directement sur un téléphone. Pour ce faire, l'environnement est isolé du reste du système (Android étant lui-même un Linux) dans lequel il est possible d'utiliser le gestionnaire de paquets `pkg` afin d'installer divers packages. Il est cependant important de noter qu'il ne s'agit ni de virtualisation ni de conteneurisation, mais bien d'une installation dans un emplacement réservé de l'application. Cela signifie qu'il n'est pas non plus possible de tout faire dans cette application (il est notamment impossible d'accéder à l'utilisateur root).

## Les applications à installer en tant que root

Sur un téléphone rooté, on peut aller beaucoup plus loin dans l'utilisation de son téléphone. Bien que ces applications soient inintéressantes pour le commun des mortels, elles peuvent s'avérer très utiles auprès d'utilisateurs avertis.

### [AdAway](https://adaway.org/)

AdAway est un bloqueur de pub capable de fonctionner avec deux modes différents. Si le téléphone n'est pas root, de la même façon que TrackerControl, il va créer une boucle VPN en local et filtrer les flux sur cette boucle, ce qui est assez peu performant et très gourmand en batterie. En revanche, en mode root, l'application met simplement à jour le fichier hosts de l'appareil afin que les publicités soient directement bloquées au niveau du système d'exploitation, ce qui est évidemment beaucoup plus optimisé.

### [NetHunter](https://store.nethunter.com/en/packages/com.offsec.nethunter/)

NetHunter est un outil développé par les équipes d'[OffSec](https://www.offsec.com/) qui sont également les développeurs de la distribution [KaliLinux](https://www.kali.org/). Cette application permet d'installer KaliLinux dans un chroot sur son téléphone et de le piloter via une interface.

### [NetHunter Terminal](https://store.nethunter.com/en/packages/com.offsec.nhterm/)

Cette application permet simplement d'utiliser la distribution fournie par NetHunter dans un terminal.

## Sources

- [Android Q upgrade for FP3](https://doc.e.foundation/q-upgrade-fp3)
- [/e/ ROM latest dev build downloads](https://images.ecloud.global/dev/FP3/)
- [Installation Magisk](https://topjohnwu.github.io/Magisk/install.html)
- [Support Bouygues: Impossibilité d'accéder à la 4G en ipv4](https://www.assistance.bouyguestelecom.fr/s/forum/question/0D5080000BVfeTSCQZ/impossibilit%C3%A9-dacc%C3%A9der-%C3%A0-la-4g-en-ipv4-)
- [eDrive](https://gitlab.e.foundation/e/os/eDrive)
