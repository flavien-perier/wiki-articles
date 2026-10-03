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

### Rôle des autorités de certifications

Les autorités de certifications ou certificate authority (CA) sont des organisations qui ont pour rôle de signer les certificats d'autres organisations (leurs clients).

Ainsi, si je souhaite créer un site web et l'exposer en https, je vais devoir respecter plusieurs étapes :
- Déclarer mon nom de domaine auprès d'un registrar (comme l'[Afnic](https://www.afnic.fr/) pour un site en .fr). Heureusement cette étape fastidieuse peut être simplifiée en passant par des intermédiaires comme [Gandi](https://www.gandi.net/) ou [OVH](https://www.ovhcloud.com/fr/domains/).
- Générer une paire de clé dans l'algorithme asymétrique de son choix (RSA, DSA, ECC...). La clé privée doit impérativement être conservée dans un lieu sécurisé.
- Contacter une autorité de certification qui va contrôler d'une façon ou d'une autre que vous êtes bien le possesseur du nom de domaine que vous souhaitez faire passer en https. Vous allez envoyer votre clé publique et ils vont vous renvoyer un certificat, qui contient votre clé publique, des métadonnées comme le nom de domaine auquel elle est rattachée, la date de début et de fin de validité du certificat... et une signature cryptographique qui pourra être vérifiée par n'importe quel ordinateur connecté à internet qui prouve que ni la clé publique ni ses métadonnées ont été altérées. C'est ce certificat qui va être envoyé aux utilisateurs de votre site et c'est grâce à lui que les navigateurs vont considérer que le site est sécurisé.

L'autorité de certification la plus utilisée pour de petits sites est [Let's Encrypt](https://letsencrypt.org/) car elle permet de délivrer des certificats gratuits et fournit un outil, le [certbot](https://certbot.eff.org/) qui permet de signer automatiquement ses certificats.

Pour fournir des certificats à ses clients, l'autorité de certification a elle-même besoin d'une paire de clé. La clé publique de n'importe quelle autorité de certification publique est présente d'office sur tous les ordinateurs ayant un système d'exploitation à jour. Les navigateurs et autres services connectés à internet utilisent cette clé publique pour vérifier que les certificats des sites sur lesquels on va sont bien valides.

Quant aux clés privées de ces autorités de certification (ou clés primaires) elles sont gardées très précieusement dans des infrastructures hautement sécurisées (incluant notamment des [HSM](https://fr.wikipedia.org/wiki/Hardware_Security_Module)). En cas de compromission de la clé privée d'une certificate authority, c'est toute l'infrastructure du web moderne qui est compromise. Si un état ou une organisation venait à utiliser la clé privée d'une autorité de certification de manière abusive, il pourrait très facilement déployer des attaques de l'homme du milieu ([MITM](https://fr.wikipedia.org/wiki/Attaque_de_l%27homme_du_milieu)) et espionner des communications a priori chiffrées.

### Rôle du cipher

### Explication du protocole TLS 1.2



### Explication du protocole TLS 1.3

## Impact des ordinateurs quantique sur les algorithmes actuels

Toutes les familles de clés asymétriques sont hypothétiquement cassables dans le cas où un ordinateur quantique serait déployé. L'algorithme qui permet de briser ces clés est l'[algorithme de Shor](https://fr.wikipedia.org/wiki/Algorithme_de_Shor).

Dans le cas de l'algorithme AES sa complexité est hypothétiquement divisée par deux. Par exemple une clé AES256 a une complexité équivalente pour une clé AES128. L'algorithme qui permet de réduire cette complexité est l'[algorithme de Grover](https://fr.wikipedia.org/wiki/Algorithme_de_Grover).

## Quels algorithmes en remplacement

Pour les algorithmes asymétriques :

- ML-KEM anciennement appelé CRYSTALS-Kyber : Pour la partie chiffrement.
- ML-DSA anciennement appelé CRYSTALS-Dilithium : Pour la partie signature.

## Contremesure applicable aujourd'hui

- Utiliser les bons ciphers.

## Evolutions à venir dans nos infrastructures

- Utiliser des clés ML-DSA (CRYSTALS-Dilithium).

## Sources

- [Barbhack : Crypto post-quantique : intérêt, enjeux, et perspectives](https://www.barbhack.fr/2023/assets/slides/barbhack23_public_deneuville.pdf)
- [USI event : La lévitation quantique: Julien Bobroff](https://www.youtube.com/watch?v=6kg2yV_3B1Q&list=PL_wN0FeVwhX5VSlQBioX21WNgRUoTTS5F)
- [Wikipedia : cryptographie post-quantique](https://fr.wikipedia.org/wiki/Cryptographie_post-quantique)

