---
title: Docker
type: WIP
categories:
  - system
  - developpement
description: Base de connaissances sur la solution de conteneurisation Docker.
author: Flavien PERIER <perier@flavien.io>
date: 2021-12-17 18:00
---

Durant la dernière décénie, l'architecture de nos centres de données et par extension celles de tous les services qu'ils hébergent a considérablement evolué. Parmis ces évolution, l'une des plus marquante est sans aucun doute la technologie de conteneurisation Docker qui a radicalement changé les approches non seulement en terme de développement des applications, mais également d'un point de vue adminisatration des plateformes.

## De la virtualisation à la conteneurisation

La vitualisation est depuis le début des années 2000 très largement utilisé pour plusieurs raisons :

- Elle permet une meilleur répartition de la charge de travail. En effet, un ordinateur en IDLE (qui ne fait rien) consomme une quantité d'énergie presque aussi importante qu'un ordinateur exploitant 100% de ses ressources. Le but est donc de faire en sorte seul quelques machines portent la plus grande charge de travail possible. Dans un centre de données cela permet de faire des économies d'énergie et donc financière très importantes.

- Il est plus aisé pour un administrateur système de faire des sauvegardes de machine virtuelle que de machine physique. Les disque dur virtuelles peuvent facilement être sauvegardés, répliqués, réutilisés...

- Les machines virtuelles offrent également la possibilitée d'installer plusieurs systèmes d'exploitation, ou versions de système d'exploitation sur une même machine physique.

Cepandnat les machines virtuelles présentent également un important défaut inhérent à leur architecture : Leur poid.

![Architecture d'une vm](https://medias.flavien.io/articles/docker/architecture-vm.svg)

En effet, comme on le voit dans le shéma précédent, chaque machine virtuelle va posséder son propre noyeau de système d'exploitation. Ce détail d'architecture n'en est pas un. Le noyeau Linux est lourd (une centaine de Mo) et rare sont les distributions qui pèsent moins de 500 Mo à l'installation. Pour un Windows il faut compter plus de 10Go pour une installation minimaliste. Cependant, le poid du noyeau ne ce répercute pas que sur le disque, il a également un coup de fonctionnement important. Autrement dit, si notre hyperviseur porte 5 machines virtuelles, on auras simultanément 6 noyeaux en fonctionnement (celui de l'hyperviseur plus les 5 des VMs) ce qui va inutilement gaspiller les ressources en RAM et en CPU de la machine.

![Architecture d'un contenur](https://medias.flavien.io/articles/docker/architecture-conteneur.svg)

La contenurisation propose une approche radicalement différente. Contrairement à des machines virtuelles, les environnements conteneurisés ne contiennent pas leur propre noyeau et utilise simplement celui de l'hyperviseur. Différentes technologies sont alors mise en place par le système hôte afin de garentir que les conteneurs sont correctement isolés les uns des autres (comme par exmple le chrooting). Les éditeurs de distribusions pour contenurs n'ont plus besoin de gérer ni le noyea ni des fonctionalitées spécifiques au hardware. En effet, l'environement Docker étant standardisé, il n'est par exemple pas utile à un conteneur d'embarquer des drivers pour carte réseau, car cet aspect est à la charge de l'hyperviseur. Ainsi, certaines distribution comme [Alpine Linux](https://www.alpinelinux.org/) arrivent à diminuer le poid de leur images à moins de 5Mo. Le poid sur disque est une chose, mais à l'ère du stockage illimité ca serait pas un argument. Les ressources matérielle nécessaires au lancement d'une application conteneurisé sont tellement faible, qu'elle ne sont pas significativement supérieur à ce que consommerais la machine hôte en terme de ressource RAM/CPU si elle lancais elle même l'application sans le conteneur. Le temps de démarrage d'un conteneur va donc ce compter en seconde quand celui d'une machine virtuelle ce compte en minutes.

## Description de l'architecture

![Architecture de Docker](https://medias.flavien.io/articles/docker/architecture-docker.svg)

### Les conteneurs

Dans Docker, un conteneur est un environnement isolé de la machne hôte, sur laquelle va s'exécuter une ou plusieurs applications.

### Les images

### La registry

### Les volumes

### Les Dockerfiles

## Impacte sur les développements

### Applications Statless

### Approche IAC (Infrastructure As Code)

## Utilisation avec du CI/CD

### Gestion des tags

## Du IaaS (Infracture As A Service) au Caas (Container As A Service)

### Réponse aux problématiques d'autoscaling

### Kubernetes

## Sources

