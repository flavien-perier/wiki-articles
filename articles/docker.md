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

La conteneurisation propose une approche radicalement différente. Contrairement à des machines virtuelles, les environnements conteneurisés ne contiennent pas leur propre noyau et utilisent simplement celui de l'hyperviseur. Différentes technologies sont alors mises en place par le système hôte afin de garantir que les conteneurs sont correctement isolés les uns des autres (comme par exemple le chrooting). Les éditeurs de distributions pour conteneurs n'ont plus besoin de gérer ni le noyau ni les fonctionnalités spécifiques au hardware. En effet, l'environnement Docker étant standardisé, il n'est par exemple pas utile à un conteneur d'embarquer des drivers pour carte réseau, car cet aspect est à la charge de l'hyperviseur. Ainsi, certaines distributions comme [Alpine Linux](https://www.alpinelinux.org/) arrivent à diminuer le poids de leurs images à moins de 5Mo. Le poids sur disque est une chose, mais à l'ère du stockage illimité ça ne serait pas un argument suffisant. Les ressources matérielles nécessaires au lancement d'une application conteneurisé sont tellement faibles, qu'elles ne sont pas significativement supérieures à ce que consommerait la machine hôte en termes de ressource RAM/CPU si elle lançait elle-même l'application sans le conteneur. Le temps de démarrage d'un conteneur va donc se compter en seconde quand celui d'une machine virtuelle se compte généralement en minutes (même si cela dépend de la façon d'on les éléments ont été packagé).

## Description de l'architecture

![Architecture de Docker](https://medias.flavien.io/articles/docker/architecture-docker.svg)

### Les conteneurs

Dans Docker, un conteneur est un environnement isolé de la machine hôte, sur laquelle va s'exécuter une application et ses dépendances. Le cycle de vie du conteneur va être calqué sur celui de cette application. L'environnement sera complètement up au moment du démarrage de cette dernière et s'éteindra en même temps qu'elle. Par exemple dans le cas d'un conteneur exécutant un serveur Nginx, si l'application s'éteint, le conteneur va également s'éteindre. Il faut donc partir du principe qu'un conteneur ne va exécuter qu'une seule application. On entend souvent dire que la conteneurisation va remplacer la virtualisation. Pour certains usages c'est vrai et c'est déjà le cas. Les clouds modernes hébergent leurs services essentiellement sur des conteneurs (même si les machines hôtes sont souvent des machines virtuelles). Cependant ce que propose Docker est davantage de remplacer [systemd](https://systemd.io/) et la notion de démon tel qu'elle est utilisé sur la plupart des serveurs.

À l'utilisation, il faut partir du principe qu'un conteneur à un cycle de vie court. Cela signifie qu'il sera rapidement détruit par un orchestrateur, un redémarrage de la machine hôte... Dans tous les cas, il faut concevoir les applications qui s'exécutent dans des environnements conteneurisés de telle sorte qu'elles soient sans état (stateless). Les états applicatifs doivent donc soit être stockés dans des volumes qui seront montés sur le conteneur à son démarrage, soit dans des services externes tels que des bases de données.

Une fois lancée une application devra exposée un ou plusieurs ports sur la machine hôte afin d'être rendu accessible pour le reste du réseau. Si le conteneur n'a pas vocation à être exposé, il ne sera visible que du réseau Docker. Si on prend l'exemple d'une application [Nextcloud](https://nextcloud.com/), le conteneur de base de donne ne sera visible que pour le conteneur du service de cloud.

### Les images

Si on devait parler de Docker comme d'un langage de programmation, les images seraient des class et les conteneurs serait les instances de ces class. Pour continuer la métaphore, l'image Docker va hériter d'une image sous-jacente, qui elle-même va très certainement hériter d'une autre image. L'image la plus basse étant généralement un système d'exploitation. Il s'agit du mécanisme de layer de Docker. Ainsi chaque layer ne contient que les différences avec celui du dessous et plusieurs images peuvent avoir des layers en commun. Ainsi quand on souhaite télécharger une image on a besoin de récupérer uniquement les layers qui ne sont pas déjà sur la machine.

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

### Les Dockerfiles

Nous venons de voir qu'il est possible de créer une Image Docker à la main en injectant des commandes dans un conteneur. Cependant, cette technique n'est pas entièrement satisfaisante, car elle obligerait les développeurs d'application soit à repackager leurs applications d'une version sur l'autre, soit à développer des scripts shell complexes pour automatiser ces actions. Un langage WSL a donc été mis à disposition dans le but d'automatiser l'enchainement des actions menant à la création d'une image. Ce langage est utilisé au sein du [Dockerfile](https://docs.docker.com/engine/reference/builder/).

### La registry

Une [registry Docker](https://docs.docker.com/registry/) est un service accessible depuis le réseau sur lequel des images vont pouvoir être mises à disposition. Par défaut, la plupart des images que nous utilisons sont stockées sur le [Docker hub](https://hub.docker.com/). Les images qui y sont stockées ne portent aucun préfixe. Comme par exemple `postgres`.

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

C'est ce dernier tag qui serait le plus approprié dans une infrastructure. En effet, il est souvent préférable d'utiliser `Alpine` quand il est proposé par les éditeurs aux autres distributions, car bien adapté à Docker et surtout très léger. Ensuite, il faut surtout favoriser les tags qui donnent le plus de détail sur le numéro de version. Cela évite quand on met à jour son infrastructure d'avoir la mauvaise surprise d'une base de données plus compatible avec l'application qu'elle a en face.

### Débogage d'environnement

#### Visualiser les logs

Les conteneurs exposent les logs du programme qui est exécuté en premier plan. Il est donc possible de visualiser les logs grâce à la commande suivante :

```bash
docker logs container-test
```

Avec cette commande le retour va contenir tous les logs depuis le lancement du conteneur. Cependant, il peut être utile d'écouter les logs au fur et à mesure qu'elles arrivent (pour déboguer un serveur web par exemple) :

```bash
docker logs -f container-test
```

#### Entrer dans un conteneur

Dans de nombreux cas, il peut être nécessaire de visualiser l'intérieur d'un conteneur que ce soit pour le déboguer, tester des modifications en live pendant un développement...

```bash
docker exec -it container-test bash
```

Il est à noter que si le système utilise Alpine Linux ou une autre distribution légère, bash ne sera pas installé, il faudra donc ce replier sur le shell de base.

```bash
docker exec -it container-test sh
```

### docker-compose

Docker compose est une application développer par la communauté Docker. Elle permet de décrire une infrastructure basée sur Docker. On va donc pouvoir décrire l'ordre de lancement des conteneurs, les réseaux sur lesquels ils vont être connectés, les volumes auxquels ils auront accès...

## Impacte sur les développements

### Applications Statless

### Approche IAC (Infrastructure As Code)

Avec Docker, on peut considérer que le livrable que les développeurs doivent fournir n'est plus un `jar` ou un `exe`, mais une image. Ainsi tout un pan des problématiques liées au système est déplacé sur ces équipes. Le principal avantage est qu'une fois que l'application tourne sur un environnement, nous pouvons garantir qu'elle fonctionnera ailleurs (le seul problème peut être les différentes architectures de processeur).

Le problème est que tous les développeurs ne sont pas formés au système. De nombreuses approches existent parmi lesquelles on peut avoir :

- La mise en place d'équipes DevOps.
- L'utilisation de plateforme PaaS (Plateforme as a Service) telle que [Heroku](https://www.heroku.com/) qui est capable d'appliquer des templates de Dockerfile automatiquement en fonction des technologies utilisées et de les déployer sur leur serveur (très utilisé par les startups, mais créer une adhérence forte à ces solutions).
- L'utilisation de framework tel que [Quarkus](https://quarkus.io/) en Java qui est capable de générer le Dockerfile et même le docker-compose en fonction des librairies utilisées. Par mesure d'optimisation, ce framework a même la capacité de placer les librairies sur un layer et le code sur un autre. Ainsi il est possible de rebuild sont image pendant le développement en à peine quelques secondes.

## Du IaaS (Infracture As A Service) au Caas (Container As A Service)

### Kubernetes

Sans entrer plus dans le détail, [Kubernetes](https://kubernetes.io/) est une solution développée pas Google et maintenu par la [Cloud Native Computing Foundation](https://www.cncf.io/) permettant d'orchestrer des conteneurs dans une infrastructure distribuée, c'est-à-dire composée de plusieurs machines physiques ou virtuelles. La solution propose de très nombreuses options :

- Gestion de l'autoscaling: L'infrastructure est capable d'augmenter ou de diminuer le nombre d'instances d'une application en fonction de critères comme la charge moyenne des instances déjà en place. Il est possible d'éditer de très nombreuses règles et donc de définir très précisément une stratégie.
- Ingress: Dans Kubernetes, les Ingres sont les points d'entrées vers l'application. Ce sont eux qui vont servir de revers-proxy et s'occuper du load-balancing. En fonction des instances Kubernetes ils peuvent être sous [Nginx](https://nginx.org/) ou plus généralement sous [Traefik](https://traefik.io/traefik/).
- Mise à jour à chaud: Quand les applications le permettent (le point bloquant étant souvent le schéma de la base de données), il est possible d'effectuer des mises à jour des applications sur l'infrastructure sans même que les utilisateurs ne le remarquent. Pour ce faire il existe diverses stratégies de redirection de flux.
- Readiness probe: commence à utiliser une application uniquement au moment ou elle a complètement fini de démarrer. L'application ne prendra donc pas de trafic entrant durant son démarrage.
- Liveness probe: Vérifie à interval régulier que l'application est bien lancée. En cas de dysfonctionnement l'infrastructure est capable de réagir en fonction de scénario. Redémarrer l'application, faire une nouvelle instance...
- Déploiement d'application réparti entre plusieurs data-center sur plusieurs contient avec adaptation du nombre d'instances et des points de sortis en fonction de la position géographique des utilisateurs.

### Réponse aux problématiques d'autoscaling

Dans Kubernetes un [pod](https://kubernetes.io/fr/docs/concepts/workloads/pods/pod/) décrit un group de conteneur. En général un seul conteneur constituera le pod, mais dans certains cas particuliers, comme avec des [Service Mesh](https://www.redhat.com/fr/topics/microservices/what-is-a-service-mesh), il peut être indispensable d'avoir plusieurs services séparés, mais agissants comme un ensemble. Ainsi les différentes entités constituant le pod ne peuvent ni être instanciées séparément ni instanciées sur des machines séparées.

Toujours dans Kubernetes, un [node](https://kubernetes.io/docs/concepts/architecture/nodes/) décrit une machine appartenant au réseau Kubernetes. En fonction de l'architecture du cloud sous-jacent, il peut s'agir d'une machine physique, ou d'une machine virtuelle.

Kubernetes possède des mécaniques qui lui permettent de répartir au mieux la charge au sein des différents nodes qui constituent son infrastructure. Il sera donc capable de créer de nouvelles instances de pod afin d'utiliser au mieux les ressources mises à sa disposition et éviter les dégradations de services. Pour les fournisseurs de cloud, cela permet également d'éviter d'allouer trop de ressources à des applications qui ne serait pas ou peu utilisé. Cette technologie permet donc de faire tourner plus d'applications sur moins de machines, à condition que les pics de montée en charge ne soient pas tous au même moment. Pour prendre un exemple, le service de streaming de jeux dans le cloud [Shadow](https://shadow.tech/) est une offre qui est utilisée par des particuliers sur leur temps personnel. Lors du [rachat par M. Klaba](https://www.lefigaro.fr/flash-eco/jeu-video-en-ligne-le-fondateur-d-ovhcloud-reprend-blade-shadow-20210430), le patron d'[OVH](https://www.ovh.com/), la nouvelle stratégie fut d'ajouter sur le même data-center des offres professionnelles. Ainsi, durant la journée les applications de bureau vont prendre la majeure partie du datacenter tandis que le soir, ces applications vont être désinstancier pour laisser place à des machines de jeux. Les pics de charges sur ces deux types d'applications étant non synchronisés, il est très facile de les faire cohabiter.

Cependant, des problématiques peuvent de nouveau survenir quand la totalité des ressources des nodes de l'application est consommée. Si Kubernetes ne répond pas directement à cette problématique, il est tout de même extrêmement facile d'ajouter un nouveau node à un cluster durant son fonctionnement et il est tout aussi facile de le supprimer. Les mécaniques de Kubernetes faisant en sorte que les ressources disponibles sur un node soient instanciées ailleurs en cas de perte. Ainsi dans le cas ou une entreprise gère elle-même son cluster Kubernetes, mais qu'elle repose sur un cloud-provider, il lui est donc possible de faire de l'autoscaling de node.

