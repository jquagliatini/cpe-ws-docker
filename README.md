# CPE Workshop: Docker

Pour la préparation du workshop il est nécessaire d'avoir installé Docker au préalable.
Le site [docs.docker.com](https://docs.docker.com), fournit toutes les procédures nécessaires
quelque soit votre système d'exploitation.

Les éléments requis sont:
* [docker engine :arrow_upper_right:](https://docs.docker.com/engine/installation/) 
* [docker compose :arrow_upper_right:](https://docs.docker.com/compose/install/)

Si à l'issue de l'installation vous êtes capables de lancer le hello world, c'est l'essentiel, ce 
n'est pas la peine de chercher plus loin.

Une clef USB sera passée pour installer les images nécessaires et éviter de compter sur le réseau.
pour ce faire récupérez le dossier cpe-ws-docker sur votre ordinateur et lancer un terminal dans
le dossier (**sur votre ordinateur !**). Une fois que c'est fait vous pouvez lancer un

```shell
$ cd ~/Documents/src/cpe-ws-docker/
$ sudo docker load -i images.tar
``` 

sinon vous pouvez les installer à l'avance:

+ `sudo docker pull node:7.4.0-alpine`
+ `sudo docker pull mariadb:10.1.21`

## But

Sur ce commit application utilise 2 containers, un contenant notre application nodejs
et l'autre la base de données [MariaDB][mariadb-image].

À l'aide de la [documentation de l'image][mariadb-image] et des scripts `docker/common.sh` et
`docker/db/build.sh`, il va vous falloir rédiger un Dockerfile permettant de créer un container
autonome qui pourra communiquer avec le container applicatif (ici `adjectives`).

La requête que l'on veux exécuter au démarrage du container :

```sql
CREATE TABLE IF NOT EXISTS adjectives (
    id int NOT NULL PRIMARY KEY AUTO_INCREMENT,
    word varchar(255) NOT NULL
);

INSERT INTO adjectives (word) VALUES ('great');
INSERT INTO adjectives (word) VALUES ('boring');
INSERT INTO adjectives (word) VALUES ('beautiful');
INSERT INTO adjectives (word) VALUES ('mesmerizing');
```

L'idée est que notre application est maintenant capable d'utiliser les variables d'environnement que
l'on a définit pour communiquer avec la BDD.
Petite astuce vous noterez l'attribut `--link` lors du run de l'application. 
Ceci permet aux containers de se retrouver par leur noms. On verra plus tard comment aller encore
plus loin avec ce mécanisme (link est plus ou moins déprécié).

*INDICE*: Cette partie est plus ou moins une question piège, à vous de trouver pourquoi.

[mariadb-image]: https://hub.docker.com/_/mariadb/ 

## Build automatisé

### Application

Pour builder l'API REST dans un container il suffit de lancer:

```shell
$ sudo ./build.sh
```

Lors du premier lancement docker va se charger d'aller récupérer les images indiquées dans le
`Dockerfile` sur [hub.docker.com](https://hub.docker.com/), ou les récupérer sur votre ordinateur si
elle a déjà été téléchargée (voir ci-dessus).

Le script se contente de créer un nom pour l'image qu'il va builder et de lancer les 2 commandes
nécessaire pour builder et lancer.
À but pédagogique je vous invite à aller le visualiser.

Si vous lancer le script plusieurs fois, vous risquez de vous retrouver avec pas mal d'images
parasites.
Vous pouver alors arrêter l'application qui tourne (ce qui va supprimer l'image actuellement
utilisée) à l'aide d'un bon vieux `Ctrl+C`, et lancer 

```shell
$ sudo ./build.sh clean
```

## Application - branche `nouns`

L'application que l'on va utiliser est un petit serveur nodejs. Si vous ne connaissez pas
(ou n'aimez pas/détestez) Javascript, ne vous inquiétez pas, on aurait très bien pu faire ça dans
n'importe quel autre langage.
L'application suivra le concept du [cadavre exquis][cadavre-exquis-wiki]: (*Nom*, Adjectif, Verbe,
Complément),chacun contenu dans un service différent.
Ce serveur web est une API REST avec quelques endpoints permettant de modifier une partie du jeu.


[cadavre-exquis-wiki]: https://www.wikiwand.com/fr/Cadavre_exquis_(jeu)

### GET /

+ Response 200 (`application/json`)
Retourne un nom au hasard

```json
{
   "noun": "dog",
    "ip": "192.168.0.33"
}
```

### POST /

Lorsqu'on ajoute un nouveau nom, la réponse consiste à le renvoyer, avec l'adresse
ip du serveur repondant.

+ Body
```json
{
    "noun": "fish"
}
```

+ Reponse 200 (`application/json`)
```json
{
    "noun": "fish",
    "ip": "192.168.0.33"
}
```

## Noms par défaut

Par défaut on utilise la liste suivant de noms (dans la langue de Shakespeare):

+ dog
+ cat
+ Jean-Louis

## Note

On a supprimé les tests qui prenaient trop de temps à maintenir.
