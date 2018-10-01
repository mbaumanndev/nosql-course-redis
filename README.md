NoSQL Course Redis
==================

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Introduction](#introduction)
- [Présentation de Redis](#pr%C3%A9sentation-de-redis)
- [Présentation de Docker](#pr%C3%A9sentation-de-docker)
- [Lancement de Docker](#lancement-de-docker)
- [Attaquons Redis](#attaquons-redis)
  - [Les types de données](#les-types-de-donn%C3%A9es)
  - [Les chaînes](#les-cha%C3%AEnes)
  - [Les hashs](#les-hashs)
  - [Les listes](#les-listes)
  - [Les sets](#les-sets)
  - [Opérations sur les clés](#op%C3%A9rations-sur-les-cl%C3%A9s)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Introduction
------------

Le but de ce premier TP va être de se familliariser avec une partie de l'environement que l'on va utiliser au cours de ce cours.

Nous allons donc introduire ici deux concepts :
- Redis, un SGBD NoSQL très répendu et doté de fonctionnalités de programation répartie,
- Docker, un environement de virtualisation de conteneurs applicatifs.

Présentation de Redis
---------------------

Redis est un SGBD NoSQL Open Source orienté stockage clé-valeur. Il a trois particularités importantes :
- La base de données est entièrement gérée en mémoire, le disque n'est utilisé que pour la persistance,
- Il possède un jeu de types de données relativement riche comparé aux autres SGBD orientés stockage clé-valeur,
- Redis peut répliquer ses données sur de nombreux serveurs secondaires.

En plus de ces particularités, il possède quelques avantages :
- Il est très rapide avec 110000 insertions à la seconde et 81000 lectures à la seconde,
- Il supporte de types de données communs, comme des set, des hash, des list,
- Les opérations sont atomiques, en cas d'accès concurents, Redis conservera la valeur mise à jour,
- Il répond à d'autres besoins que du stockage clé-valeur, et peut être utilisé pour faire du message-queueing grâce à son support Pub/Sub natif, de la gestion de données temporaires ou à courte durée de vie comme des sessions utilisateur sur un site web, des opérations de comptage, etc.

Présentation de Docker
----------------------

Docker est un logiciel libre permettant d'automatiser le déploiement d'applications dans des conteneurs logiciels. Il nous permet de faire abstraction du système hôte pour l'éxécution d'un programme donné, d'isoler le programme du système hôte, de faire des applications réparties plus facilement scalable.

Dans le cadre de ce cours, Docker va nous permettre de contourner les restrictions des postes de l'IUT afin de pouvoir éxécuter n'importe quel SGBD NoSQL sans avoir à les installer ou configurer.

Lancement de Docker
-------------------

Notre projet nécéssite le lancement de plusieurs conteneurs :
- Un conteneur redis en mode SGBD (redis dans le fichier de configuration)
- Un conteneur redis en mode cli (rcli dans le fichier de configuration)
- Un conteneur readis (PHP) pour monitorer notre serveur redis
- Un conteneur phpRedisAdmin (PHP) pour interagir avec redis

Docker nous permet de définir des dépendance entre nos conteneurs, ainsi, nos conteneur PHP et notre conteneur redis CLI dépendent tout deux du conteneur redis SGBD.

Afin de lancer notre environement de développement, nous devons éxécuter la commande suivante :

```sh
docker-compose up -d
```

Cette commande va monter notre environement en téléchargeant les conteneurs directement depuis les dépôts de Docker et en démarrant et configurant ceux-ci.

Nous allons maintenant pouvoir nous connecter à notre conteneur redis CLI pour éxécuter des commandes sur notre conteneur redis SGBD par le biais de la commande suivante :

```sh
docker-compose run rcli
```

Nous sommes ainsi connectés dans notre conteneur redis CLI directement dans l'invite de commande de redis. Nous aurons donc uniquement accès [aux commandes mises à disposition par redis](https://redis.io/commands).

À la fin de la séance, pensez à éteindre vos conteneurs avec la commande suivante :

```
docker-compose down
```

Attaquons Redis
---------------

Nous allons maintenant pouvoir nous connecter à notre conteneur redis CLI pour éxécuter des commandes sur notre conteneur redis SGBD par le biais de la commande suivante :

```sh
docker-compose run rcli
```

Nous sommes ainsi connectés dans notre conteneur redis CLI directement dans l'invite de commande de redis. Nous aurons donc uniquement accès [aux commandes mises à disposition par redis](https://redis.io/commands).

### Les types de données

Redis nous propose différents types de données que l'on peut stocker :
- Des chaînes de caractères (binary safe),
- Des listes,
- Des sets,
- Des sorted sets,
- Des hashs,
- Des tableaux de bits.

Nous nous intéresserons qu'aux chaînes, aux listes, aux hashs et aux sets.

### Les chaînes

Redis nous permet de stocker en premier lieu des châines de caractères qui sont binary safe, c'est à dire que leur taille n'est pas délimitée par un caractère particulier. Nos clés comme nos valeurs peuvent contenir des caractères spéciaux. Vous trouverez l'ensemble des commandes pour les chaînes de caractère dans [la documentation de Redis](https://redis.io/commands#string).

Nous allons commencer par stocker une simple chaîne dans Redis.

```shell
SET clef valeur
```
ou
```shell
SET clef "valeur"
```

En cas de réussite, redis vous répondra la chaîne `OK`.

La commande `SET` nous propose des options sur le comportement de la clé et sur le comportemant de l'insertion :
- On peut définir une expiration en secondes avec `EX` ou en millisecondes avec `PX`. Ex : `SET clef valeur EX 5`.
- On peut définir une condition d'insertion `NX` (définie la clé-valeur seulement si elle n'existe pas) ou `XX` (définie la clé-valeur seulement si elle existe déjà).

```shell
redis:6379> SET clef valeur XX
(nil)
redis:6379> SET clef valeur NX
OK
redis:6379> SET clef valeur NX
(nil)
redis:6379> SET clef valeur2 XX
OK
```

Maintenant que nous savons écrire des chaînes, nous allons les récupérées. Pour celà, Redis nous met à disposition une commande `GET` :

```
GET clef
```

Cette commande ne prend pas de paramètres, elle retourne la valeur en cas de réussite, et `null` (`nil`) en cas d'échec (la clé n'existe pas).

Afin d'effectuer des opérations `SET` et `GET` sur plusieurs clés simultanément, Redis nous offres les commandes `MSET` et `MGET`.

```shell
redis:6379> MSET clef1 valeur1 clef2 valeur2
OK
redis:6379> MGET clef1 clef2
1) "valeur1"
2) "valeur2"
```

En plus de nous permettre de créer des chaînes de caractères et de les récupérées, Redis nous offre des commandes afin de les manipulées directement dans le SGBD.

Les premières commandes auxquelles nous allons nous intéressé sont les commandes `INCR` et `DECR`, permettant respectivement d'incrémenter et décrémenter des valeurs. Bien que l'on parle de chaînes de caractères, si nos valeurs représentent des nombres (qu'il soient entiers ou décimaux), ces commandes vont nous permettre de les manipulées comme des nombre. En cas de réussite, Redis nous retourne la nouvelle valeur.

```shell
redis:6379> SET visites 10
OK
redis:6379> INCR visites
(integer) 11
redis:6379> DECR visites
(integer) 10
```

Les commandes `INCRBY` et `DECRBY` permettent d'avoir un comportement similaire, mais nous imposent de définir le pas.

```shell
redis:6379> INCRBY visites 10
(integer) 20
redis:6379> DECRBY visites 5
(integer) 15
```

Les commandes `SETRANGE` et `GETRANGE` permettent de manipuler des morceaux de valeurs, en définissant ou récupérant partiellement les valeurs.

```shell
redis:6379> SET hello "Hello World"
OK
redis:6379> SETRANGE hello 6 "Redis"
(integer) 11
redis:6379> GET hello
"Hello Redis"
redis:6379> GETRANGE hello 0 5
"Hello "
```

La commande `STRLEN` retourne la longueur d'une valeur, ou 0 si la clé n'existe pas.

```shell
redis:6379> SET hello "Hello World"
OK
redis:6379> STRLEN hello
(integer) 11
```

La commande `APPEND` permet d'ajouter une chaîne de caractères à la fin d'une valeur.

```shell
redis:6379> SET hello "Hello"
OK
redis:6379> APPEND hello " World !"
(integer) 13
redis:6379> GET hello
"Hello World !"
```

Contrairement à `SET`, `MSET` ne permet pas de définir des conditions d'expiration ou d'écriture. Toutefois, une commande `MSETNX` est mise à disposition pour éviter d'écraser des clés existantes.

> À vous de jouer !
> 
> En utilisant les commandes vues précédement, créez les clés suivantes :
> - user:1 avec pour valeur "Antoine"
> - user:1:city avec pour valeur "Paris"
> - user:1:age avec pour valeur "19"
> - user:1:activity avec pour valeur "Running" et une expiration dans 10 minutes
> - user:1:hobby avec pour valeur "Course à pieds"
> - user:1:weight avec pour valeur "74"
>
> Récuperez en une fois les clés que vous venez de créer pour city, age et hobby.
>
> Définissez maintenant la clé user:1 avec pour valeur "Thomas" à condition que cette clé n'éxiste pas, puis définissez la clé user:1:city à "Amiens" à condition que cette clé existe.
>
> Notre user a pris un an, et à perdu trois kilos grâce à la course à pieds. À l'aide des commandes Redis adéquates, modifiez ces valeurs. De plus, il fait le trajet régulièrement entre Amiens et Paris. Mettez à jour la valeur de la clé city à "Amiens/Paris" sans l'écraser complètement.
>
> Notre user s'est mis à la course hippique. Sans écraser toute la valeur de la clé hobby, mettez à jour ce dernier. Récupérez la longueur de la clé hobby.
>
> Pour finir, récupérez juste le fragment de valeur "Amiens" de la clé city.

### Les hashs

Notre façon de gérer les données de notre user à l'exercice de la partie précédente n'est pas des plus optimales... En effet, nous avons créer de nombreuses clés à la racine de notre base. Afin d'éviter celà, nous allons utiliser des hashs.

Les hashs permettent de structurer les données sous forme de dictionnaires : à une clé correspond une valeur. Ceci nous permet de définir des "sous-clés" dans nos clés afin de regrouper tout ce qui concerne un object particulier.

Comme pour les chaînes de caractères, la documentation détaille [les commandes disponibles pour les hashs](https://redis.io/commands#hash).

De la même façon que pour les châines, Redis propose des commandes pour écrire et récupérer des hashs. Nous allons pouvoir écrire nos hashs avec `HSET` et les lire avec `HGET`.

```shell
redis:6379> HSET dictionnaire champ valeur
(integer) 1
redis:6379> HGET dictionnaire champ
"valeur"
```

Nous pouvons également définir plusieurs champs du hash grâce à la commande `HMSET` et en récupérer plusieurs avec `HMGET`. Un commande `HSETNX` permet de faire un `HMSET` si le champ n'existe pas.

```shell
redis:6379> HMSET bestiaire chat "Être perfide" chien "Meilleur ami de l'Homme"
OK
redis:6379> HMGET bestiaire chat chien
1) "\xc3\x8atre perfide"
2) "Meilleur ami de l'Homme"
```

Nous allons pouvoir tester l'existance d'un champ dans un hash avec la commande `HEXISTS`, et supprimer des champs d'un hash avec la commande `HDEL`. `HEXISTS` retourne `1` si le champ existe, `0` le cas échéant, et `HDEL` retourne le nombre de champs supprimés.

```shell
redis:6379> HMSET bestiaire chat "Être perfide" chien "Meilleur ami de l'Homme"
OK
edis:6379> HEXISTS bestiaire poisson
(integer) 0
redis:6379> HEXISTS bestiaire chat
(integer) 1
redis:6379> HDEL bestiaire chat chien cheval
(integer) 2
```

Nous avons également à disposition différentes commandes pour récupérer des inforamtions sur nos hashs. La commande `HLEN` retourne le nombre de champs d'un hash, `HKEYS` retourne les noms de champs d'un hash, `HVALS` retourne les valeurs des champs d'un hash, `HGETALL` retourne les champs et valeurs d'un hash.

```shell
redis:6379> HMSET bestiaire chat "Être perfide" chien "Meilleur ami de l'Homme"
OK
redis:6379> HLEN bestiaire
(integer) 2
redis:6379> HKEYS bestiaire
1) "chat"
2) "chien"
redis:6379> HVALS bestiaire
1) "\xc3\x8atre perfide"
2) "Meilleur ami de l'Homme"
redis:6379> HGETALL bestiaire
1) "chat"
2) "\xc3\x8atre perfide"
3) "chien"
4) "Meilleur ami de l'Homme"
```

> À vous de jouer !
> 
> En utilisant les commandes relatives aux hashs, créez les dictionnaires suivants :
> - Un dictionnaire user:2 contenant le nom "Dupont", le prénom "Etienne", un âge à 23
> - Un dictionnaire user:3 contenant le nom "Dupond", le prénom "Charles", un âge à 21
> - `HMSET ordis hp:pavillon "HP Pavillon" hp:stream "HP Stream" hp:envy "HP ENVY" asus:vivo "Vivobook" asus:chrome "Chromebook" asus:zen "Zenbook"`
>
> C'est l'anniversaire de notre user:2, à l'aide de [la documentation](https://redis.io/commands#hash), ajoutez lui un an.
>
> Notre user:3 s'est trompé sur son âge, avec l'aide de [la documentation](https://redis.io/commands#hash), retirez-lui deux ans.
>
> À l'aide de la documentation de la commande [`HSCAN`](https://redis.io/commands/hscan), récupérez dans le dictionnaire `ordis` les ordinateurs dde marque `hp`. Rappel : Par défault, nous travaillons dans la base (cursor) `0`.
>
> Toujours à l'aide de `HSCAN`, récupérez les ordinateurs contenant `vi`.
> Toujours à l'aide de `HSCAN`, récupérez les ordinateurs finissants par `book`. Que remarquez-vous ?

### Les listes

Redis propose un ensemble de commandes pour manipuler des listes. Vous pouvez retrouver ces commandes dans [la documentation](https://redis.io/commands#list).

Tout d'abord, nous pouvons ajouter des valeurs dans nos listes. Comme d'autres commandes pour les listes, il est possible de le faire pour le début de liste (L, comme left) ou pour la fin (R, comme right). Ainsi, nous disposons de deux commandes : `LPUSH` et `RPUSH`.

Pour récupérer des valeurs, nous avons sur le même principe les commandes `LPOP` et `RPOP`. Ces deux commandes enlèvent le dernier élément inséré au bout (gauche ou droit) de la liste.

```shell
redis:6379> LPUSH liste 1 2
(integer) 2
redis:6379> RPUSH liste 3 4
(integer) 4
redis:6379> LPOP liste
"2"
redis:6379>  RPOP liste
"4"
```

On peut également insérer des éléments avec d'autres méthodes, et lire des éléments sans les supprimés de la liste. Nous pouvons parvenir à celà grâce aux commandes `LINSERT`, `LSET` et `LINDEX`.

`LINSERT` nous permet de faire de l'insertion grâce à un pivot, Redis recherche la première occurence de la valeur servant de pivot, et ajoute notre élément avant ou après celui-ci selon les options spécifiées. Cette méthode peut être coûteuse si la valeur pivot est située en fin de liste.

`LSET` nous permet d'insérer un élement à un index particulier de la lsite.

`LINDEX` à l'inverse permet de lire une valeur à un index particulier de la liste.

```shell
redis:6379> LPUSH liste 1 2 3 5 6
(integer) 5
redis:6379> LINSERT liste AFTER 5 4
(integer) 6
redis:6379> LINDEX liste 1
"5"
redis:6379> LINDEX liste 2
"4"
```
équivalent avec `RPUSH`
```shell
redis:6379> RPUSH liste 1 2 3 5 6
(integer) 5
redis:6379> LINSERT liste BEFORE 5 4
(integer) 6
redis:6379> LINDEX liste 3
"4"
```

Redis nous permet de supprimer des éléments d'une liste avec la commande `LREM`. Cette commande prend en premier paramètre la liste, en second le nombre `x` d'éléments à supprimer (si `x`> 0, on en supprime `x` en partant du début, si `x` < 0, on en supprime `x` en partant de la fin, si `x` = 0, on supprime tout les éléments correspondants), et en dernier paramètre, les valeurs à supprimer.

```shell
redis:6379> RPUSH liste 1 2 1 2 3 1 2 3 4
(integer) 9
redis:6379> LREM liste 2 1
(integer) 1
redis:6379> LRANGE liste 0 -1
1) "2"
2) "2"
3) "3"
4) "1"
5) "2"
6) "3"
7) "4"
```

> À vous de jouer !
>
> Sans regarder la documentation, à l'aide des explications précédentes, expliquer ce que fait la commande `RPOPLPUSH`, prenant en paramètres une liste source et une liste destination.
>
> En lançant à nouveau redis cli dans un nouveau terminal (`docker-compose run rcli` et à l'aide de [la documentation](https://redis.io/commands#list)), expliquez et expérimentez les commandes `BLPOP`, `BRPOP` et `BRPOPLPUSH`. Selon vous, quel peut-être l'intérêt de telles commandes ?

### Les sets

Nous verrons cette partie à la prochaine séance, patience !

### Opérations sur les clés

Nous verrons cette partie à la prochaine séance, patience !
