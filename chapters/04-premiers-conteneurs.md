## Lancer vos premiers conteneurs : Commandes de base de Docker

Vous avez installé Docker, vous comprenez les bases des images et des conteneurs — maintenant, il est temps de passer à l’action. Ce chapitre vous met les mains dans le code avec les commandes fondamentales de Docker. Vous allez non seulement apprendre à lancer, arrêter et inspecter des conteneurs, mais aussi comprendre comment ils interagissent avec votre machine.

L’objectif ici est simple : vous rendre autonome avec l’interface en ligne de commande de Docker, l’outil principal pour tout développeur qui travaille avec des conteneurs.

### Démarrer un conteneur avec `docker run`

La commande `docker run` est probablement la plus utilisée. Elle permet de créer et de démarrer un conteneur à partir d’une image. Prenons un exemple concret : vous souhaitez tester un serveur web léger comme Nginx, très populaire en Afrique francophone pour héberger des sites statiques ou des applications simples.

```bash
docker run -d -p 8080:80 --name mon-nginx nginx
```

Analysons cette commande :

- `docker run` : lance un nouveau conteneur.
- `-d` : exécute le conteneur en arrière-plan (mode détaché).
- `-p 8080:80` : mappe le port 8080 de votre machine au port 80 du conteneur. Cela signifie que vous pourrez accéder au serveur via `http://localhost:8080`.
- `--name mon-nginx` : donne un nom personnalisé au conteneur (plus facile à gérer qu’un nom aléatoire).
- `nginx` : l’image à utiliser (téléchargée automatiquement depuis Docker Hub si elle n’est pas présente localement).

Une fois exécutée, cette commande télécharge l’image `nginx` si elle n’est pas déjà en cache, crée un conteneur, et le démarre. Ouvrez votre navigateur et rendez-vous sur `http://localhost:8080` — vous verrez la page par défaut de Nginx.

> **Astuce pratique** : Si vous êtes au Sénégal, au Cameroun ou en Côte d’Ivoire et que vous développez des applications locales, utiliser Nginx via Docker vous permet de tester rapidement sans installer de serveur web global sur votre machine.

### Lister les conteneurs en cours avec `docker ps`

Pour voir quels conteneurs sont actifs, utilisez :

```bash
docker ps
```

Cela affiche une liste des conteneurs en cours d’exécution, avec des informations cruciales :
- CONTAINER ID
- IMAGE utilisée
- COMMAND lancée
- CREATED (depuis combien de temps il tourne)
- STATUS
- PORTS exposés
- NAMES attribués

Si vous souhaitez voir tous les conteneurs, y compris ceux arrêtés, ajoutez l’option `-a` :

```bash
docker ps -a
```

C’est utile pour retrouver un conteneur que vous avez lancé hier pour un test à Abidjan, même s’il n’est plus actif.

### Arrêter et redémarrer un conteneur

Vous avez un conteneur qui tourne, mais vous souhaitez l’arrêter temporairement. Utilisez `docker stop` :

```bash
docker stop mon-nginx
```

Docker envoie un signal d’arrêt propre au conteneur, qui se termine en quelques secondes. Si vous vérifiez avec `docker ps`, il ne sera plus listé. Pour le relancer :

```bash
docker start mon-nginx
```

Le conteneur redémarre exactement dans l’état où il était (mêmes ports, même configuration). Pas besoin de relancer `docker run`.

### Supprimer un conteneur avec `docker rm`

Les conteneurs arrêtés consomment peu de ressources, mais ils encombrent la liste. Pour nettoyer, utilisez `docker rm` :

```bash
docker rm mon-nginx
```

> ⚠️ Attention : cette suppression est définitive. Le conteneur est perdu, mais l’image `nginx` reste disponible pour un prochain `docker run`.

Vous pouvez supprimer plusieurs conteneurs d’un coup :
```bash
docker rm conteneur1 conteneur2
```

Ou supprimer tous les conteneurs arrêtés :
```bash
docker container prune
```

### Explorer les logs avec `docker logs`

Quand un conteneur tourne, il génère des logs — très utiles pour le débogage. Par exemple, si votre conteneur Nginx ne répond pas, consultez ses logs :

```bash
docker logs mon-nginx
```

Vous verrez les requêtes HTTP entrantes, les erreurs éventuelles, etc.

Pour suivre les logs en temps réel (comme `tail -f`), ajoutez `-f` :

```bash
docker logs -f mon-nginx
```

Imaginez que vous gérez un site e-commerce au Maroc ou au Bénin : suivre les logs en direct vous permet de détecter rapidement une panne ou une surcharge.

### Exécuter des commandes dans un conteneur avec `docker exec`

Parfois, vous avez besoin d’entrer dans un conteneur pour inspecter son système de fichiers ou lancer des commandes. C’est le rôle de `docker exec`.

Lançons un conteneur Ubuntu en arrière-plan :

```bash
docker run -d --name mon-ubuntu ubuntu sleep infinity
```

Ici, `sleep infinity` empêche le conteneur de s’arrêter immédiatement (Ubuntu s’arrête dès qu’il n’a plus de processus à exécuter).

Maintenant, accédons-y :

```bash
docker exec -it mon-ubuntu /bin/bash
```

- `-i` : mode interactif
- `-t` : alloue un terminal
- `/bin/bash` : lance le shell bash à l’intérieur du conteneur

Une fois à l’intérieur, vous êtes dans un environnement Ubuntu isolé. Vous pouvez lister les fichiers avec `ls`, installer des paquets avec `apt update`, etc.

> **Exemple concret** : Vous développez une application Django au Sénégal et vous voulez tester les dépendances dans un environnement propre. Entrer dans un conteneur Ubuntu vous permet de simuler un serveur de production sans toucher votre machine locale.

Pour sortir du conteneur, tapez `exit`.

### Vérifier les détails d’un conteneur avec `docker inspect`

Quand vous avez besoin d’informations techniques précises (adresse IP, variables d’environnement, montage de volumes, etc.), utilisez `docker inspect` :

```bash
docker inspect mon-nginx
```

La sortie est un JSON très détaillé. Par exemple, vous pouvez y trouver :
- L’adresse IP attribuée au conteneur
- Les ports exposés
- Le chemin du répertoire de travail
- Les variables d’environnement

Pour extraire une valeur spécifique, combinez avec `grep` ou utilisez le format Go (avancé, mais puissant) :

```bash
docker inspect -f '{{.NetworkSettings.IPAddress}}' mon-nginx
```

Cela affiche uniquement l’adresse IP du conteneur.

### Gérer les images avec `docker images` et `docker rmi`

Les conteneurs sont créés à partir d’images. Pour voir quelles images sont présentes sur votre machine :

```bash
docker images
```

Vous verrez :
- REPOSITORY (nom de l’image)
- TAG (version, ex: `latest`, `1.21`)
- IMAGE ID
- CREATED
- SIZE

Pour supprimer une image inutilisée :

```bash
docker rmi nginx
```

> ⚠️ Vous ne pouvez pas supprimer une image si un conteneur l’utilise (même arrêté). Supprimez d’abord les conteneurs.

### Cycle de vie d’un conteneur : résumé pratique

Voici le cycle typique d’un conteneur :

1. **Création et lancement** : `docker run`
2. **Exécution** : le conteneur tourne, expose des ports, génère des logs
3. **Arrêt** : `docker stop`
4. **Redémarrage** : `docker start`
5. **Suppression** : `docker rm`
6. **Nettoyage** : suppression des images inutiles avec `docker rmi`

Chaque étape est contrôlable via la CLI. Cela donne une grande flexibilité — par exemple, lancer un conteneur de test au Togo, l’arrêter après vérification, puis le relancer la semaine suivante sans tout reconfigurer.

### Exercice pratique : Lancez un serveur Redis

Redis est une base de données en mémoire souvent utilisée pour la mise en cache, très utile pour les applications web à fort trafic (comme un site d’actualité au Mali ou un service de mobile money au Burkina Faso).

1. Lancez un conteneur Redis :
```bash
docker run -d --name mon-redis -p 6379:6379 redis
```

2. Vérifiez qu’il tourne :
```bash
docker ps
```

3. Consultez les logs :
```bash
docker logs mon-redis
```

4. Arrêtez-le :
```bash
docker stop mon-redis
```

5. Redémarrez-le :
```bash
docker start mon-redis
```

6. Nettoyez :
```bash
docker stop mon-redis && docker rm mon-redis
```

Ce petit exercice vous permet de manipuler l’ensemble des commandes de base dans un contexte réaliste.

## Points clés

- `docker run` est la commande principale pour lancer un conteneur à partir d’une image.
- Utilisez `docker ps` pour lister les conteneurs actifs, et `docker ps -a` pour voir tous les conteneurs.
- `docker stop` arrête proprement un conteneur ; `docker start` le relance.
- `docker rm` supprime un conteneur (arrêtez-le d’abord).
- `docker logs` permet d’analyser le comportement d’un conteneur en temps réel ou après une erreur.
- `docker exec -it` ouvre un terminal à l’intérieur d’un conteneur en cours d’exécution.
- `docker inspect` donne des détails techniques complets sur un conteneur.
- `docker images` et `docker rmi` permettent de gérer les images stockées localement.
- Le cycle de vie d’un conteneur est entièrement contrôlé via la ligne de commande.
- Pratiquer avec des images comme Nginx, Ubuntu ou Redis permet de comprendre rapidement les concepts.
- Docker rend possible un environnement de développement uniforme, peu importe le pays ou le système d’exploitation — un atout majeur pour les développeurs africains qui collaborent à distance.

En maîtrisant ces commandes, vous devenez capable de tester, déployer et gérer des services rapidement — un fondement essentiel pour la suite de votre apprentissage avec Empire du Web.