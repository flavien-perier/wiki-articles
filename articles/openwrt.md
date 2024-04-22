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

Ensuite il est possible de se connecter sur l'interface `192.168.1.1`

## Sources

- [Getting Started BPI-R4](https://docs.banana-pi.org/en/BPI-R4/GettingStarted_BPI-R4)
- [Sinovoip BananaPi BPi-R4](https://openwrt.org/inbox/toh/sinovoip/bananapi_bpi-r4)
- [ISP Configurations](https://openwrt.org/docs/guide-user/network/wan/isp-configurations)
- [[TUTORIEL] Remplacer sa bbox par un routeur OpenWRT (Avec IPTV)](https://lafibre.info/remplacer-bbox/tutoriel-remplacer-sa-bbox-par-un-routeur-openwrt-avec-iptv/)