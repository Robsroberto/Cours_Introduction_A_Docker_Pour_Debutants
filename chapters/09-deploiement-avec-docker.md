## Déployer une application avec Docker sur un serveur

Déployer une application en production n’est plus réservé aux équipes d’ingénieurs DevOps. Grâce à Docker, même un développeur solo basé à Abidjan, Dakar ou Yaoundé peut déployer une application robuste sur un serveur accessible mondialement, en quelques minutes. Ce chapitre vous guide pas à pas dans le déploiement d’une application containerisée sur un serveur Linux, comme un VPS (ex : AWS EC2, DigitalOcean, OVHcloud, ou un serveur local).

Nous allons déployer une API Express.js simple, mais les principes s’appliquent à Laravel, Django, ou tout autre framework. À la fin de ce chapitre, votre application sera accessible via une URL, protégée par un reverse proxy sécurisé, et surveillée de manière basique.

### Préparer le serveur Linux

Commencez par louer un VPS. Pour ce tutoriel, prenons un exemple concret : un serveur Ubuntu 22.04 avec 2 Go de RAM sur DigitalOcean, à 10 $/mois. Après création, connectez-vous via SSH :

```bash
ssh root@votre.ip.du.serveur
```

Mettez à jour le système :

```bash
sudo apt update && sudo apt upgrade -y
```

Installez Docker et Docker Compose :

```bash
curl -fsSL https://get.docker.com | bash
sudo usermod -aG docker $USER
sudo apt install docker-compose -y
```

Déconnectez-vous et reconnectez-vous pour que les permissions Docker prennent effet.

### Transférer votre image Docker

Il existe deux façons principales de transférer une image sur un serveur : via **Docker Hub** ou un **registry privé**. Pour les débutants, Docker Hub est le plus simple.

Sur votre machine locale, taggez votre image avec votre nom d’utilisateur Docker Hub :

```bash
docker tag mon-api-express:v1 votre-pseudo-dockerhub/mon-api-express:v1
```

Puis poussez-la :

```bash
docker login
docker push votre-pseudo-dockerhub/mon-api-express:v1
```

Sur le serveur, récupérez l’image :

```bash
docker pull votre-pseudo-dockerhub/mon-api-express:v1
```

> 🔐 **Conseil sécurité** : Évitez de stocker des secrets (mots de passe, clés API) en dur dans l’image. Utilisez des variables d’environnement ou un fichier `.env` que vous ne poussez jamais sur Internet.

### Lancer l’application en arrière-plan

Ne lancez jamais un conteneur en mode interactif (`-it`) en production. Utilisez le mode détaché avec `-d` :

```bash
docker run -d -p 3000:3000 --name api-prod votre-pseudo-dockerhub/mon-api-express:v1
```

Le conteneur tourne maintenant en arrière-plan. Vérifiez son état :

```bash
docker ps
```

Vous devriez voir `api-prod` en cours d’exécution. Testez l’API localement sur le serveur :

```bash
curl http://localhost:3000/health
```

Cela devrait renvoyer une réponse comme `{ "status": "ok" }`.

Mais l’API est encore exposée sur le port 3000. Ce n’est ni sécurisé ni professionnel. Passons à l’étape suivante.

### Configurer Nginx comme reverse proxy

Nginx permet de rediriger le trafic du port 80 (HTTP) ou 443 (HTTPS) vers votre conteneur. Installez Nginx :

```bash
sudo apt install nginx -y
```

Créez un fichier de configuration pour votre site :

```bash
sudo nano /etc/nginx/sites-available/mon-api
```

Ajoutez cette configuration :

```nginx
server {
    listen 80;
    server_name api.votresite.ci;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Activez le site :

```bash
sudo ln -s /etc/nginx/sites-available/mon-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

> 💡 **Astuce locale** : Si vous n’avez pas de nom de domaine, utilisez un service comme DuckDNS ou pointez un sous-domaine gratuit (ex : `.ml`, `.ga` via Freenom) vers votre IP.

### Sécuriser avec un certificat HTTPS (Let’s Encrypt)

Un site sans HTTPS est vulnérable. Let’s Encrypt fournit des certificats gratuits. Installez Certbot :

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Générez un certificat :

```bash
sudo certbot --nginx -d api.votresite.ci
```

Certbot mettra automatiquement à jour votre configuration Nginx pour utiliser HTTPS. Votre API est maintenant sécurisée.

### Configurer le pare-feu (UFW)

Un pare-feu limite les accès au serveur. Activez UFW :

```bash
sudo ufw enable
```

Autorisez uniquement SSH, HTTP et HTTPS :

```bash
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
```

Vérifiez l’état :

```bash
sudo ufw status
```

Seuls les ports 22 (SSH), 80 (HTTP) et 443 (HTTPS) sont ouverts. C’est bon pour la sécurité.

### Gérer les conteneurs avec Docker Compose (optionnel mais recommandé)

Plutôt que de lancer des commandes `docker run` manuelles, utilisez un `docker-compose.yml` sur le serveur. Créez le fichier :

```yaml
version: '3.8'
services:
  api:
    image: votre-pseudo-dockerhub/mon-api-express:v1
    container_name: api-prod
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    ports:
      - "3000"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

Lancez l’application :

```bash
docker-compose up -d
```

Le `restart: unless-stopped` garantit que le conteneur redémarre après un crash ou un redémarrage du serveur.

### Surveillance basique

Un conteneur peut planter. Utilisez des outils simples pour surveiller.

Voir les logs en temps réel :

```bash
docker logs -f api-prod
```

Voir l’utilisation des ressources :

```bash
docker stats
```

Pour une surveillance avancée, envisagez des outils comme **Portainer** (interface web pour Docker) ou **Netdata** (monitoring système en temps réel).

### Exemple complet : déployer une API Express

Voici un résumé des étapes pour déployer une API Express :

1. **Sur votre machine locale** :
   - Dockerisez l’API avec un `Dockerfile`
   - Testez localement avec `docker-compose up`
   - Poussez l’image sur Docker Hub

2. **Sur le serveur** :
   - Installez Docker et Docker Compose
   - Récupérez l’image : `docker pull`
   - Installez Nginx et configurez le reverse proxy
   - Générez un certificat Let’s Encrypt
   - Activez le pare-feu UFW
   - Lancez l’application avec Docker Compose

3. **Surveillance** :
   - Vérifiez les logs
   - Configurez des alertes simples (ex : scripts de vérification HTTP)

> 🌍 **Cas réel africain** : Une startup à Lomé développe une API de gestion agricole. Elle utilise un VPS local à 5 $/mois, déploie avec Docker, et sécurise avec HTTPS. Les agriculteurs accèdent à l’application via un site web sécurisé, même avec une connexion 3G instable.

### Bonnes pratiques de déploiement

- **Ne jamais exécuter en root** : Créez un utilisateur dédié sur le serveur.
- **Utilisez des tags de version** : Ne poussez jamais une image `latest` en production.
- **Sauvegardez régulièrement** : Sauvegardez les volumes critiques (ex : base de données).
- **Automatisez** : Un jour, vous passerez à GitLab CI/CD ou GitHub Actions pour déployer automatiquement à chaque modification.

## Points clés

- Le déploiement avec Docker repose sur trois piliers : **transport d’image**, **reverse proxy** et **sécurité**.
- Docker Hub est idéal pour débuter ; passez à un registry privé (ex : GitLab Container Registry) pour des projets sensibles.
- Nginx agit comme un bouclier entre Internet et votre conteneur : il gère le HTTPS, le load balancing, et protège contre certaines attaques.
- Le pare-feu UFW doit être activé et configuré pour limiter les accès.
- Un conteneur en production doit être lancé en mode détaché (`-d`) ou via Docker Compose avec `restart: unless-stopped`.
- Let’s Encrypt rend l’HTTPS accessible à tous, même sur un budget limité.
- La surveillance via `docker logs` et `docker stats` est essentielle pour détecter les problèmes tôt.
- Même un développeur solo peut déployer une application professionnelle avec Docker, sans infrastructure complexe.

Vous êtes maintenant capable de déployer une application Docker sur un serveur accessible au monde entier. C’est une compétence puissante, surtout dans les écosystèmes africains où l’accès à des infrastructures cloud est de plus en plus courant. Dans le prochain chapitre, nous approfondirons la sécurité : gestion des utilisateurs, isolation des conteneurs, et bonnes pratiques pour éviter les failles.