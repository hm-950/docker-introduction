# Réponses : volumes-exercices

## Exercice 1 : perte des données sans volume

- Résultats : le premier conteneur affiche `Donnee-temporaire`, le second affiche `ABSENT`.
- Pourquoi la même image ne suffit pas : l'image ne change jamais. Le fichier a été
  écrit dans la couche modifiable du premier conteneur, qui a disparu avec lui.
  Le second conteneur repart de l'image d'origine, sans le fichier.
- Quand c'est un problème : pour une base de données, des fichiers envoyés par les
  utilisateurs ou des logs à garder. Tout serait perdu à chaque suppression ou mise
  à jour du conteneur.

## Exercice 2 : créer et inspecter un volume nommé

- Valeur de `Name` : `tp-volume`.
- `Driver` : la façon dont le volume est stocké ; `local` = sur le disque de la
  machine Docker. `Mountpoint` : le dossier où Docker range les fichiers du volume.
  `Scope` : la portée ; `local` = visible uniquement sur cette machine.
- Sur Docker Desktop, le `Mountpoint` est un chemin Linux situé dans la machine
  virtuelle de Docker (WSL), pas sur le disque Windows : il n'apparaît donc pas
  dans l'Explorateur.
- Le volume n'est lié à aucun conteneur : il existe seul et peut être monté
  dans n'importe quel conteneur.

## Exercice 3 : vérifier la persistance avec un volume

- Le fichier subsiste-t-il ? Oui. Après `docker rm tp-ecrivain`, un nouveau conteneur
  montant `tp-volume` affiche toujours `Donnee-persistante`.
- Différence avec l'exercice 1 : à l'exercice 1 j'écrivais dans `/tmp`, c'est-à-dire
  dans la couche modifiable du conteneur, détruite avec lui. Ici `/data` est un volume
  géré par Docker, stocké en dehors du conteneur.
- Qui porte la durée de vie : le **volume**. Le conteneur n'est qu'un utilisateur
  temporaire des données ; il peut être supprimé et remplacé sans les perdre.
- Remarque : `docker volume inspect tp-volume` n'indique aucun conteneur. Le volume
  ne sait pas qui l'utilise, ce qui confirme son indépendance.

## Exercice 4 : partager un volume entre deux conteneurs

- Les deux conteneurs voient-ils les mêmes fichiers ? Oui. Le lecteur a affiché
  `Message-du-conteneur-A`, puis `Message-modifie` après modification par l'écrivain,
  sans redémarrage ni recopie : il n'y a qu'un seul stockage derrière les deux montages.
- Pourquoi `readonly` pour le lecteur : parce qu'il n'a besoin que de lire. C'est le
  principe du moindre privilège, qui évite qu'un conteneur abîme par erreur des données
  dont il n'est pas responsable. J'ai vérifié : une tentative d'écriture depuis le
  montage en lecture seule échoue avec `can't create /data/interdit.txt: Read-only file system`.
- Risques si deux applications écrivaient en même temps : Docker ne pose aucun verrou.
  On s'expose à des écritures qui s'écrasent (la dernière gagne), à des fichiers
  à moitié écrits et à des conditions de course. Pour une base de données, deux
  instances sur le même volume corrompraient les fichiers de données.

## Exercice 5 : Bind Mounts avec Nginx

- Pourquoi la page change sans reconstruire l'image : le Bind Mount fait pointer
  `/usr/share/nginx/html` directement sur mon dossier `site-web`. Nginx lit donc mes
  fichiers en direct ; ils n'ont jamais été copiés dans l'image. `Version 1` est devenue
  `Version 2` après un simple rafraîchissement.
- Valeur de `Type` dans `docker inspect` : `bind`, avec `"RW": false` à cause du
  `readonly`, et `Source` =
  `C:\Users\User\docker-introduction\volumes-exercices\site-web`.
- Ce que signifie `readonly` : le conteneur peut lire le dossier monté mais ne peut rien
  y écrire ni y supprimer.
- Les fichiers locaux sont-ils supprimés avec le conteneur ? Non. Après
  `docker rm tp-nginx-bind`, `index.html` est toujours dans `site-web` avec la
  Version 2. Le dossier appartient à ma machine, pas au conteneur.

## Exercice 6 : comparer Bind Mount et volume nommé

Sans Bind Mount, `http://localhost:8086` affiche la page « Welcome to nginx! » de
l'image, et `/usr/share/nginx/html` contient `50x.html` et `index.html`, les fichiers
d'origine de l'image. Ma page personnalisée n'apparaît pas : rien ne la relie au conteneur.

| Situation | Volume nommé, Bind Mount ou aucun montage ? | Justification |
| --- | --- | --- |
| Faire persister des fichiers de base de données | Volume nommé | Docker gère l'emplacement, les droits et les performances ; c'est l'usage recommandé pour des données applicatives |
| Voir instantanément les modifications d'un site HTML local | Bind Mount | mon dossier de travail est monté tel quel, donc aucune reconstruction d'image pendant le développement |
| Exécuter une application temporaire sans données à conserver | Aucun montage | la couche modifiable suffit, et tout disparaît avec le conteneur, ce qui est voulu |
| Conserver des données lors de la recréation d'un conteneur | Volume nommé | sa durée de vie est indépendante de celle du conteneur |

- Chemin géré par Docker ou chemin choisi sur l'hôte : avec un volume nommé, je donne
  seulement un nom (`tp-volume`) et Docker décide de l'emplacement
  (`/var/lib/docker/volumes/tp-volume/_data`). C'est portable : la même commande
  fonctionne sur Windows, macOS ou Linux. Avec un Bind Mount, j'impose un chemin exact
  de ma machine, qui n'existera pas forcément ailleurs.
- Les deux permettent-ils la persistance ? Oui, dans les deux cas les données survivent
  au conteneur. La différence porte sur qui possède l'emplacement, pas sur la persistance.
- Persistance et sauvegarde ne sont pas la même chose : la persistance veut seulement dire
  que les données survivent au conteneur. Il n'en existe toujours qu'un seul exemplaire,
  au même endroit. Un `docker volume rm`, un `DROP TABLE`, une corruption ou une panne de
  disque et tout est perdu. Une sauvegarde est une copie séparée, datée et restaurable,
  stockée ailleurs.

## Exercice 7 : supprimer des volumes sans affecter d'autres projets

- `important.txt` est-il encore là après recréation ? Non. `docker run ... ls -la /data`
  montre un dossier vide. Supprimer le volume a effacé son contenu ; recréer un volume
  du même nom ne fournit qu'un contenant neuf et vide. Le nom ne ramène pas les données.
- Pourquoi `docker volume prune --all` est risqué : il supprime tous les volumes nommés
  qu'aucun conteneur n'utilise, pas seulement ceux du TP. Mes volumes `drupal_*` entrent
  exactement dans ce cas quand leurs conteneurs sont arrêtés ou supprimés : la commande
  les effacerait aussi, sans retour possible. Je supprime donc toujours un volume par
  son nom.
- Avant de supprimer le volume d'une application de production : faire une sauvegarde et
  vérifier qu'elle se restaure vraiment, repérer qui utilise le volume
  (`docker ps -a --filter volume=...`), obtenir une validation et passer par une fenêtre
  de maintenance, puis conserver la sauvegarde un certain temps avant de considérer la
  suppression comme définitive.

## Atelier : garantir la persistance d'une base PostgreSQL

### Missions 1 à 4 : ce que j'ai obtenu

- Mission 1 : volume `tp-postgres-data` créé, driver `local`,
  `Mountpoint` = `/var/lib/docker/volumes/tp-postgres-data/_data`.
- Mission 2 : conteneur `tp-postgres` lancé depuis `postgres:17` avec `POSTGRES_USER`,
  `POSTGRES_PASSWORD` et `POSTGRES_DB`, volume monté sur `/var/lib/postgresql/data`.
  `pg_isready` répond `/var/run/postgresql:5432 - accepting connections`, et `docker ps`
  affiche `5432/tcp` sans redirection vers l'hôte : le port n'est pas exposé, tout passe
  par `docker exec`.
- Mission 3 : table `apprenants` créée (`CREATE TABLE`), 3 lignes insérées
  (`INSERT 0 3`), relues avec Alice/Docker, Bilal/DevOps, Chloe/Cloud.
- Mission 4 : conteneur arrêté puis supprimé sans toucher au volume. `docker volume ls`
  montre que `tp-postgres-data` existe toujours. Après recréation du conteneur avec la
  même image, les mêmes variables et le même volume, la requête
  `SELECT * FROM apprenants ORDER BY id;` renvoie **les 3 mêmes lignes**. La maintenance
  n'a rien perdu.

### Mission 5 : analyse

1. **Pourquoi les données sont conservées malgré la suppression du conteneur :** parce que
   PostgreSQL écrit dans `/var/lib/postgresql/data`, qui n'est pas un dossier du conteneur
   mais le point de montage du volume `tp-postgres-data`. Supprimer le conteneur ne retire
   que le processus et sa couche modifiable ; les fichiers de la base restent dans le
   volume, que le nouveau conteneur retrouve en se montant au même endroit.
2. **Avec un nouveau volume vide :** l'image ne trouverait aucune base existante et
   lancerait son initialisation (`initdb`). J'obtiendrais une base `tpdb` neuve, avec
   l'utilisateur `tpuser`, mais **sans la table `apprenants`** : un `SELECT` échouerait
   avec `relation "apprenants" does not exist`. Les anciennes données ne seraient pas
   détruites pour autant, elles resteraient dans l'ancien volume devenu inutilisé.
3. **Pourquoi le volume ne remplace pas une sauvegarde :** il ne contient qu'un seul
   exemplaire vivant des données, au même endroit que la base. Il ne protège ni d'un
   `DROP TABLE` ou d'un `DELETE` malheureux, ni d'une corruption des fichiers, ni d'un
   `docker volume rm`, ni d'une panne du disque de la machine. Il ne garde aucun
   historique : impossible de revenir à l'état d'hier. Une vraie sauvegarde est un
   `pg_dump` ou un `pg_basebackup` planifié, stocké sur un autre support, et dont la
   restauration est testée.
4. **Risques lors d'un changement de version majeure :** le format interne des fichiers de
   données change d'une version majeure à l'autre. Monter un volume écrit par PostgreSQL 17
   dans un conteneur `postgres:18` ne met pas la base à jour : le serveur refuse de démarrer
   avec une erreur du type `database files are incompatible with server`. Il faut une
   migration explicite (`pg_dump` puis restauration dans la nouvelle version, ou
   `pg_upgrade`), après sauvegarde. C'est pourquoi on épingle une version précise
   (`postgres:17`) au lieu de `latest`, qui pourrait changer de majeure sans prévenir.
   Le point de montage attendu peut lui aussi évoluer selon les versions de l'image.
5. **Quand préférer un Bind Mount :** quand j'ai besoin de maîtriser l'emplacement exact
   des fichiers sur l'hôte, par exemple pour placer les données sur un disque dédié plus
   rapide ou plus grand, ou pour que l'outil de sauvegarde de l'entreprise, qui surveille
   un chemin précis, puisse les prendre en charge. C'est aussi la bonne solution pour
   injecter des fichiers de configuration ou des scripts d'initialisation en lecture seule.
   Pour les données d'une base sur Windows ou macOS, le volume nommé reste préférable :
   le Bind Mount passe par une traduction de système de fichiers qui dégrade les
   performances et pose des problèmes de droits.

## Note sur le nettoyage

Je n'ai pas lancé le nettoyage final du README. Les volumes `tp-volume`, `tp-partage`,
`tp-nettoyage` et `tp-postgres-data`, ainsi que le conteneur `tp-postgres`, existent
encore. Je n'ai jamais utilisé `docker volume prune` et n'ai touché à aucune ressource
du projet drupal.
