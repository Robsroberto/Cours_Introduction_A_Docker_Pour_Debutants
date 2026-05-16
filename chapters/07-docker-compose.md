## Orchestration avec Docker Compose : gérer plusieurs conteneurs

Lorsque vous développez des applications modernes, il est rare qu’un seul conteneur suffise. Une application web typique repose souvent sur plusieurs composants : un serveur web, une base de données, un service de mise en cache, et parfois un outil de file d’attente ou de monitoring. Gérer chacun de ces conteneurs manuellement avec des commandes `docker run` longues et complexes devient vite fastidieux et source d’erreurs.

C’est ici que **Docker Compose** entre en jeu.

Docker Compose est un outil puissant qui permet de définir et exécuter des applications composées de plusieurs conteneurs à partir d’un seul fichier de configuration : `docker-compose.yml`. Ce fichier décrit tous les services dont votre application a besoin, leurs dépendances, leurs variables d’environnement, leurs ports exposés et leurs volumes.

### Comprendre le fichier docker-compose.yml

Le fichier `docker-compose.yml` est écrit en **YAML**, un format simple et lisible qui permet d’organiser des données hiérarchiques. Il contient une série de sections qui définissent vos services, réseaux et volumes.

Voici un exemple de structure de base :

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - app
    networks:
      - app-network

  app:
    build: .
    environment:
      - DB_HOST=database
      - REDIS_URL=cache:6379
    networks:
      - app-network

  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password123
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - app-network

  cache:
    image: redis:alpine
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

Analysons ce fichier pas à pas.

- **`version`** : indique la version du format Docker Compose. La version `3.8` est couramment utilisée et offre une bonne compatibilité.
- **`services`** : chaque service correspond à un conteneur. Ici, nous avons `web`, `app`, `database` et `cache`.
- **`image`** : spécifie l’image à utiliser. Elle peut être tirée de Docker Hub ou construite localement avec `build`.
- **`ports`** : mappe les ports du conteneur vers ceux de la machine hôte. Par exemple, `80:80` signifie que le port 80 du conteneur est accessible via le port 80 de votre machine.
- **`environment`** : définit les variables d’environnement nécessaires au service. Très utile pour configurer des paramètres comme les mots de passe ou les URLs.
- **`depends_on`** : indique que le service `web` doit démarrer après le service `app`. Attention : cela ne garantit pas que l’application à l’intérieur du conteneur soit prête, seulement que le conteneur est lancé.
- **`volumes`** : permet de persister les données. Ici, les données MySQL sont sauvegardées dans un volume nommé `db-data`.
- **`networks`** : crée un réseau privé entre les conteneurs, permettant la communication interne via les noms de service (`database`, `cache`, etc.).

### Un exemple concret : un blog WordPress avec MySQL

Imaginons que vous êtes développeur au Sénégal et que vous souhaitez lancer un blog WordPress pour promouvoir une initiative locale. Vous pouvez le faire en quelques minutes avec Docker Compose.

Créez un fichier `docker-compose.yml` avec ce contenu :

```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: password123
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp-content:/var/www/html/wp-content
    depends_on:
      - mysql
    networks:
      - wp-network

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: password123
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - wp-network

volumes:
  wp-content:
  db-data:

networks:
  wp-network:
    driver: bridge
```

Enregistrez ce fichier, puis exécutez dans votre terminal :

```bash
docker-compose up -d
```

Le flag `-d` lance les conteneurs en arrière-plan. Docker Compose lit le fichier, télécharge les images nécessaires, crée les volumes et les réseaux, puis démarre les deux conteneurs.

Ouvrez votre navigateur et rendez-vous sur `http://localhost:8080`. Le setup de WordPress s’affiche ! Vous pouvez maintenant configurer votre blog sans avoir installé Apache, PHP ou MySQL sur votre machine.

### Gestion des dépendances et de la santé des services

L’un des pièges courants avec `depends_on` est de supposer qu’un service est *prêt* juste parce qu’il a démarré. Par exemple, MySQL peut être en cours de démarrage pendant plusieurs secondes. Si WordPress tente de se connecter trop tôt, il échoue.

Pour résoudre cela, Docker Compose propose l’option `healthcheck`. Voici comment l’ajouter au service MySQL :

```yaml
mysql:
  image: mysql:8.0
  environment:
    MYSQL_ROOT_PASSWORD: rootpassword
    MYSQL_DATABASE: wordpress
    MYSQL_USER: wordpress
    MYSQL_PASSWORD: password123
  volumes:
    - db-data:/var/lib/mysql
  healthcheck:
    test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
    timeout: 20s
    retries: 10
  networks:
    - wp-network
```

Avec ce `healthcheck`, Docker vérifie que MySQL est réellement opérationnel. Vous pouvez maintenant modifier le service WordPress pour qu’il attende que MySQL soit sain :

```yaml
wordpress:
  image: wordpress:latest
  ports:
    - "8080:80"
  environment:
    WORDPRESS_DB_HOST: mysql
    WORDPRESS_DB_USER: wordpress
    WORDPRESS_DB_PASSWORD: password123
    WORDPRESS_DB_NAME: wordpress
  volumes:
    - wp-content:/var/www/html/wp-content
  depends_on:
    mysql:
      condition: service_healthy
  networks:
    - wp-network
```

Grâce à `condition: service_healthy`, WordPress ne démarre que lorsque MySQL passe le test de santé.

### Bonnes pratiques pour vos projets

1. **Utilisez des versions d’images spécifiques** : évitez `latest` en production. Préférez `mysql:8.0` ou `nginx:1.21` pour plus de stabilité.
2. **Externalisez les secrets** : ne stockez jamais les mots de passe en clair dans le fichier `docker-compose.yml`. Utilisez des fichiers `.env` :
   ```env
   MYSQL_ROOT_PASSWORD=monmotdepassecomplex
   WORDPRESS_DB_PASSWORD=autremotdepasse
   ```
   Puis référencez-les dans votre fichier :
   ```yaml
   environment:
     MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
   ```
3. **Nommez vos volumes** : cela facilite la sauvegarde et la migration.
4. **Documentez votre configuration** : ajoutez des commentaires dans le fichier YAML si nécessaire, surtout dans un projet d’équipe.

### Automatisation d’une API Node.js + MongoDB + React

Prenons un autre exemple typique : une application full-stack africaine pour la gestion des coopératives agricoles. Elle comprend :

- Un frontend React sur le port 3000
- Une API Node.js sur le port 5000
- Une base MongoDB

Voici le `docker-compose.yml` correspondant :

```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    networks:
      - app-network

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      MONGODB_URI: mongodb://mongodb:27017/coopagri
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - app-network

  mongodb:
    image: mongo:6
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 10s
      retries: 5
    networks:
      - app-network

volumes:
  mongo-data:

networks:
  app-network:
    driver: bridge
```

Avec ce seul fichier, vous orchestrez une application complexe en une seule commande. Cela accélère le développement, favorise la collaboration entre développeurs au Cameroun, au Mali ou au Maroc, et assure que tout le monde travaille dans un environnement identique.

## Points clés

- Docker Compose permet de gérer plusieurs conteneurs avec un seul fichier de configuration.
- Le fichier `docker-compose.yml` décrit les services, leurs dépendances, variables d’environnement, ports, volumes et réseaux.
- Utilisez `depends_on` avec `condition: service_healthy` pour garantir que les services démarrent dans le bon ordre et sont prêts.
- Les volumes permettent de persister les données entre les redémarrages.
- Les fichiers `.env` permettent de séparer les configurations sensibles du code.
- Docker Compose est idéal pour les environnements de développement et de test, mais peut aussi être utilisé en production dans des cas simples.
- Grâce à Docker Compose, vous pouvez monter des applications complètes (WordPress, API + frontend + base) en quelques commandes, ce qui accélère considérablement le workflow des développeurs.