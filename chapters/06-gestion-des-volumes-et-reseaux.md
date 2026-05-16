## Gestion des volumes et des réseaux Docker

Dans un environnement de développement, les données et la communication entre composants sont fondamentales. Un conteneur Docker est par nature éphémère : il peut être arrêté, supprimé ou reconstruit à tout moment. Cela pose un problème crucial : que deviennent les données générées ou modifiées pendant son exécution ? De même, comment permettre à plusieurs conteneurs — par exemple une application web et une base de données — de collaborer efficacement ?

La réponse réside dans deux fonctionnalités essentielles de Docker : les **volumes** et les **réseaux**. Ces outils permettent de gérer la persistance des données et d’assurer une communication fluide entre conteneurs, même s’ils sont isolés par défaut.

### Comprendre les volumes Docker

Un volume Docker est un mécanisme qui permet de stocker des données en dehors du système de fichiers du conteneur. Sans volume, toute modification dans un conteneur (comme l’ajout d’un utilisateur dans une base MySQL) est perdue dès que le conteneur est supprimé. Les volumes résolvent ce problème en conservant les données indépendamment du cycle de vie du conteneur.

Il existe trois types principaux de volumes :

1. **Volumes anonymes**
2. **Volumes nommés**
3. **Bind mounts**

#### Volumes anonymes

Les volumes anonymes sont créés automatiquement par Docker quand un conteneur a besoin d’un emplacement pour stocker des données, sans qu’un nom explicite soit fourni. Ils sont utiles pour des données temporaires ou de cache.

Exemple :  
Lorsque vous lancez un conteneur MySQL sans spécifier de volume nommé, Docker crée un volume anonyme pour le répertoire `/var/lib/mysql`.

```bash
docker run -d --name mysql-temp mysql:8.0
```

Ce volume persiste même si vous supprimez le conteneur, mais il est difficile à réutiliser car il n’a pas de nom clair. Pour le supprimer, il faut utiliser `docker volume prune`.

#### Volumes nommés

Les volumes nommés sont explicites, faciles à gérer et recommandés pour la persistance des données critiques comme les bases de données.

Créons un volume nommé `db-data` pour stocker les données d’une base MySQL :

```bash
docker volume create db-data
docker run -d --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=ecommerce \
  -v db-data:/var/lib/mysql \
  mysql:8.0
```

Maintenant, même si vous supprimez et recréez le conteneur `mysql-db`, les données de la base resteront intactes car elles sont stockées dans le volume `db-data`.

> **Cas d’usage concret** : Imaginez une startup basée à Abidjan qui développe une plateforme de e-commerce. Elle utilise Docker pour son environnement de développement. Grâce à un volume nommé, les données des clients et des commandes ne disparaissent pas entre deux tests du développeur.

#### Bind mounts

Les bind mounts permettent de monter un répertoire **directement depuis l’hôte** (votre machine) dans un conteneur. C’est particulièrement utile en développement, car chaque modification sur votre machine est immédiatement visible dans le conteneur.

Exemple avec une application Node.js :

```bash
docker run -d --name app-dev \
  -v /home/utilisateur/projets/ecommerce:/app \
  -w /app \
  node:18 \
  npm start
```

Ici, le dossier local `/home/utilisateur/projets/ecommerce` est synchronisé avec `/app` dans le conteneur. Vous pouvez modifier un fichier localement, et l’application à l’intérieur du conteneur verra la mise à jour en temps réel.

> **Astuce Empire du Web** : En Afrique francophone, où certaines connexions internet peuvent être instables, les bind mounts permettent de travailler en local sans dépendre de dépôts distants à chaque modification.

### Gérer les réseaux Docker

Par défaut, chaque conteneur Docker fonctionne dans son propre réseau isolé. Pour qu’un conteneur communique avec un autre, ou avec l’hôte, il faut configurer un réseau approprié.

Docker propose plusieurs types de réseaux :

- **Bridge** (par défaut)
- **Host**
- **None**

#### Réseau Bridge

Le réseau **bridge** est le mode par défaut. Docker crée un réseau interne privé où les conteneurs peuvent communiquer entre eux via des noms d’hôte.

Exemple : connexion entre un conteneur web et un conteneur MySQL.

1. Créer un réseau personnalisé :

```bash
docker network create app-network
```

2. Lancer MySQL sur ce réseau :

```bash
docker run -d --name mysql-db \
  --network app-network \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=blog \
  mysql:8.0
```

3. Lancer une application Node.js qui se connecte à MySQL :

```bash
docker run -d --name blog-app \
  --network app-network \
  -p 3000:3000 \
  mon-app-node:1.0
```

Dans le code de l’application, la connexion à MySQL peut utiliser le nom du conteneur comme hôte :

```javascript
const mysql = require('mysql2');
const connection = mysql.createConnection({
  host: 'mysql-db',  // Nom du conteneur
  user: 'root',
  password: 'secret',
  database: 'blog'
});
```

Grâce au réseau personnalisé `app-network`, les deux conteneurs peuvent se parler directement, sans exposer MySQL au monde extérieur.

> **Cas d’usage** : Une équipe de développeurs à Dakar construit une application de gestion scolaire. Le front-end (React) parle au back-end (Express), qui interroge une base PostgreSQL. Tous les conteneurs sont sur le même réseau Docker, sécurisé et isolé.

#### Réseau Host

Le mode **host** fait que le conteneur partage directement le réseau de la machine hôte. Il n’y a pas de NAT ni d’isolation réseau.

Exemple :

```bash
docker run -d --network host --name nginx-host nginx:alpine
```

Le conteneur est accessible directement sur le port 80 de la machine, sans redirection de port (`-p`). C’est utile pour des performances élevées, mais **moins sécurisé**, car le conteneur a un accès direct au réseau de l’hôte.

> **Attention** : Ce mode n’est pas disponible sur Docker Desktop (Mac/Windows), uniquement sur Linux.

#### Réseau None

Le mode **none** désactive complètement le réseau pour le conteneur. Il est totalement isolé.

```bash
docker run --network none alpine ifconfig
```

Ce mode est rare, mais utile pour des tâches ponctuelles sans besoin de connexion réseau (ex : traitement de données sensibles hors ligne).

### Communication sécurisée entre conteneurs

La combinaison de volumes et de réseaux personnalisés permet de construire des architectures robustes et sécurisées.

Reprenons l’exemple d’une application de gestion de marché local, comme celle qu’un développeur à Lomé pourrait créer pour aider les petits commerçants.

- **Conteneur 1** : Base de données PostgreSQL (volume nommé `market-db`)
- **Conteneur 2** : API Flask (accès au volume de logs partagé)
- **Conteneur 3** : Interface d’administration (React)

Étapes :

1. Créer un réseau dédié :

```bash
docker network create market-net
```

2. Créer un volume pour la base :

```bash
docker volume create market-data
```

3. Lancer PostgreSQL :

```bash
docker run -d --name db-postgres \
  --network market-net \
  -v market-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=mdp123 \
  postgres:15
```

4. Lancer l’API Flask :

```bash
docker run -d --name api-flask \
  --network market-net \
  -p 5000:5000 \
  -v /home/user/logs:/app/logs \
  mon-api-flask:latest
```

5. L’API peut maintenant se connecter à `db-postgres` via le nom d’hôte, et les logs sont sauvegardés localement grâce au bind mount.

### Sauvegarder et restaurer des données

La persistance n’est complète que si vous pouvez sauvegarder les données.

Exemple : sauvegarder une base MySQL stockée dans un volume.

```bash
# Sauvegarde
docker exec mysql-db \
  mysqldump -u root -psecret ecommerce > backup.sql

# Restauration (dans un nouveau conteneur)
docker exec -i nouveau-mysql-db \
  mysql -u root -psecret ecommerce < backup.sql
```

> **Conseil Empire du Web** : Automatisez ces sauvegardes avec un script cron ou un conteneur dédié, surtout si vous hébergez des données critiques comme des factures ou des profils utilisateurs.

## Points clés

- **Les conteneurs sont éphémères** : sans volumes, les données sont perdues à la suppression du conteneur.
- **Les volumes nommés** sont idéaux pour la persistance des données critiques (ex : bases de données).
- **Les bind mounts** synchronisent un dossier local avec un conteneur — indispensables en développement.
- **Les réseaux personnalisés** permettent aux conteneurs de communiquer par nom, de manière sécurisée.
- **Le réseau bridge** est le plus utilisé pour les communications internes.
- **Le réseau host** supprime l’isolation réseau — à utiliser avec précaution.
- **Le réseau none** isole totalement le conteneur — utile pour des tâches sensibles.
- **Toujours utiliser un réseau dédié** pour les groupes de conteneurs qui doivent communiquer (ex : app + base).
- **Combinez volumes et réseaux** pour créer des environnements de développement ou de production fiables, sécurisés et faciles à maintenir.

Avec ces outils maîtrisés, vous êtes prêt à passer à l’étape supérieure : orchestrer plusieurs conteneurs ensemble avec Docker Compose, ce que nous explorerons dans le prochain chapitre.