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

## Sources

- [Getting Started BPI-R4](https://docs.banana-pi.org/en/BPI-R4/GettingStarted_BPI-R4)
- [Sinovoip BananaPi BPi-R4](https://openwrt.org/inbox/toh/sinovoip/bananapi_bpi-r4)
- [ISP Configurations](https://openwrt.org/docs/guide-user/network/wan/isp-configurations)
- [[TUTORIEL] Remplacer sa bbox par un routeur OpenWRT (Avec IPTV)](https://lafibre.info/remplacer-bbox/tutoriel-remplacer-sa-bbox-par-un-routeur-openwrt-avec-iptv/)
- [Config OpenWRT derrière ONT Bouygues](https://lafibre.info/remplacer-bbox/config-openwrt-derriere-ont-bouygues/)
