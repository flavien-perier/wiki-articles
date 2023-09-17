---
title: Docker
type: WIP
categories:
  - system
  - development
description: Base de connaissances sur la solution de conteneurisation Docker.
author: Flavien PERIER <perier@flavien.io>
date: 2021-12-17 18:00
---

Durant la dernière décennie, l'architecture de nos centres de données et par extension celles de tous les services qu'ils hébergent a considérablement évolué. Parmi ces évolutions, l'une des plus marquantes est sans aucun doute la technologie de conteneurisation Docker qui a radicalement changé les approches non seulement en termes de développement des applications, mais également d'un point de vue administration des plateformes.

## De la virtualisation à la conteneurisation

La virtualisation est depuis le début des années 2000 très largement utilisé pour plusieurs raisons :

- Elle permet une meilleure répartition de la charge de travail. En effet, un ordinateur en IDLE (qui ne fait rien) consomme une quantité d'énergie presque aussi importante qu'un ordinateur exploitant 100% de ses ressources. Le but est donc de faire en sorte seule quelques machines porte la plus grande charge de travail possible. Dans un centre de données, cela permet de faire des économies d'énergie et donc financière très importantes.

- Il est plus aisé pour un administrateur système de faire des sauvegardes de machine virtuelle que de machine physique. Les disques durs virtuels peuvent facilement être sauvegardés, répliqués, réutilisés...

- Les machines virtuelles offrent également la possibilité d'installer plusieurs systèmes d'exploitation, ou versions de système d'exploitation sur une même machine physique.

Cependant, les machines virtuelles présentent également un important défaut inhérent à leur architecture : leur poids.

![Architecture d'une vm](https://medias.flavien.io/articles/docker/architecture-vm.svg)

En effet, comme on le voit dans le schéma précédent, chaque machine virtuelle va posséder son propre noyau de système d'exploitation. Ce détail d'architecture n'en est pas un. Le noyau Linux est lourd (une centaine de Mo) et rares sont les distributions qui pèsent moins de 500 Mo à l'installation. Pour un Windows ,il faut compter plus de 10Go pour une installation minimaliste. Cependant, le poids du noyau ne se répercute pas que sur le disque, il a également un coût de fonctionnement important. Autrement dit, si notre hyperviseur porte 5 machines virtuelles, on aura simultanément 6 noyaux en fonctionnement (celui de l'hyperviseur plus les 5 des VMs) ce qui va inutilement gaspiller les ressources en RAM et en CPU de la machine.

![Architecture d'un conteneur](https://medias.flavien.io/articles/docker/architecture-conteneur.svg)

La conteneurisation propose une approche radicalement différente. Contrairement à des machines virtuelles, les environnements conteneurisés ne contiennent pas leur propre noyau et utilisent simplement celui de l'hyperviseur. Différentes technologies sont alors mises en place par le système hôte afin de garantir que les conteneurs sont correctement isolés les uns des autres (comme par exemple le chrooting). Les éditeurs de distribuions pour conteneurs n'ont plus besoin de gérer ni le noyau ni les fonctionnalités spécifiques au hardware. En effet, l'environnement Docker étant standardisé, il n'est par exemple pas utile à un conteneur d'embarquer des drivers pour carte réseau, car cet aspect est à la charge de l'hyperviseur. Ainsi, certaines distributions comme [Alpine Linux](https://www.alpinelinux.org/) arrivent à diminuer le poids de leurs images à moins de 5Mo. Le poids sur disque est une chose, mais à l'ère du stockage illimité ça ne serait pas un argument suffisant. Les ressources matérielles nécessaires au lancement d'une application conteneurisé sont tellement faibles, qu'elles ne sont pas significativement supérieures à ce que consommerait la machine hôte en termes de ressource RAM/CPU si elle lançait elle-même l'application sans le conteneur. Le temps de démarrage d'un conteneur va donc se compter en seconde quand celui d'une machine virtuelle se compte généralement en minutes (même si cela dépend de la façon d'on les éléments ont été packagés).

## Description de l'architecture

![Architecture de Docker](https://medias.flavien.io/articles/docker/architecture-docker.svg)

### Les conteneurs

Dans Docker, un conteneur est un environnement isolé de la machine hôte, sur laquelle va s'exécuter une application et ses dépendances. Le cycle de vie du conteneur va être calqué sur celui de cette application. L'environnement sera complètement up au moment du démarrage de cette dernière et s'éteindra en même temps qu'elle. Par exemple dans le cas d'un conteneur exécutant un serveur Nginx, si l'application s'éteint, le conteneur va également s'éteindre. Il faut donc partir du principe qu'un conteneur ne va exécuter qu'une seule application. On entend souvent dire que la conteneurisation va remplacer la virtualisation. Pour certains usages c'est vrai et c'est déjà le cas. Les clouds modernes hébergent leurs services essentiellement sur des conteneurs (même si les machines hôtes sont souvent des machines virtuelles). Cependant ce que propose Docker est davantage de remplacer [systemd](https://systemd.io/) et la notion de démon tel qu'elle est utilisé sur la plupart des serveurs.

À l'utilisation, il faut partir du principe qu'un conteneur à un cycle de vie court. Cela signifie qu'il sera rapidement détruit par un orchestrateur, un redémarrage de la machine hôte... Dans tous les cas, il faut concevoir les applications qui s'exécutent dans des environnements conteneurisés de telle sorte qu'elles soient sans état (stateless). Les états applicatifs doivent donc soit être stockés dans des volumes qui seront montés sur le conteneur à son démarrage, soit dans des services externes tels que des bases de données.

Une fois lancée une application devra exposée un ou plusieurs ports sur la machine hôte afin d'être rendu accessible pour le reste du réseau. Si le conteneur n'a pas vocation à être exposé, il ne sera visible que du réseau Docker. Si on prend l'exemple d'une application [Nextcloud](https://nextcloud.com/), le conteneur de base de donne ne sera visible que pour le conteneur du service de cloud.

### Les images

Si on devait parler de Docker comme d'un langage de programmation, les images seraient des class et les conteneurs serait les instances de ces class. Pour continuer la métaphore, l'image Docker va ériter d'une image sous-jacente, qui elle-même va très certainement hériter d'une autre image. L'image la plus basse étant généralement un système d'exploitation. Il s'agit du mécanisme de layer de Docker. Ainsi chaque layer ne contient que les différences avec celui du dessous et plusieurs images peuvent avoir des layers en commun. Ainsi quand on souhaite télécharger une image on a besoin de récupérer uniquement les layers qui ne sont pas déjà sur la machine.

Pour prendre un exemple, imaginons un environnement Docker dans lequel nous aurions besoin d'une application Java et de sa base de données. On va avoir schématiquement quelque chose qui ressemble à cela :

- Une image de l'application : Layer Alpine + Layer JDK + Layer de l'application
- Une image de la base de données: Layer Alpine + Layer PostgreSQL

Pour comprendre le fonctionnement, nous pouvons utiliser un exemple :

- Nous allons ici prendre une image Alpine et y exécuter la commande `touch hello.txt` :

```bash
docker run --name container-test -it alpine touch hello.txt
```

- La commande prenant fin, le conteneur s'éteint. On peut regarder ce qui a changé sur le file-system de ce conteneur par rapport à l'image sur laquelle il se base :

```bash
docker diff container-test
```

- Normalement il n'y a qu'un unique fichier de différence, celui que nous venons de créer `hello.txt`. Nous allons donc pouvoir créer une image à partir des modifications apportées à ce conteneur :

```bash
docker commit container-test image-test
```

- Ensuite nous allons pouvoir regarder les différents layers de l'image `image-test` et les comparer avec ceux de l'image `alpine` :

```bash
docker history image-test
docker history alpine
```

- Nous constatons donc que l'image `image-test` à un unique layer de plus que l'image `Alpine` et qu'il a un poids négligeable du fait qu'il ne contient que l'information comme quoi un fichier vide `hello.txt` a été créé. Maintenant que tout est terminé nous pouvons nettoyer l'environnement :

```bash
docker rm -f container-test
docker rmi -f image-test
```

### La registry

Une registry Docker est un service accessible depuis le réseau sur lequel des images vont pouvoir être mises à disposition. Par défaut, la plupart des images que nous utilisons sont stockées sur le [Docker hub](https://hub.docker.com/). Les images qui y sont stockées ne portent aucun préfixe. Comme par exemple `postgres`.

Si pour des besoins particuliers nous sommes amenés à déployer une registry à un autre endroit, les images porteront dans leur nom l'URL de cette registry. Cela pourrait donner par exemple : `flavien.io/postgres`.

Les layers sont une optimisation également très efficace dans le cas de la registry. En effet Docker, ne va récupérer que les layers qu'il n'a pas déjà à sa disposition. Dans l'exemple précédent de notre packaging de l'image `image-test`, dans le cas ou `Alpine` serait déjà présent sur la machine, seul le layer contenant le fichier `hello.txt` aurait été téléchargé.

Il est possible, pour le même nom d'image, de stocker plusieurs versions ou type types de packaging qui seront identifiés par des tags. Dans l'exemple précédent, aucun tag n'est précisé. Ce sera donc celui par défaut qui sera utilisé `latest`.

Il est à noter qu'il est possible pour l'éditeur d'une image de mettre à jour un tag. Ce qui peut donc poser des problèmes de sécurité si on ne choisit pas un tag approprié dans son infrastructure.

Pour continuer sur l'exemple de [PostgreSQL](https://www.postgresql.org/) :

- L'exemple de base, on télécharge la dernière version :

```bash
docker pull postgres
```

- Si on souhaite la dernière version sur un système  alpine :

```bash
docker pull postgres:alpine
```

- Si on souhaite la version 3.17 de la base de données sous Alpine :

```bash
docker pull postgres:alpine3.17
```

C'est ce dernier tag qui serait le plus approprié dans une infrastructure. En effet il est souvent préférable d'utiliser `Alpine` quand il est proposé par les éditeurs aux autres distributions, car bien adapté à Docker et surtout très léger. Ensuite il faut surtout favoriser les tags qui donnent le plus de détail quand au numéro de version. Cela évite quand on met à jour son infrastructure d'avoir la mauvaise surprise d'une base de données plus compatible avec l'application qu'elle a en face.

### Les Dockerfiles



### docker-compose

## Impacte sur les développements

### Applications Statless

### Approche IAC (Infrastructure As Code)

## Utilisation avec du CI/CD

### Gestion des tags

## Du IaaS (Infracture As A Service) au Caas (Container As A Service)

### Réponse aux problématiques d'autoscaling

### Kubernetes

## Sources

