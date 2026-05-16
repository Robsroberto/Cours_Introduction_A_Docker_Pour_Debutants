## La suite après Docker : Kubernetes, CI/CD et cloud

Vous venez de maîtriser Docker, ses images, ses conteneurs, ses fichiers `Dockerfile` et `docker-compose.yml`. Vous êtes désormais capable de containeriser une application, de la faire fonctionner localement, et même de la déployer sur un serveur. Mais Docker, aussi puissant soit-il, n’est qu’un maillon d’une chaîne bien plus vaste : celle du développement moderne, agile et scalable. Ce chapitre vous ouvre les portes des technologies qui viennent naturellement **après Docker**, et qui transforment un développeur individuel en acteur d’un écosystème DevOps complet.

---

### Kubernetes : orchestrer à grande échelle

Imaginez que vous développez une application e-commerce populaire au Sénégal, qui attire des milliers de visiteurs chaque jour. Votre application tourne dans plusieurs conteneurs Docker : un pour le frontend, un pour l’API, un pour la base de données, et un pour le service de paiement. Un jour, un pic de trafic survient à cause d’une promotion sur les téléphones mobiles. Votre serveur peine, les utilisateurs se plaignent du ralentissement.

C’est là que **Kubernetes** entre en jeu.

Kubernetes, souvent abrégé en **K8s**, est un système d’orchestration de conteneurs open source, initialement développé par Google. Alors que Docker Compose vous permet de gérer plusieurs conteneurs sur **une seule machine**, Kubernetes vous permet de déployer, surveiller et **mettre à l’échelle automatiquement** des centaines de conteneurs sur **plusieurs serveurs**.

Voici ce que Kubernetes peut faire pour vous :
- **Auto-scaling** : augmenter ou réduire le nombre de conteneurs selon la charge.
- **Auto-healing** : redémarrer un conteneur qui tombe en panne.
- **Équilibrage de charge** : répartir le trafic entre plusieurs instances.
- **Déploiements progressifs (rolling updates)** : mettre à jour une application sans interruption de service.

Par exemple, dans votre application e-commerce, Kubernetes peut détecter que la charge CPU dépasse 70 % et, en quelques secondes, lancer 3 nouveaux conteneurs de l’API pour absorber la demande.

Même si Kubernetes a une courbe d’apprentissage raide, des outils comme **Minikube** (pour tester localement) ou des services gérés comme **Google Kubernetes Engine (GKE)**, **Amazon EKS** ou **Azure AKS** simplifient grandement son utilisation.

---

### CI/CD : automatiser le développement avec Git

En Afrique, de plus en plus d’entreprises tech adoptent des méthodes agiles. Mais coder vite ne suffit pas : il faut aussi **déployer vite, en toute sécurité**. C’est ici qu’intervient le **CI/CD** : **Intégration Continue** (CI) et **Déploiement Continu** (CD).

Le principe est simple : à chaque fois que vous poussez du code sur GitHub ou GitLab, un ensemble de tests s’exécute automatiquement, puis l’application est déployée sur un serveur de test ou de production — **sans intervention manuelle**.

Prenons un cas concret : vous travaillez sur une application de gestion de stock pour une pharmacie à Abidjan. Chaque modification de code doit être testée, containerisée avec Docker, puis déployée sur un serveur de préproduction.

Avec **GitHub Actions**, vous pouvez automatiser tout cela grâce à un fichier comme celui-ci :

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t pharmacie-app:latest .

      - name: Run tests
        run: docker run pharmacie-app:latest npm test

      - name: Deploy to server
        run: |
          ssh user@votre-serveur.com "
            docker pull votre-docker-hub/pharmacie-app:latest
            docker stop pharmacie-app || true
            docker rm pharmacie-app || true
            docker run -d --name pharmacie-app -p 80:3000 votre-docker-hub/pharmacie-app:latest
          "
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

Ce script fait exactement ce que vous feriez manuellement — mais **automatiquement**, **rapidement**, et **sans erreur humaine**.

Des alternatives comme **GitLab CI/CD** ou **Jenkins** offrent des fonctionnalités similaires, avec des interfaces graphiques parfois plus accessibles pour les débutants.

---

### Déploiement sur le cloud : AWS, Azure, Google Cloud

Jusqu’ici, vous avez probablement déployé vos applications Docker sur un VPS (comme ceux proposés par OVH, DigitalOcean ou Scaleway). Mais le **cloud public** (AWS, Azure, Google Cloud) offre des avantages considérables pour les développeurs africains :

- **Évolutivité** : montez en puissance en quelques clics.
- **Fiabilité** : des centres de données redondants dans le monde entier.
- **Services managés** : pas besoin de gérer soi-même la base de données ou le cache.
- **Prix à l’usage** : payez seulement ce que vous consommez.

Par exemple, si vous développez une application de suivi agricole au Kenya, vous pouvez :
- Héberger votre API sur **AWS Elastic Beanstalk** (qui supporte Docker),
- Stocker vos données dans **Amazon RDS** (base de données gérée),
- Servir vos images via **Amazon S3**,
- Et tout cela peut être déployé depuis GitHub via **AWS CodePipeline**.

Google Cloud propose même des crédits gratuits pour les étudiants et startups en Afrique. Azure a lancé des centres de données au Maroc et en Afrique du Sud, réduisant la latence pour les utilisateurs locaux.

Le cloud n’est plus un luxe : c’est un levier stratégique pour innover rapidement, même avec un petit budget.

---

### Docker au cœur du DevOps

Toutes ces technologies — Kubernetes, CI/CD, cloud — convergent vers une même philosophie : **le DevOps**. Ce n’est pas un outil, mais une **culture** qui rapproche les développeurs (Dev) et les équipes d’exploitation (Ops).

Avec Docker, vous avez déjà fait un grand pas : vous livrez une application **identique** en local, en test et en production. C’est ce qu’on appelle **"infrastructure as code"**.

Maintenant, en ajoutant CI/CD, vous automatiserez les tests et déploiements. Avec Kubernetes, vous garantissez la disponibilité. Et avec le cloud, vous gagnez en flexibilité.

Le développeur africain moderne ne code plus seul dans son coin. Il fait partie d’un **pipeline automatisé**, où chaque changement de code peut devenir une fonctionnalité en ligne en quelques minutes.

---

### Ressources gratuites pour continuer

Vous souhaitez aller plus loin ? Voici une sélection de ressources **gratuites** et accessibles, spécialement choisies pour le public africain francophone :

- **Kubernetes** :  
  - [Kubernetes officiel - Tutoriels interactifs](https://kubernetes.io/fr/docs/tutorials/)  
  - [Katacoda](https://www.katacoda.com/) (environnements pratiques en ligne)

- **CI/CD** :  
  - [GitHub Learning Lab](https://lab.github.com/) – cours gratuits sur GitHub Actions  
  - [GitLab CI/CD pour les débutants](https://docs.gitlab.com/ee/ci/) (bien documenté en français)

- **Cloud** :  
  - [AWS Educate](https://aws.amazon.com/fr/education/awseducate/) – crédits gratuits pour étudiants  
  - [Google Cloud Skills Boost](https://www.cloudskillsboost.google/) – parcours gratuits avec badges

- **Formations en français** :  
  - [OpenClassrooms](https://openclassrooms.com) – parcours DevOps et cloud  
  - [Udemy (avec promotions fréquentes)](https://www.udemy.com) – cherchez "Docker Kubernetes CI/CD"  
  - [YouTube : Graven](https://www.youtube.com/@Gravenilvectuto) – tutoriels clairs sur Docker et CI/CD

- **Communautés africaines** :  
  - Rejoignez des groupes comme **AfroDevOps** sur LinkedIn ou Telegram  
  - Participez aux meetups tech à Dakar, Abidjan, Ouagadougou ou Casablanca

---

### Votre plan de montée en compétence

Voici un plan simple, sur 6 mois, pour passer de débutant Docker à développeur DevOps :

| Mois | Objectif |
|------|---------|
| 1 | Maîtriser Docker Compose et déployer une app sur un VPS |
| 2 | Apprendre GitHub Actions : automatiser les tests et déploiements |
| 3 | Découvrir le cloud : créer un compte AWS/Azure, déployer un conteneur |
| 4 | Installer Minikube, déployer une app simple sur Kubernetes |
| 5 | Intégrer CI/CD + Docker + cloud dans un projet personnel |
| 6 | Participer à un projet open source ou contribuer à une startup tech africaine |

Chaque étape vous rapproche d’un profil très recherché : un développeur qui **livre vite, bien, et à grande échelle**.

---

## Points clés

- **Kubernetes** permet d’orchestrer des conteneurs Docker à grande échelle, avec mise à l’échelle automatique et haute disponibilité.
- Le **CI/CD** (avec GitHub Actions ou GitLab CI) automatise les tests et déploiements, réduisant les erreurs et accélérant la livraison.
- Les **clouds publics** (AWS, Azure, Google Cloud) offrent des infrastructures flexibles, fiables et accessibles, même avec un petit budget.
- Docker est le socle du **DevOps**, une culture qui combine développement, automatisation et opérations.
- Des **ressources gratuites en français** existent pour apprendre Kubernetes, CI/CD et le cloud.
- Un **plan de progression clair** sur 6 mois vous permet de passer de Docker à un profil DevOps complet.
- En Afrique, ces compétences ouvrent des portes vers les startups, les freelances internationaux, et les carrières tech à l’export.

Vous avez maintenant les bases pour continuer. Docker n’était qu’un départ. Le monde du développement moderne vous attend — et il a besoin de vos talents.