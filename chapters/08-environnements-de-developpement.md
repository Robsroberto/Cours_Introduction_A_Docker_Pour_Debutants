## Dockeriser votre environnement de développement local

Imaginez que vous commencez un nouveau projet web avec un collègue basé à Dakar, tandis que vous êtes à Abidjan. Votre collègue installe tout l’environnement : PHP, MySQL, Apache… mais il utilise une version différente de PHP. Résultat : le code fonctionne chez lui, mais pas chez vous. Ce problème, connu sous le nom de *"ça marche sur ma machine"*, est un cauchemar quotidien pour de nombreux développeurs, surtout lorsqu’on dépend d’outils comme XAMPP ou MAMP, qui imposent une configuration globale sur la machine.

Docker résout ce problème en permettant de **containeriser tout votre environnement de développement**. Chaque projet a ses propres conteneurs : PHP, base de données, serveur web — le tout isolé, reproductible, et portable.

### Remplacer XAMPP, MAMP et les machines virtuelles

Les outils comme XAMPP ou MAMP sont simples à installer, mais ils ont des limites majeures. D’abord, ils installent des services *globalement* sur votre machine. Si vous travaillez sur plusieurs projets, l’un nécessitant PHP 7.4 et l’autre PHP 8.2, vous êtes coincé. De plus, ces outils sont souvent lents à démarrer et peuvent entrer en conflit avec d’autres logiciels.

Les machines virtuelles (VM) offrent plus d’isolation, mais elles sont lourdes : elles virtualisent tout le système, consomment beaucoup de RAM et de CPU, et mettent du temps à démarrer.

Docker, lui, utilise le noyau de votre machine hôte et crée des conteneurs légers, rapides, et spécifiques à chaque projet.  
Un conteneur pour PHP 7.4, un autre pour PHP 8.2 ? Aucun problème. Chaque projet a son propre *environnement complet*, démarré en quelques secondes.

### Exemple concret : un environnement PHP + Apache + MySQL + phpMyAdmin

Prenons un exemple typique : vous développez un site web en PHP avec une base de données MySQL. Traditionnellement, vous auriez installé tout cela manuellement. Avec Docker, vous créez un fichier `docker-compose.yml` pour tout orchestrer.

Voici à quoi ressemble ce fichier :

```yaml
version: '3.8'

services:
  web:
    image: php:8.2-apache
    ports:
      - "8080:80"
    volumes:
      - ./src:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: monsite
      MYSQL_USER: devuser
      MYSQL_PASSWORD: devpass
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: rootpass
    depends_on:
      - db

volumes:
  db_data:
```

Avec ce seul fichier, vous créez :
- Un serveur web Apache avec PHP 8.2
- Une base MySQL configurée automatiquement
- Une interface phpMyAdmin accessible via `http://localhost:8081`

Pour lancer l’ensemble, exécutez :
```bash
docker-compose up -d
```

En moins de 30 secondes, votre environnement est prêt. Vous déposez votre code PHP dans le dossier `src`, et il est automatiquement synchronisé avec le conteneur grâce au *volume*.

### Utiliser des variables d’environnement avec .env

Dans le monde réel, les mots de passe, les clés API ou les versions de logiciels changent selon les environnements (développement, production). Docker permet de gérer cela via un fichier `.env`.

Créez un fichier `.env` à la racine de votre projet :

```env
MYSQL_ROOT_PASSWORD=rootpass123
MYSQL_DATABASE=monprojet_dev
MYSQL_USER=dev
MYSQL_PASSWORD=securepass
PHP_VERSION=8.2
```

Puis modifiez votre `docker-compose.yml` pour utiliser ces variables :

```yaml
db:
  image: mysql:8.0
  environment:
    MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    MYSQL_DATABASE: ${MYSQL_DATABASE}
    MYSQL_USER: ${MYSQL_USER}
    MYSQL_PASSWORD: ${MYSQL_PASSWORD}
  ports:
    - "3306:3306"
  volumes:
    - db_data:/var/lib/mysql
```

Docker lit automatiquement le fichier `.env` s’il est présent. Cela permet de **séparer la configuration du code**, une pratique essentielle en développement moderne.

> ⚠️ **Astuce** : n’oubliez jamais d’ajouter `.env` à votre `.gitignore` pour ne pas exposer vos mots de passe sur GitHub !

### Intégration avec VS Code : développement dans un conteneur

Visual Studio Code (VS Code) propose une extension puissante : **Dev Containers**. Elle permet de *développer directement à l’intérieur d’un conteneur Docker*, comme si vous étiez sur une machine distante.

Voici comment l’utiliser :
1. Installez l’extension "Dev Containers" dans VS Code.
2. Ouvrez votre projet PHP.
3. Cliquez sur l’icône en bas à gauche (><) > "Reopen in Container".
4. VS Code détecte votre `docker-compose.yml` et lance les conteneurs.
5. Vous pouvez maintenant éditer votre code, exécuter des commandes PHP, et même utiliser le terminal du conteneur.

L’avantage ? Vous n’avez **rien à installer localement** : ni PHP, ni Composer, ni MySQL. Tout tourne dans le conteneur. Parfait pour les jeunes développeurs en Afrique qui veulent éviter les installations complexes sur des machines modestes.

### Exemple Node.js + MongoDB + Redis

Docker n’est pas limité à PHP. Prenons un autre cas courant : une application Node.js avec MongoDB pour la base de données et Redis pour la mise en cache.

Voici le `docker-compose.yml` correspondant :

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    depends_on:
      - mongo
      - redis

  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  mongo_data:
```

Et votre `Dockerfile` dans le dossier du projet :

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Lancez avec `docker-compose up --build`, et votre application Node.js démarre, connectée à MongoDB et Redis. Tout cela, sans toucher à votre système.

### Bonnes pratiques pour un développement efficace

1. **Un projet, un dossier, un docker-compose.yml**  
   Chaque projet a sa propre configuration. Pas de conflits, pas de pollution systémique.

2. **Utilisez des images officielles**  
   Privilégiez `mysql`, `redis`, `node`, `php` du Docker Hub. Elles sont maintenues, sécurisées, et bien documentées.

3. **Nommez vos conteneurs et volumes**  
   Ajoutez `container_name: monapp-db` dans votre `docker-compose.yml` pour mieux les identifier.

4. **Nettoyez régulièrement**  
   Les volumes et images s’accumulent. Utilisez `docker system prune -a` de temps en temps pour libérer de l’espace.

5. **Documentez votre environnement**  
   Ajoutez un `README.md` avec les commandes : `docker-compose up`, `docker-compose down`, etc. Cela aide les nouveaux membres de l’équipe.

### Gagner du temps, éviter les bugs

Grâce à Docker, un nouveau développeur qui rejoint votre projet peut :
1. Cloner le dépôt
2. Lancer `docker-compose up`
3. Accéder à l’application dans 30 secondes

Plus besoin de lire des pages de documentation d’installation. Plus de souci de version. Ce gain de temps est précieux, surtout dans des contextes où les ressources sont limitées.

En Afrique, où l’accès à une connexion internet rapide n’est pas toujours garanti, pouvoir travailler hors-ligne avec un environnement complet est un avantage énorme. Docker permet aussi de **former plus rapidement** : un formateur à Bamako peut partager un projet Dockerisé, et ses apprenants à Conakry ou à Lomé peuvent l’exécuter immédiatement.

## Points clés

- Docker remplace avantageusement XAMPP, MAMP et les machines virtuelles en offrant des environnements légers, rapides et isolés.
- Avec `docker-compose.yml`, vous orchestrez plusieurs services (PHP, MySQL, Redis, etc.) en un seul fichier.
- Le fichier `.env` permet de gérer les variables sensibles et spécifiques à chaque environnement, en les gardant hors du code source.
- L’intégration avec VS Code via "Dev Containers" permet de développer entièrement dans un conteneur, sans rien installer localement.
- Chaque projet doit avoir sa propre configuration Docker pour éviter les conflits et assurer la reproductibilité.
- Docker réduit drastiquement le temps d’installation et d’intégration des nouveaux développeurs, un atout majeur pour les équipes africaines en croissance.
- En adoptant Docker, vous standardisez votre workflow, ce qui améliore la collaboration, la qualité du code, et la vitesse de livraison.

Avec ce chapitre, vous avez maintenant les clés pour **dockeriser n’importe quel projet de développement local**. Que vous travailliez sur un site en PHP, une API en Node.js, ou une application full-stack, Docker vous permet de démarrer en 30 secondes, partout dans le monde.