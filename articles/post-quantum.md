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

Il s'agit d'une famille d'algorithmes qui permet de transformer une entité numérique (une chaîne de caractères, ou un chiffre) en une courte chaîne de caractères. En théorie, il n'est pas possible de retrouver l'élément d'origine à partir d'un hash, cependant, si la même entité est hashée sur deux ordinateurs différents avec le même algorithme, alors on devrait retrouver le même hash dans les deux cas.

Là aussi il existe de nombreux algorithmes qui permettent de hasher des éléments :

- MD5
- sha

### Explication d'un échange Diffie-Hellman de base

Le but d'un échange Diffie-Hellman est de créer un canal de communication sécurisé entre deux interlocuteurs. Ici Alice et Bob.

Cette méthode peut être appliquée quel que soit le support, mais elle est à la base de toutes les communications chiffrées sur internet.

Voici le déroulé d'un échange Diffie-Hellman :
- Au préalable Alice et Bob disposent tous deux d'une paire de clés générée dans un algorithme asymétrique de leur choix : clé privée / clé publique.
- Ils vont échanger leur clé publique.
- Chacun de son côté va générer un secret qui va correspondre à un nombre ou une chaîne aléatoire.
- Ils vont échanger leur secret, donc Bob va envoyer la chaîne qu'il a générée à Alice en la chiffrant avec la clé publique de cette dernière et Alice va faire la même chose avec la chaîne qu'elle a générée et la clé publique de Bob.
- Les deux interlocuteurs vont alors pouvoir déchiffrer le message qui leur a été envoyé grâce à leur clés privées respectives.
- Ils sont donc à ce stade tous les deux en possession du secret de l'autre et de leur propre secret.
- Ils n'ont donc plus qu'à additionner leur propre secret avec celui de leur correspondant·e pour obtenir un nouveau secret.
- Leurs futurs échanges pourront donc être réalisés en utilisant un algorithme symétrique dont la clé de chiffrement sera le secret obtenu à l'étape précédente. 

Cette méthode est extrêmement fiable, mais elle a un défaut, si Alice et Bob essaient de communiquer à travers un canal non fiable, un attaquant qui arriverait à intercepter tous leurs messages depuis le début et disposant lui-même d'une paire de clés privée / clé publique pourrait lui-même se faire passer pour Alice auprès de Bob et de Bob auprès d'Alice. C'est une attaque de l'homme du milieu active ([MITM](https://fr.wikipedia.org/wiki/Attaque_de_l%27homme_du_milieu)).

Pour se prémunir de cela il existe deux solutions :
- Avoir une mécanique permettant à l'un des deux interlocuteurs de certifier que la clé publique qu'il a reçue de la part de l'autre interlocuteur est bien à celui qu'elle prétend être et pas à un tiers. C'est la mécanique standard sur le protocole https et cette mécanique est une autorité de certification.
- Avoir une clé publique qui a déjà été échangée au préalable par un moyen fiable afin que lors du début d'un échange l'un des deux interlocuteurs ait déjà la clé publique de l'autre. C'est ce que fait SSH par exemple quand on s'authentifie par clé.

### Rôle des autorités de certification

Les autorités de certification ou certificate authority (CA) sont des organisations qui ont pour rôle de signer les certificats d'autres organisations (leurs clients).

Ainsi, si je souhaite créer un site web et l'exposer en https, je vais devoir respecter plusieurs étapes :
- Déclarer mon nom de domaine auprès d'un registrar (comme l'[Afnic](https://www.afnic.fr/) pour un site en .fr). Heureusement cette étape fastidieuse peut être simplifiée en passant par des intermédiaires comme [Gandi](https://www.gandi.net/) ou [OVH](https://www.ovhcloud.com/fr/domains/).
- Générer une paire de clés dans l'algorithme asymétrique de son choix (RSA, DSA, ECC...). La clé privée doit impérativement être conservée dans un lieu sécurisé.
- Contacter une autorité de certification qui va contrôler d'une façon ou d'une autre que vous êtes bien le possesseur du nom de domaine que vous souhaitez faire passer en https. Vous allez envoyer votre clé publique et ils vont vous renvoyer un certificat, qui contient votre clé publique, des métadonnées comme le nom de domaine auquel elle est rattachée, la date de début et de fin de validité du certificat... et une signature cryptographique qui pourra être vérifiée par n'importe quel ordinateur connecté à internet qui prouve que ni la clé publique ni ses métadonnées ont été altérées. C'est ce certificat qui va être envoyé aux utilisateurs de votre site et c'est grâce à lui que les navigateurs vont considérer que le site est sécurisé.

L'autorité de certification la plus utilisée pour de petits sites est [Let's Encrypt](https://letsencrypt.org/) car elle permet de délivrer des certificats gratuits et fournit un outil, le [certbot](https://certbot.eff.org/) qui permet de signer automatiquement ses certificats.

Pour fournir des certificats à ses clients, l'autorité de certification a elle-même besoin d'une paire de clés. La clé publique de n'importe quelle autorité de certification publique est présente d'office sur tous les ordinateurs ayant un système d'exploitation à jour. Les navigateurs et autres services connectés à internet utilisent cette clé publique pour vérifier que les certificats des sites sur lesquels on va sont bien valides.

Quant aux clés privées de ces autorités de certification (ou clés primaires) elles sont gardées très précieusement dans des infrastructures hautement sécurisées (incluant notamment des [HSM](https://fr.wikipedia.org/wiki/Hardware_Security_Module)). En cas de compromission de la clé privée d'une certificate authority, c'est toute l'infrastructure du web moderne qui est compromise. Si un état ou une organisation venait à utiliser la clé privée d'une autorité de certification de manière abusive, il pourrait très facilement déployer des attaques de l'homme du milieu et espionner des communications a priori chiffrées.

### Rôle du cipher

Dans un échange TLS, le cipher permet de définir quels sont les algorithmes qui vont être utilisés. Sur toutes les versions de SSL/TLS, il permet de définir l'algorithme symétrique et la fonction de hashage. Mais depuis TLS 1.2 il laisse aussi la possibilité de sélectionner un algorithme asymétrique. En effet TLS 1.2 laisse la possibilité de créer une paire de clés temporaire. Ainsi la paire de clés de base du serveur (celle signée par l'autorité de certification) ne sert plus qu'à faire des opérations de signature et la paire de clés temporaire ne sert quant à elle plus qu'à faire des opérations de chiffrement. Cette mécanique n'apparaît plus dans TLS 1.3 (2018), car il s'agit aujourd'hui du mode par défaut. La clé de chiffrement est maintenant toujours différente de la clé de signature.

Voici quelques exemples de cipher TLS 1.2 avec et sans clé temporaire :

Voici quelques exemples de cipher TLS 1.3 :

### Explication du protocole TLS 1.3

Comme dit dans la partie précédente, aujourd'hui les méthodes de chiffrement sont 

## Impact des ordinateurs quantiques sur les algorithmes actuels

Toutes les familles de clés asymétriques sont hypothétiquement cassables dans le cas où un ordinateur quantique serait déployé. L'algorithme qui permet de briser ces clés est l'[algorithme de Shor](https://fr.wikipedia.org/wiki/Algorithme_de_Shor). Cet algorithme permet de ramener la complexité de la décomposition d'un nombre en facteurs premiers (problème sous-jacent de RSA) ou la résolution d'un logarithme discret (problématique derrière DSA ou ECC) en un temps polynomial (un temps fini).

Dans le cas des algorithmes symétriques, leur complexité est divisée. Par exemple une clé AES256 a une complexité équivalente à une clé AES128 pour un ordinateur quantique. L'algorithme qui permet de réduire cette complexité est l'[algorithme de Grover](https://fr.wikipedia.org/wiki/Algorithme_de_Grover).

L'algorithme de Grover impacte aussi les fonctions de hashage. Donc un hash en SHA256 a une complexité équivalente à un SHA128 pour un ordinateur quantique.

## Quels sont les risques que cela présente

Le jour où une entreprise ou un gouvernement disposera d'un ordinateur quantique suffisamment puissant pour appliquer les algorithmes de Shor et de Grover à nos méthodes de chiffrement actuelles est noté le [Q-day](https://fr.wikipedia.org/wiki/Menace_quantique). C'est le moment où nous ne pourrons plus faire confiance dans des algorithmes comme RSA.

- [Harvest now, decrypt later](https://fr.wikipedia.org/wiki/R%C3%A9colter_maintenant,_d%C3%A9chiffrer_plus_tard) : C'est une technique employée massivement par les services de renseignements. Il y a des types de canaux qui, même s'ils sont chiffrés aujourd'hui, peuvent avoir de la valeur dans le futur. Il peut donc être intéressant pour un attaquant de stocker de la donnée chiffrée dans l'espoir d'être en mesure de la décrypter plus tard. On peut prendre comme exemple l'[ambassade des États-Unis à Paris](https://fr.wikipedia.org/wiki/Ambassade_des_%C3%89tats-Unis_en_France) qui dispose d'une station d'écoute sur son toit pour, entre autres, espionner les communications entre l'Élysée et Matignon. La France ne réagit pas car les communications sont chiffrées et les États-Unis font le pari qu'ils seront capables de décrypter ces données plus tard quand ils auront atteint la suprématie quantique. Il s'agit donc d'un risque à prendre en compte dès aujourd'hui.
  - Un risque qui en découle pour les utilisateurs d'internet. Étant donné qu'on ne peut pas dire si notre navigation sur internet est enregistrée ou non à un instant t, cela signifie qu'on peut partir du principe qu'elle l'est au moins à certains moments. Quand on s'authentifie à un site web, notre mot de passe est envoyé au serveur et il est hashé côté serveur (dans la majorité des cas). Comme on l'a vu on peut garder une confiance dans l'algorithme de hashage côté serveur, mais en revanche un attaquant qui a sauvegardé tout notre échange avec le serveur, peut le décrypter s'il dispose d'un ordinateur quantique. Il peut donc retrouver notre mot de passe en clair. Si un jour nous passons le Q-day, il serait bon de changer tous ses mots de passe par précaution.
- Briser une autorité de certification : Avec un ordinateur quantique, il serait possible de retrouver la clé privée d'une autorité de certification à partir de sa clé publique. Une fois cette clé retrouvée l'attaquant pourrait facilement mettre en place des attaques de l'homme du milieu sans pouvoir être détecté. Il s'agit d'un risque qui n'a pas forcément besoin d'être anticipé. Il supprime juste la confiance qu'on peut avoir dans les certificats traditionnels à partir du moment où le Q-day est passé. Mais cela n'a aucun impact sur les données déjà chiffrées.

## Quels algorithmes en remplacement

En 2016, l'organisme de normalisation américain [NIST](https://www.nist.gov/) a lancé un grand concours dans lequel les différents mathématiciens, cryptographes et autres laboratoires de recherche pouvaient proposer des algorithmes qui auraient pour but de remplacer les algorithmes de chiffrement asymétrique actuels. Ils ont reçu 82 soumissions et en 2024 ils ont normalisé les 3 seuls qui n'ont pas été brisés.

- [FIPS203](https://csrc.nist.gov/pubs/fips/203/final) ou ML-KEM anciennement appelé CRYSTALS-Kyber : Pour la partie chiffrement.
- [FIPS204](https://csrc.nist.gov/pubs/fips/204/final) ou ML-DSA anciennement appelé CRYSTALS-Dilithium : Pour la partie signature.
- [FIPS205](https://csrc.nist.gov/pubs/fips/205/final) ou SLH-DSA anciennement appelé SPHINCS+ : Un autre algorithme de signature.

Il faut appliquer une méfiance modérée à ces différents algorithmes. Non pas qu'ils ne sont pas fiables, mais que comme dit précédemment sur 82 candidats seuls 3 n'ont pas été brisés et cela ne donne aucune garantie sur leur résistance dans le futur.

Pour les algorithmes symétriques, on peut continuer d'utiliser les algorithmes connus, il faut "simplement" augmenter leur complexité afin que même divisée par deux par un ordinateur quantique, la sécurité des secrets chiffrés ne soit pas mise à mal.

## Contre-mesure applicable aujourd'hui

- Utiliser les bons ciphers.

## Évolutions à venir dans nos infrastructures

- Utiliser des clés ML-DSA (CRYSTALS-Dilithium).

## Sources

- [Barbhack : Crypto post-quantique : intérêt, enjeux, et perspectives](https://www.barbhack.fr/2023/assets/slides/barbhack23_public_deneuville.pdf)
- [USI event : La lévitation quantique: Julien Bobroff](https://www.youtube.com/watch?v=6kg2yV_3B1Q&list=PL_wN0FeVwhX5VSlQBioX21WNgRUoTTS5F)
- [Wikipedia : cryptographie post-quantique](https://fr.wikipedia.org/wiki/Cryptographie_post-quantique)

