## Les concepts fondamentaux : Images, conteneurs, registres et Dockerfile

### Une image, c’est comme une recette de cuisine

Imaginez que vous voulez préparer un bon *thiéboudienne* chez vous. Avant de commencer, vous avez besoin d’une recette détaillée : les ingrédients, les étapes de cuisson, les épices, et même la durée de cuisson du riz. Cette recette, même si elle n’est pas encore utilisée, contient toute l’information nécessaire pour créer le plat.

En Docker, une **image** est exactement cela : une recette complète pour créer une application. Elle contient tout ce dont le logiciel a besoin pour fonctionner — le système d’exploitation de base, les bibliothèques, les dépendances, le code de l’application, et les instructions d’exécution. Mais comme une recette, l’image **n’est pas encore vivante**. Elle ne fait rien tant qu’on ne l’utilise pas.

Une image Docker est **immuable**, c’est-à-dire qu’elle ne change pas une fois créée. Si vous voulez modifier quelque chose, vous n’éditez pas l’image directement — vous en créez une nouvelle, comme si vous réécriviez la recette pour une version améliorée du plat.

Les images sont structurées en **couches**. Chaque instruction dans une recette ajoute une couche : découper les légumes, faire revenir l’oignon, ajouter le poisson, etc. En Docker, chaque commande dans un fichier de construction (qu’on verra plus tard) crée une nouvelle couche. Cela permet de **réutiliser** les couches communes entre différentes images, ce qui économise du temps et de l’espace.

Par exemple, si vous avez deux applications qui utilisent toutes les deux Ubuntu comme base, Docker n’a besoin de télécharger cette couche de base qu’une seule fois. C’est comme utiliser la même casserole pour deux recettes différentes — vous ne la rachetez pas à chaque fois.

Voici un exemple simplifié de ce que contient une image Docker :
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3
COPY app.py /app/
CMD ["python3", "/app/app.py"]
```

Chaque ligne crée une couche. L’image finale est comme un empilement de ces étapes, prête à être utilisée.

### Un conteneur, c’est le plat servi sur la table

Maintenant que vous avez la recette (l’image), vous pouvez enfin cuisiner. Le *thiéboudienne* est préparé, servi dans une assiette, et vous pouvez le manger. Ce plat **en cours d’utilisation** est un **conteneur**.

Un conteneur Docker est une **instance exécutable** d’une image. C’est là que l’application tourne réellement. Contrairement à l’image (statique), le conteneur est **dynamique** : il consomme de la mémoire, du processeur, peut lire ou écrire des fichiers, et communiquer avec d’autres programmes.

Par exemple, si votre image est une recette de *soupe kandia*, le conteneur est la soupe chaude que vous mangez maintenant. Vous pouvez avoir plusieurs assiettes (conteneurs) issues de la même recette (image), chacune pouvant être modifiée indépendamment — une un peu plus salée, une autre avec plus de lait de coco.

Mais attention : par défaut, les conteneurs sont **éphémères**. Cela signifie que si vous éteignez le conteneur, tout ce qui a été modifié à l’intérieur disparaît — comme si vous jetiez l’assiette après le repas. Si vous redémarrez une nouvelle instance, vous repartez toujours de la recette d’origine.

C’est une force : cela garantit que chaque lancement est **prévisible et propre**. Mais si vous voulez garder des données (comme une base de données), il faudra utiliser des **volumes**, un concept que nous verrons plus tard.

Voici comment démarrer un conteneur à partir d’une image :
```bash
docker run -d --name mon_site ubuntu:20.04
```
Ici, Docker prend l’image `ubuntu:20.04`, crée un conteneur nommé `mon_site`, et le lance en arrière-plan (`-d`). Le conteneur fonctionne dans son propre environnement isolé, sans interférer avec le reste du système.

### Les registres : les bibliothèques d’images

Vous ne créez pas toujours vos propres recettes. Parfois, vous trouvez une bonne recette de *mafé* sur un blog culinaire et vous l’utilisez directement. De la même manière, vous n’êtes pas obligé de créer toutes vos images Docker à la main.

Un **registre Docker** est un endroit où les images sont stockées et partagées. Le plus connu est **Docker Hub**, une sorte de "bibliothèque mondiale" d’images Docker. Des millions d’images y sont disponibles : Ubuntu, Python, MySQL, Nginx, Node.js, etc.

Par exemple, si vous voulez lancer un serveur web avec Nginx, vous n’avez pas besoin de tout configurer vous-même :
```bash
docker run -p 8080:80 nginx
```
Docker va automatiquement chercher l’image `nginx` sur Docker Hub, la télécharger si nécessaire, puis créer un conteneur à partir de cette image. En quelques secondes, un serveur web est opérationnel sur votre machine, accessible via `http://localhost:8080`.

C’est comme télécharger une application depuis un magasin d’applications, sauf que c’est une application serveur, prête à tourner.

Vous pouvez aussi **pousser** (push) vos propres images sur un registre. Par exemple, une startup ivoirienne qui développe une application de gestion de marchés locaux peut créer une image personnalisée, la tester, puis la partager avec son équipe ou ses clients via un registre privé.

### Le Dockerfile : le cahier de recettes technique

Si vous voulez créer votre propre plat — disons un *attiéké grillé aux crevettes* — vous allez écrire votre propre recette. En Docker, ce cahier de recettes s’appelle un **Dockerfile**.

Un Dockerfile est un fichier texte qui contient une suite d’instructions pour construire une image Docker. Chaque ligne est une commande précise : installer un logiciel, copier des fichiers, définir une variable d’environnement, ou indiquer quel programme lancer au démarrage.

Voici un exemple concret : vous êtes développeur au Sénégal et vous travaillez sur un site web en Python utilisant Flask.

```dockerfile
# On part d'une image de base avec Python
FROM python:3.9-slim

# On crée un répertoire dans le conteneur pour notre app
WORKDIR /app

# On copie les dépendances
COPY requirements.txt .

# On installe les bibliothèques nécessaires
RUN pip install -r requirements.txt

# On copie le code source
COPY . .

# On expose le port 5000 (Flask)
EXPOSE 5000

# Commande à exécuter au démarrage
CMD ["python", "app.py"]
```

Ce Dockerfile décrit étape par étape comment construire l’environnement pour votre application. Une fois prêt, vous pouvez construire l’image avec la commande :
```bash
docker build -t mon_app_flask .
```

Et lancer un conteneur :
```bash
docker run -p 5000:5000 mon_app_flask
```

Votre application est désormais accessible sur `http://localhost:5000`, sans avoir à installer Python ou Flask sur votre machine locale. Vous pouvez la partager facilement avec un collègue au Maroc ou en Côte d’Ivoire — il lui suffit de récupérer le Dockerfile et de lancer les mêmes commandes.

### Éphémère vs durable : comprendre la nature des conteneurs

Un point crucial à retenir est que **les conteneurs sont conçus pour être éphémères**. Cela peut sembler étrange au début : pourquoi créer quelque chose qui disparaît ?

Mais c’est justement ce qui rend Docker si puissant. En étant éphémères, les conteneurs deviennent **prévisibles et reproductibles**. Si un conteneur plante, vous en créez un nouveau à partir de l’image — comme remplacer une assiette cassée par une nouvelle, identique à la précédente.

Cela change radicalement la façon dont on gère les applications. Plus besoin de "réparer" un serveur en panne. On le remplace. C’est comme avoir un stock infini de *doumbous* prêts à être servis : si l’un tombe par terre, on en sort un autre du sac.

Cependant, pour les données importantes (comme les profils utilisateurs ou les commandes d’e-commerce), on utilise des **volumes** ou des bases de données externes. C’est comme garder la recette précieuse dans un coffre, même si l’assiette est jetée.

### Exemple concret : un blog africain en conteneur

Imaginons que vous développez un blog pour partager des contes traditionnels africains. Vous utilisez WordPress, qui a besoin de PHP et de MySQL.

Au lieu d’installer tout cela manuellement sur chaque machine, vous pouvez :

1. Télécharger l’image officielle de WordPress depuis Docker Hub.
2. Créer un conteneur avec :
   ```bash
   docker run -d -p 80:80 --name mon_blog wordpress
   ```
3. Accéder à `http://localhost` et commencer à rédiger.

Tout est isolé, propre, et peut être déployé sur n’importe quel serveur compatible Docker — au Bénin, au Cameroun, ou sur un cloud international.

---

## Points clés à retenir

- Une **image Docker** est une recette immuable contenant tout ce dont une application a besoin pour fonctionner.
- Un **conteneur** est une instance exécutable d’une image — c’est là que l’application tourne.
- Les images sont composées de **couches**, ce qui permet de les construire efficacement et de les partager.
- Les **registres** comme Docker Hub sont des bibliothèques d’images prêtes à l’emploi.
- Un **Dockerfile** est un fichier qui décrit comment construire une image personnalisée, étape par étape.
- Les conteneurs sont **éphémères** par défaut : ils peuvent être détruits et recréés sans perte de qualité, tant que les données critiques sont sauvegardées ailleurs.
- Docker permet de standardiser le développement et le déploiement, peu importe la machine ou le pays — un atout majeur pour les développeurs africains travaillant dans des équipes distribuées.

En maîtrisant ces concepts, vous posez les bases solides pour utiliser Docker efficacement, que ce soit pour tester une application localement ou la déployer à grande échelle.