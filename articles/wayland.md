---
title: Expérimentation Arch + Wayland
type: WIKI
categories:
  - system
description: Expérimentation de l'installation d'un système Arch avec Wayland.
author: Flavien PERIER <perier@flavien.io>
date: 2021-02-22 18:00
---

## Pourquoi Wayland

À mon sens, la principale problématique des environnements Linux aujourd'hui est qu'un utilisateur ne sachant pas utiliser son OS en ligne de commande (ce qui représente la grande majorité des utilisateurs) aura de très grandes difficultés à utiliser sa distribution correctement. La plupart des distributions Linux ont donc accumulé un important retard en termes d'expérience utilisateur par rapport à Microsoft et Apple.

Sur les systèmes d'exploitation Linux, c'est dans la plupart des cas le composant X11 qui va permettre au système d'exploitation de gérer l'affichage de toute interface. Le problème est que ce composant est vieillissant et qu'il souffre de nombreux problèmes d'architectures impactant la fiabilité la sécurité du composant. Heureusement, un nouveau projet est actuellement en cours de développement nommé Wayland. Sa conception est différente de celle de X11 et devrait corriger bon nombre de ses problèmes.

Comme je viens de le dire, Wayland est encore en cours de développement. Cependant, de premières versions stables sont déjà sorties et j’aimerais effectuer quelques expérimentations sur ce nouveau système d'affichage. Mais comme en s'en doute tous, il ne suffit pas de désinstaller X11 puis installer Wayland pour que tout fonctionne comme avant. En effet, les applications conçues pour utiliser X11 ne sont pas nécessairement compatibles avec Wayland... Par exemple le gestionnaire de bureau que j'ai pour habitude d'utiliser [XFCE](https://xfce.org/), n'est pas compatible, nous allons donc utiliser [Weston](https://gitlab.freedesktop.org/wayland/weston) qui est l'implémentation officielle de Wayland.

Maintenant que j'ai justifié mes choix de Wayland et Weston, je me dois de justifier l'utilisation d'une distribution Arch Linux plutôt qu'une autre. La principale raison, est qu'en utilisant une Arch nous allons partir d'une page blanche sur laquelle nous allons devoir installer couche après couche les services dont nous aurons besoin. En faisant cela, nous allons réduire les probabilités d'erreur de compatibilité entre des logiciels préinstallés et Wayland.

## Installation

Quand nous démarrons sur le disque d'installation d'Arch Linux, il n'y a aucune interface pour nous accueillir et aucun système d'installation automatisé. Nous allons donc nous même depuis cet utilitaire construire les premières couches de notre système afin qu'il soit un minimum fonctionnel.

### Installation de la distribution Arch

Pour passer le clavier en azerty :

```bash
loadkeys fr
```

Pour le reste de l'installation de la base de la distribution il suffit de suivre [ce tuto](https://wiki.archlinux.org/index.php/Installation_guide) sur le wiki officiel. Il est très bien fait et permettra d'avoir une base de système d'exploitation prête à l'emploi.

Pour mon installation je vais avoir besoin de 3 partitions :

- Une partition EFI de 300M pour le Grub. (/dev/sda1)
- Une partition SWAP mesurant par convention entre 1.5 et 2x la taille de la ram (/dev/sda3)
- Use partition root prenant le reste de la place (/dev/sda2)

Voici les différentes commandes que j'ai exécutées après avoir partitionné mon disque :

```bash
ip link
timedatectl set-ntp true

mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkswap /dev/sda3

mount /dev/sda2 /mnt
swapon /dev/sda3

pacstrap /mnt base linux linux-firmware grub systemd dhcpcd pacman sudo

genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt

echo "wayland-test" > /etc/hostname

# Tools configuration
echo "%sudo	ALL=(ALL:ALL) ALL" >> /etc/sudoers
groupadd sudo

# Local configuration
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc
echo "fr_FR.UTF-8 UTF-8" > /etc/locale.gen
echo "LANG=fr_FR.UTF-8" > /etc/locale.conf
echo "KEYMAP=fr" > /etc/vconsole.conf
locale-gen

# Network configuration
cat << EOL > /etc/hosts
127.0.0.1 localhost
::1       localhost
EOL
cat << EOL > /etc/resolv.conf
nameserver 208.67.222.222
nameserver 208.67.220.220
nameserver 1.1.1.1
nameserver 1.0.0.1
nameserver 151.80.222.79
EOL
systemctl enable dhcpcd

# Grub installation
mkdir /boot/efi
mount /dev/sda1 /boot/efi
grub-install /dev/sda --target=i386-pc
grub-mkconfig -o /boot/grub/grub.cfg

# User configuration
passwd
useradd -m admin
usermod -a -G sudo admin
passwd admin
curl -s https://raw.githubusercontent.com/flavien-perier/linux-shell-configuration/master/linux-shell-configuration.sh | bash -

exit
shutdown -h now
```

### Installation de Wayland + Weston

Maintenant que nous disposons d'une Arch pleinement fonctionnelle, nous allons pouvoir installer notre environnement graphique :

```bash
sudo pacman -S wayland pulseaudio weston
sudo pacman -S adwaita-icon-theme adobe-source-code-pro-fonts
```

Le problème qui se pose est qu’à ce niveau-là nous ne pourrons pas installer beaucoup d'autres applications sans un serveur X... Heureusement, les développeurs de Wayland ont mis au point un mock nommé xwayland permettant de régler un certain nombre de problèmes de compatibilité.

```bash
sudo pacman -S xorg-xwayland
```

Étape suivante nous allons forcer les différentes technologies de construction de fenêtre à utiliser Wayland. Pour ce faire nous allons passer par les variables d'environnement et l'installation de différents paquets :

```bash
sudo pacman -S qt5-wayland qt6-wayland glfw-wayland

cat << EOL >> /etc/environment
GDK_BACKEND=wayland
QT_QPA_PLATFORM=wayland
CLUTTER_BACKEND=wayland
SDL_VIDEODRIVER=wayland
EOL
```

Bon nombre de technologies supportent donc déjà Wayland de manière native. Il en est cependant une qui n'est toujours pas supportée, [Electron](https://www.electronjs.org/). Cette technologie permet aux développeurs de créer des clients lourds à base de langage issu du web (HTML, CSS JavaScript). Nous ne pourrons donc pas installer [Discord](https://discordapp.com/) ou [VisualStudio Code](https://code.visualstudio.com/) sur notre Distribution.

### Installation du navigateur

Aujourd'hui les usages des clients lourds diminuant et ceux des clients légers augmentant, le composant le plus important lors d'une installation est le navigateur. Nous allons donc nous tourner du côté de [Firefox](https://www.mozilla.org/fr/firefox/) qui affiche une compatibilité complète de la solution moyennant la déclaration d'une variable d'environnement :

```bash
export MOZ_ENABLE_WAYLAND=1 firefox

cat << EOL >> /etc/environment
MOZ_ENABLE_WAYLAND=1 firefox
EOL

sudo pacman -S firefox
```

### Installation d'un explorateur de fichier

Quel que soit le système d'exploitation, il est toujours important de pouvoir gérer ses fichiers. C'est pour cette raison que nous allons installer le gestionnaire de fichiers de [Gnome](https://www.gnome.org/), à savoir [Nautilus](https://wiki.gnome.org/Apps/Files). Ce choix étant motivé par le fait que Gnome est l'un des environnements qui possèdent la meilleure intégration de Wayland.

```bash
sudo pacman -S nautilus
```

### Configuration de Weston

Sur cette dernière phase, nous allons configurer notre environnement afin d'y intégrer tout ce que nous avons fait précédemment :

```bash
cat << EOL > ~/.config/weston.ini
[core]
xwayland=true

[keyboard]
keymap_layout=fr

[shell]
cursor-theme=Adwaita
cursor-size=16
panel-position=bottom

[output]
name=LVDS1
mode=1920x1080
transform=90

[terminal]
font=Source Code Pro Black
font-size=15

[launcher]
icon=/usr/share/icons/Adwaita/24x24/apps/utilities-terminal-symbolic.symbolic.png
path=/usr/bin/weston-terminal

[launcher]
icon=/usr/share/icons/Adwaita/24x24/apps/system-file-manager-symbolic.symbolic.png
path=/usr/bin/nautilus

[launcher]
icon=/usr/share/icons/hicolor/24x24/apps/firefox.png
path=/usr/bin/firefox
EOL
```

## Conclusion

Wayland est une promesse intéressante. Cependant cette installation nous montre qu'il est pour le moment compliqué de se passer complètement du serveur X.

De plus, Weston a davantage à un rôle de présentateur que de solutions pérennes ! Cette installation n'est donc absolument pas viable pour un usage réel.

Personnellement pour une installation que j'utiliserais quotidiennement, je partirais sur un environnement Gnome qui affiche une très bonne compatibilité avec Wayland et accepterais que certaines de mes applications se lancent avec X11. Même si, actuellement, je préfère attendre que XFCE soit compatible et que les équipes de Manjaro s'occupent de déployer la solution dans mon système.

## Sources

- [Arch wiki: Installation guide](https://wiki.archlinux.org/index.php/Installation_guide)
- [Arch wiki: Wayland](https://wiki.archlinux.org/index.php/Wayland)
- [Arch wiki: Weston](https://wiki.archlinux.org/index.php/Weston)
- [Arch wiki: Connexion au réseau](https://wiki.archlinux.fr/Connexions_reseau)
- [Arch wiki: Firefox](https://wiki.archlinux.org/index.php/Firefox)
- [How to Install GRUB on Arch Linux (UEFI)](https://fasterland.net/how-to-install-grub-on-arch-linux-uefi.html)
- [How To Manually Install Arch Linux on a KVM VPS via VNC](https://www.linuxbabe.com/linux-server/install-arch-linux-on-kvm-vps)
