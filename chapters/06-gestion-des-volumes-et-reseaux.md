## Gestion des volumes Docker : Préserver vos données

Lorsqu’un conteneur s’arrête ou est supprimé, toutes les données créées à l’intérieur de son système de fichiers sont perdues par défaut. Cela pose un problème majeur dans les applications réelles, notamment pour les bases de données. Imaginez un site de gestion de microcrédit au Sénégal : si le conteneur MySQL s’arrête et que toutes les données des prêts sont effacées, cela serait catastrophique.

Pour éviter cela, Docker propose un mécanisme appelé **volumes**. Un volume est un espace de stockage persistant, géré par Docker, qui existe indépendamment du cycle de vie du conteneur. Même si le conteneur est supprimé, les données du volume restent intactes.

### Créer un volume persistant pour MySQL

Prenons un exemple concret : une base de données MySQL utilisée par une application de gestion de santé dans un hôpital au Cameroun. Chaque patient, chaque consultation, chaque médicament est stocké dans cette base de données. Il est impératif que ces informations soient conservées, même après un redémarrage du serveur.

Voici comment créer un volume Docker pour persister les données MySQL :

```bash
docker volume create mysql_data
```

Ce simple commande crée un volume nommé `mysql_data`. Ensuite, nous lançons un conteneur MySQL en montant ce volume sur le répertoire `/var/lib/mysql`, là où MySQL stocke ses données :

```bash
docker run -d \
  --name mysql_db \
  -e MYSQL_ROOT_PASSWORD=secret123 \
  -e MYSQL_DATABASE=hopital_db \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0
```

Le paramètre `-v mysql_data:/var/lib/mysql` indique à Docker de lier le volume `mysql_data` au répertoire interne du conteneur. Si vous arrêtez ce conteneur, puis le relancez avec le même volume, toutes les données seront toujours là.

### Vérifier et gérer vos volumes

Vous pouvez lister tous les volumes existants avec :

```bash
docker volume ls
```

Pour inspecter un volume spécifique :

```bash
docker volume inspect mysql_data
```

Cela affiche des informations comme le chemin sur le système hôte, les métadonnées, etc.

Si vous souhaitez supprimer un volume (attention : les données seront perdues), utilisez :

```bash
docker volume rm mysql_data
```

> ⚠️ Attention : Ne supprimez jamais un volume contenant des données critiques sans sauvegarde préalable.

### Autres types de stockage : bind mounts

Outre les volumes Docker, il existe les **bind mounts**, qui permettent de monter un répertoire du système hôte directement dans un conteneur. C’est utile pendant le développement, par exemple pour synchroniser du code.

Imaginons un développeur à Abidjan qui travaille sur une API Node.js. Il peut monter son répertoire local dans le conteneur pour voir les modifications en temps réel :

```bash
docker run -d \
  --name api_dev \
  -v /home/user/projets/api-node:/app \
  -w /app \
  node:18 \
  npm start
```

Ici, chaque changement dans `/home/user/projets/api-node` sera immédiatement visible dans le conteneur. C’est efficace, mais moins portable qu’un volume, car cela dépend du chemin du système hôte.

Les volumes sont donc préférables pour les données persistantes, tandis que les bind mounts sont idéaux pour le développement local.

---

## Réseaux Docker : Faire communiquer les conteneurs

Dans une application réelle, les composants sont rarement isolés. Une application web a besoin d’un conteneur backend, qui lui-même communique avec une base de données. Sans réseau, ces conteneurs ne peuvent pas se "voir" entre eux.

Par défaut, Docker crée un réseau bridge appelé `bridge`, mais il a des limites : les conteneurs s’y connectent par adresse IP, ce qui est peu pratique et peu lisible.

C’est pourquoi Docker permet de créer des **réseaux personnalisés**. Ces réseaux offrent une communication sécurisée et nommée entre conteneurs.

### Créer un réseau personnalisé

Créons un réseau dédié à une application de gestion scolaire utilisée dans une école au Bénin. Cette application comprend deux composants :
- Un conteneur API (backend)
- Un conteneur web (frontend)

Commençons par créer un réseau :

```bash
docker network create ecole_net
```

Ce réseau permettra aux conteneurs de communiquer via leur nom, comme s’ils étaient sur un même serveur local.

### Lancer les conteneurs sur le même réseau

Supposons que vous ayez une image `api-ecole:1.0` et une image `web-ecole:1.0`. Vous pouvez maintenant les lancer sur le réseau `ecole_net` :

```bash
docker run -d --name api_server --network ecole_net api-ecole:1.0
```

```bash
docker run -d --name web_client --network ecole_net -p 8080:80 web-ecole:1.0
```

Désormais, depuis le conteneur `web_client`, vous pouvez accéder à l’API via son nom :

```bash
curl http://api_server:3000/eleves
```

Plus besoin de connaître l’IP du conteneur API ! Docker gère automatiquement la résolution de noms à l’intérieur du réseau personnalisé.

### Pourquoi les réseaux personnalisés sont essentiels

Les réseaux personnalisés offrent plusieurs avantages :

- **Résolution de noms** : les conteneurs peuvent se contacter par nom.
- **Isolation** : seuls les conteneurs sur le même réseau peuvent communiquer.
- **Sécurité** : pas d’exposition inutile à l’extérieur.
- **Flexibilité** : vous pouvez connecter ou déconnecter des conteneurs dynamiquement.

Par exemple, si vous ajoutez plus tard un conteneur de messagerie (pour envoyer des SMS aux parents), vous pouvez le relier au même réseau :

```bash
docker run -d --name sms_gateway --network ecole_net sms-service:1.0
```

Et l’API pourra l’appeler via `http://sms_gateway:5000/send`.

### Explorer les réseaux existants

Vous pouvez lister tous les réseaux Docker avec :

```bash
docker network ls
```

Pour voir les détails d’un réseau :

```bash
docker network inspect ecole_net
```

Cela affiche la liste des conteneurs connectés, leurs adresses IP, et la configuration du réseau.

Vous pouvez aussi connecter un conteneur existant à un autre réseau :

```bash
docker network connect ecole_net mysql_db
```

Et le déconnecter si besoin :

```bash
docker network disconnect ecole_net mysql_db
```

---

## Cas pratique : Application web complète avec données persistantes et réseau

Reprenons l’exemple d’une application de suivi agricole utilisée par des coopératives au Mali. Elle comprend :
- Une base de données MySQL (avec données persistantes)
- Une API Node.js
- Un frontend React

Voici comment tout assembler :

### Étape 1 : Créer le volume pour la base de données

```bash
docker volume create agricole_db_data
```

### Étape 2 : Créer le réseau

```bash
docker network create agricole_net
```

### Étape 3 : Lancer la base de données

```bash
docker run -d \
  --name db_mysql \
  --network agricole_net \
  -e MYSQL_ROOT_PASSWORD=agri2025 \
  -e MYSQL_DATABASE=coop_db \
  -v agricole_db_data:/var/lib/mysql \
  mysql:8.0
```

### Étape 4 : Lancer l’API

```bash
docker run -d \
  --name api_agri \
  --network agricole_net \
  -e DB_HOST=db_mysql \
  -e DB_USER=root \
  -e DB_PASSWORD=agri2025 \
  agri-api:1.0
```

L’API utilise `db_mysql` comme nom d’hôte pour se connecter à la base.

### Étape 5 : Lancer le frontend

```bash
docker run -d \
  --name web_agri \
  --network agricole_net \
  -p 3000:80 \
  agri-web:1.0
```

Le frontend peut maintenant appeler l’API via `http://api_agri:5000`, et l’API accède à la base via `db_mysql`.

Grâce à cette architecture :
- Les données de production agricole sont sauvegardées même si le conteneur MySQL redémarre.
- Les composants communiquent de manière fiable via des noms clairs.
- L’application est modulaire, facile à mettre à jour ou scaler.

---

## Points clés

- **Les volumes Docker** permettent de persister les données au-delà du cycle de vie des conteneurs. Ils sont essentiels pour les bases de données (MySQL, PostgreSQL, etc.).
- Utilisez `docker volume create` pour créer un volume, et `-v nom_volume:/chemin` pour le monter dans un conteneur.
- **Les bind mounts** sont utiles pour le développement, mais moins portables que les volumes.
- **Les réseaux personnalisés** permettent une communication sécurisée et nommée entre conteneurs.
- Créez un réseau avec `docker network create`, puis lancez vos conteneurs avec `--network nom_reseau`.
- À l’intérieur d’un même réseau, les conteneurs peuvent se contacter par leur nom de conteneur.
- Cette combinaison (volumes + réseaux) est fondamentale pour construire des applications réalistes, fiables, et évolutives avec Docker.
- Dans votre prochain projet — qu’il s’agisse d’un e-commerce, d’un système de gestion, ou d’un outil éducatif — pensez dès le départ à la persistance des données et à l’interconnexion des services.