## Lancement de vos premiers conteneurs
Maintenant que vous avez installé et configuré Docker sur votre machine, il est temps de passer à l'action et de lancer vos premiers conteneurs. Pour cela, nous allons utiliser des images simples comme Nginx ou BusyBox. Ces images sont idéales pour les débutants car elles sont légères et faciles à utiliser.

### Utilisation de l'image Nginx
L'image Nginx est un excellent choix pour les débutants. Nginx est un serveur web léger et performant qui peut être utilisé pour servir des pages web statiques. Pour lancer un conteneur Nginx, vous pouvez utiliser la commande suivante :
```bash
docker run -p 8080:80 nginx
```
Cette commande créé un nouveau conteneur à partir de l'image Nginx et le lance. Le paramètre `-p 8080:80` mappe le port 8080 de votre machine hôte au port 80 du conteneur. Cela signifie que vous pouvez accéder au serveur web Nginx en allant à l'adresse `http://localhost:8080` dans votre navigateur.

### Utilisation de l'image BusyBox
L'image BusyBox est une autre image légère et polyvalente qui peut être utilisée pour de nombreux objectifs. BusyBox est un système d'exploitation minimaliste qui fournit un ensemble d'outils de base pour la gestion de fichiers, la gestion de réseaux, etc. Pour lancer un conteneur BusyBox, vous pouvez utiliser la commande suivante :
```bash
docker run -it busybox
```
Cette commande créé un nouveau conteneur à partir de l'image BusyBox et le lance en mode interactif. Le paramètre `-it` permet d'avoir un accès interactif au conteneur, ce qui signifie que vous pouvez exécuter des commandes et interagir avec le conteneur comme si vous étiez à l'intérieur.

## Gestion de vos conteneurs
Maintenant que vous avez lancé vos premiers conteneurs, il est important de savoir comment les gérer. La gestion de conteneurs implique plusieurs tâches, notamment la liste des conteneurs en cours d'exécution, l'arrêt des conteneurs, la suppression des conteneurs, etc.

### Liste des conteneurs en cours d'exécution
Pour lister les conteneurs en cours d'exécution, vous pouvez utiliser la commande suivante :
```bash
docker ps
```
Cette commande affiche une liste des conteneurs en cours d'exécution, y compris leur ID, leur nom, leur image, leur statut, etc.

### Arrêt des conteneurs
Pour arrêter un conteneur, vous pouvez utiliser la commande suivante :
```bash
docker stop <ID_conteneur>
```
Remplacez `<ID_conteneur>` par l'ID du conteneur que vous souhaitez arrêter. Vous pouvez trouver l'ID du conteneur en utilisant la commande `docker ps`.

### Suppression des conteneurs
Pour supprimer un conteneur, vous pouvez utiliser la commande suivante :
```bash
docker rm <ID_conteneur>
```
Remplacez `<ID_conteneur>` par l'ID du conteneur que vous souhaitez supprimer. Vous pouvez trouver l'ID du conteneur en utilisant la commande `docker ps`.

## Inspection des conteneurs
Il est souvent utile de pouvoir inspecter les conteneurs pour obtenir des informations sur leur configuration, leur état, etc. Pour inspecter un conteneur, vous pouvez utiliser la commande suivante :
```bash
docker inspect <ID_conteneur>
```
Cette commande affiche des informations détaillées sur le conteneur, y compris sa configuration, son état, ses réseaux, etc.

## Bonnes pratiques de nettoyage et de gestion des ressources
Il est important de nettoyer régulièrement les conteneurs et les images pour éviter de gaspiller des ressources. Voici quelques bonnes pratiques à suivre :

* Supprimez régulièrement les conteneurs et les images inutilisés.
* Utilisez la commande `docker system prune` pour supprimer les conteneurs et les images inutilisés.
* Utilisez la commande `docker volume prune` pour supprimer les volumes inutilisés.

## Exemples concrets pour les développeurs africains francophones
Supposons que vous êtes un développeur web en Afrique qui souhaite créer un site web pour une entreprise locale. Vous pouvez utiliser Docker pour créer un serveur web Nginx et héberger votre site web. Voici un exemple de commande pour créer un conteneur Nginx :
```bash
docker run -p 8080:80 -v /chemin/vers/votre/site/web:/usr/share/nginx/html nginx
```
Cette commande créé un nouveau conteneur à partir de l'image Nginx et le lance. Le paramètre `-v` mappe le répertoire `/chemin/vers/votre/site/web` de votre machine hôte au répertoire `/usr/share/nginx/html` du conteneur. Cela signifie que vous pouvez accéder à votre site web en allant à l'adresse `http://localhost:8080` dans votre navigateur.

## Points clés
* Lancez vos premiers conteneurs à l'aide des commandes `docker run` et `docker ps`.
* Gérez vos conteneurs à l'aide des commandes `docker stop`, `docker rm` et `docker inspect`.
* Suivez les bonnes pratiques de nettoyage et de gestion des ressources pour éviter de gaspiller des ressources.
* Utilisez les commandes `docker system prune` et `docker volume prune` pour supprimer les conteneurs et les volumes inutilisés.
* Utilisez les images officielles de Docker pour créer des conteneurs légers et performants.
* Mappez les ports et les répertoires pour accéder à vos conteneurs et vos applications.
* Utilisez les commandes `docker` pour automatiser la création et la gestion de vos conteneurs.