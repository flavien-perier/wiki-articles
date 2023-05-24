---
title: Environnement Android
type: WIKI
categories:
  - system
description: Environement Android Orienté vie privée.
author: Flavien PERIER <perier@flavien.io>
date: 2020-11-10 19:00
---

Avant toute chose, cet article traite de manipulation technique qui pourraient dans certains cas causer un certain nombre de dysfonctionnements voir des pertes de données. Je n'assumerais en aucun cas la responsabilité de tels problèmes.

Dans cet article nous allons nous intéresser à la mise en place d'un environnement orient vie privée et sécurité sur un téléphone portable.

## Le téléphone

Au niveau du téléphone, j'ai personnellement fait le choix de m'orienter sur un [Fairphone 3](https://www.fairphone.com/). Il s'agit d'un téléphone modulaire présentant un grand nombre d'avantages, tant au niveau éthique, que d'un point de vue purement technique.

Tout d'abord, il est important de souligner que l'entreprise essaye de s'orienter le plus possible vers des matériaux issus du commerce équitable. Pour en savoir plus sur cet aspect, je recommande vivement d'aller consulter le [blog](https://www.fairphone.com/fr/blog/) de l'entreprise.

L'autre aspect du Fairphone qui va plus nous intéresser dans cet article, c'est son aspect modulaire. Ce téléphone étant conçu dans une optique de durabilité, il est possible de changer en quelques poignées de minutes n'importe quels composants. Cela peut se faire tant au niveau hardware (changement de l'écran, caméra, carte mère...) qu'au niveau software. En effet, contrairement à beaucoup d'autres constructeurs, Fairphone fait en sorte qu'il soit facile de remplacer le système d'exploitation par défaut de son téléphone.

L'inconveignet souvent répété de ce téléphone est sa puissance jugée trop faible pour un téléphone de ce prix (plus ou moins 450 €). Mais soyons réalistes, si vous ne jouez pas à des jeux 3D, son processeur [Snapdragon 632](https://www.qualcomm.com/products/snapdragon-632-mobile-platform) et ses 4Go de RAM devraient être largement suffisants pour tous vos usages. Quant au niveau du prix, il faut bien mettre dans la balance le fait que le téléphone étant fait pour durer le support des mises à jour sera poussé tant que possible (5 ans minimum) contrairement à la plupart des constructeurs (hors Apple) qui partent sur une base de 18 mois. Pour ceux qui se disent qu'ils peuvent continuer à utiliser leur téléphone sans mise à jour de sécurité, je tiens simplement à rappeler qu'Android est aujourd'hui l'un des systèmes d'exploitation les plus attaqués et que par conséquent, ne pas le mettre revient à mettre en péril les données qu'on y stocke, notamment quand on sait que beaucoup d'utilisateurs n'hésitent pas à installer leurs applications bancaires favorites sur leur téléphone.

## Le système d'exploitation

Comme je l'ai dit précédemment, le Fairphone est compatible avec d'autres variantes d'Android que celle qui nous est généreusement fournie par Google.

La variante d'Android la plus connue est [Lineage OS](https://lineageos.org/) (anciennement [Cyanogen Mod](https://fr.wikipedia.org/wiki/CyanogenMod)) qui nous propose un Android sans couche Google installée par défaut. Comme le projet est open source, il existe d'autres distributions qui s'appuient sur cette dernière et notamment celle qui va nous intéresser, [/e/](https://e.foundation/).

Le slogan de /e/ est "Your data is YOUR data!" (Vos données sont vos données). L'objectif est ici de mettre en place un certain nombre de mécaniques qui vont permettre de garantir au mieux la vie privée de l'utilisateur. L'autre avantage de cette distribution est qu'elle se trouve en [partenariat](https://www.fairphone.com/fr/2020/04/30/keeping-your-data-safe-with-e-os/) direct avec l'entreprise Fairphone et que par conséquent les développeurs s'appliquent à ce que le système soit le plus stable possible sur cet équipement. Il est donc possible d'[installer](https://doc.e.foundation/devices/FP3/install) facilement ce système sur un téléphone, ou tout simplement en [acheter](https://esolutions.shop/fr/shop/murena-fairphone-3-plus-fr/) un déjà préinstallé.

## Faire une sauvegarde

Si le téléphone est actuellement utilisé, il est possible de sauvegarder une grande partie des données déjà en place grâce à des commandes ADB. Pour ce faire, je conseille cette page [GitHub d'AnatomicJC](https://gist.github.com/AnatomicJC/e773dd55ae60ab0b2d6dd2351eb977c1).

Tout d'abord, il faut vérifier qu'[ADB](https://www.flavien.io/wiki/manjaro.md#android-adb) est bien installé sur son PC afin de pouvoir manipuler son mobile.

Ensuite il faut activer le mode développeur sur le téléphone (aller dans les paramètres, puis dans "À propos du téléphone" et cliquer 7 fois sur le numéro du build). Une fois le menu développeur visible s'y rendre et activer le "debogage USB".

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

## Installation de [/e/](https://e.foundation/) + root avec [Magisk](https://github.com/topjohnwu/Magisk/) sur Fairphone 3

En premier lieu, il faut télécharger les différents composants pour l'installation :

- [/e/ canal stable](https://images.ecloud.global/stable/FP3/) ou [/e/ canal dev](https://images.ecloud.global/dev/FP3/)
- L'APK [Magisk 24.2](https://github.com/topjohnwu/Magisk/releases/tag/v24.2)

Ensuite, il faut accéder au mode recovry du téléphone (l'éteindre et le rallumer en maintenant les bouton "marche/arrêt" et "volume +" enfoncés) afin d'installer le système (toutes les données utilisateurs vont être perdues).

```bash
unzip *

fastboot flashing unlock

fastboot -w

fastboot flash system_a system.img -S 522239K
fastboot flash boot_a boot.img
fastboot flash vendor_a vendor.img -S 522239K
fastboot flash dtbo_a dtbo.img
fastboot flash vbmeta_a vbmeta.img

fastboot flash system_b system.img -S 522239K
fastboot flash boot_b boot.img
fastboot flash vendor_b vendor.img -S 522239K
fastboot flash dtbo_b dtbo.img
fastboot flash vbmeta_b vbmeta.img
```

Une fois le système de base installé, il est possible de rooter le téléphone :

- Dans un premier temps, il va falloir le démarrer et y installer l'application Magisk (grâce à l'APK précédemment récupérée).
- Temps que le téléphone est branché à un ordinateur, nous allons en profiter pour déposer sur le stockage de l'appareil le fichier `boot.img`.
- Depuis l'application Magisk il faut sélectionner le bouton `installer` depuis l'onglet `Magisk`. Par la suite, on sélectionne le fichier `boot.img`. L'application va alors appliquer ses correctifs root sur le fichier.
- On récupère le fichier généré dans le dossier Téléchargement et on le dépose sur l'ordinateur.
- On redémarre le téléphone sur le bootloader.
- On tape les commandes suivantes sur un terminale de son ordinateur :

```bash
fastboot flash boot_a magisk_patched-*.img
fastboot flash boot_b magisk_patched-*.img
```

Remarque : Avec le root le bootloader ne devra jamais être reverrouillé. Visiblement une vérification d'intégrité est effectuée à ce moment-là et le root est considéré comme une anomalie.

## Configuration de l'APN Bouygues Telecom

L'[APN](https://fr.wikipedia.org/wiki/Access_Point_Name) est une configuration permettant de se connecter au réseau mobile de l'opérateur afin de pouvoir envoyer et recevoir des MMS ainsi que d'accéder à internet au travers de la 3G/4G/5G.

Cependant dans leur infinie sagesse les opérateurs peuvent nous passer en IPv6. Le futur d'internet, plus sécurisé, sur lequel on pourra brancher plus d'appareils... Et sur lequel de nombreux services ne fonctionnent pas encore : [Discord](https://discord.com/), [ProtonVPN](https://protonvpn.com/) et biens d'autres.

Si on utilise ces services, il faut repasser en IPv4 pour que ca fonctionne et pour cela il faut la configuration APN adéquate :

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

## Installation des polices de caractères

Cette manipulation n'est pas vraiment utile, elle consiste simplement à rajouter une police de caractère ([JetBrains mono](https://www.jetbrains.com/lp/mono/) dans le cas présent) dans l'os.

- Vérifier que le "Débogage usb" et le "Déboggage rooté" sont activés dans les "Options pour les développeurs".
- On connecte le téléphone à l'ordinateur.
- On tape les commandes suivantes :

```bash
adb usb
adb root
cd /tmp
wget https://download.jetbrains.com/fonts/JetBrainsMono-2.304.zip
unzip JetBrainsMono-2.304.zip
cd fonts/ttf
adb push * /system/fonts
adb reboot
```

### Applications Par Default

Par défaut le système d'exploitation contient un minimum d'applications pour être utilisable (Dialer, SMS, Launcher, Calendar...). Bien évidemment, je ne vais pas parler de toutes ces applications, déjà parce que ça prendrait beaucoup de temps, mais surtout parce que ça n'apporterait pas grand-chose. je vais donc me contenter de parler de quelques-unes d'entre elles :

#### [MicroG](https://microg.org/download.html)

Comme dit plus tôt, ce système d'exploitation est conçu pour communiquer le moins possible avec Google. Seul problème, Android et son Framework sont développés par Google. Les interactions avec les serveurs de la société sont donc nombreuses et toutes les coupés causeraient de grave dysfonctionnement pour de très nombreuses applications.

Heureusement, des développeurs se sont penchés sur le sujet et ont conçu MicroG. Ce dernier simule le comportement des composants Google sur un téléphone utilisant un système Android natif en envoyant le moins d'informations possible sur internet. Cette application est donc un très bon équilibre entre téléphone fonctionnel et vie privée.

#### Le store

/e/ ayant pour objectif de préserver la vie privée de ses utilisateurs, l'implémentation du Google Play Store aurait été un très mauvais choix... C'est pour cette raison que le système d'exploitation possède son propre métastore nommé App Lounge.

Pourquoi un métastore et pas un store ? Parce que l'entreprise ne stocke pas elle-même les applications, mais nous les fournit en allant les chercher directement sur le [Google Play Store](https://play.google.com/) ou sur [F-Droid](https://f-droid.org/).

Cela présente l'avantage d'avoir un catalogue très fourni (puisque constitué de la fusion de deux très grands stores Android), sans risque d'installer une application non certifiée (puisque les applications sont certifiées sur ces deux stores) et de ne pas compromettre la vie privée des utilisateurs puisque les serveurs de /e/ servent de proxy pour aller récupérer les applications sur les stores tiers.

Parmi les fonctionnalités amusantes de ce store, il y a la possibilité d'afficher une note de confidentialité calculée sur le nombre de traqueurs implémenté et la quantité d'autorisations demandées par une application. Tous ces résultats sont obtenus grâce au service [Exodus](https://exodus-privacy.eu.org/).

#### Le cloud

Par défaut /e/ propose la possibilité de ce connecter à une instance [NextCloud](https://nextcloud.com/) afin d'y synchroniser ses données. Il est donc possible d'y sauvegarder des photos, agenda, contacts... sans passer par un GAFAM, voir, sauvegarder toutes ses données sur une instance auto hébergée. Heureusement, pour les utilisateurs ne souhaitant pas consacrer du temps à la mise en place d'une instance, la fondation propose son propre cloud (ce dernier n'offre malheureusement que quelques mégaoctets). Suffisant néanmoins pour synchroniser des contacts et agendas (il faudra laisser de côté la sauvegarde des photos).

#### Le navigateur

Forcément pas de Google Chrome en vue, mais du [Chromium](https://www.chromium.org/) (la version open source du navigateur). Quant au moteur de recherche par défaut, là encore, pas de Google, mais un métamoteur aux couleurs de /e/ localisé à l'URL [spot.ecloud.global](https://spot.ecloud.global/). Ce dernier nous restitue une agrégation des résultats trouvée sur d'autres moteurs de recherches tels que [Qwant](https://www.qwant.com/), [DuckDuckGo](https://duckduckgo.com/), [Wikipédia](https://fr.wikipedia.org/)... Toutes les requêtes envoyées par ce métamoteur à des tiers sont routées à travers le réseau [Tor](https://www.torproject.org/). L'architecture de ce moteur semble donc très confidentielle et même si je ne l'utilise pas personnellement je trouve sa conception très intéressante.

#### Le launcher

Appelé Bliss Lancher, le lanceur d'application de la distribution est une copie de l'interface Apple, avec des fonctionnalités en moins et des bugs en plus. Pour avoir testé beaucoup d'autre launchers tel que [NovaLauncher](https://novalauncher.com/) ou [Lawnchair](https://lawnchair.app/), je peux affirmer qu'il manque beaucoup de fonctionnalités comme la gestion des gestes, un tiroir et des paramètres de manière générale. Cependant, contrairement à d'autres lanceurs d'applications tels que [Mikcrosoft Launcher](https://apkpure.com/fr/microsoft-launcher/com.microsoft.launcher) celui-ci a les immenses avantages de respecter notre vie privée, de ne passe pas son temps à interagir avec le réseau, d'être open source et économe en batterie pour une application de ce type. Il ne s'agit donc clairement pas de l'application de l'année, mais à l'avantage de faire ce qu'on lui demande.

#### [Magic Earth](https://www.magicearth.com/)

Magic Earth est un simple GPS basé sur [Open Street Map](https://www.openstreetmap.org/) avec une interface très propre et intuitive qui propose entre autres choses de télécharger les cartes en local. Grâce à cette application, il est possible d'utiliser le téléphone en tant que GPS sans accéder aux données. Cela permet de réduire sa chauffe, lui permettre de rester plus longtemps allumé et de ne pas être épié par un quelconque [GAFAM](https://fr.wikipedia.org/wiki/G%C3%A9ants_du_Web) durant un voyage.

#### [OpenKeychain](https://www.openkeychain.org/)

Simple application pour gérer vos [clés de chiffrement asymétrique](https://fr.wikipedia.org/wiki/Cryptographie_asym%C3%A9trique). Il est possible d'utiliser cette application pour signer/chiffrer des SMS, textes ou tout autre fichier.

## Les applications à installer

### L'antivirus

Android étant un système d'exploitation particulièrement vulnérable, il est donc important de posséder un antivirus digne de ce nom.

J'ai personnellement utilisé l'Antivirus [Kaspersky Internet Security](https://www.kaspersky.fr/internet-security) pendant plusieurs années. Cependant ce dernier étant conçu en Russie par d’anciens membres des services de renseignement, plusieurs entreprises et organismes conseillent d'éviter son utilisation dans le contexte de [guerre entre la Russie et l'Ukraine](https://fr.wikipedia.org/wiki/Guerre_du_Donbass). Un [article de Zataz](https://www.zataz.com/kaspersky-dangereux-ou-pas/) détail la situation en pesant les arguments pour et contre l'utilisation de Kapersky.

Même si je n'ai aucune preuve tangible que cet antivirus est néfaste (et au demeurant je ne pense personnellement pas qu'il le soit), il vaut mieux s'appuyer sur une solution sur laquelle on peut avoir toute confiance. C'est pourquoi je me suis orienté vers [Avira security](https://www.avira.com/fr/free-antivirus-android).

De manière générale les performances des antivirus sont changeantes d'une année sur l'autre. Il peut donc être pertinent de consulter fréquemment le site [AV-Test](https://www.av-test.org/fr/antivirus/portables/) afin de voir les derniers comparatifs de ces solutions de sécurité.

### Les réseaux sociaux

Pour les utilisateurs de réseaux sociaux, il est évident qu'il est pratique d'avoir accès à ces derniers directement depuis son mobile.

Le premier conseil que je peux donner, est de bien définir les réseaux que vous utiliser afin d'éviter d'installer du superflux.

Le second conseil que je peux donner est de privilégier quand c'est possible les versions "lite" de vos réseaux sociaux. Ces dernières sont développées pour les téléphones peu puissants (ce qui n'est pas le cas du Fairphone) et effectuent donc moins d'opérations (et sont donc un peu plus respectueuses de la vie privée). En voici une liste :

- [Facebook lite](https://apkpure.com/fr/facebook-lite/com.facebook.lite)
- [Messenger lite](https://apkpure.com/fr/messenger-lite/com.facebook.mlite)
- [Instagram lite](https://apkpure.com/fr/instagram-lite/com.instagram.lite)

### [KDE Connect](https://kdeconnect.kde.org/)

Logiciel permettant d'utiliser son téléphone depuis son ordinateur à travers le réseau local. Il est ainsi possible d'envoyer et de recevoir des SMS sur son PC, ou encore de partager son presse-papier entre les deux appareils.

En fonction de l'environnement de bureau installé sur l'ordinateur de l'utilisateur, il sera possible d'opter pour une implémentation ou une autre.

- Pour les utilisateurs sous [KDE](https://kde.org/), c'est bien évidemment l'implémentation officielle qui est recommandée.
- Sous [Gnome](https://www.gnome.org/), il sera possible d'utiliser [GSConnect](https://github.com/GSConnect/gnome-shell-extension-gsconnect).
- Enfin pour les utilisateurs de [XFCE](https://xfce.org/), il sera possible d'utiliser [Valent](https://valent.andyholmes.ca/). Un paragraphe est dédié à ce logiciel dans [la section du wiki consacré à Manjaro](https://www.flavien.io/wiki/manjaro.md#valent).

### [NewPipe](https://newpipe.schabi.org/)

Aujourd'hui je peux affirmer que je me passe de tous les services Google à une exception près, YouTube. Heureusement sur mobile il existe une application qui me permet d'accéder au contenu de YouTube sans installer l'application YouTube officielle. Il est possible de regarder les vidéos en arrière-plan (pour écouter de la musique, même si je déconseille de le faire, car écouter de la musique sur YouTube n'est pas un acte écoresponsable en termes de GreenIt), de les télécharger, de ne pas avoir les pubs et de faire des playlists locales (qui ne sont pas sauvegardées sur les serveurs de Google).

### [Flym](https://github.com/FredJul/Flym)

Un lecteur de flux RSS léger et open source.

### [Bookeen](https://apkpure.com/fr/bookeen/fr.bookeen.qboo.reader)

[Bookeen](https://bookeen.com/) est une entreprise française spécialisée dans la construction d'eBook et la diffusion de livre au format numérique depuis [leur store](https://www.bookeenstore.com/).

Leur application permet simplement de reprendre la lecture sur téléphone la ou on l'a laissé sur liseuse, ou simplement d'ouvrir un fichier ePub. 

### [Collabora Office](https://www.collaboraoffice.com/)

Pendant assez longtemps, la distribution possédait son implémentation d'une visionneuse de document office. Cependant, étant donné qu'il n'y a pas d'implémentation officielle, cela semble compliqué à maintenir dans le temps (d'autant plus que l'implémentation qui avait été choisie n'était pas des plus efficaces).

Cependant, Collabora semble [avoir le soutient de la fondation Libre Office](https://fr.libreoffice.org/download/android-et-ios/) et ce trouve être basé sur leur solution. Il s'agit donc pour moi de la meilleure solution pour le moment.

### [Aegis](https://getaegis.app/)

Aegis est une application de calcul des seconds facteurs d'authentification complètement open source. Elle à l'avantage de ne comporter aucun traqueur et de la possibilité d'être déverrouillé par empreinte digitale. Il est possible d'importer facilement les données d'une autre application 2FA tel que [AndOTP](https://github.com/andOTP/andOTP), [Google Authentificator](https://apkpure.com/fr/google-authenticator/com.google.android.apps.authenticator2), [Microsoft Authentificator](https://apkpure.com/fr/microsoft-authenticator/com.azure.authenticator) et bien d'autres.

### [KeePassDX](https://www.keepassdx.com/)

KeePassDX est comme son nom l'indique une application permettant de gérer des fichiers KeePass. Elle est légère, et comme très autres outils de cette sélection, open source.

### [PhyPhox](https://phyphox.org/)

Une application permettant d'exploiter les différents capteurs présents dans un téléphone portable afin de faire des mesures physiques. Il est par exemple possible de mesurer l'accélération d'un déplacement, la Transformée de Fourier d'un son, la luminosité... Ces mesures sont bien souvent limitées par l'imprécision des capteurs d'un téléphone, mais peuvent néanmoins donner des ordres de grandeur.

### [TrackerControl](https://trackercontrol.org/)

Cette application permet de faire une boucle VPN en local afin d'analyser tous les flux sortants. Cela permet de mettre en évidence qu’elles sont les applications qui envoient des informations vers de tierces entreprises et de bloquer ces flux non désirés.

### [Termux](https://termux.com/)

Termux est une application permettant d'utiliser différents outils Linux directement sur un téléphone. Pour ce faire, l'environnement est isolé du reste du système (Android étant lui-même un Linux) dans lequel il est possible d'utiliser le gestionnaire de paquets `pkg` afin d'installer divers packages. Il est cependant important de noter qu'il ne s'agit ni de virtualisation ni de conteneurisation, mais bien d'une installation dans un emplacement réservé de l'application. Cela signifie qu'il n'est pas non plus possible de tout faire dans cette application (il est notamment impossible d'accéder à l'utilisateur root).

## Les applications à installer en temps que root

Sur un téléphone rooter, on peut aller beaucoup plus loin dans l'utilisation de son téléphone. Bien que ces applications soient inintéressantes pour le commun des mortels, elles peuvent s'avérer très utiles auprès d'utilisateurs avertis.

### [AdAway](https://adaway.org/)

AdAway est un bloqueur de pub capable de fonctionner avec deux modes différents. Si le téléphone n'est pas root, de la même façon que TrackerControl, il va créer une boucle VPN en local et filtrer les flux sur cette boucle. Ce qui est assez peu performant et très gourmand en batterie. En revanche en mode root, l'application met simplement à jour le fichier host de l'appareil afin que les publicités soient directement bloquées au niveau du système d'exploitation. Ce qui est évidemment beaucoup plus optimisé.

### [Kernel Adiutor](https://f-droid.org/en/packages/com.nhellfire.kerneladiutor/)

Une application permettant de modifier les paramètres du kernel d'Android. Il est par exemple possible de modifier la fréquence de ses processeurs. Personnellement, je la baisse de quelques MHz afin de gagner en autonomie et limiter la chauffe de mon téléphone (ce qui n'est pas forcément pertinent en fonction de l'utilisation qu'on a de son appareil).

### [Linux deploy](https://apkpure.com/linux-deploy/ru.meefik.linuxdeploy)

Permets d'installer un conteneur Linux sur un téléphone portable. Plusieurs distributions sont disponibles [Arch](https://archlinux.org/), [Debian](https://www.debian.org/), [Kali](https://www.kali.org/)...

Contrairement à Termux, s'agissant de véritable conteneurisation, il est possible d'utiliser toutes les fonctionnalités d'un véritable Linux.

### [NetHunter Store](https://store.nethunter.com/)

Développée par les équipes de [Kali Linux](https://www.kali.org/), la distribution [NetHunter](https://www.kali.org/docs/nethunter/) contient un grand nombre d'outils de pentest.

Cette distribution n'étant disponible que pour un nombre restreint d'appareils (et parce qu'un téléphone sert à autre chose que rechercher des vulnérabilités sur un réseau), nous pouvons seulement installer le store de NetHunter sur leur site. Une fois cette application installée, on pourra installer les autres composants de NetHunter.

Une autre façon d'accéder à ces applications (façon que je préfère personnellement) est d'installer le store [F-droid](https://f-droid.org/) (sur lequel est basé le celui de NetHunter), d'aller dans les paramètres et de rajouter le dépôt `https://store.nethunter.com/repo?fingerprint=7E418D34C3AD4F3C37D7E6B0FACE13332364459C862134EB099A3BDA2CCF4494`.

#### [NetHunter](https://store.nethunter.com/en/packages/com.offsec.nethunter/)

Le composant principal de l'écosystème, c'est lui qui va permettre de télécharger et installer KaliLinux sur le téléphone.

#### [NetHunter Terminal](https://store.nethunter.com/en/packages/com.offsec.nhterm/)

Le terminal qui va permettre d'accéder en ligne de commande à Android et au système Linux installé sur le téléphone.

## Sources

- [Android Q upgrade for FP3](https://doc.e.foundation/q-upgrade-fp3)
- [/e/ ROM latest dev build downloads](https://images.ecloud.global/dev/FP3/)
- [Installation Magisk](https://topjohnwu.github.io/Magisk/install.html)
- [Support Bouygues: Impossibilité d'accéder à la 4G en ipv4](https://www.assistance.bouyguestelecom.fr/s/forum/question/0D5080000BVfeTSCQZ/impossibilit%C3%A9-dacc%C3%A9der-%C3%A0-la-4g-en-ipv4-)