---
title: OpenWrt
type: WIP
categories:
  - system
  - security
description: Configuration d'un modem sous OpenWrt.
author: Flavien PERIER <perier@flavien.io>
date: 2024-03-16 18:00
---

Au niveau du hardware l'équipement utilisé va être une carte [Banana Pi BPI-R4 Wifi 7](https://www.banana-pi.org/en/bananapi-router/155.html). Cette dernière présente notamment deux emplacements SFP qui vont permettre de rajouter un module fibre pour être raccordé directement au réseau de l'opérateur. En plus de cela cette carte possède 4 ports RJ45, plutôt pratique pour connecter les appareils domestiques directement en filaire et compatible WiFi 7. Bref cette carte propose de bien meilleures caractéristiques que les box les plus haut de gamme des opérateurs. En option superflue, il est même possible de rajouter jusqu'à 3 cartes SIM afin d'avoir une continuité d'activité en cas de rupture du réseau fibre.

Au niveau du software c'est la distribution [OpenWrt](https://openwrt.org/) qui va être utilisée. Cette distribution Linux étant spécifiquement faite pour être installée sur des routeurs.

## Installation

Dans un premier temps, il faut récupérter l'image OpenWRT pour carte micro sd sur leur lien [Google Drive](https://drive.google.com/file/d/146CUGBRC0ce5uN9nCM08Jegc51abAz1b/view).

Dans un second temps, on peut utiliser l'outil mis à disposition par Banana Pi afin d'écrire l'image sur la carte.

```bash
sudo pacman -S pv
wget https://raw.githubusercontent.com/BPI-SINOVOIP/bpi-tools/master/bpi-copy -O /tmp/bpi-copy
chmod 700 /tmp/bpi-copy
sudo /tmp/bpi-copy ~/Downloads/mtk-bpi-r4-SD-*-New.img /dev/sdd
```

Ensuite il est nécessaire de se connecter sur l'interface `192.168.1.1` afin de configurer le modem. Dans les options du system il est noteament possible de configurer un accès SSH, ce qui peut ce révéler pratique pour les opérations qui vont suivre.

## Connexion à l'opérateur

Afin de ce connecter à un opérateur il va falloir deux informations :

- L'addresse mac du modem fourni par l'opérateur: 
- Le numéro client: Qui est inscrit en haut de ca facture.

Une fois ces informations en notre pocession, il faut modifer le fichier `/etc/config/network` pour y ajouter la configuration suivantes :

```
config device
	option name 'eth2'
	option macaddr '$MAC_ADDRESS'
  option ipv6 '0'

config device
	option type '8021q'
	option ifname 'eth2'
	option vid '100'
	option macaddr '$MAC_ADDRESS'
	option name 'eth2.100'
  option ipv6 '0'

config interface 'wan'
	option proto 'dhcp'
	option clientid '$CLIENT_ID'
	option vendorid 'BYGTELIAD'
	option hostname '*'
	option macaddr '$MAC_ADDRESS'
	option device 'eth2.100'
  option delegate '0'
  option peerdns '0'
  option dns '1.1.1.1 1.0.0.1'
```

## Sources

- [Getting Started BPI-R4](https://docs.banana-pi.org/en/BPI-R4/GettingStarted_BPI-R4)
- [Sinovoip BananaPi BPi-R4](https://openwrt.org/inbox/toh/sinovoip/bananapi_bpi-r4)
- [ISP Configurations](https://openwrt.org/docs/guide-user/network/wan/isp-configurations)
- [[TUTORIEL] Remplacer sa bbox par un routeur OpenWRT (Avec IPTV)](https://lafibre.info/remplacer-bbox/tutoriel-remplacer-sa-bbox-par-un-routeur-openwrt-avec-iptv/)