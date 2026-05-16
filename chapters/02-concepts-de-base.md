## Les concepts fondamentaux de Docker : Images, conteneurs, registres

### Comprendre l’image Docker : un modèle immuable

Imaginez que vous êtes un cuisinier dans un restaurant au Sénégal, et que vous préparez un plat emblématique : le thieboudienne. Pour que chaque assiette soit identique en goût et en qualité, vous suivez une recette précise. Cette recette, avec ses ingrédients, ses étapes, et ses instructions exactes, est stockée dans un carnet que vous ne modifiez jamais. Vous pouvez l’utiliser autant de fois que nécessaire, mais elle reste inchangée.

En Docker, une **image** est exactement comme cette recette. C’est un modèle immuable, une "photographie" d’un environnement logiciel à un instant donné. Elle contient tout ce dont une application a besoin pour fonctionner : le système d’exploitation, les bibliothèques, les dépendances, les variables d’environnement, et le code de l’application elle-même.

Une image Docker n’est pas un programme exécutable en soi. C’est une base, un modèle à partir duquel on va créer des instances fonctionnelles. Elle est construite de manière hiérarchique, grâce à des "couches" (layers). Chaque instruction dans la recette (comme installer un logiciel ou copier un fichier) ajoute une nouvelle couche à l’image. Grâce à ce système, Docker peut réutiliser efficacement les couches communes entre plusieurs images, ce qui optimise le stockage et la vitesse de distribution.

Par exemple, une image pour une application web en PHP pourrait inclure :
- Une version de Linux (comme Debian ou Alpine),
- Un serveur web (Apache ou Nginx),
- Le langage PHP,
- Une base de données légère (comme SQLite),
- Et enfin, le code de votre site web.

Tout cela est empaqueté dans une seule image, prête à être utilisée n’importe où.

### Le conteneur : une instance exécutable de l’image

Revenons à notre cuisine. Vous avez la recette du thieboudienne. Maintenant, vous décidez de préparer un plat. Vous sortez les ingrédients, faites chauffer la marmite, et après 45 minutes, vous servez une assiette fumante. Ce plat, c’est l’**instance** de la recette.

En Docker, le **conteneur** est cette instance. C’est une version exécutable d’une image. Lorsque vous "lancez" une image avec Docker, vous créez un conteneur qui fonctionne dans un environnement isolé sur votre machine. Ce conteneur peut démarrer, s’arrêter, redémarrer, ou être supprimé, sans affecter l’image d’origine.

Voici un exemple concret :  
Supposons que vous ayez une image appelée `webapp:1.0` (le `:1.0` indique la version). Vous pouvez créer plusieurs conteneurs à partir de cette même image :

```bash
docker run -d --name site-prod webapp:1.0
docker run -d --name site-test webapp:1.0
```

Ici, deux conteneurs sont lancés à partir de la même image. L’un s’appelle `site-prod` (pour la production), l’autre `site-test` (pour les tests). Même s’ils partagent la même base, ils fonctionnent indépendamment. Si vous modifiez quelque chose dans le conteneur `site-test`, cela n’affecte pas `site-prod`, ni l’image d’origine.

Un point essentiel à comprendre : les conteneurs sont **éphémères**. Cela signifie qu’ils peuvent être détruits et recréés facilement. C’est une philosophie fondamentale de Docker : au lieu de "réparer" un serveur qui plante, on le remplace par un nouveau conteneur frais, basé sur une image fiable.

### Les registres Docker : des bibliothèques d’images

Maintenant, imaginons que vous ne possédez pas la recette du thieboudienne. Vous souhaitez la trouver rapidement, fiable et testée. Vous allez dans une grande bibliothèque culinaire où des milliers de recettes sont classées, versionnées et accessibles à tous. Vous la téléchargez, et vous pouvez commencer à cuisiner.

En Docker, ce rôle de bibliothèque est joué par les **registres**. Un registre est un dépôt centralisé où sont stockées les images Docker. Le plus connu est **Docker Hub**, un registre public gratuit, comparable à GitHub pour le code.

Docker Hub contient des millions d’images officielles ou communautaires. Par exemple :
- `nginx:latest` – un serveur web léger,
- `mysql:8.0` – une base de données MySQL,
- `python:3.11-slim` – un environnement Python allégé.

Vous pouvez les utiliser directement sans avoir à tout reconstruire. Par exemple, pour lancer un serveur Nginx :

```bash
docker run -p 8080:80 nginx:latest
```

Cette commande dit à Docker :
1. Va sur Docker Hub,
2. Télécharge l’image `nginx:latest`,
3. Crée un conteneur à partir de cette image,
4. Mappe le port 8080 de votre machine au port 80 du conteneur.

En quelques secondes, un serveur web fonctionnel est opérationnel. C’est puissant, surtout pour un développeur au Maroc ou en Côte d’Ivoire qui veut tester rapidement une technologie sans configurer manuellement un serveur.

Mais Docker Hub n’est pas le seul registre. Les entreprises utilisent souvent des **registres privés** (comme Amazon ECR, Google Container Registry, ou un registre auto-hébergé) pour stocker leurs images internes, sécurisées et spécifiques à leurs projets.

### Versionner les images : pourquoi c’est crucial

Dans une cuisine, deux versions d’une recette peuvent exister : une ancienne (avec beaucoup de sel) et une nouvelle (plus saine). Vous ne voulez pas que le client reçoive la mauvaise version. De même, dans Docker, **le versionnage des images est fondamental**.

Chaque image peut avoir un **tag** (étiquette), souvent utilisé pour indiquer la version. Par exemple :
- `ubuntu:20.04`
- `node:18-alpine`
- `monapp:v1.2`

Le tag `latest` est souvent utilisé par défaut, mais il peut être trompeur : il pointe vers la dernière version poussée, qui n’est pas toujours stable. En production, il est fortement recommandé d’utiliser des tags explicites (`v1.0`, `2.3.1`, etc.) pour garantir la reproductibilité.

Prenons un cas concret : un développeur à Abidjan déploie une application avec `python:3.9`. Deux mois plus tard, une mise à jour de l’image `python:3.9` introduit une incompatibilité. Si le développeur redéploie sans version explicite, son application peut casser. En revanche, avec `python:3.9.18`, il reste sur une version connue et stable.

### Illustration concrète : une application africaine dans Docker

Prenons l’exemple d’un développeur à Dakar qui crée une application de gestion de marchés locaux, en Python et Flask. Voici comment les concepts s’appliquent :

1. **Image** : Il crée un `Dockerfile` qui décrit l’environnement :
   ```Dockerfile
   FROM python:3.10-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY . .
   CMD ["python", "app.py"]
   ```
   Ensuite, il construit l’image :
   ```bash
   docker build -t gest-marche:v1.0 .
   ```

2. **Conteneur** : Il lance l’application :
   ```bash
   docker run -d -p 5000:5000 --name mon-marche gest-marche:v1.0
   ```
   Un conteneur nommé `mon-marche` démarre, accessible via `http://localhost:5000`.

3. **Registre** : Pour partager l’image avec son équipe à Bamako, il pousse l’image sur Docker Hub :
   ```bash
   docker tag gest-marche:v1.0 moncompte/gest-marche:v1.0
   docker push moncompte/gest-marche:v1.0
   ```
   Son collègue à Bamako peut alors la récupérer et lancer sa propre instance :
   ```bash
   docker run -d -p 5000:5000 moncompte/gest-marche:v1.0
   ```

Tout fonctionne exactement de la même manière, qu’on soit à Dakar, à Lomé ou à Paris. C’est la puissance de Docker.

## Points clés à retenir

- Une **image Docker** est un modèle immuable, comparable à une recette de cuisine. Elle contient tout ce dont une application a besoin pour fonctionner.
- Un **conteneur** est une instance exécutable d’une image. Il est isolé, éphémère, et peut être lancé, arrêté ou supprimé à tout moment.
- Les images sont organisées en couches, ce qui permet un partage efficace des ressources et une construction rapide.
- Un **registre** (comme Docker Hub) est un dépôt centralisé pour stocker et partager des images Docker.
- Le **versionnage** des images via des tags (`v1.0`, `2.1`, etc.) est essentiel pour assurer la stabilité et la reproductibilité des environnements.
- Utiliser des tags explicites est une meilleure pratique que d’utiliser `latest`, surtout en production.
- Grâce à ces concepts, Docker permet de créer des environnements cohérents, reproductibles, et portables, peu importe la machine ou le pays.

Ces fondations sont cruciales. Dans les chapitres suivants d’Empire du Web, vous passerez à la pratique : installer Docker, lancer vos premiers conteneurs, et créer vos propres images.