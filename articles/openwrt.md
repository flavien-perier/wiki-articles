---
title: OpenWrt
type: WIKI
categories:
  - system
  - security
description: Configuration d'un modem sous OpenWrt.
author: Flavien PERIER <perier@flavien.io>
date: 2024-03-16 18:00
---

Au niveau du hardware, l'équipement utilisé va être une carte [Banana Pi BPI-R4 Wifi 7](https://www.banana-pi.org/en/bananapi-router/155.html). Cette dernière présente notamment deux emplacements SFP qui vont permettre de rajouter un module fibre pour être raccordé directement au réseau de l'opérateur. En plus de cela, cette carte possède 4 ports RJ45, plutôt pratique pour connecter les appareils domestiques directement en filaire et compatible avec le WiFi 7. Bref, cette carte propose de bien meilleures caractéristiques que les box les plus hauts de gamme des opérateurs. En option superflue, il est même possible de rajouter jusqu'à 3 cartes SIM afin d'avoir une continuité d'activité en cas de rupture du réseau fibre.

Au niveau du software, c'est la distribution [OpenWrt](https://openwrt.org/) qui va être utilisée. Cette distribution Linux étant spécifiquement faite pour être installée sur des routeurs.

## Installation

- Dans un premier temps, il faut récupérer l'image pour carte sd sur le site [d'OpenWRT](https://firmware-selector.openwrt.org/?version=SNAPSHOT&target=mediatek%2Ffilogic&id=bananapi_bpi-r4).

- Dans un second temps, on peut insérer la carte et y écrire l'OS avec la commande :

```bash
zcat openwrt-mediatek-filogic-bananapi_bpi-r4-sdcard.img.gz | sudo dd of=/dev/sdd bs=1M status=progress
```

- La troisième étape consiste à brancher son pc à la carte et effectuer quelques configurations de base en SSH :

```bash
ssh root@192.168.1.1 passwd
ssh-keygen -f ~/.ssh/openwrt -t rsa -b 4096
ssh-copy-id -i ~/.ssh/openwrt root@192.168.1.1
```

## Mise à jour

OpenWRT utilise le gestionnaire de paquets apk. Il suffit de faire un `apk update` puis un `apk upgrade` pour mettre la distribution à jour.

Un routeur étant un équipement réseau particulièrement sensible il est conseillé de le faire assez régulièrement.

## [CrowdSec](https://www.crowdsec.net/)

CrowdSec est une suite de sécurité. Un bon scénario est d'installer un serveur CrowdSec quelque part dans son infrastructure. Ce dernier va analyser les fichiers de logs produits par nos serveurs (un Nginx par exemple) afin de prendre des décisions concernant des blocages d'IP.

Cependant, même si ce serveur CrowdSec prend des décisions de blocage, il faut un autre composant qui va faire la jonction entre CrowdSec et un firewall. Ce composant est nommé un `Bouncer`. Il en existe différents types que ce soit pour iptables, le firewall Windows... Et bien évidemment pour OpenWRT.

Tout d'abord sur la machine qui a CrowdSec d'installé (donc pas notre routeur), il faut enregistrer la machine avec un token d'authentification :

```bash
cscli bouncers add OpenWrt
```

Ensuite, au niveau du routeur, il faut installer notre configuration en mettant le bon nom d'hôte pour la machine CrowdSec et la bonne api key :

```bash
apk add crowdsec-firewall-bouncer

cat << EOF > /etc/config/crowdsec
config bouncer
	option enabled '1'
	option ipv4 '1'
	option ipv6 '1'
	option api_url 'http://$CROWDSEC_IP:8080/'
	option api_key '$CROWDSEC_API_KEY'
	option deny_action 'drop'
	option deny_log '0'
	option log_prefix 'crowdsec: '
	option log_level 'info'
	option filter_input '1'
	option filter_forward '1'
	list interface 'br-wan'
EOF

service crowdsec-firewall-bouncer reload
```

## Configuration

Les fichiers de configuration d'OpenWRT se situent dans `/etc/config`. C'est ce dossier qu'il faudra versionner ou au moins sauvegarder pour garder une trace de sa configuration réseau.

## Sources

- [OpenWRT : ISP Configurations](https://openwrt.org/docs/guide-user/network/wan/isp-configurations)
- [OpenWRT : Sinovoip BananaPi BPi-R4](https://openwrt.org/inbox/toh/sinovoip/bananapi_bpi-r4)
- [OpenWRT : CrowdSec](https://openwrt.org/docs/guide-user/services/crowdsec)
- [LaFibre.info : [TUTORIEL] Remplacer sa bbox par un routeur OpenWRT (Avec IPTV)](https://lafibre.info/remplacer-bbox/tutoriel-remplacer-sa-bbox-par-un-routeur-openwrt-avec-iptv/)
- [LaFibre.info : Config OpenWRT derrière ONT Bouygues](https://lafibre.info/remplacer-bbox/config-openwrt-derriere-ont-bouygues/)
- [Getting Started BPI-R4](https://docs.banana-pi.org/en/BPI-R4/GettingStarted_BPI-R4)
