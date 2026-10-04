---
title: Cryptographie post-quantique
type: WIP
categories:
  - system
  - security
description: Impact de la cryptographie post-quantique sur la sécurité moderne.
author: Flavien PERIER <perier@flavien.io>
date: 2026-09-25 19:00
---

## Les bases de la cryptographie

### Algorithmes asymétriques

Les algorithmes asymétriques se découpent en deux éléments distincts ; une clé publique et une clé privée.

Ces algorithmes possèdent différentes propriétés :

- La clé publique permet de chiffrer la donnée tandis que la clé privée permet de la déchiffrer.
- À partir de la clé privée on peut retrouver la clé publique.
- On peut signer une donnée à partir de la clé privée et vérifier l'authenticité de la signature grâce à la clé publique.

Il existe plusieurs algorithmes qui permettent d'implémenter cette mécanique :

- RSA : Basé sur un problème de factorisation de nombres premiers.
- DSA : Basé sur un problème de logarithme discret.
- Les courbes elliptiques : Basé sur un problème de logarithme discret sur des courbes elliptiques.

### Algorithmes symétriques

Les algorithmes symétriques permettent de chiffrer une donnée avec le même secret que celui qui va permettre de le déchiffrer. Ainsi lors d'un échange de donnée chiffrée, les deux interlocuteurs doivent convenir à l'avance du secret à utiliser.

Il existe plusieurs algorithmes qui permettent d'implémenter cette mécanique :

- AES : Algorithmes permettant d'associer différentes méthodes pour chiffrer des données à partir d'un secret.
- chacha : Un autre algorithme de chiffrement.

### Algorithme de hashage

Il s'agit d'une famille d'algorithme qui permet de transformer une entitée numérique (une chaine de caractère, ou un chiffrer) en une courte chaine de caractère. En théorie, il n'est pas possible de retrouver l'élément d'origine à partir d'un hash, cependant, si la même entitée est hashé sur deux ordinateurs différents avec le même algorithme, alors on devrait retrouver le même hash dans les deux cas.

La aussi il existe de nombreux algorithmes qui permettent de hasher des éléments :

- MD5
- sha

### Explication d'un échange diff-helman de base

Le but d'un échange diff-helman est de créer un canal de communication sur entre deux interlocuteurs. Ici Alice et Bob.

Cette méthode peut être appliquée quelque soit le support, mais elle est à la base de toute les communications chiffrées sur internet.

Voici le déroulé d'un échange diff-helman :
- Au préalable Alice et Bob disposent tous deux d'une pair de clé généré dans un algorithme asymétrique de leur choix : clé privée / clé public.
- Ils vont échanger leur clé public.
- Chacun de son côté va générer un secret qui va correspondre à un nombre ou une chaine aléatoire.
- Ils vont échanger leur secret, donc Bob va envoyer la chaine qu'il a généré à Alice en la chiffrant avec la clé public de cette dernière et Alice va faire la même chose avec la chine qu'elle a généré et la clé public de Bob.
- Les deux interlocuteurs vont alors pouvoir déchiffrer le message qui leur a été envoyé grâce à leur clé privés respectives.
- Ils sont donc à ce stade tous les deux en posession du secret de l'autre et de leur propre secret.
- Ils n'ont donc plus qu'a aditionner leur propre secret avec celui de leur correspondant·e pour obtenir un nouveau secret.
- Leur futurs échanges pourront donc être réaliser en utilisant un algorithme symétrique dont la clée de chiffrement sera le secret obtenu à l'étape précédente. 

Cette méthode est extrèmement fiable, mais elle a un default, si Alice et Bob essaye de communiquer à travers un canal non fiable, un attatquant qui arriverait à intercepter tous leurs message depuis le début et disposant lui même d'une pair de clé privée / clé public pourrait lui même se faire passer pour Alice auprès de Bob et de Bob auprès d'Alice. C'est une attaque de l'homme du milieu active ([MITM](https://fr.wikipedia.org/wiki/Attaque_de_l%27homme_du_milieu)).

Pour ce prémunir de cela il existe deux solution :
- Avoir une mécanique permettant à l'un des deux interlocuteur de certifier que la clé public qu'il a recu de la part de l'autre interlocuteur est bien à celui qu'elle prétend être et pas à un tiers. C'est la mécanique standard sur le protocol https et cette mécanique est une autoritée de certification.
- Avoir une clé public qui a déjà été échangé au préalable par un moyen fiable afin que lors du début d'un échange l'un des deux interlocuteurs ai déjà la clé public de l'autre. C'est ce que fait SSH par exemple quand on s'authentifie par clé.

### Rôle des autorités de certifications

Les autorités de certifications ou certificate authority (CA) sont des organisations qui ont pour rôle de signer les certificats d'autres organisations (leurs clients).

Ainsi, si je souhaite créer un site web et l'exposer en https, je vais devoir respecter plusieurs étapes :
- Déclarer mon nom de domaine auprès d'un registrar (comme l'[Afnic](https://www.afnic.fr/) pour un site en .fr). Heureusement cette étape fastidieuse peut être simplifiée en passant par des intermédiaires comme [Gandi](https://www.gandi.net/) ou [OVH](https://www.ovhcloud.com/fr/domains/).
- Générer une paire de clé dans l'algorithme asymétrique de son choix (RSA, DSA, ECC...). La clé privée doit impérativement être conservée dans un lieu sécurisé.
- Contacter une autorité de certification qui va contrôler d'une façon ou d'une autre que vous êtes bien le possesseur du nom de domaine que vous souhaitez faire passer en https. Vous allez envoyer votre clé publique et ils vont vous renvoyer un certificat, qui contient votre clé publique, des métadonnées comme le nom de domaine auquel elle est rattachée, la date de début et de fin de validité du certificat... et une signature cryptographique qui pourra être vérifiée par n'importe quel ordinateur connecté à internet qui prouve que ni la clé publique ni ses métadonnées ont été altérées. C'est ce certificat qui va être envoyé aux utilisateurs de votre site et c'est grâce à lui que les navigateurs vont considérer que le site est sécurisé.

L'autorité de certification la plus utilisée pour de petits sites est [Let's Encrypt](https://letsencrypt.org/) car elle permet de délivrer des certificats gratuits et fournit un outil, le [certbot](https://certbot.eff.org/) qui permet de signer automatiquement ses certificats.

Pour fournir des certificats à ses clients, l'autorité de certification a elle-même besoin d'une paire de clé. La clé publique de n'importe quelle autorité de certification publique est présente d'office sur tous les ordinateurs ayant un système d'exploitation à jour. Les navigateurs et autres services connectés à internet utilisent cette clé publique pour vérifier que les certificats des sites sur lesquels on va sont bien valides.

Quant aux clés privées de ces autorités de certification (ou clés primaires) elles sont gardées très précieusement dans des infrastructures hautement sécurisées (incluant notamment des [HSM](https://fr.wikipedia.org/wiki/Hardware_Security_Module)). En cas de compromission de la clé privée d'une certificate authority, c'est toute l'infrastructure du web moderne qui est compromise. Si un état ou une organisation venait à utiliser la clé privée d'une autorité de certification de manière abusive, il pourrait très facilement déployer des attaques de l'homme du milieu et espionner des communications a priori chiffrées.

### Rôle du cipher

Dans un échange TLS, le cipher permet de définir quels sont les algorithmes qui vont être utilisés. Sur toutes les versions de SSL/TLS, il permet de définir l'algorithme symétrique et la fonction de hashage. Mais depuis TLS1.2 il laisse aussi la possibilité de sélectionner un algorithme asymétrique. En effet TLS1.2 laisse la possibilité de créer une pair de clé temporaire. Ainsi la pair de clé de base du serveur (celle signée par l'autoritée de certification) ne sert plus qu'a faire des opérations de signature et la pair de clé temporaire ne sert quand à elle plus qu'a faire des opérations de chiffrement. Cette mécanique n'apparait plus dans TLS1.3 (2018), car il s'agit aujourd'hui du mode par default. La clé de chiffrement est maintenant toujours différente de la clé de signature.

Voici quelques exemples de cypher TLS1.2 avec et sans clé temporairte :

Voici quelques exemples de cypher TLS1.3 :

### Explication du protocole TLS 1.3

Comme dit dans la partie précédente, aujourd'hui les méthodes de chiffrement sont 

## Impact des ordinateurs quantique sur les algorithmes actuels

Toutes les familles de clés asymétriques sont hypothétiquement cassables dans le cas où un ordinateur quantique serait déployé. L'algorithme qui permet de briser ces clés est l'[algorithme de Shor](https://fr.wikipedia.org/wiki/Algorithme_de_Shor). Cet algorithme permet de ramener la complexité de la décomposition d'un nombre en facteur premier (problème sous jacent de RSA) ou la résolution d'un logarithme discret (problématique dernière DSA ou ECC) en un temps polynomial (un temps finfi).

Dans le cas des algorithme symétrique, leur complexité est divisée. Par exemple une clé AES256 a une complexité équivalente à une clé AES128 pour un ordinateur quantique. L'algorithme qui permet de réduire cette complexité est l'[algorithme de Grover](https://fr.wikipedia.org/wiki/Algorithme_de_Grover).

L'algorithme de Grover impact aussi les fonctions de hashage. Donc un hash en SHA256 a une complexité équivalente à un SHA128 pour un ordinateur quantique.

## Quels sont les risques que cela présente

Le jour ou une entreprise ou un gouvernement disposera d'un ordinateur quantique suffisament puissant pour appliquer les algorithmes de Shor et de Grover à nos méthodes de chiffrment actuel est noté le [Q-day](https://fr.wikipedia.org/wiki/Menace_quantique). C'est le moment ou nous ne pourrons plus faire confiance dans des algorithmes comme RSA.

- [Harvest now, decrypt later](https://fr.wikipedia.org/wiki/R%C3%A9colter_maintenant,_d%C3%A9chiffrer_plus_tard) : C'est une technique employté massivement pas les services de renseignements. Il y a des types de canaux qui, même si ils sont chiffré aujourd'hui, peuvent avoir de la valeur dans le futur. Il peut donc être intéréssent pour un attaquant de stocker de la donnée chiffrée dans l'espoir d'être en mesure de la décrypter plus tard. On peut prendre comme exemple l'[ambassade des états unis à Parias](https://fr.wikipedia.org/wiki/Ambassade_des_%C3%89tats-Unis_en_France) qui dispose d'une station d'écoute sur son toit pour, entre autre, espionner les communications entre l'Elysée et Matignon. La france ne réagit pas car les communications sont chiffrées et les Etats-Unis font le paris qu'ils seront capable de décrypter ces données plus tard quand ils auront attein la suprémassie quantique. Il s'agit donc d'un risque a prendre en compte des aujourd'hui.
  - Un risque qui en découle pour les utilisatuers d'internet. Etant données qu'on ne peut pas dire si notre navigation sur internet est enrengistrée ou non à un instant t, cela signifie qu'on peut partir du principe qu'elle l'est au moins à certain moments. Quand on s'authentifie à un site web, notre mot de passe est envoyée au serveur et il est hashé côté serveur (dans la majorité des cas). Comme on l'a vu on peu garder une confiance dans l'algorithme de hashage côté serveur, mais en revanche un attaquant qui a sauvegardé tout notre échange avec le serveur, peut le décrypter si il dispose d'un ordinateur quantique. Il peut donc retrouver notre mot de passe en clair. Si un jour nous passont le Q-day, il serait bon de changer tout ses mot de passe par précaution.
- Briser une autoritée de certification : Avec un ordinateur quantique, il serait possible de retrouver la clée privée d'une autoritée de certification à partir de sa clée public. Une fois cette clé retrouvée l'attaquant pourrait facilement mettre en place des attaques de l'homme du milieu sans pouvoir être détecté. Il s'agit d'un risque qui n'a pas forcément besoin d'être anticipé. Il supprime juste la confiance qu'on peut avoir dans les certificats traditionnel à partir du moment ou le Q-day est passé. Mais cela n'a aucun impact sur les données déjà chiffrées.

## Quels algorithmes en remplacement

En 2016, l'organisme de normalisation Américain [NIST](https://www.nist.gov/) a lancé un grand concour dans lequel les différents méthématiciens, cryptographe et autre laboratoire de recherche pouvaient proposer des algorithmes qui auraient pour but de remplacer les algorithmes de chiffrement asymétrique actuel. Ils ont recu 82 soumissions et en 2024 ils ont normalisé les 3 seuls qui n'ont pas été brisé.

- [FIPS203](https://csrc.nist.gov/pubs/fips/203/final) ou ML-KEM anciennement appelé CRYSTALS-Kyber : Pour la partie chiffrement.
- [FIPS204](https://csrc.nist.gov/pubs/fips/204/final) ou ML-DSA anciennement appelé CRYSTALS-Dilithium : Pour la partie signature.
- [FIPS205](https://csrc.nist.gov/pubs/fips/205/final) ou SLH-DSA anciennement appelé SPHINCS+ : Un autre algorithme de signature.

Il faut appliquer une méfiance modérée à ces différents algorithmes. Non pas qu'ils ne sont pas fiable, mais que comme dit précédemment sur 82 candidats seuls 3 n'ont pas été brisée et cela ne donne aucune garentie sur leur résistancer dans le futur.

Pour les algorithmes symétriques, on peut continuer d'utiliser les algorithmes connues, il faut "simplement" augmenter leur complexité afin que même divisée par deux par un ordinateur quantique, la sécurité des secrets chiffrés ne soit pas mise à mal.

## Contremesure applicable aujourd'hui

- Utiliser les bons ciphers.

## Evolutions à venir dans nos infrastructures

- Utiliser des clés ML-DSA (CRYSTALS-Dilithium).

## Sources

- [Barbhack : Crypto post-quantique : intérêt, enjeux, et perspectives](https://www.barbhack.fr/2023/assets/slides/barbhack23_public_deneuville.pdf)
- [USI event : La lévitation quantique: Julien Bobroff](https://www.youtube.com/watch?v=6kg2yV_3B1Q&list=PL_wN0FeVwhX5VSlQBioX21WNgRUoTTS5F)
- [Wikipedia : cryptographie post-quantique](https://fr.wikipedia.org/wiki/Cryptographie_post-quantique)

