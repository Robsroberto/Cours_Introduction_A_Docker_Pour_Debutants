## Sécurité et bonnes pratiques dans l’utilisation de Docker

L'utilisation de Docker simplifie grandement le développement et le déploiement d'applications, mais elle introduit aussi de nouveaux vecteurs d'attaques si elle n’est pas utilisée avec précaution. La sécurité dans Docker ne doit pas être une réflexion a posteriori. Elle doit être intégrée dès le début, dans chaque étape : création d’images, configuration des conteneurs, déploiement, et maintenance.

Dans les écosystèmes technologiques africains, où les ressources sont parfois limitées — comme la bande passante ou l’accès à des serveurs performants —, il est encore plus crucial d’adopter des pratiques sécurisées. Un conteneur mal configuré peut exposer des données sensibles, ralentir le réseau, ou même permettre l’accès non autorisé à un serveur mutualisé. Ce chapitre vous donne les outils et les réflexes pour éviter ces risques.

### Appliquer le principe du moindre privilège

Le principe du moindre privilège signifie qu’un conteneur ne doit disposer que des permissions strictement nécessaires à son fonctionnement. Par défaut, Docker exécute les conteneurs en tant qu’utilisateur `root`. Cela peut être dangereux : si un attaquant parvient à s’échapper du conteneur (container breakout), il obtient des droits élevés sur l’hôte.

Pour éviter cela, créez un utilisateur non-root à l’intérieur de l’image. Voici un exemple dans un `Dockerfile` :

```dockerfile
FROM nginx:alpine

# Création d'un utilisateur dédié
RUN addgroup -g 1001 -S appuser && \
    adduser -u 1001 -S appuser -G appuser

# Changer le propriétaire des fichiers statiques
COPY --chown=appuser:appuser ./html /usr/share/nginx/html

# Basculer vers l'utilisateur non-root
USER appuser

# Lancer le service
CMD ["nginx", "-g", "daemon off;"]
```

Avec cette configuration, même si une vulnérabilité est exploitée, l’attaquant ne pourra pas modifier des fichiers système ou installer des logiciels sans droits d’administration.

### Utiliser des images fiables et les mettre à jour

Beaucoup de développeurs utilisent des images publiques depuis Docker Hub sans vérifier leur provenance. Certaines images peuvent contenir des logiciels malveillants, des backdoors, ou des dépendances obsolètes. Une bonne pratique consiste à :

- Privilégier les **images officielles** (marquées "Official" sur Docker Hub).
- Utiliser des **tags spécifiques** au lieu de `latest`. Par exemple, préférez `nginx:1.25-alpine` plutôt que `nginx:latest` pour garantir la reproductibilité.
- Mettre régulièrement à jour les images de base pour intégrer les correctifs de sécurité.

Prenons un exemple concret : un développeur à Abidjan déploie une application avec `node:16`. Six mois plus tard, une vulnérabilité critique est publiée dans cette version. Si l’image n’est pas mise à jour, l’application reste exposée, même si le code source est sécurisé.

Vous pouvez automatiser les mises à jour avec des outils comme **Dependabot** (sur GitHub) ou intégrer des scans dans votre CI/CD.

### Scanner les images pour détecter les vulnérabilités

Il existe des outils capables d’analyser vos images Docker et de lister les vulnérabilités connues dans les paquets installés. L’un des plus populaires est **Trivy**, un scanner open source léger.

Voici comment l’utiliser :

```bash
# Télécharger et installer Trivy (Linux)
wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.48.0_Linux-64bit.tar.gz
tar zxvf trivy_0.48.0_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/

# Scanner une image
trivy image my-app:latest
```

Le résultat affichera les vulnérabilités classées par niveau de gravité (LOW, MEDIUM, HIGH, CRITICAL). Vous pourrez alors décider de mettre à jour une dépendance ou de changer de base d’image.

Dans un contexte africain, où la bande passante peut être limitée, il est judicieux de scanner localement avant de pousser l’image vers un registre distant. Cela évite de transférer des images compromises.

### Éviter les fuites de données sensibles

Les variables d’environnement et les fichiers de configuration (comme `.env`) ne doivent jamais être intégrés directement dans une image Docker. Une fois l’image construite, toute personne ayant accès à celle-ci peut extraire ces informations.

Par exemple, ce `Dockerfile` est **dangereux** :

```dockerfile
FROM node:18
COPY . /app
WORKDIR /app
# Mauvaise pratique : les variables sont dans l'image
ENV DB_PASSWORD=supersecret123
RUN npm install
CMD ["npm", "start"]
```

La solution ? Utiliser des **fichiers de secrets** ou des gestionnaires de configuration externes.

En développement, utilisez un fichier `.env` chargé via Docker Compose :

```yaml
version: '3.8'
services:
  app:
    build: .
    env_file:
      - .env
    ports:
      - "3000:3000"
```

Et dans le `.env` (non versionné dans Git) :

```
DB_HOST=db
DB_USER=admin
DB_PASSWORD=mon_mot_de_passe_sécurisé
```

En production, envisagez des outils comme **Hashicorp Vault** ou les secrets natifs de **Docker Swarm**.

### Prévenir les échappatoires de conteneurs

Un échappatoire de conteneur (container escape) survient quand un attaquant parvient à exécuter des commandes sur la machine hôte à partir du conteneur. Cela peut arriver si le conteneur a accès à des fonctionnalités système sensibles.

Pour réduire ce risque :

- **Évitez les conteneurs en mode privilégié** (`--privileged`).
- **Ne montez pas le socket Docker** (`/var/run/docker.sock`) sauf cas très spécifiques.
- **Désactivez les fonctionnalités système inutiles** avec `--cap-drop`.

Exemple de commande sécurisée :

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE -p 80:8080 mon-app
```

Ici, on retire toutes les capacités Linux, puis on ajoute uniquement celle nécessaire pour écouter sur le port 80.

### Audit de sécurité avec Docker Bench Security

**Docker Bench Security** est un script open source qui teste votre environnement Docker contre les meilleures pratiques de sécurité définies par le Centre for Internet Security (CIS).

Pour l’utiliser :

```bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh
```

Le script produit un rapport détaillé avec des points de contrôle comme :
- L’activation du journal de sécurité
- La configuration du daemon Docker
- La présence de conteneurs en mode privilégié

Même dans un environnement restreint — comme un VPS basé au Sénégal avec peu de ressources —, ce scan peut être exécuté ponctuellement pour vérifier la sécurité.

### Bonnes pratiques adaptées aux contextes africains

Dans de nombreux pays africains, les développeurs travaillent avec :
- Des connexions Internet intermittentes
- Des serveurs mutualisés ou partagés
- Un accès limité à des outils cloud coûteux

Voici des conseils pratiques pour ces réalités :

1. **Préférez les images légères** comme `alpine` ou `distroless`. Elles consomment moins de bande passante et de stockage.
2. **Construisez localement, poussez rarement**. Évitez de repousser des images à chaque modification. Utilisez des tags de version clairs.
3. **Formez votre équipe**. Dans un projet collaboratif à Dakar ou à Yaoundé, la sécurité est collective. Partagez les bonnes pratiques via des checklists ou des revues de code.
4. **Utilisez des registres privés locaux** si possible. Certains hubs régionaux ou universitaires proposent des registres Docker sécurisés et rapides.

### Surveillance continue et revue des journaux

La sécurité ne s’arrête pas au déploiement. Il est essentiel de surveiller les conteneurs en production.

Commandes utiles :
```bash
# Voir les logs d’un conteneur
docker logs mon-app

# Surveiller l’activité réseau
docker stats

# Lister les conteneurs en cours
docker ps --no-trunc
```

Automatisez la conservation des logs avec des outils comme **Fluentd** ou **ELK Stack**, même sur un petit serveur. Même un simple script qui sauvegarde les logs quotidiennement peut faire la différence après une intrusion.

## Points clés

- **Ne jamais exécuter de conteneurs en tant que root** : créez un utilisateur dédié dans vos images.
- **Mettez à jour régulièrement vos images de base** pour intégrer les correctifs de sécurité.
- **Scannez vos images** avec des outils comme Trivy avant chaque déploiement.
- **Protégez les données sensibles** : n’intégrez jamais de secrets dans les images Docker.
- **Évitez les privilèges excessifs** : désactivez les capacités Linux inutiles avec `--cap-drop`.
- **Utilisez Docker Bench Security** pour auditer votre configuration.
- **Adaptez vos pratiques** à la réalité locale : limites de bande passante, ressources partagées, accès restreint.
- **Surveillez en continu** : les logs et les performances des conteneurs sont des indicateurs précieux d’activités suspectes.

En appliquant ces principes, vous transformez Docker d’un simple outil de développement en un pilier sécurisé de votre infrastructure. Chez Empire du Web, nous croyons que la sécurité n’est pas un luxe, mais un droit fondamental — surtout dans les écosystèmes émergents où chaque ressource compte.