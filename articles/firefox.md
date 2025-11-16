---
title: Environnement Firefox
type: WIKI
categories:
  - system
description: Configuration d'un navigateur Firefox orienté vie privée.
author: Flavien PERIER <perier@flavien.io>
date: 2020-09-25 12:00
---

L'application la plus utilisée sur un ordinateur est dans la majorité des cas le navigateur. Il est donc primordial d'attacher une attention toute particulière à ce logiciel pour tout ce qui touche à la sécurité et à la vie privée.

Pour ma part, j'utilise le navigateur Mozilla Firefox. Ce navigateur possède à mon sens deux avantages qui ne sont pas des moindres :

Tout d'abord, il est open source. Le code source est donc à livre ouvert et chacun peut à la fois contribuer à l'amélioration de l'outil et vérifier qu'il ne comporte pas de portes dérobées. Pour ceux que ça intéresse, le code est disponible [sur les dépôts de Mozilla](https://hg.mozilla.org/mozilla-central/).

Il est respectueux de la vie privée. Contrairement à son concurrent Google Chrome, la société Mozilla accorde beaucoup d'importance à la sécurité des données des utilisateurs.

## Configuration

D'un point de vue de la configuration, le menu le plus important est celui concernant les aspects privacy :

![Configuration "Vie privée et sécurité"](https://medias.flavien.io/articles/firefox/configuration-privacy.webp)

Avec la configuration proposée plus haut, le navigateur devrait avoir un niveau de sécurité optimal et un minimum en termes de privacy (sur ce point, ce sont les extensions qui vont faire l'essentiel du travail).

Pour aller plus loin, il est possible de cliquer sur le bouton "Paramètres" en face de "Vider l’historique lors de la fermeture de Firefox" afin de préciser quels éléments conserver et lesquels supprimer au moment de la fermeture du navigateur.

![Configuration "Paramètres d'effacement de l'historique"](https://medias.flavien.io/articles/firefox/configuration-history.webp)

Les options proposées vont faire en sorte que les caches des sites web vont être conservés (donc les sites ne seront pas entièrement retéléchargés d'une visite sur l'autre ce qui économise de la bande passante), mais les données qui les concernent seront supprimées.

Le seul défaut de cette configuration est qu'il faudra se reconnecter sur tous les sites à chaque redémarrage du navigateur.

### Configuration avancée

Il est également possible d'aller plus loin en allant dans les paramètres avancés du navigateur. Pour ce faire, il suffit de taper `about:config` dans la barre de recherche. Par la suite, plusieurs optimisations peuvent être effectuées :

#### Désactivation de la géolocalisation

- `geo.enabled`: false

#### Désactivation de la télémétrie en profondeur

- `browser.send_pings`: false
- `browser.newtabpage.activity-stream.feeds.telemetry`: false
- `browser.newtabpage.activity-stream.telemetry`: false
- `browser.ping-centre.telemetry`: false
- `toolkit.telemetry.bhrPing.enabled`: false
- `toolkit.telemetry.firstShutdownPing.enabled`: false
- `toolkit.telemetry.pioneer-new-studies-available`: false
- `toolkit.telemetry.newProfilePing.enabled`: false
- `toolkit.telemetry.reportingpolicy.firstRun`: false
- `toolkit.telemetry.shutdownPingSender.enabled`: false
- `toolkit.telemetry.unified`: false
- `toolkit.telemetry.updatePing.enabled`: false
- `toolkit.telemetry.archive.enabled`: false
- `datareporting.healthreport.uploadEnabled`: false
- `datareporting.policy.dataSubmissionEnabled`: false

#### Désactivation des animations

- `layout.css.animation-composition.enabled`: false
- `dom.animations-api.compositing.enabled`: false
- `dom.animations-api.getAnimations.enabled`: false
- `dom.animations-api.timelines.enabled`: false

#### Désactivation des fonctionnalités d'intelligence artificielle

- `browser.ml.enabled`: false
- `browser.ml.chat.enabled`: false
- `browser.tabs.groups.smart.enabled`: false

## Extensions

### Vie privée

#### [uBlock Origin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)

![uBlock Origin logo](https://medias.flavien.io/articles/firefox/ublock-origin.webp)

Cette extension permet de bloquer les pubs que l'on peut trouver sur différents sites internet, ou avant les vidéos YouTube.

#### [Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)

![Privacy Badger logo](https://medias.flavien.io/articles/firefox/privacy-badger.webp)

Cette extension permet de bloquer les traqueurs présents sur le web. Cela va par exemple empêcher les modules [Google Analytics](https://fr.wikipedia.org/wiki/Google_Analytics) de s'exécuter.

Il existe actuellement de nombreuses extensions permettant d'améliorer la vie privée. Mais les trois principales qui ont pour objectif de limiter le champ d'action des traqueurs sont :
- [Ghostery](https://addons.mozilla.org/en-US/firefox/addon/ghostery) qui est également un AdBlock, mais qui cause de nombreux dysfonctionnements, car trop agressive.
- [Disconnect](https://addons.mozilla.org/en-US/firefox/addon/disconnect/) qui est plutôt efficace, mais moins populaire que les deux autres.
- [Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/) qui se base sur des méthodes d'apprentissage pour bloquer les contenus indésirables.

#### [NoScript](https://addons.mozilla.org/en-US/firefox/addon/noscript/)

![NoScript logo](https://medias.flavien.io/articles/firefox/noscript.webp)

Il s'agit d'une extension très puissante, mais qui va nécessiter une attention toute particulière. En effet, NoScript va tout simplement bloquer tous les contenus JavaScript au sein des sites web que nous visitons. Le problème est qu'en 2020, beaucoup de sites fonctionnent exclusivement avec des frameworks écrits dans ce langage de programmation. Il va donc falloir régulièrement ajouter des exceptions afin que les sites puissent continuer à fonctionner.

Il existe cependant au sein du logiciel une option nommée "Définir temporairement les sites de haut niveau comme FIABLES" permettant d'autoriser automatiquement les scripts provenant du même domaine que les sites que nous visitons.

![Configuration de NoScript](https://medias.flavien.io/articles/firefox/noscript-configuration.webp)

#### [ClearURLs](https://addons.mozilla.org/en-US/firefox/addon/clearurls/)

![ClearURLs logo](https://medias.flavien.io/articles/firefox/clearurls.webp)

Une autre extension ayant pour but de renforcer la vie privée. Cette dernière nettoie les URL des informations superflues qui pourraient donner des informations aux sites quant au chemin que l'utilisateur a emprunté dans sa navigation.

Dans les paramètres de cette application, je désactive l'option `Filtrer ETag`. En effet, les [ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) sont une composante importante de la mécanique de cache de nos navigateurs, les désactiver revient à rendre cette mécanique inopérante. Ce que je trouve personnellement très dommage.

#### [Decentraleyes](https://addons.mozilla.org/en-US/firefox/addon/decentraleyes/)

![Decentraleyes logo](https://medias.flavien.io/articles/firefox/decentraleyes.webp)

Avec les extensions précédentes, nous pouvons bloquer de manière assez efficace les différents composants d'un site web ayant pour vocation de nous traquer. Cependant, parmi toutes les méthodes pouvant être utilisées pour nous suivre à la trace, on peut trouver les CDN. Un CDN est un site mettant à la disposition d'autres sites différents contenus tels que des scripts JavaScript, des polices de caractères... Les systèmes de tracking peuvent donc profiter du fait que les clients vont envoyer des requêtes pour récupérer ces contenus afin de savoir sur quel site nous sommes. Le problème, c'est que ces contenus sont bien souvent nécessaires au bon fonctionnement d'un site. Par exemple, de nombreux sites ont besoin de jQuery pour fonctionner, ce dernier étant généralement délivré à travers des CDN. L'extension Decentraleyes va simplement régler le problème en téléchargeant les scripts délivrés par les CDN sur la machine de l'utilisateur. De cette manière, le nombre de requêtes transmises aux CDN se trouve grandement réduit. De plus, pour les connexions limitées, ceci a aussi pour avantage d'éviter les retéléchargements inutiles de contenus que l'on a déjà. Ce qui permet d'économiser de la bande passante.

#### [Random User-Agent](https://addons.mozilla.org/en-US/firefox/addon/random_user_agent/)

![Random User-Agent logo](https://medias.flavien.io/articles/firefox/random_user_agent.webp)

Lorsqu'un navigateur envoie une requête à un site web, il envoie dans sa requête un User-Agent. Cette information permet au site de savoir quel navigateur est utilisé, sa version et éventuellement le système d'exploitation qui le porte.

Grâce à cette extension, cette information est falsifiée et régénérée toutes les 10 minutes. Ainsi, les traqueurs peuvent très bien croire que vous utilisez Google Chrome sur un Mac alors que vous utilisez en réalité Firefox sur un Windows. Cela vous rend plus difficile à suivre et protège donc un peu plus votre anonymat.

Cependant, cette application peut aussi causer quelques petits désagréments. Par exemple, si on pense que vous êtes sur Internet Explorer ou sur un mobile, il est probable que certains sites n'activent pas les mêmes options. Je vous conseille donc de configurer la liste des User-Agents que l'application peut utiliser et de ne cocher que les "Firefox", "Chrome" et "Edge" sur "Windows", "Mac" et "Linux". Il s'agit des combinaisons les plus supportées et qui, en général, posent le moins de problèmes.

![Configuration de Random User-Agent](https://medias.flavien.io/articles/firefox/random_user_agent-configuration.webp)

Cette extension peut également poser quelques problèmes sur des sites de téléchargement, où l’on va nous proposer de télécharger les versions "Mac" ou "Linux" de nos applications favorites.

#### [Auto Tab Discard](https://addons.mozilla.org/en-US/firefox/addon/auto-tab-discard/)

![Auto Tab Discard logo](https://medias.flavien.io/articles/firefox/auto-tab-discard.webp)

Cette extension permet d'optimiser la gestion de la mémoire au sein des onglets qui ne sont pas au premier plan.

### Développement

Les extensions présentées dans cette catégorie ne doivent pas rester installées en permanence. En effet, certaines d'entre elles peuvent fortement dégrader les performances du navigateur et donc nuire à l'expérience utilisateur.

#### [Cookie Quick Manager](https://addons.mozilla.org/en-US/firefox/addon/cookie-quick-manager/)

![Cookie Quick Manager logo](https://medias.flavien.io/articles/firefox/cookie-quick-manager.webp)

Un simple gestionnaire de Cookies. Permet de les modifier, de les supprimer ou encore de les sauvegarder.

#### [ModHeader](https://addons.mozilla.org/en-US/firefox/addon/modheader-firefox/)

![Cookie Quick Manager logo](https://medias.flavien.io/articles/firefox/modheader.webp)

Permet de rajouter des informations dans l’en-tête des requêtes envoyées par le navigateur.

#### [Shodan](https://addons.mozilla.org/en-US/firefox/addon/shodan-addon/)

![Shodan logo](https://medias.flavien.io/articles/firefox/shodan.webp)

[Shodan](https://www.shodan.io/) est un site référençant, pour chaque site et adresses IP accessibles depuis Internet, les services exposés. De rapides tests de sécurité sont également effectués (tests de mot de passe par défaut en fonction de l'équipement, version d'un service possédant une vulnérabilité de sécurité référencée, mauvaise configuration...).

Cette extension permet donc tout simplement d'accéder à toutes ces informations sur les sites que nous visitons. Elle permet ainsi de contrôler facilement si une infrastructure n'expose pas trop d'informations par rapport à ce qu'elle devrait et que les différents services exposés ne souffrent pas d'anomalies de configuration.

#### [Wappalyzer](https://addons.mozilla.org/en-US/firefox/addon/wappalyzer/)

![GreenIT-Analysis logo](https://medias.flavien.io/articles/firefox/wappalyzer.webp)

[Wappalyzer](https://www.wappalyzer.com/) est une application permettant d'identifier les différentes technologies utilisées pour un site web donné. Que ce soit des technologies ayant servi à le concevoir telles que [WordPress](https://wordpress.com/), [FontAwesome](https://fontawesome.com/) ou [Bootstrap](https://getbootstrap.com/)... Ou encore des technologies serveur telles que [Nginx](https://nginx.org/) ou [Apache HTTPD](https://httpd.apache.org/).

#### [GreenIT-Analysis](https://addons.mozilla.org/en-US/firefox/addon/greenit-analysis/)

![GreenIT-Analysis logo](https://medias.flavien.io/articles/firefox/gereenit-analysis.webp)

Cette application open source permet d'analyser les éléments d'un site web qui ne seraient pas conformes à une démarche d'écoconception. Elle peut donc nous aider afin de savoir que regarder pour améliorer les performances de son site et réduire son empreinte énergétique.

### Confort

#### [I still don't care about cookies](https://addons.mozilla.org/en-US/firefox/addon/istilldontcareaboutcookies/)

![I still don't care about cookies logo](https://medias.flavien.io/articles/firefox/i-still-dont-care-about-cookies.webp)

Cette application est un fork en open source de [I don't care about cookies](https://addons.mozilla.org/en-US/firefox/addon/i-dont-care-about-cookies/) qui avait été racheté par Avast.

Si vous en avez marre de tous les bandeaux "Acceptez les cookies", cette application se charge automatiquement de cliquer sur "J'accepte" (ce n’est pas comme si on nous laisse vraiment le choix d'un autre côté) et de nous cacher les bandeaux en question. Simple et efficace pour une navigation sans encombre.

#### [SponsorBlock](https://addons.mozilla.org/en-US/firefox/addon/sponsorblock/)

![SponsorBlock logo](https://medias.flavien.io/articles/firefox/sponsorblock.webp)

Si les AdBlocks permettent de bloquer les pubs hors des vidéos sur YouTube, il est beaucoup plus difficile de bloquer les placements de produit pour "NordVPN" ou autre "RAID: Shadow Legends" au sein même des vidéos. C'est pourtant bien ce que fait cette extension grâce à une communauté d'utilisateurs motivés qui taguent les moments des vidéos durant lesquelles des placements de produits sont effectués et permettent donc aux autres utilisateurs de sauter ces moments.
