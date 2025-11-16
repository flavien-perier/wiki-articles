---
title: Configuration d'une montre sous AsteroidOS
type: WIKI
categories:
  - system
description: Configuration d'une montre connectée avec le système d'exploitation AsteroidOS.
author: Flavien PERIER <perier@flavien.io>
date: 2022-03-27 18:00
---

Avant toute chose, cet article traite de manipulations techniques qui pourraient, dans certains cas, causer un certain nombre de dysfonctionnements voire des pertes de données. Je n'assumerai en aucun cas la responsabilité de tels problèmes.

En préambule, je tiens à préciser que je vois personnellement d'un assez mauvais œil les montres connectées. Elles sont pour moi un gadget supplémentaire n'apportant pas de réelles innovations en matière d'usage, mais sont une source inépuisable de données pour ceux qui nous les vendent (données de déplacement, paramètres de santé...). Un système d'exploitation pour ce type d'objet a néanmoins retenu mon attention. Il s'agit d'[AsteroidOS](https://asteroidos.org/).

## Le système

AsteroidOS est un système d'exploitation pour montre connectée basé sur Linux. Les interfaces sont développées en [QT](https://www.qt.io/) et sont affichées grâce à [Wayland](https://wayland.freedesktop.org/).

Si on veut être honnête, ce système d'exploitation est très en retard par rapport à ce que peut proposer [Google WearOS](https://wearos.google.com/) ou une [Apple Watch](https://www.apple.com/fr/watch/) et ne propose que très peu d'options supplémentaires par rapport à une montre non connectée. Néanmoins, le projet est intéressant et les montres qui le supportent étant assez anciennes, il est possible de s'en procurer à moindre coût d'occasion (et donc à moindre impact environnemental).

On trouve comme fonctionnalités intégrées :

- L'affichage de l'heure.
- [Un agenda](https://github.com/AsteroidOS/asteroid-calendar).
- [Un chronomètre](https://github.com/AsteroidOS/asteroid-stopwatch).
- [Un minuteur](https://github.com/AsteroidOS/asteroid-timer).
- [Un réveil](https://github.com/AsteroidOS/asteroid-alarmclock).
- [Une calculatrice](https://github.com/AsteroidOS/asteroid-calculator).
- [Une boussole](https://github.com/AsteroidOS/asteroid-compass).
- [Une lampe torche](https://github.com/AsteroidOS/asteroid-calendar) (en réalité cela met tout simplement l'écran en blanc, ce qui ne génère pas non plus de grande quantité de lumière).
- [L'affichage de notre fréquence cardiaque à un instant T](https://github.com/AsteroidOS/asteroid-hrm) (pas de suivi ou d'enregistrement des courbes, rendant cet OS inutile pour les sportifs).
- [Boutons de gestion de la musique](https://github.com/AsteroidOS/asteroid-music) depuis la montre (pause, volume, musique suivante/précédente).
- [La météo](https://github.com/AsteroidOS/asteroid-weather).
- [Un petit jeu](https://github.com/AsteroidOS/asteroid-diamonds) inspiré de [2048](https://github.com/gabrielecirulli/2048).

## La montre

Ayant acheté une montre connectée dans l'unique objectif d'essayer cet OS, je suis parti sur celle qui avait la meilleure compatibilité à savoir la [LG Watch Urban](https://www.lg.com/fr/montres-connectees-lg-g-watch/lg-Watch-Urbane-W150).

Elle a notamment l'énorme avantage d'être connectable directement à un ordinateur sans devoir faire de bricolage au niveau du hardware.

Son bracelet est facilement changeable puisqu'il s'agit d'attache standard pour montre. Et en cas de problème au niveau hardware, il semblerait que les équipes de [iFixit](https://fr.ifixit.com/News/7178/lg-watch-urbane) aient réussi à la démonter sans trop de difficultés.

## Installation

Pour l'installation, il faut au préalable vérifier qu'[ADB](https://www.flavien.io/wiki/manjaro.md#android-adb) est bien installé sur son ordinateur.

Pour la suite, [le tutoriel présent sur le site](https://asteroidos.org/install/bass/) suffit pour installer facilement l'OS.

Je détaille néanmoins ici les étapes d'installation :

- Dans un premier temps, il faut télécharger les fichiers de la dernière version de WearOS pour la montre : [Le kernel](https://androidfilehost.com/?fid=529152257862690904), [Le recovery](https://www.androidfilehost.com/?fid=457095661767128807) et [La ROM](https://androidfilehost.com/?fid=745425885120695961).

- Puis les fichiers d'AsteroidOS : [Les données utilisateur](https://release.asteroidos.org/nightlies/bass/asteroid-image-bass.rootfs.ext4) et [La ROM](https://release.asteroidos.org/nightlies/bass/zImage-dtb-bass.fastboot).

- La stabilité de distribution étant ce qu'elle est, il peut-être bon de remplacer le recovery par [TWRP](https://dl.twrp.me/bass/). Cela peut permettre d'effectuer des backups.

- Ensuite, il faut accéder au bootloader de l'appareil. Pour ce faire, il suffit d'éteindre la montre puis de la rallumer. Au démarrage quand le logo "LG" apparait il faut faire glisser son doigt du coin supérieur gauche de l'écran au coin inférieur droit.

- Ensuite il faut brancher l'appareil à un ordinateur via le câble USB.

- Puis taper les commandes :

```bash
fastboot oem unlock

fastboot flash recovery recovery.img
fastboot flash system MM-system-ext4-*.img
fastboot flash boot Skin1980-boot-*.img

fastboot continue
# Attendre que l'os de base ait fini de s'installer avant de revenir sur fastboot et continuer.

fastboot flash recovery twrp-*-bass.img
fastboot flash userdata asteroid-image-bass.rootfs.ext4
fastboot flash boot zImage-dtb-bass.fastboot

fastboot oem lock

fastboot continue
```

## Application téléphone

Par défaut, il est possible d'utiliser [l'application mobile officielle d'AsteroidOs](https://f-droid.org/fr/packages/org.asteroidos.sync/). Cependant, cette dernière est assez peu maintenue et relativement instable. Il est donc préférable d'utiliser [GadgetBridge](https://f-droid.org/fr/packages/nodomain.freeyourgadget.gadgetbridge/), une alternative plus complète également en mesure de piloter des montres qui ne fonctionnent pas sous AsteroidOs.

## Connexion Wi-Fi

Pour se connecter à la montre, il faut aller dans les paramètres de la montre puis configurer le mode USB sur `Mode ADB`, puis taper les commandes sur un pc pendant que la montre est connectée :

```bash
adb usb
adb shell
```

Ensuite, pour se connecter en WiFi, il faut taper les commandes :

```bash
WIFI_SSID=*********
WIFI_PASSWORD=*********

rm -Rf /var/lib/connman/wifi*

connmanctl enable wifi
connmanctl scan wifi

WIFI=`connmanctl services | grep $WIFI_SSID | tr -s " " | cut -f3 -d ' '`

cat << EOL > /var/lib/connman/$WIFI.config
[service_wifi_$WIFI]
Type = wifi
Name = $WIFI_SSID
Passphrase = $WIFI_PASSWORD
Security = psk
AutoConnect=true
EOL

ifconfig wlan0 down
ifconfig wlan0 up

systemctl restart connman
```

## Watchface

J'utilise une watchface custom pour la montre. Pour l'installer, il suffit de taper les commandes depuis un pc :

```bash
cd /tmp
git clone -q --depth 1 -- https://github.com/flavien-perier/asteroid-watchface.git
cd asteroid-watchface
adb push usr/ /
adb shell systemctl restart user@1000
```

## System

Le gestionnaire de paquet de la distribution est `opkg`. Il s'utilise globalement comme `apt-get` sur les systèmes [Debian](https://www.debian.org/).

Il faut cependant être attentif à ne pas faire d'`opkg dist-upgrade`. Cela pose un problème au moment de la mise à jour de la [BusyBox](https://www.busybox.net/) et casse complètement l'OS... Donc obligé de tout réinstaller.

De même, des upgrades globaux (`opkg upgrade`) peuvent introduire des instabilités plus ou moins graves... Chose malheureusement symptomatique des distributions mal ou insuffisamment testées. Une fois la distribution installée, le plus sage est donc malheureusement de ne plus toucher à rien.

## Affichage de l'IP après connexion au WiFi

L'objectif de ce script est simplement d'afficher l'IP de la montre avec une notification quand cette dernière se connecte à un WiFi.

```bash
echo '#!/bin/sh

if [ ! -f /tmp/network-state ]
then
  echo "ready" > /tmp/network-state
fi

STATE=`connmanctl state | grep State | tr -d " " | cut -f2 -d=`
LAST_STATE=`cat /tmp/network-state`

if [ $STATE = "ready" ]
then
  ping -q -c 1 1.1.1.1
  if [ $? -ne 0 ]
  then
    STATE="offline"
  fi
fi

if [ $STATE = $LAST_STATE ]
then
  exit
fi

if [ $STATE = "ready" ]
then
  IP=`ip -br -4 a show wlan0 | tr -s " " | cut -f3 -d " " | cut -f1 -d "/"`
  su -l ceres -c "notificationtool -o add --icon=ios-wifi --application=\"System\" --urgency=3 --hint=\"x-nemo-preview-summary WiFi\" --hint=\"x-nemo-preview-body IP: $IP\" \"WiFi\" \"IP: $IP\""
fi

echo $STATE > /tmp/network-state' | tee /home/root/notification-wifi
chmod 550 /home/root/notification-wifi

cat << EOL > /etc/systemd/system/notification-wifi.service
[Unit]
Description=Notification WiFi
Wants=notification-wifi.timer

[Service]
ExecStart=/home/root/notification-wifi
Type=oneshot

[Install]
WantedBy=network-online.target
EOL

cat << EOL > /etc/systemd/system/notification-wifi.timer
[Unit]
Description=Notification WiFi
Requires=notification-wifi.service

[Timer]
Unit=notification-wifi.service
OnUnitInactiveSec=2m

[Install]
WantedBy=timers.target
EOL

systemctl enable notification-wifi.service
systemctl enable notification-wifi.timer
systemctl start notification-wifi.service
systemctl start notification-wifi.timer
systemctl daemon-reload
```

## Sources

- [AsteroidOS](https://asteroidos.org/)
- [AsteroidOS LG G Watch Urbane](https://asteroidos.org/install/bass/)
- [AsteroidOS Documentation IP Connection](https://asteroidos.org/wiki/ip-connection/)
- [Forum XDA](https://forum.xda-developers.com/t/rom-kernel-ext4all-by-skin1980-m1d64s-update-10-12-2016.3341598/)
