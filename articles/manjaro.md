---
title: Environnement Manjaro
type: WIKI
categories:
  - system
description: Installation d'un environnement de développement sous Manjaro.
author: Flavien PERIER <perier@flavien.io>
date: 2020-09-30 18:00
---

Cet article traite de la mise en place d'un poste de développeur sous Manjaro. L'objectif consiste à avoir un système d'exploitation stable et sécurisé qui réponde à un maximum d'usage.

![Desktop](https://medias.flavien.io/articles/manjaro/desktop.webp)

![Desktop Neofetch](https://medias.flavien.io/articles/manjaro/desktop-neofetch.webp)

## Mise en place du système

Pour ma part, je me base sur la version minimale de la distribution [Manjaro](https://manjaro.org/downloads/official/xfce/) avec l'environnement [XFCE](https://xfce.org/). Cet environnement est plus léger que [Gnome](https://www.gnome.org/) ou [KDE](https://kde.org/), mais présente néanmoins tous les avantages d'un véritable gestionnaire de bureau. Malgré sa légèreté, il n'y a pas de compromis sur les fonctionnalités.

Quant au choix de Manjaro, il s'agit d'une distribution de type rolling, c'est-à-dire qu'elle est constamment mise à jour et qu'on n’aura donc pas de réinstallation complète du système à faire toutes les x années pour le passage à la version majeure suivante. Ce type de distribution est assez adapté à des configurations desktop, mais complètement inadapté à des configurations serveurs, car les paquets et le noyau du système d'exploitation étant constamment mis à jour vers les dernières versions, des instabilités peuvent fréquemment survenir. Inacceptable pour un système de production, mais rarement vraiment bloquant (du moins pas longtemps, jusqu’à la mise à jour suivante) sur un pc personnel.

L'autre avantage de Manjaro, est qu'elle est basée sur la distribution [Arch Linux](https://archlinux.org/). Cette dernière disposant d'une communauté très importante mettant à jour un [Wiki](https://wiki.archlinux.org/) réputé comme étant très complet. Cette source de documentation peut donc servir à se débloquer dans de très nombreuses situations. À mon sens les avantages de Manjaro sur Arch sont la simplicité et la stabilité. En effet, Arch étant une base de système d'exploitation quasiment vierge, c'est à l'utilisateur d'installer les composants dont il aura besoin. Cela nécessite une bonne connaissance de tous ces modules (et beaucoup de temps à lire leur documentation) et cela résulte bien souvent sur des installations instables. Manjaro offre une base déjà installée et configurée et permet donc d'être utilisé par des personnes ayant une moindre connaissance des composants Linux. Il est néanmoins très enrichissant d'installer et de configurer une Arch, mais je réserve cela davantage à de l'expérimentation qu'à la mise en place d'un système d'exploitation destiné à être utilisé tous les jours.

### Mise à jour de Manjaro

Maintenant que Manjaro est installé, nous allons le mettre à jour grâce à l'un des gestionnaires de paquets intègres de la distribution. Pour ma part, j'utilise `pacman`, qui est le gestionnaire de paquet de la distribution Arch. Il est relativement difficile à utiliser et très peu intuitif, c'est pour cette raison que les développeurs de la distribution Manjaro ont rajouté un outil permettant de simplifier son utilisation `pamac`. Pour les utilisateurs venant d'une distribution basée sur [Debian](https://www.debian.org/), ou [RedHat](https://www.redhat.com/), il semblera sans aucun doute plus intuitif.

La raison pour lequel j'utilise néanmoins `pacman` est qu'il offre beaucoup plus de possibilités que sa surcouche. Ce wiki peut néanmoins être suivi en recopiant les commandes `pacman` et par la suite utilisé à titre personnel `pamac`. Ce dernier étant simplement une surcouche, cela ne devrait pas poser de problème de compatibilité.

#### Quelques commandes Pacman

- Mise à jour de tous les paquets système :

```bash
sudo pacman -Syyu
```

- Installation d'un paquet :

```bash
sudo pacman -S <package_name>
```

- Suppression d'un paquet :

```bash
sudo pacman -Rsn <package_name>
```

- Recherche de paquet dans les dépôts :

```bash
sudo pacman -Ss <package_name>
```

- Recherche de paquets installés :

```bash
pacman -Qs <package_name>
```

- Recherche de paquets orphelins :

```bash
sudo pacman -Qdt
```

- Nettoyage du cache pacman :

```bash
sudo pacman -Scc
```

### Gestionnaires de paquets

La plupart des outils disposants d'une interface qui vont être installée dans ce wiki sont installés grâce à [Flatpak](https://www.flatpak.org/). Il s'agit d'un gestionnaire de paquets qui permet d'installer des applications avec un système d'isolation. Cela va permettre de garantir un certain niveau de sécurité pour des applications possédant une GUI.

```bash
sudo pacman -S flatpak
flatpak remote-add --user flathub https://flathub.org/repo/flathub.flatpakrepo
```

À d'autres moments, il sera nécessaire d'utiliser des paquets provenant de la communauté (les [AUR d'Arch Linux](https://aur.archlinux.org/)). Pour ce faire, il faut installer `yay` qui est un gestionnaire de paquets qui prend la succession de `yaourt`.

```bash
cd /tmp
sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

yay -Syyu
```

Pour ceux qui voudraient utiliser leur système pour faire du pentest, il est possible d'installer les repos de [BlackArch](https://www.blackarch.org/). Grâce à ces derniers, il sera possible d'installer la plupart des outils disponibles sur d'autres distributions orientées sécurités telles que [KaliLinux](https://www.kali.org/).

```bash
curl -s https://blackarch.org/strap.sh | sudo bash -
```

### Installation des headers

Parfois, certains logiciels vont avoir besoin des headers Linux afin d'être compilés. Il est donc bon d'avoir la dernière version de ces derniers installés sur ça machine.

```bash
sudo pacman -S linux`uname -r | cut -f1,2 -d. | tr -d "."`-headers
```

### Configuration du réseau

Par défaut, Manjaro utilise [NetworkManager](https://networkmanager.dev/) qui permet de gérer automatiquement le démon est en règle générale plutôt efficace. Mon seul problème avec la configuration par défaut est que le DNS que l'on va inscrire dans le `resolve.conf` sera automatiquement remplacé par celui fourni par le DHCP. Je vais donc forcer manuellement l'utilisation des DNS [OpenDNS](https://www.opendns.com/), [Cloudflare](https://www.cloudflare.com/fr-fr/dns) et [OpenNIC](https://www.opennic.org/).

```bash
cat << EOL | sudo tee /etc/NetworkManager/conf.d/90-dns.conf
[main]
dns=none
EOL

cat << EOL | sudo tee /etc/resolv.conf
nameserver 208.67.222.222
nameserver 208.67.220.220
nameserver 1.1.1.1
nameserver 1.0.0.1
nameserver 151.80.222.79
EOL
```

### Mise en place d'un second disque chiffré

Pour cette installation, nous allons utiliser une architecture assez classique dans le monde de Linux. Avoir un SSD pour accueillir le système (monté sur le `/`) et un second SSD pour accueillir les données (monté sur le `/home`).

Pour maximiser la sécurité du système, nous allons également chiffrer le SSD contenant les données. De nombreuses distributions Linux modernes proposent lors de l'installation de chiffrer les données sur une partition [LVM](https://wiki.archlinux.fr/LVM) (Logical Volume Manager). Je ne suis personnellement pas entièrement convaincu par cette option, car LVM est une couche d'abstraction entre les volumes physiques et les volumes logiques. Cette solution permet par exemple de créer une unique partition reposant sur plusieurs disques physiques, ou de créer une partition avec des clusters non adjacents. À mon sens, cette technologie prend son intérêt sur une infrastructure serveur, car elle va offrir de la flexibilité sur le stockage. Cependant, je suis beaucoup plus sceptique quant à son utilisation dans un ordinateur personnel ou un volume physique va correspondre à un volume logique. C'est pour cette raison que je vais plutôt m'orienter sur l'utilisation de [LUKS](https://gitlab.com/cryptsetup/cryptsetup) (Linux Unified Key Setup) avec son utilitaire `cryptsetup`. L'avantage de cette technologie est qu'elle est directement intégrée dans le noyau Linux et va donc garantir un bon niveau de performance.

- Dans un premier temps, il faut rechercher l'emplacement du second disque grâce à la commande `sudo fdisk -l`. Dans la plupart des cas, il sera localisé dans `/dev/sdb1`.

- Par la suite, pour chiffrer le lecteur, il faudra utiliser la commande `sudo cryptsetup luksFormat --type luks2 /dev/sdb`. Lors de cette opération, toutes les données stockées sur le lecteur seront perdues, il faut donc être extrêmement vigilant lors de cette opération.

- Maintenant que le lecteur est chiffré, nous allons pouvoir le formater et le monter à un endroit ou nous pourrons copier les fichiers que nous voudrons garder quand il sera en `/home`.

```bash
sudo mkdir -p /mnt/data
sudo cryptsetup luksOpen /dev/sdb home
sudo mkfs.btrfs /dev/mapper/home
sudo mount /dev/mapper/home /mnt/data

sudo cp -Rp $HOME /mnt/data
```

- Après avoir copié tous les éléments dont nous aurons besoin, nous pouvons démonter le lecteur.

```bash
sudo umount /mnt/data
sudo cryptsetup luksClose home
```

- Nous allons à présent faire en sorte que le lecteur soit déchiffré automatiquement au démarrage de la machine. Pour ce faire, nous allons l'ajouter à la `crypttab` du système.

```bash
sudo su
echo "home	/dev/sdb	none	luks2" >> /etc/crypttab
```

- Maintenant, nous avons plus qu'a ajouté le volume déchiffré à la `fstable`.

```bash
sudo su
echo "/dev/mapper/home	/home	btrfs	defaults 0 0" >> /etc/fstab
```

### Configuration du Shell

Pour configurer mon shell j'ai développé un petit script que je maintiens sur github : [linux-shell-configuration](https://github.com/flavien-perier/linux-shell-configuration).

Ce dernier introduit plusieurs raccourcis ainsi que des configurations pour les Shells [bash](https://www.gnu.org/software/bash/), [fish](https://fishshell.com/) et [zsh](https://www.zsh.org/).

Pour l'installer, il suffit de taper la ligne de commande :

```bash
curl -s https://sh.flavien.io/shell.sh | sudo bash -
```

### Apparence

Sans plus entrer dans les détails, j'utilise personnellement le thème [Sweet Dark](https://www.xfce-look.org/p/1253385/) avec le pack d'icônes [Sweet Rainbow](https://www.opendesktop.org/p/1284047/) ainsi que les polices de caractères de [JetBrains](https://www.jetbrains.com/lp/mono/) qui ont été pensées pour être optimisées pour le développement.

Pour mettre en place ma configuration, il suffit d'exécuter le script :

```bash
curl -s https://sh.flavien.io/xfce.sh | bash -
```

Le thème `Sweet Dark` étant un thème [GTK2](https://www.gtk.org/), les applications développées avec [QT](https://www.qt.io/) ne sont pas compatibles. Étant donné que les applications QT que j'utilise passent toutes par flatpak, j'installe simplement le thème [Adwaita](https://www.gnomelibre.fr/tag/adwaita/) sur cet environnement.

```bash
flatpak install --user org.kde.KStyle.Adwaita
```

### Nommage des dossiers

Pour des questions d'habitude, il est souvent plus simple d'utiliser la convention de nommage des dossiers en anglais. Pour les renommer dans XFCE, il d'utiliser les commandes suivantes :

```bash
echo "en_EN" > ~/.config/user-dirs.locale

mv ~/Bureau ~/Desktop
mv ~/Téléchargements ~/Downloads
mv ~/Modèles ~/Templates
mv ~/Public ~/Public
mv ~/Documents ~/Documents
mv ~/Musique ~/Music
mv ~/Images ~/Pictures
mv ~/Vidéos ~/Videos

echo 'XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"' | tee ~/.config/user-dirs.dirs
```

### PC portable disposant de plusieurs GPUs

Personnellement, je dispose actuellement d'un Asus ZenBook UX410UQK. Il s'agit d'un très bon PC disposant de deux cartes graphiques: un IGP Intel (Intel HD Graphics 620) et une carte graphique NVIDIA (NVIDIA GeForce 940MX). Le problème de cette configuration sous Linux, c'est que le système va surexploiter le GPU Nvidia qui est bien plus énergivore que le GPU Intel. L'autonomie de la machine va donc s’en trouver grandement affectée et la chauffe constante risque d'affecter la durée de vie de l'appareil...

Dans cette configuration, c'est donc le logiciel `optimus-manager` qui va permettre de passer d'un GPU à l'autre.

```bash
sudo pacman -S intel-media-driver
sudo mhwd -a pci nonfree 0300

yay -S libsndio-61-compat
sudo pacman -S libva-utils

sudo pacman -S optimus-manager
sudo systemctl enable optimus-manager
```

Par la suite, il n'y a plus qu'à redémarrer la machine et utiliser les commandes:

- `sudo optimus-manager --switch nvidia`: Pour utiliser le GPU NVIDIA.
- `sudo optimus-manager --switch integrated`: Pour utiliser IGP Intel.
- `sudo optimus-manager --switch hybrid`: Un fonctionnement similaire à Windows ou le GPU Nvidia sera utilisé pour les taches gourmandes en ressources et l'IGP Intel le reste du temps.

Il est à noter qu'à chaque fois que la carte graphique est changée, il faudra redémarrer l'interface avec `sudo systemctl restart lightdm`.

Quand optimus-manager est en mode hybride, il suffit de passer quelques variables d'environnement à un programme pour qu'il démarre sur la carte graphique. Par exemple pour lancer Steam dans son flatpak :

```bash
flatpak run --env=__NV_PRIME_RENDER_OFFLOAD=1 --env=__GLX_VENDOR_LIBRARY_NAME="nvidia" --env=__VK_LAYER_NV_optimus="NVIDIA_only" --branch=stable --arch=x86_64 --command=/app/bin/steam --file-forwarding com.valvesoftware.Steam @@u %U @@
```

Si le programme que l'on souhaite lancer est une simple application, on peut l'appeler en ligne de commande à travers la commande `prime-run`.

Enfin pour afficher les processus qui utilisent actuellement le GPU, il suffit d'utiliser la commande : `nvidia-smi`.

Il est également possible de configurer optimus-manager dans `/etc` afin de rendre la configuration permanente :

```bash
cat << EOL | sudo tee /etc/optimus-manager/optimus-manager.conf
[optimus]
switching=none
pci_power_control=no
pci_remove=no
pci_reset=no
auto_logout=yes
startup_mode=hybrid
startup_auto_battery_mode=integrated
startup_auto_extpower_mode=nvidia

[intel]
driver=modesetting
accel=
tearfree=
DRI=3
modeset=yes

[amd]
driver=modesetting
tearfree=
DRI=3

[nvidia]
modeset=yes
PAT=yes
DPI=96
ignore_abi=no
allow_external_gpus=no
options=overclocking
dynamic_power_management=no
dynamic_power_management_memory_threshold=
EOL
```

### Installation de l'antivirus

Pour garantir un niveau de sécurité minimal, la présence d'un antivirus est nécessaire (et oui même sous Linux). [Clamav](https://www.clamav.net/) n'est pas forcément le plus performant du marché, mais il est très léger, configurable et très utilisé par la communauté Linux :

```bash
sudo pacman -S clamav
```

Certains considèrent que la base de signature par défaut n'est pas suffisante. C'est pour cela qu'il est possible de l'étendre avec une base non officielle. La base de [Fangfrisch](https://rseichter.github.io/fangfrisch/) est censé être plutôt efficace dans son genre et a l'avantage d'être disponible en AUR.

```bash
yay -S python-fangfrisch

echo '[DEFAULT]
db_url = sqlite:////var/lib/fangfrisch/db.sqlite
local_directory = /var/lib/fangfrisch/signatures
log_level = WARNING
max_size = 100MB

[malwarepatrol]
enabled = no
receipt = you_forgot_to_configure_receipt
interval = 1d
integrity_check = disabled
product = 8
prefix = https://lists.malwarepatrol.net/cgi/getfile?product=${product}&receipt=${receipt}&list=
url_clamav_basic = ${prefix}clamav_basic
filename_clamav_basic = malwarepatrol.db

[sanesecurity]
enabled = yes
interval = 2h
prefix = http://ftp.swin.edu.au/sanesecurity/
!url_foxhole_all_cdb = ${prefix}foxhole_all.cdb
!url_foxhole_all_ndb = ${prefix}foxhole_all.ndb
!url_foxhole_mail = ${prefix}foxhole_mail.cdb
!url_scamnailer = ${prefix}scamnailer.ndb
!url_winnow_phish_complete = ${prefix}winnow_phish_complete.ndb
url_badmacro = ${prefix}badmacro.ndb
url_blurl = ${prefix}blurl.ndb
url_bofhland_cracked_url = ${prefix}bofhland_cracked_URL.ndb
url_bofhland_malware_attach = ${prefix}bofhland_malware_attach.hdb
url_bofhland_malware_url = ${prefix}bofhland_malware_URL.ndb
url_bofhland_phishing_url = ${prefix}bofhland_phishing_URL.ndb
url_foxhole_filename = ${prefix}foxhole_filename.cdb
url_foxhole_generic = ${prefix}foxhole_generic.cdb
url_foxhole_js_cdb = ${prefix}foxhole_js.cdb
url_foxhole_js_ndb = ${prefix}foxhole_js.ndb
url_hackingteam = ${prefix}hackingteam.hsb
url_junk = ${prefix}junk.ndb
url_jurlbl = ${prefix}jurlbl.ndb
url_jurlbla = ${prefix}jurlbla.ndb
url_lott = ${prefix}lott.ndb
url_malwareexpert_fp = ${prefix}malware.expert.fp
url_malwareexpert_hdb = ${prefix}malware.expert.hdb
url_malwareexpert_ldb = ${prefix}malware.expert.ldb
url_malwareexpert_ndb = ${prefix}malware.expert.ndb
url_malwarehash = ${prefix}malwarehash.hsb
url_phish = ${prefix}phish.ndb
url_phishtank = ${prefix}phishtank.ndb
url_porcupine = ${prefix}porcupine.ndb
url_rogue = ${prefix}rogue.hdb
url_scam = ${prefix}scam.ndb
url_shelter = ${prefix}shelter.ldb
url_spamattach = ${prefix}spamattach.hdb
url_spamimg = ${prefix}spamimg.hdb
url_spear = ${prefix}spear.ndb
url_spearl = ${prefix}spearl.ndb
url_winnow_attachments = ${prefix}winnow.attachments.hdb
url_winnow_bad_cw = ${prefix}winnow_bad_cw.hdb
url_winnow_extended_malware = ${prefix}winnow_extended_malware.hdb
url_winnow_extended_malware_links = ${prefix}winnow_extended_malware_links.ndb
url_winnow_malware = ${prefix}winnow_malware.hdb
url_winnow_malware_links = ${prefix}winnow_malware_links.ndb
url_winnow_phish_complete_url = ${prefix}winnow_phish_complete_url.ndb
url_winnow_spam_complete = ${prefix}winnow_spam_complete.ndb

[securiteinfo]
enabled = no
customer_id = you_forgot_to_configure_customer_id
interval = 1h
max_size = 20MB
prefix = https://www.securiteinfo.com/get/signatures/${customer_id}/
!url_0hour = ${prefix}securiteinfo0hour.hdb
!url_old = ${prefix}securiteinfoold.hdb
!url_securiteinfo_mdb = ${prefix}securiteinfo.mdb
!url_spam_marketing = ${prefix}spam_marketing.ndb
url_android = ${prefix}securiteinfoandroid.hdb
url_ascii = ${prefix}securiteinfoascii.hdb
url_html = ${prefix}securiteinfohtml.hdb
url_javascript = ${prefix}javascript.ndb
url_pdf = ${prefix}securiteinfopdf.hdb
url_securiteinfo = ${prefix}securiteinfo.hdb
url_securiteinfo_ign2 = ${prefix}securiteinfo.ign2

[urlhaus]
enabled = yes
interval = 10m
url_urlhaus = https://urlhaus.abuse.ch/downloads/urlhaus.ndb'  | sudo tee /etc/fangfrisch/fangfrisch.conf

sudo chown root:clamav /etc/fangfrisch/fangfrisch.conf
sudo chmod 640 /etc/fangfrisch/fangfrisch.conf

sudo -u clamav fangfrisch --conf /etc/fangfrisch/fangfrisch.conf initdb

sudo systemctl enable fangfrisch.timer
sudo systemctl start fangfrisch.timer
```

Pour mettre la base de données du logiciel, tapez la commande :

```bash
sudo freshclam
```

Le script suivant va permettre à l'antivirus de passer tous les jours en ne consommant que 10% de la puissance d'un coeur du CPU au maximum (ce qui va éviter de maltraiter le matériel) en analysant les fichiers les plus récemment modifiés en premier :

```bash
echo '#!/bin/bash

mkdir -p /var/virus
chmod -R 000 /var/virus

freshclam
if [ $? -ne 0 ]
then
    rm /var/log/clamav/freshclam.log
    freshclam
fi

touch /var/log/clamav/scan.log

find / -type f ! -size 0 -printf "%Ts,%p\n" | sort -nr | cut -f2- -d, > /tmp/antivirus-files
clamscan -i -l /var/log/clamav/scan.log --move=/var/virus --file-list=/tmp/antivirus-files &

cpulimit -l 10 -p `pgrep clamscan`

rm /tmp/antivirus-files
chmod -R 000 /var/virus' | sudo tee /etc/cron.daily/antivirus
sudo chmod 544 /etc/cron.daily/antivirus
```

Ce dernier va permettre à l'antivirus de se mettre à jour toute les 24 heures puis d'effectuer une analyse. Dans l'hypothèse où un virus serait trouvé au sein de vos fichiers personnels, il serait transféré dans le dossier `/var/virus`.

### Mise en place d'un pare-feu

Autre composante importante de la politique de sécurité, le pare-feu. L'objectif va être ici de limiter au strict minimum les interactions avec le réseau. Malheureusement, certains logiciels tels que Discord nécessitent de grandes plages d'ouverture de port, ce qui réduit la sécurité du système. Un compromis, est de n'autoriser les connections à ces plages de port qu'à l'utilisateur principal. Il ne faut donc pas hésiter à supprimer ces règles si les programmes auxquels elles se rapportent ne sont pas installés.

Pour faire cela, nous allons utiliser le proxy `iptables` intégré à la plupart des distributions Linux.

```bash
sudo su

# Reset parameters
iptables -t filter -F
iptables -t filter -X

# Traffic Blocking
iptables -t filter -P INPUT DROP
iptables -t filter -P FORWARD DROP
iptables -t filter -P OUTPUT DROP

# Authorization of already established connections
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# Loopback authorization
iptables -t filter -A INPUT -i lo -j ACCEPT
iptables -t filter -A OUTPUT -o lo -j ACCEPT

# Allow PING to user
iptables -t filter -A OUTPUT -p icmp --icmp-type 8 -j ACCEPT -m owner --uid-owner 1000

# Authorization to open port

## dns
iptables -t filter -A OUTPUT -p udp --dport 53 -d 208.67.222.222 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 53 -d 208.67.220.220 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 53 -d 1.1.1.1 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 53 -d 1.0.0.1 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 53 -d 151.80.222.79 -j ACCEPT

## ntp
iptables -t filter -A OUTPUT -p udp --dport 123 -j ACCEPT -m owner --uid-owner 979

## http/s
iptables -t filter -A OUTPUT -p tcp --dport 80 -j ACCEPT -m owner --uid-owner 0
iptables -t filter -A OUTPUT -p tcp --dport 443 -j ACCEPT -m owner --uid-owner 0
iptables -t filter -A OUTPUT -p tcp --dport 80 -j ACCEPT -m owner --uid-owner 1000
iptables -t filter -A OUTPUT -p tcp --dport 443 -j ACCEPT -m owner --uid-owner 1000

## QUIC
iptables -t filter -A OUTPUT -p udp --dport 80 -j ACCEPT -m owner --uid-owner 1000
iptables -t filter -A OUTPUT -p udp --dport 443 -j ACCEPT -m owner --uid-owner 1000

## ssh
iptables -t filter -A OUTPUT -p tcp --dport 22 -j ACCEPT -m owner --uid-owner 1000

## ftp
iptables -t filter -A OUTPUT -p tcp --dport 21 -j ACCEPT -m owner --uid-owner 1000

## whoIs
iptables -t filter -A OUTPUT -p tcp --dport 43 -j ACCEPT -m owner --uid-owner 1000

## mail (Outlook)
iptables -t filter -A OUTPUT -p tcp --dport 587 -j ACCEPT -m owner --uid-owner 1000
iptables -t filter -A OUTPUT -p tcp --dport 993 -j ACCEPT -m owner --uid-owner 1000

## protonvpn
iptables -t filter -A OUTPUT -p udp --dport 1194 -j ACCEPT -m owner --uid-owner 0
iptables -A OUTPUT -o tun+ -j ACCEPT -m owner --uid-owner 0

## Discord
iptables -t filter -A OUTPUT -p udp --dport 50000:65535 -j ACCEPT -m owner --uid-owner 1000

## Steam (Remote Play)
iptables -t filter -A INPUT -p udp --dport 27031:27036 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 27036 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 27000:27100 -j ACCEPT -m owner --uid-owner 1000

## Sonos
iptables -t filter -A INPUT -p tcp --dport 1400 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 1400 -j ACCEPT -m owner --uid-owner 1000

### Sonos with Spotify
iptables -t filter -A INPUT -p udp --dport 1900 -j ACCEPT
iptables -t filter -A INPUT -p udp --dport 5353 -j ACCEPT
iptables -t filter -A OUTPUT -p udp --dport 5353 -j ACCEPT -m owner --uid-owner 1000

# Synergy
iptables -t filter -A INPUT -p tcp --dport 24800 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 24800 -j ACCEPT -m owner --uid-owner 1000

# Valent
iptables -t filter -A INPUT -p tcp --dport 1714:1764 -j ACCEPT
iptables -t filter -A INPUT -p udp --dport 1714:1764 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 1714:1764 -j ACCEPT -m owner --uid-owner 1000
iptables -t filter -A OUTPUT -p udp --dport 1714:1764 -j ACCEPT -m owner --uid-owner 1000

# Protections

## DDos (https://javapipe.com/blog/iptables-ddos-protection/)

### Drop invalid packets
iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP

### Drop TCP packets that are new and are not SYN
iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP

### Drop SYN packets with suspicious MSS value
iptables -t mangle -A PREROUTING -p tcp -m conntrack --ctstate NEW -m tcpmss ! --mss 536:65535 -j DROP

### Block packets with bogus TCP flags
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN FIN,SYN -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,RST FIN,RST -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,ACK FIN -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,URG URG -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,FIN FIN -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,PSH PSH -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL ALL -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL NONE -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL FIN,PSH,URG -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,FIN,PSH,URG -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -j DROP

### Drop fragments in all chains
iptables -t mangle -A PREROUTING -f -j DROP

### Limit connections per source IP
iptables -A INPUT -p tcp -m connlimit --connlimit-above 111 -j REJECT --reject-with tcp-reset

### Limit RST packets
iptables -A INPUT -p tcp --tcp-flags RST RST -m limit --limit 2/s --limit-burst 2 -j ACCEPT
iptables -A INPUT -p tcp --tcp-flags RST RST -j DROP

### Limit new TCP connections per second per source IP
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m limit --limit 60/s --limit-burst 20 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -j DROP

## Port scan
iptables -N port-scanning
iptables -A port-scanning -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s --limit-burst 2 -j RETURN
iptables -A port-scanning -j DROP

# Save table
iptables-save > /etc/iptables.rules

cat << EOL > /etc/systemd/system/custom-firewall.service
[Unit]
Description=Custom firewall
After=network.target
Before=docker.service
Before=libvirtd.service

[Service]
ExecStart=iptables-restore /etc/iptables.rules
Type=oneshot

[Install]
WantedBy=multi-user.target
EOL

systemctl daemon-reload
systemctl enable custom-firewall
systemctl start custom-firewall
```

Enfin, nous allons ajouter quelques commandes :

- `firewall stop`: Pour désactiver la protection.
- `firewall start`: Pour réactiver la protection.
- `firewall input-stop`: Pour désactiver la protection pour le trafic entrant.
- `firewall output-stop`: Pour désactiver la protection pour le trafic sortant.

```bash
chmod 700 ~/bin

echo '#!/bin/bash

USER_ID=$(id -u)

case $1 in
start)
  sudo iptables -t filter -P INPUT DROP
  sudo iptables -t filter -P FORWARD DROP
  sudo iptables -t filter -P OUTPUT DROP
;;
stop)
  sudo iptables -t filter -P INPUT ACCEPT
  sudo iptables -t filter -P FORWARD ACCEPT
  sudo iptables -t filter -P OUTPUT ACCEPT
;;
input-stop)
  sudo iptables -t filter -P INPUT ACCEPT
;;
output-stop)
  sudo iptables -t filter -P OUTPUT ACCEPT
;;
full-start)
  sudo iptables-restore /etc/iptables.rules
;;
full-stop)
  sudo iptables -t filter -F
  sudo iptables -t filter -X
  sudo iptables -t filter -P INPUT ACCEPT
  sudo iptables -t filter -P FORWARD ACCEPT
  sudo iptables -t filter -P OUTPUT ACCEPT
;;
allow-local)
  sudo iptables -t filter -A OUTPUT -p tcp -d 10.0.0.0/8 -j ACCEPT -m owner --uid-owner $USER_ID
  sudo iptables -t filter -A OUTPUT -p udp -d 10.0.0.0/8 -j ACCEPT -m owner --uid-owner $USER_ID
  sudo iptables -t filter -A OUTPUT -p tcp -d 192.168.0.0/16 -j ACCEPT -m owner --uid-owner $USER_ID
  sudo iptables -t filter -A OUTPUT -p udp -d 192.168.0.0/16 -j ACCEPT -m owner --uid-owner $USER_ID
  sudo iptables -t filter -A OUTPUT -p tcp -d 172.16.0.0/12 -j ACCEPT -m owner --uid-owner $USER_ID
  sudo iptables -t filter -A OUTPUT -p udp -d 172.16.0.0/12 -j ACCEPT -m owner --uid-owner $USER_ID
;;
*)
  echo "Usage: firewall (start|stop|input-stop|output-stop|full-start|full-stop|allow-local)"
  exit -1
;;
esac' | tee ~/bin/firewall

chmod 500 ~/bin/firewall
chmod 500 ~/bin
```

## Outils de développement

Je fais personnellement le choix d'installer mes différents outils directement au niveau de mon système et non pas dans des conteneurs Flatpak. S'agissant de mes outils de travail, je fais cela afin de réduire au maximum les désagréments que pourrait causer la non-accessibilité d'une ressource du système depuis un conteneur. Il est cependant possible d'installer la plupart de ces outils au travers de Flatpak.

### Outils [JetBrains](https://www.jetbrains.com/)

Disposant d'un pack [JetBrains](https://www.jetbrains.com/) complet, il m'est possible d'installer les différents IDEs de l'entreprise à partir de la [JetBrains toolbox](https://www.jetbrains.com/toolbox-app/). C'est au travers de cette interface qu'il est par la suite possible d'installer [Intellij](https://www.jetbrains.com/idea/), [Clion](https://www.jetbrains.com/fr-fr/clion/), [PyCharm](https://www.jetbrains.com/pycharm/), [DataGrip](https://www.jetbrains.com/datagrip/)... et de les maintenir à jour.

```bash
cd /tmp
wget https://download-cdn.jetbrains.com/toolbox/jetbrains-toolbox-2.1.3.18901.tar.gz -O /tmp/toolbox.tar.gz
tar xvf /tmp/toolbox.tar.gz
mv jetbrains* jetbrains
./jetbrains/jetbrains-toolbox
```

Il est également possible d'utiliser GraalVM pour lancer ces différentes IDEs ce qui aura un impacte extrêmement positif sur leurs performances.

### [VisualStudio code](https://code.visualstudio.com/)

Sur les dépôts pacman, il n'y a pas de trace de [Microsoft VisualStudio Code](https://code.visualstudio.com/). En revanche on peut trouver un fork nommé [Code OSS](https://appimage.github.io/Code_OSS/) sur lequel les traqueurs Microsoft ont été enlevés et les composants propriétaires remplacés (le logo par exemple). Ce fork est également compatible avec la plupart des extensions de l'IDE de base.

```bash
sudo pacman -S code

code --install-extension ms-azuretools.vscode-docker
code --install-extension ms-kubernetes-tools.vscode-kubernetes-tools

code --install-extension ms-vscode.hexeditor
code --install-extension ms-python.python
code --install-extension ms-toolsai.jupyter
code --install-extension redhat.vscode-xml
code --install-extension redhat.vscode-yaml
code --install-extension efoerster.texlab

code --install-extension arjun.swagger-viewer
code --install-extension janisdd.vscode-edit-csv
code --install-extension naumovs.color-highlight

code --install-extension k--kato.intellij-idea-keybindings
```

### Environnement Node.js

Quand on travaille sur différents projets JavaScript, on peut être amené à utiliser différentes versions de [Node.js](https://nodejs.org/en/) en fonction des projets. Il est alors possible de passer par [NVM (Node Version Manager)](https://github.com/nvm-sh/nvm), qui va permettre de changer de version de Node.js en seulement quelques commandes.

```bash
sudo pacman -S nvm

echo "source /usr/share/nvm/init-nvm.sh" >> ~/.bashrc
echo "source /usr/share/nvm/init-nvm.sh" >> ~/.zshrc

nvm install --lts
nvm use stable
```

### Environnement Java/Kotlin

De la même façon que NVM gère les versions de Node.js, il est possible d'utiliser [SdkMan](https://sdkman.io/) afin de gérer les versions d'[Open JDK](https://openjdk.java.net/), [GraalVM](https://www.graalvm.org/) ou encore [Maven](https://maven.apache.org/) nécessaire au fonctionnement de nos projets.

```bash
curl -s "https://get.sdkman.io" | bash
echo 'source "$HOME/.sdkman/bin/sdkman-init.sh"' >> ~/.profile
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Fish compatibility

echo '#!/usr/bin/fish

function sdk
  bash -c "source $HOME/.sdkman/bin/sdkman-init.sh && sdk $argv"
end

for VERSION in $HOME/.sdkman/candidates/* ;
  set -gx PATH $PATH $VERSION/current/bin
end' | tee ~/.config/fish/conf.d/sdkman.fish

# Install GraalVM 21
sdk install java 21-graal
sdk install maven
sdk use java 21-graal
```

### Environnement [QT](https://www.qt.io/)

Pour designer de petites interfaces en C++.

```bash
sudo pacman -S qtcreator
```

### Autres langages

Même si je ne code pas directement dans ces technologies, le [GO](https://golang.org/) de Google et le [Rust](https://www.rust-lang.org/) de Mozilla, sont deux langages de programmation de bas niveau qui sont en train de s'imposer. Je les installe afin d'avoir la possibilité de compiler différents projets quand c'est nécessaire.

```bash
sudo pacman -S go rust
```

### Android ADB

Pour ceux qui travaillent de près ou de loin avec [Android](https://www.android.com/), l'outil [ADB](https://developer.android.com/studio/command-line/adb) est indispensable.

```bash
sudo pacman -S android-tools
```

## Installation de [Docker](https://www.docker.com/) et [Podman](https://podman.io/)

Pour installer Docker :

```bash
sudo pacman -S docker docker-buildx docker-compose
 
echo '{
  "features": {
    "buildkit" : true
  }
}' > /etc/docker/daemon.json

sudo systemctl enable docker
sudo systemctl start docker

sudo groupadd docker

echo 'alias docker="sudo docker"' >> $HOME/.alias
```

Pour installer Podamn :

```bash
sudo pacman -S podman

sudo usermod --add-subuids 100000-165535 --add-subgids 100000-165535 $USER

podman system migrate
```

### Installation de [K3s](https://k3s.io/)


K3s est une implémentation légère de [Kubernetes](https://kubernetes.io/). Elle va permettre d'effectuer des tests directement depuis un environnement de développement.

```bash
curl -sfL https://get.k3s.io | sh -

sudo systemctl disable k3s
```

## Utilisation de [KVM](https://www.linux-kvm.org/page/Main_Page)

Avant de commencer, il est indispensable de vérifier au préalable que la virtualisation est activée au niveau du BIOS de la machine hôte. Si ce n'est pas le cas, il sera impossible d'utiliser [KVM](https://www.linux-kvm.org/page/Main_Page) (Kernel-based Virtual Machine).

Il existe dans l'univers Linux de nombreuses solutions de virtualisation. Cependant, le choix de KVM plutôt qu'une autre repose sur le fait qu'il s'agisse d'un hyperviseur de niveau 1, ce qui signifie qu'il est directement intégré au niveau du noyau du système d'exploitation. Cette solution est donc particulièrement efficace dans un environnement Linux.

```bash
sudo pacman -S virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat x11-ssh-askpass

sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service

sudo usermod -a -G libvirt $USER
sudo usermod -a -G kvm $USER
```

Par la suite pour récupérer une image Windows, il faut se rendre sur [le site officiel de Microsoft](https://www.microsoft.com/en-us/software-download/windows11) et télécharger l'image ISO du système d'exploitation.

L'outil `virt-manager` va permettre de manager les différentes machines virtuelles au travers d'une interface. Il fourni également un certain nombre de profils déjà configuré pour faciliter la mise en place de Machines virtuelles avec des configurations complexes (comme c'est le cas d'un Windows 11 avec son TPM).

Pour installer facilement des Machines virtuelles en ligne de commande, il est aussi possible d'utiliser le script [Quickemu](https://github.com/quickemu-project/quickemu) avec la commande suivante :

```bash
yay -S quickemu
```

Par la suite pour créer une machine virtuelle MacOsX il suffira de taper la commande :

```bash
quickget macos big-sur
quickemu --fullscreen --display spice --vm macos-big-sur.conf
```

### Partage de dossier

SMB avec son implémentation [Samba](https://www.samba.org/) est une solution qui peut s'avérer indispensable quand il s'agit de transférer des fichiers d'une machine à l'autre. Cependant, ce protocole très utilisé est très prisé par les hackers. Installer un serveur Samba directement sur ça machine hôte afin de partager des fichiers avec une machine virtuelle ne semble donc pas être une bonne solution. Une alternative plus sécurisée consiste simplement à utiliser une implémentation de Samba pour Docker. De cette façon, en cas de vulnérabilité, le système hôte n'est pas exposé, son arborescence non plus et pas même les utilisateurs. 

Le script suivant permet d'exposer facilement le dossier `~/Public` sur un lecteur Samba :

```bash
chmod 700 ~/bin
echo '#!/bin/bash

case $1 in
start)
    sudo iptables -t filter -A OUTPUT -p udp --dport 137 -j ACCEPT
    sudo iptables -t filter -A OUTPUT -p udp --dport 138 -j ACCEPT
    sudo iptables -t filter -A OUTPUT -p tcp --dport 139 -j ACCEPT
    sudo iptables -t filter -A OUTPUT -p tcp --dport 445 -j ACCEPT
    sudo iptables -t filter -A OUTPUT -p udp --dport 445 -j ACCEPT

    sudo docker run --rm -d --name smb -p 137:137/udp -p 138:138/udp -p 139:139 -p 445:445 -p 445:445/udp -v $HOME/Public:/share/folder elswork/samba -u "1000:1000:$USER:$USER:password" -s "Public:/share/folder:rw:$USER"
    sudo docker logs smb
;;
stop)
    sudo iptables -t filter -A OUTPUT -p udp --dport 137 -j DROP
    sudo iptables -t filter -A OUTPUT -p udp --dport 138 -j DROP
    sudo iptables -t filter -A OUTPUT -p tcp --dport 139 -j DROP
    sudo iptables -t filter -A OUTPUT -p tcp --dport 445 -j DROP
    sudo iptables -t filter -A OUTPUT -p udp --dport 445 -j DROP

    sudo docker rm -f smb
;;
*)
    echo "Usage: smb (start|stop)"
    exit -1
;;
esac' | tee ~/bin/smb

chmod 500 ~/bin/smb
chmod 500 ~/bin
```

Avec ce script, il suffit de taper les commandes `smb start` ou `smb stop` afin d'activer ou désactiver le partage de fichier. De plus, le script va se charge d'ouvrir ou fermer dynamiquement les ports utilisés au niveau du firewall.

### Copie de disque

Les disques utilisés par Qemu sont par default au format `qcow2` ce format à l'avantage d'adapté sa taille sur disque en focntion de ce qu'il va contenir. Cependant, dans le cas ou on supprime des fichiers volumineux dans le disque virtuel, sa taille réservé sur le disque physique ne diminue pas. Il peut alors être bon de réecrir le disque afin de réoptimiser sa structure. Pour se faire il suffit d'utiliser la commande :

```bash
qemu-img convert -O qcow2 old.qcow2 new.qcow2
```

## Logiciels

### Désinstallation des logiciels inutiles

Même en version lite, Manjaro contient de nombreux outils qui ne sont pas réellement utiles. Ce chapitre va se concentrer sur leur désinstallation.

- Manjaro Hello est un menu au démarrage de l'OS qui présente la distribution Manjaro.

```bash
sudo pacman -Rsn manjaro-hello manjaro-documentation-en
```

- Parole est lecteur de média installé par défaut avec la distribution. On installera VLC plus tard.

```bash
sudo pacman -Rsn parole
```

- Qpdfview est un lecteur de fichier PDF. Vu le nombre de vulnérabilités relevées sur ce format de fichier je trouve personnellement plus sûr d'utiliser un logiciel isolé dans un conteneur Flatpak.

```bash
sudo pacman -Rsn qpdfview
```

- Mousepad est un éditeur de texte. Inutile de rajouter de nouveaux éditeurs quand Vim, VScode et tous les outils JetBrains sont déjà installés.

```bash
sudo pacman -Rsn mousepad
```

- Orage est un calendrier.

```bash
sudo pacman -Rsn orage
```

- Gufw est un configurateur de firewall. Cet outil est complètement redondant avec les configurations `iptables` proposées plus haut. De plus, il propose des fonctionnalités beaucoup moins avancées que ce qu'il est possible de faire en ligne de commande.

```bash
sudo pacman -Rsn gufw
```

- Gparted est un logiciel de gestion de partitions. Personnellement je préfère effectuer ce type de manipulations sur les lecteurs directement en ligne de commandes avec `fdisk` et `mkfs`.

```bash
sudo pacman -Rsn gparted
```

- GTK hash est un outil en interface graphique permettant de calculer des hash. Personnellement, les outils en ligne de commande `md5sum` ou `sha256sum` me conviennent parfaitement.

```bash
sudo pacman -Rsn gtkhash-thunar gtkhash
```

- Gcolor2 est un logiciel permettant d'afficher des codes couleur. La plupart des outils de gestion de code et de design embarquent déjà ce type de fonctionnalité.

```bash
sudo pacman -Rsn gcolor2
```

- La surcouche Manjaro pour XFCE n'est pas nécessaire dans le cas où un autre thème est déjà installé.

```bash
sudo pacman -Rsn manjaro-xfce-minimal-settings manjaro-application-utility manjaro-browser-settings kvantum-theme-matchama kvantum-qt5
sudo pacman -Rsn matcha-gtk-theme gnome-icon-theme
sudo pacman -Rsn xcursor-simpleandsoft xcursor-vanilla-dmz-aa
sudo pacman -Rsn noto-fonts noto-fonts-cjk terminus-font
sudo pacman -Rsn `pacman -Qsq texlive`
```

### [KeePassXC](https://keepassxc.org/)

KeePassXC est pour moi un composant central dans mon architecture de sécurité. Non seulement il stocke la totalité des mots de passe de mes comptes, mais il contient aussi mes différentes clés pour mes connexions SSH (GitHub et GitLab comprises). Le lien de démarrage que je propose permet donc de rajouter au logiciel l'autorisation d'accéder aux clés SSH du système.

```bash
flatpak install --user org.keepassxc.KeePassXC

cat << EOL > ~/.local/share/applications/org.keepassxc.KeePassXC.desktop
[Desktop Entry]
Name=KeePassXC
GenericName=Password Manager
GenericName=Gestionnaire de mot de passe
Comment=Community-driven port of the Windows application “KeePass Password Safe”
Exec=/usr/bin/flatpak run --socket=ssh-auth --branch=stable --arch=x86_64 --command=keepassxc --file-forwarding org.keepassxc.KeePassXC @@ %f @@
Icon=keepassxc
StartupWMClass=keepassxc
StartupNotify=true
Terminal=false
Type=Application
Categories=Utility;Security;Qt;
MimeType=application/x-keepass2;
X-Flatpak=org.keepassxc.KeePassXC
EOL
```

### [Firefox](https://www.mozilla.org/fr/firefox/new/)

Par défaut, Manjaro utilise le navigateur [Midori](https://astian.org/midori-browser/). Ce dernier est conçu pour être léger, mais n'est pas vraiment populaire et n'est pas installé dans un Flatpak. Un remplacement par Firefox en version Flatpak semble donc pertinent.

```bash
sudo pacman -Rsn midori
flatpak install --user org.mozilla.firefox
```

### [Brave Browser](https://brave.com/)

Brave Browser est un énième navigateur basé sur [Chromium](https://www.chromium.org/). Cependant, contrairement à la plupart des alternatives, il est orienté vie privée. Malheureusement, certains sites (très rares) ne fonctionnent pas sur Firefox, il est donc toujours pertinent d'avoir un navigateur de la famille Chrome sous la main pour ce type de situation.

```bash
flatpak install --user com.brave.Browser
```

### [TorBrowser](https://www.torproject.org/)

Pour accéder au réseau Tor.

```bash
flatpak install --user com.github.micahflee.torbrowser-launcher
```

### [W3m](http://w3m.sourceforge.net/)

Parfois, il peut être pratique de disposer d'un navigateur en ligne de commande. Pour du scripting, ou tout simplement pour rechercher : "comment réparer ses drivers Nvidia sous Linux".

```bash
sudo pacman -S w3m
```

### [ProtonMail](https://protonmail.com/)

Quand on utilise Linux, les notions de sécurité et de vie privée sont importantes. Donc, utiliser une boite e-mail sécurisée ça l'est tout autant. Le ProtonMailBridge permet de se connecter de manière sécurisée à son compte ProtonMail avec un client lourd de messagerie telle que Thunderbird.

```bash
flatpak install --user ch.protonmail.protonmail-bridge
```

Il est à noter que le ProtonMail Bridge peut démarrer en réduit. Pour ce faire, utilisez cette commande :

```bash
flatpak run --branch=stable --arch=x86_64 --command=protonmail-bridge ch.protonmail.protonmail-bridge  --no-window
```

### [ProtonVPN](https://protonvpn.com/)

Sur Linux, il est possible d'utiliser ProtonVPN directement en ligne de commande ce qui peut permettre de scripter les connections/déconnections.

```bash
yay -S protonvpn-cli
```

Par la suite, il suffit de faire la commande suivante pour s'authentifier à son compte Proton :

```bash
protonvpn-cli login
```

Pour se connecter au serveur le plus rapide (si le besoin est de sécuriser sa connexion sur un hotspot public) :

```bash
protonvpn-cli c -f
```

Pour se connecter à un pays en particulier avec son [code pays](https://fr.wikipedia.org/wiki/Liste_des_codes_pays_UIC) (si le besoin est de contourner une géorestriction) :

```bash
protonvpn-cli c --cc CH
```

Pour se déconnecter du VPN :

```bash
protonvpn-cli d
```

### [Thunderbird](https://www.thunderbird.net/fr/)

Pour les utilisateurs du ProtonMailBridge, il faut penser à désactiver la sécurité pour le SMTP entre Thunderbird et le bridge. Il semblerait que Thunderbird n'autorise plus du tout les certificats autosignés générés par le Bridge pour les messages sortants.

```bash
flatpak install --user org.mozilla.Thunderbird
```

### [Discord](https://discord.com/)

Discord est surement l'application de communication la plus populaire du moment. Elle est cependant développée en JavaScript dans un conteneur [Electron](https://www.electronjs.org/) et ne se soucie pas vraiment des utilisateurs Linux. Un patch du nom de [Vencord](https://vencord.dev/) a donc été développé par la communauté afin d'améliorer la performance de l'application et enlever plusieurs options de tracking.

```bash
flatpak install --user dev.vencord.Vesktop
```

Pour lancer Discord en mode réduit il est possible d'utiliser la ligne de commande suivante :

```bash
flatpak run --branch=stable --arch=x86_64 --command=startvesktop dev.vencord.Vesktop --start-minimized
```

### [Matrix](https://matrix.org/)

Matrix est un protocole de communication ouvert qui supporte aussi bien les conversations écrites qu'en audio/vidéo. Il existe plusieurs implémentations, mais la plus populaire est nommée Matrix (Il s'agit de l'implémentation officielle).

```bash
flatpak install --user im.riot.Riot
```

### [Eye of GNOME](https://github.com/GNOME/eog)

La visionneuse d'image de Gnome. Elle est plutôt simple à utiliser et affiche les métdatas des images (heure, position GPS si disponible...).

```bash
flatpak install --user org.gnome.eog
```

### [Gimp](https://www.gimp.org/)

Le célèbre éditeur d'images bitmap très complet et très adapté pour la retouche de photo. Mon seul problème avec ce logiciel est son ergonomie.

```bash
flatpak install --user org.gimp.GIMP
```

### [Krita](https://krita.org/)

Tout comme GIMP, Krita est un éditeur d'image au format bitmap. Ce logiciel est cependant plus adapté pour le dessin.

```bash
flatpak install --user org.kde.krita
```

### [Inkscape](https://inkscape.org/)

Un éditeur d'images vectorielles.

```bash
flatpak install --user org.inkscape.Inkscape
```

### [Blender](https://www.blender.org/)

Rien de tel que Blender pour créer de magnifiques modèles 3D.

```bash
flatpak install --user org.blender.Blender
```

### [VLC](https://www.videolan.org/)

Le célèbre lecteur de fichiers multimédias français.

```bash
flatpak install --user org.videolan.VLC
```

### [Audacity](https://www.audacityteam.org/)

Un logiciel simple et efficace pour enregistrer et éditer du son.

```bash
flatpak install --user org.audacityteam.Audacity
```

### [Soundux](https://github.com/Soundux/Soundux)

Soundux est un simple logiciel pour jouer des sons lors de conversations audio.

```bash
flatpak install --user io.github.Soundux
```

### [Spotify](https://www.spotify.com/fr/)

Parce qu'on ne peut pas loger toute la musique du monde sur son petit SSD...

```bash
flatpak install --user com.spotify.Client
```

### [Noson](https://github.com/janbar/noson-app)

Pour ceux, qui comme moi, possèdent du matériel [Sonos](https://www.sonos.com/fr-fr/home), l'application Noson permet de piloter les équipements de la marque. Ce logiciel va permettre en prime de streamer les musiques qui sont présentes sur le disque dur, chose qu'il n'est pas possible de faire aussi simplement sur Windows avec le client Sonos officielles.

```bash
flatpak install --user io.github.janbar.noson
```

Afin d'accélérer l'accès à l'enceinte au démarrage de l'application, j'ai configuré ma box avec un bail DHCP permanent afin que mon périphérique Sonos ait toujours la même adresse IP d'attribuée (dans mon cas `192.168.1.5`). Ensuite il ne reste plus qu'à modifier le raccourci de lancement de l'application pour lui appliquer le paramètre `--deviceurl=http://192.168.1.5:1400`. Avec cette configuration, l'application n'a pas besoin de découverte réseau étant donné qu'elle sait d'avance où se situe l'enceinte.

```bash
cat << EOL > ~/.local/share/applications/io.github.janbar.noson.desktop
[Desktop Entry]
Name=noson
Exec=/usr/bin/flatpak run --branch=stable --arch=x86_64 --command=noson-app io.github.janbar.noson --deviceurl=http://192.168.1.5:1400/
Icon=io.github.janbar.noson
Terminal=false
Type=Application
Categories=AudioVideo;Audio;
X-Flatpak=io.github.janbar.noson
Comment=
Path=
StartupNotify=false
```

### [NoiseTorch](https://github.com/noisetorch/NoiseTorch)

Un logiciel pour filtrer les bruits parasites d'un micro :

```bash
yay -S noisetorch
```

### [NextCloud](https://nextcloud.com/)

Pour ceux qui veulent déployer leur propre solution de stockage de fichier qui ne serait pas hébergé chez Google, Microsoft ou Amazon.

```bash
flatpak install --user com.nextcloud.desktopclient.nextcloud
```

### [LibreOffice](https://fr.libreoffice.org/)

Pack Office gratuit pour Linux... Pour les gros utilisateurs des outils Microsoft Word, Excel et PowerPoint, je ne peux cependant pas en toute bonne foi prétendre que LibreOffice égale Microsoft 365. Si ces logiciels sont vos outils de travail principaux, ne passez pas sous Linux et continuez d'utiliser Microsoft 365 sur votre Windows ou sur votre Mac.

```bash
flatpak install --user org.libreoffice.LibreOffice
```

### [Evince](https://wiki.gnome.org/Apps/Evince)

Un autre logiciel Gnome. Celui-ci permet simplement d'afficher des fichiers PDF. Il embarque notamment de la reconnaissance de caractère au sein des images. Ce qui permet de faire des copier-coller de documents photocopiés.

```bash
flatpak install --user org.gnome.Evince
```

### [Scribus](https://www.scribus.net/)

Scribus est un éditeur de PDF.

```bash
flatpak install --user net.scribus.Scribus
```

### [Calibre](https://calibre-ebook.com/)

Calibre est un puissant outil de lecture/écriture d'e-book. Il accepte différents formats de fichiers, dont notamment les epub.

```bash
flatpak install --user com.calibre_ebook.calibre
```

Personnellement, je dispose d'une liseuse [Bookeen Diva HD](https://bookeen.com/products/diva-hd) (produit d'excellente qualité et Française au demeurant). Afin de centraliser mon catalogue de livre acquis sur différentes plateformes, je les stocke au format epub sans DRM. Pour cela, j'utilise Calibrea avec l'extension [DeDRM_tools](https://github.com/apprenticeharper/DeDRM_tools).

### [FontForge](https://fontforge.org/)

Un éditeur de police de caractères.

```bash
flatpak install --user org.fontforge.FontForge
```

### [Antidote](https://www.antidote.info/fr)

Sans aucun doute le meilleur correcteur orthographique (payant) du marché.

Téléchargez l'application Antidote [sur le site officiel](https://services.druide.com/client/).

```bash
cd /tmp
tar xvf ~/Downloads/Antidote*
mv Antidote* Antidote
./Antidote/Installation.bash
```

### [Pandoc](https://pandoc.org/)

Pandoc est un outil très puissant permettant de convertir de nombreux formats de fichier texte dans d'autres formats de fichier texte. Il est par exemple possible de transformer un fichier Markdown en document PDF en lui appliquant une feuille de style au format [LaTeX](https://www.latex-project.org/). Il est donc possible de rédiger des documents de qualité professionnelle simplement en Markdown et de les convertir plus tard dans le format que nous souhaitons exporter, pdf, doc, html, LaTex, epub...

Je vois plusieurs avantages à cette solution par rapport au traditionnel Microsoft Word, mais la plus importante à mon sens est le découpage du flux de travail. En effet, je considère que lors de la rédaction d'un document nous avons 3 grandes phases de travail, l'écriture du contenu, une phase de relecture durant lesquelles nous pouvons corriger fautes d'orthographe et mauvaises tournures de phrases et enfin la mise en page. Le problème avec Microsoft Word est que nous devons gérer ces 3 phases simultanément, ce qui a pour inconvénient de nous rendre moins efficaces sur chacune d'entre elles et résulte sur des documents de moindre qualité. La plupart des universitaires ce sont donc tournés vers le format LaTex, mais le problème reste similaire, avec ses nombreuses annotations ce format reste lourd et nous devons donc gérer la rédaction et une partie de la mise en page simultanément. Le meilleur compromis à mon sens est le Markdown qui est très léger en écriture et surtout très intuitif.

Ainsi avec les différents logiciels que j'utilise, je peux garantir un flux de travail efficace sur ces trois axes. Je commence par rédiger en Markdown le contenu de mon document dans un simple IDE (généralement Visual Studio Code ou VIM), puis je corrige les fautes grâce à Antidote et enfin j'utilise Pandoc pour la mise en page. J'ai créé [un projet GitHub](https://github.com/flavien-perier/pandoc-template) qui me sert de base pour mes nouveaux projets. Il utilise la feuille de style LaTex [Eisvogel](https://github.com/Wandmalfarbe/pandoc-latex-template) afin de mettre en forme mes documents Markdown dans le format PDF.

```bash
pacman -Syyu pandoc pandoc-citeproc
pacman -Syyu `pacman -Ss texlive | cut -f 1 -d " " | grep -v "^$"`
```

### [NewsFlash](https://gitlab.com/news-flash/news_flash_gtk)

NewsFlash est un simple lecteur de flux RSS.

```bash
flatpak install --user com.gitlab.newsflash
```

### [Valent](https://valent.andyholmes.ca/)

Interface ce basant sur [KDE Connect](https://kdeconnect.kde.org/) permettant d'utiliser son téléphone depuis son ordinateur à travers le réseau local. Il est ainsi possible d'envoyer et de recevoir des SMS sur son PC, ou encore de partager son presse-papier entre les deux appareils.

```bash
flatpak install --user https://valent.andyholmes.ca/valent.flatpakref
```

Pour pouvoir utiliser l'application, il faudra également installer sur son appareil Android l'application [KDE Connect](https://www.flavien.io/wiki/android.md#kde-connect).

### [OBS](https://obsproject.com/)

Étant donné que les fonctionnalités de Stream de Discord ne fonctionnent qu'à moitié sous Linux (on a l'image, mais pas le son), il faut passer par de tierces solutions afin de pouvoir partager son écran.

OBS présente l'avantage de permettre de streamer sur de nombreuses plateformes tel que [Twitch](https://www.twitch.tv/), [YouTube](https://www.youtube.com/) ou [PeerTube](https://sepiasearch.org/).

```bash
flatpak install --user com.obsproject.Studio
```

### [Steam](https://store.steampowered.com/)

La célèbre plateforme de jeux. Actuellement, Valve (la société à l'origine de Steam) travail afin de rendre les jeux vidéos plus accessibles sur les systèmes d'exploitation Linux. Ils ont actuellement leur propre distribution [SteamOS](https://store.steampowered.com/steamos) basée sur Arch, qui utilise leur technologie [ProtonDB](https://www.protondb.com/) (basée sur [Wine](https://www.winehq.org/)) afin de lancer des jeux Windows sur Linux.

```bash
flatpak install --user com.valvesoftware.Steam
```

Par la suite pour activer ProtonDb il suffit de ce rendre dans l'application puis: `Steam` > `Paramètres` > `Steam Play` > `Activer Steam Play pour tous les autres titres` > `Proton 6.3-8`.

### [Xpadneo](https://atar-axis.github.io/xpadneo/)

Par défaut, les [manettes de Xbox](https://www.microsoft.com/fr-fr/store/b/xboxcontrollers) ne fonctionnent pas en Bluetooth sur Manjaro (ainsi que sur de nombreuses autres distributions). Il est toujours possible de les faire fonctionner en filaire, mais ce n'est pas toujours d'un confort absolu. Le logiciel Xpadneo va simplement être là pour patcher ce problème et permettre d'utiliser une manette sans avoir besoin de la brancher.

```bash
sudo pacman -S dkms
yay -S xpadneo-dkms
```

### [Moonlight](https://moonlight-stream.org/)

Moonlight est un logiciel de streaming pour pouvoir récupérer le flux vidéo de la technologie de streaming Nvidia.

```bash
flatpak install com.moonlight_stream.Moonlight
```

Au niveau du serveur, il est possible d'utiliser la technologie de streaming du [Nvidia Shield](https://www.nvidia.com/fr-fr/shield/). Mais il semblerait que l'option la plus viable soit [Sunshine](https://github.com/LizardByte/Sunshine) qui est une réimplémentation open source du protocole.

### [AntiMicroX](https://github.com/AntiMicroX/antimicroX)

Logiciel permettant de faire le mapping des touches d'une manette de Xbox sur les touches d'un clavier.

```bash
flatpak install --user io.github.antimicrox.antimicrox
```

### [Synergy](https://symless.com/synergy)

Synergy est un logiciel permettant d'utiliser différents ordinateurs comme s'il s'agissait d'écrans. L'ordinateur principal (celui disposant d'un clavier et d'une souris) sera le serveur et tous les autres les clients.

Il s'agit d'une application payante, mais open source. Un fork populaire est [Barrier](https://github.com/debauchee/barrier). Cependant ce dernier ne supporte pas Wayland et n'est pas maintenu. La meilleure solution est donc à mon sens de payer.

```bash
wget "https://api-functions.stage.a.symless.com/download-log?synergyVersion=3.0.79.1-rc3&operatingSystem=Linux&architecture=flatpak&downloadUrl=https%3A%2F%2Frc.symless.com%2Fsynergy3%2Fv3.0.79.1-rc3%2Fsynergy-linux_x64-libssl3-v3.0.79.1-rc3.flatpak&userId=1452232" -O /tmp/synerg.flatpak

flatpak install /tmp/synerg.flatpak
```

### [Solaar](https://pwr-solaar.github.io/Solaar/)

Un logiciel permettant de gérer les claviers et souris [Logitech](https://www.logitech.com/) (personnellement, j'utilise un clavier [MX Mechanical Mini](https://www.logitech.com/fr-fr/products/keyboards/mx-mechanical.html) avec une souris [MX Master 3](https://www.logitech.com/fr-fr/products/mice/mx-master-3s.910-006559.html)).

```bash
sudo pacman -S solaar 
```

### [Tmux](https://github.com/tmux/tmux)

J'ai utilisé pendant un grand moment [Terminator](https://github.com/janisozaur/terminator) comme émulateur de terminal afin de pouvoir multiplexer mes consoles. Cependant, XFCE dispose déjà d'un terminal codé en C et donc extrêmement performant et bien intégré avec le reste de l'environnement. Le seul défaut de ce dernier est qu'il ne gère pas le split d'écran. La solution que j'ai choisie, est donc de l'associer avec des logiciels de multiplexage plus bas niveau tel que [GNU Screen](https://www.gnu.org/software/screen/) ou [Tmux](https://github.com/tmux/tmux).

```bash
sudo pacman -S tmux xclip

cat << EOL > ~/.tmux.conf
set-option -g default-shell /usr/bin/fish
set -g default-command /usr/bin/fish

bind '"' split-window -c "#{pane_current_path}"
bind % split-window -h -c "#{pane_current_path}"

set -g status off
set -g history-limit 999999999
set -g mouse on

setw -g mode-keys vi

set-option -s set-clipboard off

bind P paste-buffer
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X rectangle-toggle
unbind -T copy-mode-vi Enter
bind-key -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel 'xclip -se c -i'
bind-key -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel 'xclip -se c -i'
EOL

tmux source-file ~/.tmux.conf
```

### [TLPUI](https://github.com/d4nj1/TLPUI)

Sur Linux [TLP](https://github.com/linrunner/TLP) est un démon permettant de gérer au mieux l'autonomie de sa machine. Sur Manjaro cet outil est préinstallé dans le système d'exploitation. TLPUI est tout simplement une interface pour cet outil.

```bash
sudo pacman -S tlpui
```

### [fwupd](https://fwupd.org/)

Fwupd est un démon permettant de maintenir à jour les firmwares des composants d'un pc.


```bash
sudo pacman -S fwupd
sudo systemctl start fwupd
```

Pour rechercher les mises à jour, on utilise la commande :

```bash
fwupdmgr get-updates
```

Et pour effectuer la mise à jour :

```bash
fwupdmgr update
```

### [Kleopatra](https://www.openpgp.org/software/kleopatra/)

Une petite interface pour la gestion de nos clés privées OpenPGP.

```bash
sudo pacman -S kleopatra
```

### [Ghidra](https://ghidra-sre.org/)

Le puissant outil de reverse engineering de la NSA pour faire de l'analyse approfondie sur du code compilé.

```bash
flatpak install --user org.ghidra_sre.Ghidra
```

### [WireShark](https://www.wireshark.org/)

Un excellent analyseur de réseau pour ceux qui, comme moi, ne maitrisent pas toute la puissance de `tcpdump`.

```bash
flatpak install --user org.wireshark.Wireshark
```

### [MacChanger](https://www.kali.org/tools/macchanger/)

Un simple script permettant de changer d'adresse mac. Cela peut-être relativement utile dans les lieux publics ou une limite de temps d'utilisation est imposée.

```bash
sudo pacman -S macchanger
```

Par exemple pour obtenir une nouvelle adresse mac sur la carte `eth0` on peut taper :

```bash
macchanger -a eth0
```

Pour revenir à l'adresse de base :

```bash
macchanger -p eth0
```

Enfin dans certains cas (en général des scénarios d'attaque) il peut-être utile de prendre une adresse mac spécifique :

```bash
macchanger --mac=XX:XX:XX:XX:XX:XX eth0
```

### [Nmap](https://nmap.org/)

Un outil permettant de scanner un réseau et de déterminer pour chaque machine trouvée quels sont les services exposés.

```bash
sudo pacman -S nmap
```

### [Magika](https://github.com/google/magika)

Magika est un outil développé par Google permettant d'identifier le type d'un fichier. Cet outil est bien plus puissant dans sa capacité d'identification que la commande `file` intégrée à la plupart des systèmes.

```bash
yay -S python-magika
```

### [Insomnia](https://insomnia.rest/)

Pour permettre aux développeurs back-end de tester les APIs REST qu'ils développent. Le logiciel est assez similaire à [Postman](https://www.postman.com/), mais offre à mon sens une meilleure gestion des formats [Swagger](https://swagger.io/) et [OpenAPI](https://www.openapis.org/).

```bash
flatpak install --user rest.insomnia.Insomnia
```

### [Deluge](https://www.deluge-torrent.org/)

Un client torrent complet.

```bash
flatpak install --user org.deluge_torrent.deluge
```

### [P7zip](http://p7zip.sourceforge.net/)

Le `7z` est l'un, si ce n'est pas le format qui offre le meilleur taux de compression du moment, le rendant particulièrement efficace pour tout ce qui touche à l'archivage. Seul problème, 7zip n'existe que sur Windows. Heureusement, avec p7zip il est possible d'utiliser ce format en ligne de commande depuis son Linux.

```bash
sudo pacman -S p7zip
```

Par la suite, pour créer une archive avec un taux de compression maximal utilisez la commande :

```bash
7z a -mx=9 archive.7z file1 file2
```

Et pour la décompresser, utilisez la commande :

```bash
7z x archive.7z
```

### [JQ](https://stedolan.github.io/jq/)

JQ est un outil permettant de traiter des fichiers json en ligne de commande.

```bash
sudo pacman -S jq
```

Voici un exemple simple visant à extraire les valeurs "a", "b" et "c" au sein du fichier JSON.

```bash
echo '{"values": [{"name": "a"}, {"name": "b"}, {"name": "c"}]}' | jq -cM ".values[].name"
```

### [bat](https://github.com/sharkdp/bat)

Bat est un clone de la commande `cat` réécrit en Rust. Cette commande prend en charge la pagination, la coloration syntaxique, git et dispose de quelques paramètres qui peuvent s'avérer bien utiles.

```bash
sudo pacman -S bat
```

Par exemple pour afficher seulement les lignes 2 à 5 d'un fichier :

```bash
bat -r 2:5 file.md
```

### [CpuLimit](https://github.com/opsengine/cpulimit)

Un petit outil permettant de limiter l'utilisation du processeur à un processus.

```bash
sudo pacman -S cpulimit
```

Les paramètres de `cpulimit` sont :

- `-e`: Le nom d'un processus.
- `-p`: Le PID d'un processus.
- `-l`: La limite d'utilisation du processeur en pourcentage à appliquer au processus ciblé.

### [Smartmontools](https://www.smartmontools.org/)

Smartmontools est un outil permettant de récupérer [les données SMART](https://fr.wikipedia.org/wiki/Self-Monitoring,_Analysis_and_Reporting_Technology) concernant l'état de santé d'un disque dur ou SSD : Vitesse d'écriture, lecture si des secteurs ont été détectés comme défaillant... Faire des analyses de temps à autre peut s'avérer pertinent afin d'éventuellement faire passer un disque dur ou SSD interne en simple support de sauvegarde externe externe.

Pour faire une analyse, il suffit de taper la commande (en remplaçant `sda` par le disque à analyser) :

```bash
sudo smartctl -H /dev/sda
```

Si tout va bien, le résultat du test devrait être `PASSED`.

Pour avoir le rapport complet, il suffit de rajouter l'argument `-a` :

```bash
sudo smartctl -H /dev/sda -a
```

### [WhoIs](https://www.whois.com/)

Permets de donner un certain nombre d'informations concernant une adresse IP (localisation, FAI...).

```bash
sudo pacman -S whois
```

### [Fastfetch](https://github.com/fastfetch-cli/fastfetch)

L'outil [Neofetch](https://github.com/dylanaraps/neofetch) était une vraie religion chez les utilisateurs de Linux. Cependant son créateur à malheureusement décidé d'arrêter ça maintenance.

Le projet Fastfetch a donc pris le relais.

Il s'agit aucun doute de l'outil le plus inutile de la sélection... Il permet d'afficher un certain nombre de caractéristiques de sa machine sous un format coloré. Un vrai Linuxien se doit de toujours laisser un Neofetch/Fastfetch tourné dans un coin de terminale sur chaque capture d'écran qu'il envoie. Cela permet de prouver au reste de la communauté la supériorité de sa distribution.

```bash
sudo pacman -S fastfetch
```

### [Conky](https://wiki.archlinux.org/title/Conky)

Conky est un outil permettant de rajouter des informations personnalisées sur le bureau. On peut par exemple l'utiliser pour afficher son IP, la fréquence des différents coeurs du processeur, la consommation de RAM, la température de tel ou tel composant...

```bash
sudo pacman -S conky
mkdir -p ~/.config/conky

cat << EOL > ~/.config/conky/conky.conf
conky.config = {
    alignment = 'bottom_right',
    background = false,
    border_width = 1,
    cpu_avg_samples = 2,
    default_color = 'white',
    default_outline_color = 'white',
    default_shade_color = 'white',
    double_buffer = true,
    draw_borders = false,
    draw_graph_borders = true,
    draw_outline = false,
    draw_shades = false,
    extra_newline = false,
    font = 'JetBrains Mono NL Medium:size=15',
    gap_x = 60,
    gap_y = 60,
    minimum_height = 5,
    minimum_width = 500,
    net_avg_samples = 2,
    no_buffers = true,
    out_to_console = false,
    out_to_ncurses = false,
    out_to_stderr = false,
    out_to_x = true,
    own_window = true,
    own_window_class = 'Conky',
    own_window_transparent = false,
    own_window_type = 'desktop',
    own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',
    own_window_argb_visual = true,
    own_window_argb_value = 0,
    show_graph_range = false,
    show_graph_scale = false,
    stippled_borders = 0,
    update_interval = 3.0,
    uppercase = false,
    use_spacer = 'none',
    use_xft = true,
    xinerama_head = 1,
}

conky.text = [[
${color grey}CPU:
${color}${exec expr `sensors | grep 'Core 0' | cut -d+ -f2 | cut -d\( -f1 | cut -d. -f1`}°C${goto 85}1: ${cpubar cpu0 6,100} ${cpu cpu0}%${goto 300}3: ${cpubar cpu2 6,100} ${cpu cpu2}%
${color}${exec expr `sensors | grep 'Core 1' | cut -d+ -f2 | cut -d\( -f1 | cut -d. -f1`}°C${goto 85}2: ${cpubar cpu1 6,100} ${cpu cpu1}%${goto 300}4: ${cpubar cpu3 6,100} ${cpu cpu3}%

${color grey}Memory:
${color}RAM${goto 120}${membar 6,100} ${memperc}%${goto 300}(${mem})
${color}SWAP${goto 120}${swapbar 6,100} ${swapperc}%${goto 300}(${swap})

${color grey}Storage:
${color}/${color}${goto 120}${fs_bar 6,100 /} ${fs_used_perc /}%${goto 300}(${fs_used /})
${color}/home${color}${goto 120}${fs_bar 6,100 /home} ${fs_used_perc /home}%${goto 300}(${fs_used /home})

${color grey}System:
${color}Kernel${goto 120}${exec uname -r}
${color}Uptime${goto 120}${uptime}
${color}Process${goto 120}${processes}
${color}Podman${goto 120}${exec podman ps -q | wc -l}
${color}Docker${goto 120}${exec pgrep -f -U 0 -c "docker run"}
${color}KVM${goto 120}${exec pgrep -f -c qemu-system-x86_64}

${color grey}Network:
${if_existing /proc/net/route wlp2s0}${color}Local${goto 120}${addr wlp2s0}
${color}Public${goto 120}${texeci 7200 curl -s ipinfo.io/ip}
${color}Download${goto 120}${downspeedf wlp2s0}k/s
${color}Upload${goto 120}${upspeedf wlp2s0}k/s
${else}${if_existing /proc/net/route enp0s20f0u4u4}${color}Local${goto 120}${addr enp0s20f0u4u4}
${color}Public${goto 120}${texeci 7200 curl -s ipinfo.io/ip}
${color}Download${goto 120}${downspeedf enp0s20f0u4u4}}k/s
${color}Upload${goto 120}${upspeedf enp0s20f0u4u4}k/s
${else}${color}Local${goto 120}-
${color}Public${goto 120}-
${color}Download${goto 120}-k/s
${color}Upload${goto 120}-k/s${endif}${endif}
$hr

${color grey}Name${goto 215}CPU${goto 305}MEM
${color grey}${top name 1}${color}${goto 190}${top cpu 1}%${goto 280}${top mem 1}%
${color grey}${top name 2}${color}${goto 190}${top cpu 2}%${goto 280}${top mem 2}%
${color grey}${top name 3}${color}${goto 190}${top cpu 3}%${goto 280}${top mem 3}%
${color grey}${top name 4}${color}${goto 190}${top cpu 4}%${goto 280}${top mem 4}%
${color grey}${top name 5}${color}${goto 190}${top cpu 5}%${goto 280}${top mem 5}%
]]
EOL

cat << EOL > ~/.config/autostart/conky.desktop
[Desktop Entry]
Type=Application
Exec=sh -c "sleep 10; conky;"
Name=Conky
Comment=Autostart conky at login
EOL
```

## Configuration de l'environnent applicatif

### Faire en sorte que les applications Flatpak puissent utiliser le thème XFCE

L'isolation proposée par Flatpak présente de nombreux avantages en termes de sécurité, mais peu également causés quelques désagréments en termes d'usage. L'un des problèmes, qui peut au premier abord paraitre anodin, est le fait d'avoir le même thème entre chacune de ses applications. En effet Flatpak n'ayant pas accès au dossier `~/.themes`, les applications auront tout le temps leur thème par défaut. Si vous êtes un utilisateur d'un thème standard tel qu'[Adwaita dark pour GTK](https://github.com/axxapy/Adwaita-dark-gtk2) et que ce thème est disponible dans le Flathub, vous pourrez simplement l'installer et Flatpak se chargera de l'activer (par exemple: `flatpak install --user org.gtk.Gtk3theme.Adwaita-dark`). Cependant, s’il s'agit d'un thème non présent dans le store ou custom, il n'est pas possible de l'utiliser dans les applications Flatpak. Une solution, est donc de copier les fichiers du thème utilisé dans les runtimes `freedesktop`, `kde` et `gnome` de Flatpak. Le problème est qu'il faut refaire les copies à chaque nouvelle version de ces runtimes. L'avantage est qu'avec cette solution il n'est pas nécessaire de modifier les configurations par défaut de Flatpak ni réduire le niveau d'isolation.

```bash
for PLATFORM in `ls $HOME/.local/share/flatpak/runtime | grep "Platform$"`
do
    for VERSION in `ls $HOME/.local/share/flatpak/runtime/$PLATFORM/x86_64`
    do
        cp -R $HOME/.themes/* $HOME/.local/share/flatpak/runtime/$PLATFORM/x86_64/$VERSION/active/files/share/themes
        cp -R $HOME/.icons/* $HOME/.local/share/flatpak/runtime/$PLATFORM/x86_64/$VERSION/active/files/share/icons
        cp -R $HOME/.fonts/* $HOME/.local/share/flatpak/runtime/$PLATFORM/x86_64/$VERSION/active/files/share/fonts
    done
done
```

### Réparer les liens cliquables Flatpak

Il arrive que les liens cliquables ne mènent plus au navigateur pour les applications Flatpak. Ceci est lié à un problème de configuration de [xdg-desktop-portal](https://github.com/flatpak/xdg-desktop-portal). Ce composant permet de faire la passerelle entre la machine hôte de Flatpak pour un certain nombre de tâches.

La première chose à faire est de vérifier que `xdg-desktop-portal-gtk` soit bien installé sur le système (si l'environnement de bureau est XFCE) :

```bash
sudo pacman -S xdg-desktop-portal-gtk
```

Par la suite, il faut changer la configuration pour l'utilisateur :

```bash
systemctl --user import-environment XDG_CURRENT_DESKTOP
mkdir -p ~/.config/xdg-desktop-portal/
cat << EOL > ~/.config/xdg-desktop-portal/xfce-portals.conf
[preferred]
default=gtk;
EOL
echo 'XDG_CURRENT_DESKTOP=XFCE' | tee -a /etc/environment
systemctl --user restart xdg-desktop-portal.service xdg-desktop-portal-gtk.service
```
Une fois ces actions effectuées, les liens devraient être de nouveau cliquables dans les applications XFCE.

### Activation des programmes par défaut

Sur Linux, il ne suffit pas forcément qu'un programme soit installé pour qu'il soit utilisé par défaut pour ouvrir certains types de fichiers depuis l'explorateur de fichier (`thunar`) ou avec la commande `exo-open`. Pour utiliser la plupart des programmes précédemment installés comme programme par défaut il suffit de mettre à jour le fichier `~/.config/mimeapps.list` :

```bash
cat << EOL > ~/.config/mimeapps.list
[Default Applications]

application/octet-stream=code-oss.desktop;
application/x-gettext-translation=code-oss.desktop;
application/xml=code-oss.desktop;
application/x-wine-extension-ini=code-oss.desktop;
application/pdf=org.gnome.Evince.desktop;

application/vnd.oasis.opendocument.text=org.libreoffice.LibreOffice.desktop;
application/vnd.oasis.opendocument.spreadsheet=org.libreoffice.LibreOffice.desktop;
application/vnd.oasis.opendocument.presentation=org.libreoffice.LibreOffice.desktop;
application/vnd.oasis.opendocument.graphics=org.libreoffice.LibreOffice.desktop;
application/vnd.ms-excel=org.libreoffice.LibreOffice.desktop;
application/vnd.ms-powerpoint=org.libreoffice.LibreOffice.desktop;
application/msword=org.libreoffice.LibreOffice.desktop;
application/vnd.openxmlformats-officedocument.spreadsheetml.sheet=org.libreoffice.LibreOffice.desktop;
application/vnd.openxmlformats-officedocument.presentationml.presentation=org.libreoffice.LibreOffice.desktop;
iapplication/vnd.openxmlformats-officedocument.wordprocessingml.document=org.libreoffice.LibreOffice.desktop;

text/plain=code-oss.desktop;

x-scheme-handler/jetbrains=jetbrains-toolbox.desktop;

image/jpeg=org.gnome.eog.desktop;
image/png=org.gnome.eog.desktop;
image/webp=org.gnome.eog.desktop;
image/gif=org.gnome.eog.desktop;
image/bmp=org.gnome.eog.desktop;
image/svg+xml=org.gnome.eog.desktop;

video/mpeg=org.videolan.VLC.desktop;
video/mp4=org.videolan.VLC.desktop;

audio/mpeg=org.videolan.VLC.desktop;

x-scheme-handler/http=org.mozilla.firefox.desktop;
x-scheme-handler/https=org.mozilla.firefox.desktop;
x-scheme-handler/mailto=org.mozilla.Thunderbird.desktop;
EOL
xdg-settings set default-web-browser org.mozilla.firefox.desktop
```

### Ajout de modèles de fichiers

Les utilisateurs de Windows connaissent tous l'option "clic droit + créer un document + nouveau document Word" très pratique quand on a besoin de manipuler beaucoup de fichiers sans passer nécessairement par les logiciels. Heureusement, XFCE propose une option assez similaire, il est donc possible directement depuis thunar de créer des templates de fichier à instancier avec un simple "clic droit + créer un document".

Dans l'exemple suivant, les templates vont servir à instancier différentes bases de code dans plusieurs langages :

```bash
cat << EOL > ~/Templates/bash.sh
#!/bin/bash

echo Hello world !

exit 0
EOL

cat << EOL > ~/Templates/c.c
#include <stdlib.h>
#include <stdio.h>

int main(int arcc, char *argv[]) {
  printf("Hello world!\n");
  return 0;
}
EOL

cat << EOL > ~/Templates/c++.cpp
#include <iostream>

using namespace std;

int main(int arcc, char *argv[]) {
  cout << "Hello world !" << endl;
  return 0;
}
EOL

cat << EOL > ~/Templates/html.html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <title>Title</title>
  <link href="./style.css" />
</head>
<body>
  <nav>
  </nav>

  <header>
  </header>

  <section>
    <h1>Section</h1>

    <article>
      Hello world!
    </article>
  </section>

  <footer>
  </footer>

  <script src="./script.js"></script>
</body>
</html>
EOL

cat << EOL > ~/Templates/markdown.md
---
title: Title
description: Description
author: Flavien PERIER <perier@flavien.io>
date: 2020-09-27 18:00
---

## Title

Content
EOL

chmod 440 ~/Templates/*
chmod 550 ~/Templates
```

### Quelques scripts

Voici quelques scripts à rajouter dans `~/bin` qui peuvent s'avérer utiles.

#### update

Un petit script qui met à jour toutes nos applications avec les différents gestionnaires de paquets installés sur l'OS et qui injecte les thèmes dans les différentes plateformes Flatpak :

```bash
#!/bin/bash

set -e

copy_theme() {
    SRC=$1
    DEST=$2

    mkdir -p $DEST
    chmod 700 $DEST

    for SRC_FILE in `ls $SRC`
    do
        DEST_PATH=$DEST/$SRC_FILE
        if [ ! -e $DEST_PATH ]
        then
            cp -Rf $SRC/$SRC_FILE $DEST_PATH
            find $DEST_PATH -type f -exec chmod 600 {} \;
            find $DEST_PATH -type d -exec chmod 700 {} \;
        fi
    done

    chmod 500 $DEST
}

source "$HOME/.sdkman/bin/sdkman-init.sh"

echo "Update pacman"
sudo pacman --noconfirm -q -Syyu
sudo pacman --noconfirm -q -S linux`uname -r | cut -f1,2 -d. | tr -d "."`-headers

echo "Update yay"
yay --noconfirm -q -Syyu

echo "Update Docker"
sudo docker images --format "{{.Repository}}:{{.Tag}}" | xargs -L1 sudo docker pull -q &

echo "Update Podman"
podman images --format "{{.Repository}}:{{.Tag}}" | xargs -L1 podman pull -q &

echo "Update SDK"
sdk update &

echo "Update Flatpak"
flatpak update --noninteractive -y &

wait

echo "Update Flatpak themes"
for PLATFORM in `ls $HOME/.local/share/flatpak/runtime`
do
    for VERSION in `ls $HOME/.local/share/flatpak/runtime/$PLATFORM/x86_64`
    do
        FLATPAK_DIR=$HOME/.local/share/flatpak/runtime/$PLATFORM/x86_64/$VERSION/active/files/share

        copy_theme $HOME/.themes $FLATPAK_DIR/themes &
        copy_theme $HOME/.icons $FLATPAK_DIR/icons &
        copy_theme $HOME/.fonts $FLATPAK_DIR/fonts &

        wait
    done
done

echo "End of update"
```

#### clean

Comme son nom l'indique, ce script permet de nettoyer différents éléments du système d'exploitation :

```bash
#!/bin/bash

CLEAN_CACHE=0

if [[ $* == "-c" ]] || [[ $* == "--cache" ]]
then
    CLEAN_CACHE=1
elif [[ $* == "-h" ]] || [[ $* == "--help" ]]
then
    echo "-c, --cache           Clean cache"
    exit 0
fi

PROTECTED_FOLDER=".ssh .themes .icons .fonts Templates Pictures Videos Music"
PROTECTED_FILETYPE="pdf zip tar gz 7z wav mp3 mp4 mkv mov"

get-used-space() {
   df | tr -s " " | cut -f3 -d " " | tail -n +2 | sed -z 's/\n/+/g;s/+$/\n/' | bc
}

USED_SPACE_BEFORE=`get-used-space`

# Clean up the package manager
sudo pacman -Scc --noconfirm
yay -Scc --noconfirm
sudo pip cache purge
pip cache purge
flatpak uninstall --unused -y
sudo pamac remove --orphans --no-confirm

# Cleans up Docker and Podman
sudo docker container prune -f
sudo docker volume prune -f
sudo docker network prune -f
sudo docker system prune -f
podman container prune -f
podman volume prune -f
podman network prune -f
podman system prune -f

# Cleans up the user session
rm -Rf $HOME/Downloads/*
rm -Rf $HOME/.local/share/.Trash
rm -f $HOME/.xsession-errors*
rm -f $HOME/.local/share/recently-used.xbel*
find $HOME -type f -iname "*.old" -delete

# Cleans up cache
if [ $CLEAN_CACHE -eq 1 ]
then
    sudo rm -Rf /var/cache/*
    sudo rm -Rf /root/.cache/*
    rm -Rf $HOME/.cache/*
    find $HOME/.var/app/ -type d -name "cache" -exec rm -Rf {}/* \;
fi

# Configuration of user rights
sudo chmod -R go-rwx $HOME
sudo find $HOME ! -user $USER -exec chown $USER {} \;
sudo find $HOME ! -group $USER -exec chgrp $USER {} \;
sudo find $HOME -type d ! -perm 700 -exec chmod 700 {} \; 
chmod -R 500 $HOME/bin
chmod 777 $HOME/Public
chmod 750 $HOME
setfacl -R --remove-all $HOME
setfacl -m g:libvirt-qemu:rx $HOME
find $HOME/Vms -type d ! -perm 550 -exec chmod 550 {} \; 
find $HOME/Vms -type f ! -perm 770 -exec chmod 770 {} \; 
sudo chgrp -R libvirt-qemu $HOME/Vms
for FOLDER in $PROTECTED_FOLDER
do
    find $HOME/$FOLDER -type d ! -perm 500 -exec chmod 500 {} \;
    find $HOME/$FOLDER -type f ! -perm 400 -exec chmod 400 {} \;
done
for FILETYPE in $PROTECTED_FILETYPE
do
    find $HOME -type f -name "*.$FILETYPE" -exec chmod ugo-wx {} \;
done

# Cleans up the system
sudo find /etc -iname "*.old" -delete
sudo find /etc -iname "*-" -delete
sudo find /etc -iname "*~" -delete

# Delete unused logs
find $HOME -type f -iname "*.log" -delete
find $HOME -type f -iname "*.log.[0-9]" -delete
sudo journalctl --vacuum-time=1d
sudo find /var/log -type f -iname "*.[0-9]" -delete
sudo find /var/log -type f -iname "*.[0-9].log" -delete
sudo find /var/log -type f -iname "*.old" -delete
sudo rm -f /var/log/optimus-manager/daemon/*.log
sudo rm -f /var/log/optimus-manager/switch/*.log

# Delete history
sudo rm -f /root/.bash_history
rm -f $HOME/.bash_history
rm -f $HOME/.python_history
rm -f $HOME/.node_repl_history
rm -f $HOME/.wget-hsts
rm -f $HOME/.lesshst
yes all | sudo fish -c "history delete all"

yes all | fish -c "history delete --prefix 'sudo pacman '"
yes all | fish -c "history delete --prefix 'pacman '"
yes all | fish -c "history delete --prefix 'yay '"
yes all | fish -c "history delete --prefix 'flatpak '"

yes all | fish -c "history delete --prefix clear"
yes all | fish -c "history delete --prefix ls"
yes all | fish -c "history delete --prefix ll"
yes all | fish -c "history delete --prefix 'rm '"
yes all | fish -c "history delete --prefix 'mv '"
yes all | fish -c "history delete --prefix 'rmdir '"
yes all | fish -c "history delete --prefix 'touch '"

yes all | fish -c "history delete --prefix 'tar '"
yes all | fish -c "history delete --prefix '7z '"
yes all | fish -c "history delete --prefix 'unzip '"
yes all | fish -c "history delete --prefix 'unrar '"

USED_SPACE_AFTER=`get-used-space`

RECOVERED_SPACE=`echo "scale=3; ($USED_SPACE_BEFORE - $USED_SPACE_AFTER) / 1000000" | bc`

echo "Recovered space: $RECOVERED_SPACE Go"
```

#### Spotify-diff

Un script un peu particulier puisqu'il permet d'afficher la différence entre les musiques présentes dans le dossier `~/Music` et une playlist [Spotify](https://www.spotify.com/). Ce script exploite d'un côté les métadonnées des fichiers et de l'autre l'API public Spotify pour fonctionner. Pour l'utiliser, il suffit de mettre la playlist en public et de remplacer les `**********` de `SPOTIFY_PLAYLIST_ID` par l'identifiant de la playlist en question.

```bash
#!/bin/bash

SPOTIFY_PLAYLIST_ID=************
SPOTIFY_BEARER=`curl 'https://open.spotify.com/get_access_token?reason=transport&productType=web_player' | jq -cM ".accessToken" | sed s/\"//g`

BREAK=0
OFFSET=0

IFS=$'\n'
while [ $BREAK -eq 0 ]
do
        COUNT=0

        for LINE in `curl "https://api.spotify.com/v1/playlists/$SPOTIFY_PLAYLIST_ID/tracks?offset=$OFFSET&limit=100" -H "authorization: Bearer $SPOTIFY_BEARER" | jq -cM ".items[].track"`
        do
                ARTISTS=`echo $LINE | jq -cM .artists[].name | sed s/\"//g | tr "\n" ";" | sed -e "s/;/, /g" -e "s/, \$//g"`
                TITLE=`echo $LINE | jq -cM .name | sed s/\"//g`
                echo "$ARTISTS : $TITLE"
                COUNT=`expr $COUNT + 1`
        done

        if [ $COUNT -eq 100 ]
        then
                OFFSET=`expr $OFFSET + 100`
        else
                BREAK=1
        fi
done | sort > /tmp/spotify-music.txt

IFS=$';'
for FILE in `ls ~/Music | tr "\n" ";"`
do
        ARTISTS=`ffprobe ~/Music/$FILE 2>&1 | grep artist | sed "s/ *artist *: //g"`
        TITLE=`ffprobe ~/Music/$FILE 2>&1 | grep title | head -n 1 | sed "s/ *title *: //g"`
        echo "$ARTISTS : $TITLE"
done | sort > /tmp/local-music.txt

diff -iyd /tmp/spotify-music.txt /tmp/local-music.txt
```

## Conclusion

Je pense que la principale différence entre un utilisateur de Windows ou de Mac par rapport à un utilisateur de Linux et que l'un adapte son usage à ce que lui propose son système et l'autre adapte son système à son besoin. L'installation que je propose dans cet article n'est pas une installation orientée légèreté, beaucoup de logiciels sont installés, mais ils répondent tous à un besoin et aucun d'entre eux n'est superflux (sauf peut-être neofetch). Libre à vous à présent de vous inspirer (ou non) de mon installation et de construire la vôtre, qui répondra à votre besoin.

Il est à noter que cette page est régulièrement mise à jour et que par extension, des paragraphes puissent être incomplets à un instant donné.

Si vous rencontrez des problèmes, n'hésitez pas à me contacter par mail sur [perier@flavien.io](mailto:perier@flavien.io).

## Sources

- [Arch wiki - Documentation pacman](https://wiki.archlinux.fr/Pacman)
- [Arch wiki - Discord](https://wiki.archlinux.org/index.php/Discord)
- [Arch wiki - Podman](https://wiki.archlinux.org/title/Podman)
- [Manjaro wiki - Optimus Manager](https://wiki.manjaro.org/index.php?title=Optimus_Manager)
- [Manjaro wiki - Configure Graphics Cards](https://wiki.manjaro.org/index.php/Configure_Graphics_Cards)
- [Manjaro wiki - Networking](https://wiki.manjaro.org/index.php/Networking)
- [Documentation Ubuntu - cryptsetup](https://doc.ubuntu-fr.org/cryptsetup)
- [How to install Virtual Machine Manager (KVM) in Manjaro and Arch Linux](https://www.fosslinux.com/2484/how-to-install-virtual-machine-manager-kvm-in-manjaro-and-arch-linux.htm)
- [Adding Custom Themes to Flatpak Apps](https://forums.linuxmint.com/viewtopic.php?t=284418)
- [Unable to connect to Browser Plugin or SSH Agent](https://github.com/flathub/org.keepassxc.KeePassXC/issues/19)
- [[HOW TO] Run Firefox *and* KeePassXC in a flatpak and get the KeePassXC-Browser add-on to work](https://discussion.fedoraproject.org/t/how-to-run-firefox-and-keepassxc-in-a-flatpak-and-get-the-keepassxc-browser-add-on-to-work/19452?u=rugk)
- [KVM - Fix Missing Default Network](https://blog.programster.org/kvm-missing-default-network)
- [DDoS Protection With IPtables: The Ultimate Guide](https://javapipe.com/blog/iptables-ddos-protection/)
- [Copy and Paste in Tmux](https://www.rockyourcode.com/copy-and-paste-in-tmux/)
- [Configure your firewall to work with Sonos](https://support.sonos.com/s/article/688)
- [Ports nécessaires à Steam](https://help.steampowered.com/fr/faqs/view/2EA8-4D75-DA21-31EB)
- [Forum Manjaro : Link in flatpak apps won’t open anymore on click since last update](https://forum.manjaro.org/t/link-in-flatpak-apps-wont-open-anymore-on-click-since-last-update/149907/22)
- [Manjaro forum - Manjaro System repair / integrity check?](https://forum.manjaro.org/t/manjaro-system-repair-integrity-check/154486/11)