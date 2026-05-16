## Orchestration avec Docker Compose : Gérer plusieurs conteneurs

Travailler avec un seul conteneur est utile, mais les applications modernes sont rarement composées d’un seul service. Un site web typique, par exemple, combine un serveur web (comme Nginx ou Apache), une base de données (comme MySQL ou PostgreSQL) et souvent un backend (comme une API en Node.js ou Python). Gérer ces conteneurs un par un devient rapidement complexe, fastidieux, et sujet aux erreurs.

C’est ici que **Docker Compose** entre en jeu. Cet outil, fourni avec Docker, permet de **définir, configurer et exécuter des applications multi-conteneurs** à l’aide d’un seul fichier de configuration : `docker-compose.yml`. Grâce à lui, vous pouvez lancer toute votre application avec une seule commande : `docker-compose up`.

### Comprendre le fichier docker-compose.yml

Le cœur de Docker Compose est le fichier `docker-compose.yml`. Il est écrit en **YAML** (YAML Ain’t Markup Language), un format lisible et structuré. Ce fichier décrit tous les services qui composent votre application, ainsi que leurs dépendances, réseaux, volumes, et variables d’environnement.

Voici une structure typique :

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password123
      MYSQL_DATABASE: app_db
    volumes:
      - db_data:/var/lib/mysql

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      - db

volumes:
  db_data:
```

Analysons chaque section.

### Les services : les briques de votre application

La section `services` définit chaque conteneur qui sera lancé. Chaque service correspond à un conteneur.

- **`web`** : Utilise l’image `nginx:alpine`, expose le port 80, et monte un dossier local (`./html`) dans le conteneur. Utile pour servir un site statique.
- **`db`** : Lance une base MySQL. On définit des variables d’environnement pour configurer le mot de passe et la base de données par défaut.
- **`backend`** : Construit une image à partir d’un Dockerfile situé dans `./backend`. Cela permet de personnaliser l’application (par exemple, une API Node.js).

La directive `depends_on` indique que le service `backend` dépend de `db`. Cela signifie que Docker Compose lancera d’abord la base de données avant le backend. Attention : cela ne garantit pas que la base est **prête** à recevoir des connexions, seulement qu’elle a démarré.

### Les volumes : persister les données

Dans l’exemple, deux types de volumes sont utilisés.

Le premier est un **volume nommé** (`db_data`). Il est déclaré dans la section `volumes` en bas du fichier. Docker gère son emplacement sur le disque, ce qui garantit que les données de la base MySQL persistent même si le conteneur est supprimé.

Le second est un **montage de dossier local** (`./html:/usr/share/nginx/html`). Cela permet de développer localement : tout changement dans le dossier `html` sur votre machine est immédiatement visible dans le conteneur.

Exemple concret : imaginez un développeur à Abidjan qui travaille sur un site d’e-commerce. Il modifie un fichier HTML dans son éditeur. Grâce au volume, le navigateur affiche la version mise à jour sans avoir à reconstruire l’image.

### Les réseaux : connecter les conteneurs

Par défaut, Docker Compose crée un **réseau privé** pour tous les services du fichier. Cela signifie que chaque conteneur peut communiquer avec les autres en utilisant leur nom de service comme nom d’hôte.

Par exemple, depuis le service `backend`, vous pouvez vous connecter à la base de données via l’URL : `mysql://db:3306/app_db`. Le nom `db` est automatiquement résolu par Docker.

Cela simplifie énormément la configuration : pas besoin de gérer des adresses IP statiques ou des ports exposés publiquement. C’est idéal pour un environnement de développement sécurisé et isolé.

Si besoin, vous pouvez définir des réseaux personnalisés :

```yaml
networks:
  app-network:
    driver: bridge
```

Puis attribuer un réseau à un service :

```yaml
services:
  web:
    image: nginx
    networks:
      - app-network
```

### Démarrer, arrêter et inspecter l’application

Une fois le fichier `docker-compose.yml` prêt, place à l’action.

Depuis le répertoire contenant le fichier, exécutez :

```bash
docker-compose up
```

Cela construit les images si nécessaire (comme pour le service `backend`), télécharge les images manquantes (comme `mysql:8.0`), crée les volumes, les réseaux, puis démarre tous les conteneurs.

Pour lancer en arrière-plan, ajoutez `-d` :

```bash
docker-compose up -d
```

Pour arrêter tous les services :

```bash
docker-compose down
```

Cette commande arrête les conteneurs et les supprime. Les volumes nommés **ne sont pas supprimés** par défaut, donc vos données de base restent intactes.

Pour arrêter **et** supprimer les volumes (attention, perte de données !) :

```bash
docker-compose down -v
```

Vous pouvez aussi inspecter l’état des services :

```bash
docker-compose ps
```

Cela affiche tous les conteneurs en cours d’exécution, leur statut, et les ports exposés.

Besoin de voir les logs en temps réel ?

```bash
docker-compose logs -f
```

Le flag `-f` suit les logs, comme `tail -f`. Très utile pour déboguer une API qui ne se connecte pas à la base.

### Exemple concret : Une application de gestion scolaire

Imaginons un développeur à Dakar qui crée une application pour suivre les élèves d’une école. L’application comprend :

- Un frontend React (dans un conteneur Nginx)
- Une API en Python (Flask)
- Une base PostgreSQL

Voici un `docker-compose.yml` adapté :

```yaml
version: '3.8'

services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./frontend/build:/usr/share/nginx/html

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/school_db
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: school_db
    volumes:
      - pg_data:/var/lib/postgresql/data

volumes:
  pg_data:
```

Avec ce fichier, le développeur peut :

1. Construire son backend avec un `Dockerfile` dans `./backend`
2. Générer son frontend React et placer le build dans `./frontend/build`
3. Lancer toute l’application avec `docker-compose up`

Plus besoin de configurer manuellement chaque service. En cas de problème, il relance tout avec `down` puis `up`.

### Bonnes pratiques avec Docker Compose

- **Toujours versionner votre `docker-compose.yml`** : Il fait partie du code source. Toute l’équipe peut reproduire l’environnement exact.
- **Utilisez des variables d’environnement** : Pour les mots de passe ou les URLs, utilisez un fichier `.env` :

  ```env
  DB_PASSWORD=secret123
  ```

  Puis dans le `docker-compose.yml` :

  ```yaml
  environment:
    MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
  ```

- **Nommez clairement vos services** : Évitez `service1`, `service2`. Préférez `api`, `cache`, `worker`.
- **Limitez les ressources si nécessaire** :

  ```yaml
  backend:
    image: myapp
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
  ```

Cela évite qu’un seul conteneur monopolise tout le serveur.

## Points clés

- Docker Compose permet de gérer **plusieurs conteneurs** comme une seule application.
- Le fichier `docker-compose.yml` décrit les services, volumes, réseaux et variables.
- Les conteneurs communiquent entre eux via un **réseau privé** automatique, en utilisant leur nom de service comme hôte.
- Les **volumes nommés** conservent les données même après l’arrêt des conteneurs.
- Les commandes `up`, `down`, `ps` et `logs` simplifient grandement le contrôle de l’application.
- Un seul fichier de configuration remplace des dizaines de commandes Docker manuelles.
- Docker Compose est idéal pour les environnements de développement, mais peut aussi être utilisé en production avec précaution.

Avec Docker Compose, vous passez d’un outil de conteneurisation à un outil d’**orchestration légère**, essentiel pour construire des applications modernes, même avec des ressources limitées. C’est un levier puissant pour les développeurs africains qui veulent innover rapidement, sans infrastructure lourde.