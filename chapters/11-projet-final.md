## Projet final : Construire et déployer une application full-stack avec Docker

Vous avez parcouru un chemin significatif depuis vos premiers pas avec Docker. À présent, il est temps de mettre en pratique toutes vos compétences dans un projet complet, réaliste et valorisable. Ce projet vous permettra de containeriser une application full-stack moderne composée d’un frontend React, d’un backend Node.js/Express et d’une base de données PostgreSQL, le tout orchestré via Docker Compose. Vous terminerez par un déploiement sur un VPS avec un serveur Nginx en reverse proxy et un certificat SSL gratuit via Let’s Encrypt.

Ce type d’architecture est aujourd’hui la norme dans les startups tech d’Afrique francophone comme Jumia, Wave ou même des incubateurs comme CTIC Dakar ou ActivSpaces Douala. Maîtriser ce workflow vous donne un avantage concret sur le marché du développement.

### Architecture de l’application

Notre application simule une plateforme de gestion de marchés locaux — une solution utile pour des entrepreneurs africains qui veulent digitaliser les ventes dans les souks ou maquis. Elle comporte trois composants principaux :

- **Frontend (React)** : interface utilisateur accessible via navigateur
- **Backend (Node.js/Express)** : API REST qui gère les produits et les utilisateurs
- **Base de données (PostgreSQL)** : stockage persistant des données

Nous allons créer un fichier `docker-compose.yml` pour orchestrer ces trois services, chacun dans son propre conteneur.

### Étape 1 : Préparer l’environnement local

Commencez par organiser votre structure de dossiers :

```
marche-en-ligne/
├── frontend/
├── backend/
├── nginx/
└── docker-compose.yml
```

Placez vos projets React et Node.js dans les dossiers respectifs. Si vous n’avez pas de code existant, créez une simple API avec Express et une app React basique avec `create-react-app`.

### Étape 2 : Créer les Dockerfiles

#### Dockerfile pour le frontend (React)

Dans `frontend/Dockerfile` :

```dockerfile
# Utilisation de l’image officielle Node.js en version slim
FROM node:18-alpine AS builder

# Création du répertoire de travail
WORKDIR /app

# Copie des dépendances
COPY package*.json ./
RUN npm install

# Copie du code source
COPY . .

# Build de l’application React
RUN npm run build

# Phase finale avec serveur léger
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]
```

#### Dockerfile pour le backend (Node.js)

Dans `backend/Dockerfile` :

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5000
CMD ["node", "server.js"]
```

> 💡 Astuce : Utilisez `.dockerignore` pour éviter de copier `node_modules`, `npm-debug.log`, etc.

#### Configuration de PostgreSQL

Pas besoin de Dockerfile pour PostgreSQL — nous utiliserons l’image officielle dans `docker-compose.yml`.

### Étape 3 : Orchestration avec Docker Compose

Créez un fichier `docker-compose.yml` à la racine :

```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=postgres
      - DB_USER=devuser
      - DB_PASS=devpass
      - DB_NAME=marche_db
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpass
      POSTGRES_DB: marche_db
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  pgdata:
```

> ⚠️ Attention : Ne jamais utiliser des mots de passe en clair en production. Utilisez des `.env` ou des secrets Docker.

### Étape 4 : Configurer Nginx comme reverse proxy

Créez `nginx/nginx.conf` :

```nginx
server {
    listen 80;
    server_name monmarche.sn;  # Remplacer par votre domaine

    location / {
        proxy_pass http://frontend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/ {
        proxy_pass http://backend:5000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Testez localement avec `docker-compose up --build`. Accédez à `http://localhost` pour voir l’application.

### Étape 5 : Déploiement sur un VPS

Choisissez un VPS abordable — des fournisseurs comme **OVH**, **Scaleway** ou **DigitalOcean** sont accessibles depuis l’Afrique avec des tarifs à partir de 5€/mois.

#### Connexion au serveur

```bash
ssh root@votre-ip-vps
```

Installez Docker et Docker Compose :

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
sudo apt install docker-compose -y
```

Transférez votre projet :

```bash
scp -r marche-en-ligne root@votre-ip-vps:/home/marche-en-ligne
```

Sur le VPS, accédez au dossier et lancez :

```bash
cd /home/marche-en-ligne
docker-compose up -d
```

Votre app est maintenant accessible sur `http://votre-ip-vps`.

### Étape 6 : Mettre en place HTTPS avec Let’s Encrypt

Installez **Certbot** pour un certificat SSL gratuit :

```bash
sudo apt install certbot -y
```

Obtenez un certificat (vous devez posséder un nom de domaine, ex: `monmarche.sn`) :

```bash
certbot certonly --standalone -d monmarche.sn
```

> 💡 Astuce : Des domaines `.sn`, `.bf`, `.ci` sont très abordables et disponibles sur des registrars comme **OVH** ou **Gandi**.

Modifiez `nginx.conf` pour activer HTTPS :

```nginx
server {
    listen 443 ssl;
    server_name monmarche.sn;

    ssl_certificate /etc/letsencrypt/live/monmarche.sn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/monmarche.sn/privkey.pem;

    location / {
        proxy_pass http://frontend:3000;
        proxy_set_header Host $host;
    }

    location /api/ {
        proxy_pass http://backend:5000;
        proxy_set_header Host $host;
    }
}

server {
    listen 80;
    server_name monmarche.sn;
    return 301 https://$server_name$request_uri;
}
```

Redémarrez Nginx :

```bash
docker-compose restart nginx
```

Félicitations ! Votre application est désormais sécurisée avec HTTPS et accessible partout en Afrique et dans le monde.

### Bonnes pratiques de déploiement

- **Utilisez des variables d’environnement** : Créez un fichier `.env` pour séparer la configuration du code.
- **Ne jamais exposer les ports internes** : Utilisez Nginx comme point d’entrée unique.
- **Sauvegardez régulièrement vos volumes** : En particulier la base de données PostgreSQL.
- **Surveillez les logs** : `docker-compose logs -f` vous aide à diagnostiquer les erreurs.

### Ajouter ce projet à votre portfolio

Ce projet est parfait pour montrer votre maîtrise de Docker. Ajoutez-le à votre CV, GitHub ou site personnel avec une courte description :

> « Application full-stack containerisée avec Docker, déployée sur un VPS avec Nginx et SSL. Utilisée pour gérer un marché local digital — solution applicable aux PME africaines. »

Vous pouvez aussi en faire une démo vidéo de 2 minutes pour LinkedIn, en expliquant comment Docker simplifie le déploiement — un atout pour décrocher des missions ou des emplois dans les écosystèmes tech africains.

## Points clés

- Un projet full-stack peut être entièrement containerisé avec Docker Compose en séparant les services (frontend, backend, base de données).
- Le fichier `docker-compose.yml` permet de définir les dépendances, les réseaux et les volumes de manière claire.
- Le déploiement sur un VPS nécessite l’installation de Docker, le transfert des fichiers et le lancement des conteneurs en arrière-plan.
- Nginx agit comme reverse proxy pour router le trafic vers les bons services.
- Let’s Encrypt fournit des certificats SSL gratuits, rendant l’application sécurisée et professionnelle.
- Ce projet est directement applicable dans les contextes africains, notamment pour digitaliser les petites entreprises locales.
- En maîtrisant ce workflow, vous êtes prêt à travailler sur des projets réels et à intégrer des équipes de développement modernes.

Ce projet marque une étape clé dans votre formation avec Empire du Web. Vous ne jouez plus avec des conteneurs — vous en maîtrisez la puissance pour construire des applications du monde réel.