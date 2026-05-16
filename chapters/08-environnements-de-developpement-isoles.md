## Créer des environnements de développement isolés avec Docker

Lorsque vous travaillez sur un projet web ou une application, l’un des défis les plus fréquents est la gestion des environnements : celui de votre machine locale, celui de vos collègues, et celui du serveur de production. Combien de fois avez-vous entendu : *« Ça fonctionne sur mon ordinateur »*, mais pas ailleurs ? Ce problème, connu sous le nom de *"it works on my machine"*, est un gouffre de productivité.

Docker permet de mettre fin à ces incohérences en créant des environnements de développement parfaitement isolés, reproductibles et identiques à l’environnement de production. Ce chapitre vous montre comment exploiter cette puissance pour travailler efficacement, sans craindre les conflits de dépendances ou les configurations incompatibles.

### Pourquoi isoler son environnement de développement ?

Imaginons que vous développez une plateforme de e-commerce pour un entrepreneur sénégalais. Cette application utilise PHP 8.1, MySQL 8, et Redis pour la mise en cache. Parallèlement, vous participez à un autre projet – un système de gestion scolaire au Cameroun – qui nécessite PHP 7.4 et PostgreSQL. Si vous installez ces versions directement sur votre machine, les conflits sont inévitables.

Avec Docker, chaque projet vit dans son propre conteneur, avec ses propres versions de PHP, de base de données, et de toutes les dépendances. Votre machine hôte reste propre, et vous pouvez basculer d’un projet à l’autre sans aucune interférence.

L’isolation offre aussi un autre avantage majeur : la reproductibilité. Lorsqu’un nouveau développeur rejoint l’équipe au Bénin, il peut lancer le projet en une seule commande. Plus besoin de passer des heures à configurer un environnement manuellement.

### Créer un environnement PHP avec Docker

Prenons l’exemple d’une application PHP simple, comme une page de connexion pour un site de vente de produits artisanaux au Maroc. Voici comment créer un environnement complet avec Apache, PHP et MySQL.

Commencez par créer un fichier `docker-compose.yml` à la racine de votre projet :

```yaml
version: '3.8'

services:
  web:
    image: php:8.1-apache
    ports:
      - "8080:80"
    volumes:
      - ./src:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: artisanat_db
      MYSQL_USER: artisanat_user
      MYSQL_PASSWORD: artisanat_pass
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

Dans ce fichier, deux services sont définis :
- `web` : un conteneur basé sur l’image officielle PHP avec Apache.
- `db` : un conteneur MySQL avec une base de données préconfigurée.

Le dossier `./src` sur votre machine est monté dans le conteneur, ce qui vous permet de modifier le code localement tout en le voyant s’exécuter dans l’environnement PHP.

Créez maintenant un fichier `src/index.php` :

```php
<!DOCTYPE html>
<html>
<head>
    <title>Bienvenue sur Artisanat Maroc</title>
</head>
<body>
    <h1>Connexion au site d'artisanat</h1>
    <p>Environnement PHP fonctionnel avec Docker !</p>
    <?php
    $link = mysqli_connect("db", "artisanat_user", "artisanat_pass", "artisanat_db");
    if ($link) {
        echo "<p>Connexion à la base de données réussie.</p>";
    } else {
        echo "<p>Échec de connexion : " . mysqli_connect_error() . "</p>";
    }
    ?>
</body>
</html>
```

Lancez l’ensemble avec la commande :

```bash
docker-compose up -d
```

Ouvrez votre navigateur et rendez-vous sur `http://localhost:8080`. Vous verrez la page s’afficher, avec la confirmation que PHP est bien connecté à la base de données.

### Environnement Python : Application de gestion scolaire

Maintenant, passons à un projet différent : une application Python utilisée pour suivre les notes des élèves dans un collège au Mali. Ce projet utilise Flask, PostgreSQL et Redis.

Créez un nouveau répertoire pour ce projet, avec un `docker-compose.yml` :

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - ./app:/app
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: ecole_db
      POSTGRES_USER: ecole_user
      POSTGRES_PASSWORD: ecole_pass
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  pg_data:
```

Créez un fichier `Dockerfile` dans le même répertoire :

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Et votre fichier `app.py` (simplifié) :

```python
from flask import Flask
import psycopg2
import redis

app = Flask(__name__)

@app.route('/')
def home():
    try:
        conn = psycopg2.connect(
            host="postgres",
            database="ecole_db",
            user="ecole_user",
            password="ecole_pass"
        )
        cache = redis.Redis(host='redis', port=6379)
        cache.set('visits', 1)
        return "Système de gestion scolaire opérationnel ! Visite enregistrée."
    except Exception as e:
        return f"Erreur : {str(e)}"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Un fichier `requirements.txt` contient les dépendances :

```
Flask==2.3.2
psycopg2-binary==2.9.6
redis==4.6.0
```

Lancez l’application :

```bash
docker-compose up --build
```

Rendez-vous sur `http://localhost:5000`. Le message s’affiche, et vous savez que chaque composant fonctionne dans son propre conteneur, sans aucune installation globale sur votre machine.

### Environnement Node.js : Plateforme de livraison au Kenya

Enfin, considérons une application Node.js pour une startup kényane de livraison de nourriture. Elle utilise Express, MongoDB et un front-end en React.

Voici un `docker-compose.yml` adapté :

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "3001:3001"
    environment:
      MONGO_URI: mongodb://mongo:27017/livraison
    depends_on:
      - mongo

  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

volumes:
  mongo_data:
```

Chaque service (`backend`, `frontend`, `mongo`) est isolé. Le backend Node.js communique avec MongoDB via le nom de service `mongo`, et le frontend interagit avec le backend via `http://localhost:3001`.

### Avantages concrets pour les développeurs africains

Dans de nombreux contextes africains, les équipes de développement sont parfois dispersées géographiquement, travaillant depuis différents pays avec des configurations techniques variées. Docker standardise l’environnement de travail, ce qui réduit drastiquement le temps d’intégration des nouveaux membres.

De plus, dans les régions où la bande passante est limitée, Docker permet de ne télécharger qu’une fois les images de base (comme `node:18`, `python:3.9`, etc.), puis de les réutiliser pour plusieurs projets.

Enfin, lorsque vient le moment de passer en production, souvent sur des serveurs cloud comme AWS ou DigitalOcean, l’environnement local reproduit fidèlement le serveur. Cela réduit les erreurs de déploiement et accélère le time-to-market.

---

## Points clés

- Docker permet de créer des environnements de développement **isolés**, **reproductibles** et **fidèles à la production**.
- Chaque projet peut utiliser des versions spécifiques de langages, bases de données et dépendances, sans conflit avec d’autres projets.
- **Docker Compose** est l’outil idéal pour orchestrer plusieurs conteneurs (application, base de données, cache, etc.) dans un même projet.
- Des exemples concrets comme une **plateforme e-commerce**, un **système de gestion scolaire** ou une **application de livraison** montrent que Docker s’adapte à des besoins variés, même dans des contextes africains.
- L’utilisation de Docker améliore la **collaboration en équipe**, car tous les développeurs travaillent dans le même environnement.
- Il **préserve la machine hôte** de l’installation de multiples versions de logiciels et réduit les bugs liés aux différences de configuration.
- Grâce à Docker, passer du développement local à la production devient plus fluide, plus rapide, et plus fiable.

En maîtrisant la création d’environnements isolés, vous gagnez en autonomie, en professionnalisme, et en efficacité – des atouts essentiels pour réussir dans l’écosystème numérique africain en pleine expansion.