---
title: Environement Firefox
type: WIKI
categories:
  - system
description: Configuration d'un navigateur Firefox orienté vie privée.
author: Flavien PERIER <perier@flavien.io>
date: 2020-09-25 12:00
---

L'application la plus utilisée sur un ordinateur est dans la majorité des cas le navigateur. Il est donc primordial d'attacher une attention toute particulière à ce logiciel pour tout ce qui touche à la sécurité et à la vie privée.

Pour ma part, j'utilise le navigateur Mozilla Firefox. Ce navigateur possède à mon sens deux avantages qui ne sont pas des moindres :

Tout d'abord il est open source. Le code source est donc à livre ouvert et chacun peu à la fois contribuer à l'amélioration de l'outil et vérifier qu'il ne comporte pas de portes dérobées. Pour ceux que ça intéresse, le code est disponible [sur les dépôts de Mozilla](https://hg.mozilla.org/mozilla-central/).

Il est respectueux de la vie privée. Contrairement à son concurrent Google Chrome, la société Mozilla accorde beaucoup d'importance à la sécurité des données des utilisateurs.

## Configuration

D'un point de vue configuration, le seul menu dont je vais parler est celui se nommant "Vie privée et sécurité".

![Configuration "Vie privée et sécurité"](https://medias.flavien.io/articles/firefox/configuration-privacy.webp)

Dans le premier menu nommé "Protection renforcée contre le pistage", je vous invite à cocher la case "personalisée" et cocher les 4 cases "Cookies", "Contenu utilisé pour le pistage", "Mineurs de cryptomonnaies" et "Détecteurs d’empreinte numérique". Pour les cookies je conseille de préciser "Tous les cookies tiers". Le navigateur nous signale que cela peut entrainer des dysfonctionnements sur certains sites, mais ayant cette option activée depuis des années je n'ai jamais constaté de problèmes particuliers liés à ce paramétrage. Je signale quand même que le blocage de la totalité des cookies peut quant à lui empêcher l'utilisateur de s'authentifier sur la plupart des sites proposant des comptes.

Sur le paramétrage suivant, je conseille d'activer l'option "ne pas me pister" et de la mettre sur "Toujours". Il s'agit ici d'une information que les navigateurs transmettent aux sites web que vous consulter leur dément de ne pas vous traquer. Il est cependant important de souligner qu'il s'agit simplement d'une convention et que les sites web ne sont pas obligés de tenir compte de cette requête.

Pour le paramétrage "Cookies et données de sites", je coche pour ma part l'option "Supprimer les cookies et les données des sites à la fermeture de Firefox". Cela signifie qu'à chaque fois que je vais fermer mon navigateur, je serais automatiquement déconnecté de tous les comptes sur lesquels je m'étais authentifiés.

Par la suite, pour le menu "Identifiants et mots de passe" je n'autorise personnellement pas à Firefox de se souvenir de mes mots de passe. Il est cependant important de souligner que durant ces dernières années, le coffre-fort de mot de passe de l'application a vu son niveau de sécurité augmenter. Si vous souhaitez l'utiliser, je vous conseille de cocher la case "Utiliser un mot de passe principal" et d'utiliser un mot de passe fort afin de protéger tous vos autres comptes. Vous pouvez aussi cocher la case "afficher des alertes pour les mots de passe de sites concernés par des fuites de données" qui va permettre à Firefox de vous afficher des alertes si un compte que vous utilisez s'est malencontreusement retrouvé dans une fuite de données. Cette fonctionnalité est une simple implémentation du site [Firefox monitor](https://monitor.firefox.com/), sur lequel je vous conseille vivement de vérifier régulièrement la présence ou non d'identifiants vous concernant dans des fuites de données.

Dans la partie historique, j'applique une philosophie identique à celle des cookies, à savoir que rien n'est conservé au-delà de ma session d'utilisation de Firefox. Décochez donc les 2 options "Conserver l’historique de navigation et des téléchargements" et "Conserver l’historique des recherches et des formulaires" puis cochez l'option "Vider l’historique lors de la fermeture de Firefox". Une fois que c'est fait, rendez-vous dans le menu des "paramètres d'effacement de l'historique" et cochez toutes les cases à l'exception de "cache" et "Données de sites web hors connexion". Ces deux paramètres permettent aux sites web de stocker des médias et autres composants de sites sur votre ordinateur et donc de ne pas les réactualiser à chaque fois que vous vous rendez sur le même site. Ces éléments n'ont qu'un faible impacte sur votre vie privée (voir pas d'impacte du tout), mais ont un impacte important en termes de Green-it étant donné qu'ils nous permettent d'éviter beaucoup d'appels réseau. C'est pourquoi je préfère ne pas les supprimer. Si vous voulez savoir quel type d'élément se cache derrière chacun de ces boutons, je vous conseille [la documentation Mozilla sur le sujet](https://support.mozilla.org/fr/kb/supprimer-historique-recent).

![Configuration "Paramètres d'effacement de l'historique"](https://medias.flavien.io/articles/firefox/configuration-history.webp)

Pour ce qui concerne l'autocomplétion dans la barre d'adresse, j'ai tendance à ne cocher que "Les marque-pages" afin de n'utiliser la barre de recherche que pour naviguer entre les favoris que j'ai déjà définis.

Pour les "Permissions", je conseille de "Bloquer les fenêtres popup" et de "Prévenir lorsque les sites essaient d’installer des modules complémentaires".

Pour la "Collecte de données par Firefox et utilisation", je décoche toutes les cases. Mon navigateur n'envoie donc aucune donnée d'utilisation à Mozilla. Cependant, si à un quelconque moment vous rencontrez des problèmes sur votre navigateur, je vous invite à recoucher ces cases afin que les développeurs de Firefox puissent obtenir les informations qui leur permettraient de reproduire le bug, pour par la suite le corriger.

Par la suite, pour tous les paramètres de sécurité, je vous invite, bien évidemment, à cocher toutes les cases dans "Protection contre les contenus trompeurs et les logiciels dangereux".

Enfin pour le "Mode HTTPS uniquement" je vous conseillerais de cocher la case "Activer le mode HTTPS uniquement dans toutes les fenêtres". Cela va forcer votre navigateur à utiliser le HTTPS plutôt que le HTTP quand cela est possible.

## Extensions

### Vie privée

#### [uBloc Origin](https://addons.mozilla.org/fr/firefox/addon/ublock-origin/)

![uBloc Origin logo](https://medias.flavien.io/articles/firefox/ublock-origin.webp)

Cette extension permet de bloquer les pubs que l'on peut trouver sur différents sites internet, ou avant les vidéos YouTube.

#### [Disconnect](https://addons.mozilla.org/fr/firefox/addon/disconnect/)

![Disconnect logo](https://medias.flavien.io/articles/firefox/disconnect.webp)

Disconnect a pour objectif de bloquer les différents traqueurs implémentés au sein des sites web. Cette extension rend donc le pistage des utilisateurs plus difficile.

#### [NoScript](https://addons.mozilla.org/fr/firefox/addon/noscript/)

![NoScript logo](https://medias.flavien.io/articles/firefox/noscript.webp)

Il s'agit d'une extension très puissante, mais qui va nécessiter une attention toute particulière. En effet, NoScript va tout simplement bloquer tous les contenus JavaScript au sein des sites web que nous visitons. Le problème est qu'en 2020, beaucoup de sites fonctionnant exclusivement avec des frameworks écrits dans ce langage de programmation. Il va donc falloir régulièrement ajouter des exceptions afin que les sites puissent continuer à fonctionner.

Il existe cependant au sein du logiciel une option nommée "Définir temporairement les sites de haut niveau comme FIABLES" permettant d'autoriser automatiquement les scripts provenant du même domaine que les sites que nous visitons.

![Configuration de NoScript](https://medias.flavien.io/articles/firefox/noscript-configuration.webp)

#### [ClearURLs](https://addons.mozilla.org/fr/firefox/addon/clearurls/)

![ClearURLs logo](https://medias.flavien.io/articles/firefox/clearurls.webp)

Une autre extension ayant pour but de renforcer la vie privée. Cette dernière nettoie les URLs des informations superflux qui pourrait donner des informations aux sites quant au chemin que l'utilisateur à emprunter dans ça navigation.

Dans les paramètres de cette application, je désactive l'option `Filtrer ETag`. En effet les [ETag](https://developer.mozilla.org/fr/docs/Web/HTTP/Headers/ETag) sont une composante importante de la mécanique de cache de nos navigateurs, les désactiver revient à rendre cette mécanique inopérante. Ce que je trouve personnellement très dommage.

#### [Decentraleyes](https://addons.mozilla.org/fr/firefox/addon/decentraleyes/)

![Decentraleyes logo](https://medias.flavien.io/articles/firefox/decentraleyes.webp)

Avec les extensions précédentes, nous pouvons bloquer de manière assez efficace les différents composants d'un site web ayant pour vocation de nous traquer. Cependant parmi toutes les méthodes pouvant être utilisées pour nous suivre à la trace on peut trouver les CDN. Un CDN est un site mettant à la disposition d'autres sites différents contenus tel que des scripts JavaScript, des polices de caractères... Les systèmes de tracking peuvent donc profiter du fait que les clients vont envoyer des requêtes pour récupérer ces contenus afin de savoir sur quel site nous sommes. Le problème c'est que ces contenus sont bien souvent nécessaires au bon fonctionnement d'un site. Par exemple, de nombreux sites ont besoin de JQuery pour fonctionner, ce dernier étant généralement délivré à travers des CDN. L'extension Decentraleyes va simplement régler le problème en téléchargeant les scripts délivrés par les CDN sur la machine de l'utilisateur. De cette manière, le nombre de requêtes transmises aux CDN se trouve grandement réduit. De plus pour les connexions limitées ceci a aussi pour avantage d'éviter les retéléchargements inutiles de contenu que l'on a déjà. Ce qui permet d'économiser de la bande passante.

#### [Random User-Agent](https://addons.mozilla.org/fr/firefox/addon/random_user_agent/)

![Random User-Agent logo](https://medias.flavien.io/articles/firefox/random_user_agent.webp)

Lorsqu'un navigateur envoie une requête à un site web, il envoie dans sa requête un User-Agent. Cette information permet au site de savoir quel navigateur est utilisé, sa version et éventuellement le système d'exploitation qui le porte.

Grâce à cette extension, cette information et falsifiée et régénérée toutes les 10 minutes. Ainsi, les trekkeurs peuvent très bien croire que vous utilisez Google Chrome sur un Mac alors que vous utilisez en réalité Firefox sur un Windows. Cela vous rend plus difficile à suivre et protège donc un peu plus votre anonymat.

Cependant, cette application peut aussi causer quelques petits désagréments. Par exemple, si on pense que vous êtes sur Internet Explorer ou sur un mobile il est probable que certains sites n'activent pas les mêmes options. Je vous conseille donc de configurer la liste des User-Agents que l'application peut utiliser et de ne cocher que les "Firefox", "Chrome" et "Edge" sur "Windows", "Mac" et "Linux". Il s'agit des combinaisons les plus supportées et qui, en général, posent le moins problème.

![Configuration de Random User-Agent](https://medias.flavien.io/articles/firefox/random_user_agent-configuration.webp)

Cette extension peut également poser quelques problèmes sur des sites de téléchargement, ou l'on va nous proposer de télécharger les versions "Mac" ou "Linux" de nos applications favorites.

#### [Auto Tab Discard](https://addons.mozilla.org/en-US/firefox/addon/auto-tab-discard/)

![Auto Tab Discard logo](https://medias.flavien.io/articles/firefox/auto-tab-discard.webp)

Cette extension permet d'optimiser la gestion de la mémoire au sein des onglets qui ne sont pas au premier plan.

### Développement

#### [Cookie Quick Manager](https://addons.mozilla.org/en-US/firefox/addon/cookie-quick-manager/)

![Cookie Quick Manager logo](https://medias.flavien.io/articles/firefox/cookie-quick-manager.webp)

Un simple gestionnaire de Cookies. Permets de les modifier, de les supprimer ou encore de les sauvegarder.

#### [ModHeader](https://addons.mozilla.org/fr/firefox/addon/modheader-firefox/)

![Cookie Quick Manager logo](https://medias.flavien.io/articles/firefox/modheader.webp)

Permets de rajouter des informations dans l'entête des requêtes envoyées par le navigateur.

#### [Shodan](https://addons.mozilla.org/en-US/firefox/addon/shodan_io/)

![Shodan logo](https://medias.flavien.io/articles/firefox/shodan_io.webp)

[Shodan](https://www.shodan.io/) est un site référençant pour chaque site et IPs accessible depuis internet les services exposés. De rapides tests de sécurités automatiques sont également effectués de manière automatique (test de mot de passe par défaut en fonction de l'équipement, version d'un service possédant une vulnérabilité de sécurité référence, mauvaise configuration...).

Cette extension permet donc tout simplement d'accéder à toutes ces informations sur les sites que nous visitons. Elle permet donc de contrôler facilement si une infrastructure n'expose pas trop d'information par rapport à ce qu'elle devrait et que les différents services exposés ne souffrent pas d'anomalies de configuration.

### [GreenIT-Analysis](https://addons.mozilla.org/fr/firefox/addon/greenit-analysis/)

![GreenIT-Analysis logo](https://medias.flavien.io/articles/firefox/gereenit-analysis.webp)

Cette application open source permet d'analyser les éléments d'un site web qui ne serait pas conforme à une démarche d'écoconception. Elle peut donc nous aider afin de savoir quoi regarder pour améliorer les performances de son site et réduire son empreinte énergétique.

### Confort

#### [Ecosia](https://addons.mozilla.org/fr/firefox/addon/ecosia-the-green-search/)

![Ecosia logo](https://medias.flavien.io/articles/firefox/ecosia.webp)

Ecosia est un métamoteur basé sur Bing qui utilise l'argent généré par nos recherches pour planter des arbres. Il s'agit donc d'une application à bilan carbone compensé, c'est-à-dire qu'elle tente de compenser la pollution engendrée par leur serveur en plantant des arbres qui vont absorber ce CO2. En plus de cela, l'application respecte la vie privée des utilisateurs et l'entreprise s'engage à ne pas revendre de données.

#### [I don't care about cookies](https://addons.mozilla.org/fr/firefox/addon/i-dont-care-about-cookies/)

![I don't care about cookies logo](https://medias.flavien.io/articles/firefox/i-dont-care-about-cookies.webp)

Si vous en avez marre de tous les bandeaux "Acceptez les cookies", cette application se charge automatiquement de cliquer sur "J'accepte" (ce n’est pas comme si on nous laissez vraiment le choix d'un autre côté) et de nous cacher les bandeaux en question. Simple et efficace pour une navigation sans encombre.

#### [SponsorBlock](https://addons.mozilla.org/en-US/firefox/addon/sponsorblock/)

![SponsorBlock logo](https://medias.flavien.io/articles/firefox/sponsorblock.webp)

Si les AdBlocks permettent de bloquer le les pubs hors des vidéos sur YouTube, il est beaucoup plus difficile de bloquer les placements de produit pour "NordVPN" ou autre "RAID: Shadow Legends" au sein même des vidéos. C'est pourtant bien ce que fait cette extension grâce à une communauté d'utilisateurs motivés qui taguent les moments des vidéos durant lesquelles des placements de produits sont effectués et permettent donc aux autres utilisateurs de sauter ces moments.

#### [Carbonalyser](https://addons.mozilla.org/fr/firefox/addon/carbonalyser/)

![Carbonalyser logo](https://medias.flavien.io/articles/firefox/carbonalyser.webp)

Un simple petit outil, qui évalue globalement l'impacte carbone de son utilisation d'internet. Il est à noter que dans le cas où les extensions présentées sont installées la consommation du navigateur sera plus faible que pour un utilisateur qui ne possède aucune extension.

