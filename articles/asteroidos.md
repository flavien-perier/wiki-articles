---
title: Configuration d'une montre sous AsteroidOS
type: WIKI
categories:
  - system
description: Configuration d'une montre connectée avec le système d'exploitation AsteroidOS.
author: Flavien PERIER <perier@flavien.io>
date: 2022-03-27 18:00
---

Avant toute chose, cet article traite de manipulation technique qui pourraient dans certains cas causer un certain nombre de dysfonctionnements voir des pertes de données. Je n'assumerais en aucun cas la responsabilité de tels problèmes.

En préambule, je tiens à préciser que je vois personnellement d'un assez mauvais œil les montres connectées. Elles sont pour moi un gadget supplémentaire n'apportant pas de réelles innovations en matière d'usage, mais sont une source inépuisable de données pour ceux qui nous les vendent (données de déplacement, paramètres de santé...). Un système d'exploitation pour ce type d'objet a néanmoins retenu mon attention. Il s'agit d'[AsteroidOS](https://asteroidos.org/).

## Le système

AsteroidOS est un système d'exploitation pour montre connectée basé sur Linux. Les interfaces sont développées en [QT](https://www.qt.io/) et sont affichées grâce à [Wayland](https://wayland.freedesktop.org/).

Si on veut être honnête ce système d'exploitation est très en retard par rapport à ce que peux proposer [Google WearOS](https://wearos.google.com/) ou une [Apple Watch](https://www.apple.com/fr/watch/) et ne propose que très peu d'options supplémentaires par rapport à une montre non connectée. Néanmoins le projet est intéressent et les montres qui le supporte étant assez anciennes, il est possible de s'en procurais à moindre cout d'occasion (et donc à moindre impacte environnementale).

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

Elle a notamment l'énorme l'avantage d'être connectable directement à un ordinateur sans devoir faire de bricolage au niveau du hardware.

Son bracelet est facilement changeable puisqu'il s'agit d'attache standard pour montre. Et en cas de problème au niveau hardware, il semblerait que les équipes de [iFixit](https://fr.ifixit.com/News/7178/lg-watch-urbane) ait réussi à la démonter sans trop de difficultés.

## Installation

Pour l'installation il faut au préalable vérifier qu'[ADB](https://www.flavien.io/wiki/manjaro.md#android-adb) est bien installé sur son ordinateur.

Pour la suite [le tutoriel présent sur le site](https://asteroidos.org/install/bass/) suffit pour installer facilement l'OS.

Je détaille néanmoins ici les étapes d'installation :

- Dans un premier temps, il faut télécharger les fichiers de la dernière version de WearOS pour la montre : [Le kernel](https://androidfilehost.com/?fid=529152257862690904) et [La ROM](https://androidfilehost.com/?fid=745425885120695961).

- Puis les fichiers d'AsteroidOS : [Les données utilisateur](https://release.asteroidos.org/nightlies/bass/asteroid-image-bass.ext4) et [La ROM](https://release.asteroidos.org/nightlies/bass/zImage-dtb-bass.fastboot).

- Ensuite, il faut accéder au bootloader de l'appareil. Pour ce faire, il suffit d'éteindre la montre puis de la rallumer. Au démarrage quand le logo "LG" apparait il faut faire glisser son doigt du coin supérieur gauche de l'écran au coin inférieur droit.

- Ensuite il faut brancher l'appareil à un ordinateur via le câble USB.

- Puis taper les commandes :

```sh
fastboot oem unlock

fastboot flash system MM-system-ext4-*.img
fastboot flash boot Skin1980-boot-*.img

fastboot flash userdata asteroid-image-bass.ext4
fastboot flash boot zImage-dtb-bass.fastboot

fastboot oem lock

fastboot continue
```

## Connection Wi-Fi

Pour se connecter à la montre, il faut aller dans les paramètres de la montre puis configurer le mode USB sur `Mode ADB`, puis tapant les commandes sur un pc pendant que la montre est connectée :

```sh
adb usb
adb shell
```

Ensuite pour ce connecter en WiFi il faut taper les commandes : 

```sh
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

## Connection SSH

Dans un premier temps, depuis un pc, il faut générer une clé privée pour la connexion SSH et la transférer vers l'équipement :

```sh
MONTRE_IP=192.168.X.X

ssh-keygen -f ~/.ssh/montre -t rsa -b 4096
ssh-copy-id -i ~/.ssh/montre ceres@$MONTRE_IP

cat << EOL >> ~/.ssh/config
Host montre
    HostName $MONTRE_IP
    User ceres
    Port 22
    IdentityFile ~/.ssh/montre
    IdentitiesOnly yes
EOL
```

Ensuite, se connecter sur la montre avec la commande `ssh ceres@MONTRE_IP` puis taper les commandes :

```sh
passwd
su
cat << EOL > /etc/ssh/sshd_config
PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes
Compression no
ClientAliveInterval 15
ClientAliveCountMax 4
Subsystem       sftp    /usr/libexec/sftp-server
EOL
systemctl reload sshd
```

## System

Le gestionnaire de paquet de la distribution est `opkg`. Il s'utilise globalement comme `apt-get` sur les systèmes [Debian](https://www.debian.org/).

La seule notre très importante est de ne pas faire d'`opkg dist-upgrade`. Cela pose un problème au moment de la mise à jour de la [BusyBox](https://www.busybox.net/) et casse complètement l'OS... Donc obligé de tout réinstaller.

## Script d'optimisation de la batterie

`crontab` n'est malheureusement pas installé sur cet OS et aucune alternative n'est disponible dans les repos d'`opkg`. L'objectif est donc de créer des scripts sous forme de machine à état appelé à des moments précis à l'aide de `systemd`.

Ce premier script possède 4 états :

- `night`: Pendant la nuit le Bluetooth, et le WiFi sont coupés, la luminosité et à son minimum et le seul état vers lequel on peut aller depuis cet état est `day`.
- `day`: État de transition permettant de réactiver Bluetooth et WiFi. Depuis cet état on peut aller vers `indoor` ou `outdoor`.
- `indoor`: La montre est à proximité d'un réseau WiFi connu. Elle est donc en intérieur. La luminosité est à son minimum.
- `outdoor`: La montre n'est pas à proximité d'un réseau WiFi connu. Elle est donc en extérieur. La luminosité est à son maximum.

```sh
echo '#!/bin/sh

if [ $# -ne 1 ]
then
  exit 1
fi

if [ ! -f /tmp/alim-state ]
then
  echo "init" > /tmp/alim-state
fi

LAST_STATE=`cat /tmp/alim-state`

case $1 in
day)
  mcetool --set-display-brightness 100
  mcetool --set-power-saving-mode disabled
  mcetool --set-low-power-mode enabled
  ifconfig wlan0 up
  systemctl start sensorfwd
  systemctl start ofono
  systemctl start connman
  systemctl start bluetooth
  systemctl start sshd

  echo "day" > /tmp/alim-state
  echo "init" > /tmp/network-state
;;
night)
  mcetool --set-display-brightness 1
  mcetool --set-power-saving-mode enabled
  mcetool --set-low-power-mode disabled
  ifconfig wlan0 down
  systemctl stop sensorfwd
  systemctl stop ofono
  systemctl stop connman
  systemctl stop bluetooth
  systemctl stop sshd

  echo "night" > /tmp/alim-state
;;
indoor)
  if [ $LAST_STATE == "night" ] || [ $LAST_STATE == "indoor" ]
  then
    exit 0
  fi

  mcetool --set-display-brightness 1
  mcetool --set-power-saving-mode enabled
  mcetool --set-low-power-mode disabled
  systemctl stop sensorfwd
  systemctl start sshd

  echo "indoor" > /tmp/alim-state
;;
outdoor)
  if [ $LAST_STATE == "night" ] || [ $LAST_STATE == "outdoor" ]
  then
    exit 0
  fi

  mcetool --set-display-brightness 100
  mcetool --set-power-saving-mode enabled
  mcetool --set-low-power-mode disabled
  systemctl stop sensorfwd
  systemctl stop sshd

  echo "outdoor" > /tmp/alim-state
;;
*)
  exit 1
esac' | tee /home/root/alim
chmod 550 /home/root/alim

cat << EOL > /etc/systemd/system/alim-day.service
[Unit]
Description=Switch alim state to day
Wants=alim-day.timer

[Service]
ExecStart=/home/root/alim day
Type=oneshot

[Install]
WantedBy=app.slice
EOL

cat << EOL > /etc/systemd/system/alim-day.timer
[Unit]
Description=Switch alim state to day
Requires=alim-day.service

[Timer]
Unit=alim-day.service
OnCalendar=*-*-* 7:00:00

[Install]
WantedBy=timers.target
EOL

cat << EOL > /etc/systemd/system/alim-night.service
[Unit]
Description=Switch alim state to night
Wants=alim-night.timer

[Service]
ExecStart=/home/root/alim night
Type=oneshot

[Install]
WantedBy=app.slice
EOL

cat << EOL > /etc/systemd/system/alim-night.timer
[Unit]
Description=Switch alim state to night
Requires=alim-night.service

[Timer]
Unit=alim-night.service
OnCalendar=*-*-* 23:00:00

[Install]
WantedBy=timers.target
EOL

systemctl enable alim-day.service
systemctl enable alim-day.timer
systemctl enable alim-night.service
systemctl enable alim-night.timer
systemctl start alim-day.service
systemctl start alim-day.timer
systemctl start alim-night.service
systemctl start alim-night.timer
systemctl daemon-reload
```

## Affichage de l'IP après connexion au WiFi

L'objectif de ce script est simplement d'afficher l'IP de la montre avec une notification quand cette dernière se connecte à un WiFi. Par la même occasion, c'est ce script qui va entrainer les passages aux états `indoor` et `outdoor` du script d'optimisation de la batterie.

```sh
echo '#!/bin/sh

if [ ! -f /tmp/network-state ]
then
  echo "init" > /tmp/network-state
fi

STATE=`connmanctl state | grep State | tr -d " " | cut -f2 -d=`
LAST_STATE=`cat /tmp/network-state`

if [ $STATE = "online" ]
then
  ping -q -c 1 flavien.io
  if [ $? -ne 0 ]
  then
    STATE="offline"
  fi
fi

if [ $STATE = $LAST_STATE ]
then
  exit
fi

if [ $STATE = "online" ]
then
  /home/root/alim indoor

  IP=`ip -br -4 a show wlan0 | tr -s " " | cut -f3 -d " " | cut -f1 -d "/"`
  su -l ceres -c "notificationtool -o add --icon=ios-wifi --application=\"System\" --urgency=3 --hint=\"x-nemo-preview-summary WiFi\" --hint=\"x-nemo-preview-body IP: $IP\" \"WiFi\" \"IP: $IP\""
fi

if [ $LAST_STATE = "online" ]
then
  /home/root/alim outdoor
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

## Source

- [AsteroidOS](https://asteroidos.org/)
- [AsteroidOS LG G Watch Urbane](https://asteroidos.org/install/bass/)
- [AsteroidOS Documentation IP Connection](https://asteroidos.org/wiki/ip-connection/)
- [Forum XDA](https://forum.xda-developers.com/t/rom-kernel-ext4all-by-skin1980-m1d64s-update-10-12-2016.3341598/)
