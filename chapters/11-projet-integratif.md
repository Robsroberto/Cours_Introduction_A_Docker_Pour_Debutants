## Projet intégratif : Application web complète avec Docker

Vous avez maintenant acquis les fondamentaux de Docker : création d’images, gestion de conteneurs, configuration de réseaux et volumes, et orchestration avec Docker Compose. Il est temps d’appliquer toutes ces compétences dans un projet concret, réaliste, et directement utilisable dans votre portfolio.

Dans ce chapitre, vous allez construire une **application web full-stack** composée d’un frontend en React, d’un backend en Node.js (Express) et d’une base de données MongoDB. Tout sera conteneurisé, orchestré via `docker-compose.yml`, et déployé localement, puis sur un serveur distant (VPS) accessible depuis Internet. Ce projet simule une situation réelle que vous pourriez rencontrer en tant que développeur freelance au Sénégal, développeur full-stack en Côte d’Ivoire, ou membre d’une startup tech au Maroc.

### Préparation de l’environnement de travail

Avant de commencer, assurez-vous que Docker et Docker Compose sont installés sur votre machine. Si ce n’est pas le cas, retournez au Chapitre 3 pour les étapes d’installation.

Créez un dossier projet nommé `app-fullstack-docker` :

```bash
mkdir app-fullstack-docker
cd app-fullstack-docker
```

À l’intérieur, créez trois sous-dossiers :
- `frontend` : pour l’application React
- `backend` : pour l’API Node.js
- `mongodb` : pour les configurations éventuelles de MongoDB

### Étape 1 : Frontend avec React

Générons une application React dans le dossier `frontend`. Utilisez `create-react-app` :

```bash
npx create-react-app frontend
```

Une fois terminé, créez un fichier `Dockerfile` dans `frontend/` :

```Dockerfile
# Utilisation de l'image de base pour React
FROM node:18-alpine

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste du code
COPY . .

# Construire l'application pour production
RUN npm run build

# Serveur léger pour servir le build
FROM nginx:alpine
COPY --from=0 /app/build /usr/share/nginx/html
EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]
```

Ce Dockerfile utilise un *multi-stage build* pour d’abord construire l’application React, puis la servir via Nginx. C’est une bonne pratique pour réduire la taille de l’image finale.

### Étape 2 : Backend avec Node.js

Créez un fichier `server.js` dans le dossier `backend/` :

```javascript
const express = require('express');
const mongoose = require('mongoose');
const app = express();
const PORT = 5000;

// Middleware
app.use(express.json());

// Connexion à MongoDB
mongoose.connect('mongodb://mongo:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Schéma simple
const userSchema = new mongoose.Schema({ name: String });
const User = mongoose.model('User', userSchema);

// Route de test
app.get('/api/users', async (req, res) => {
  const users = await User.find();
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const user = new User({ name: req.body.name });
  await user.save();
  res.status(201).json(user);
});

app.listen(PORT, () => {
  console.log(`Backend en cours d'exécution sur le port ${PORT}`);
});
```

Créez ensuite `backend/Dockerfile` :

```Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5000
CMD ["node", "server.js"]
```

Installez les dépendances dans `backend/` :

```bash
cd backend
npm init -y
npm install express mongoose
```

### Étape 3 : Configuration avec Docker Compose

À la racine du projet (`app-fullstack-docker/`), créez un fichier `docker-compose.yml` :

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
      - NODE_ENV=development
    depends_on:
      - mongo
    networks:
      - app-network

  mongo:
    image: mongo:6
    volumes:
      - mongodb_data:/data/db
    ports:
      - "27017:27017"
    networks:
      - app-network

volumes:
  mongodb_data:

networks:
  app-network:
    driver: bridge
```

Ce fichier orchestre les trois services :
- `frontend` : sert l’application React sur `http://localhost:3000`
- `backend` : expose l’API sur `http://localhost:5000`
- `mongo` : base de données MongoDB persistante grâce à un volume nommé

Remarquez que le backend se connecte à MongoDB via l’URL `mongodb://mongo:27017/myapp`. Le nom `mongo` est résolu automatiquement par Docker grâce au réseau partagé `app-network`.

### Lancement local de l’application

Depuis la racine du projet, lancez :

```bash
docker-compose up --build
```

Docker va :
1. Construire les images `frontend` et `backend`
2. Télécharger l’image `mongo`
3. Créer le réseau et les volumes
4. Démarrer les trois conteneurs

Une fois lancé, ouvrez votre navigateur :
- Frontend : [http://localhost:3000](http://localhost:3000)
- API : [http://localhost:5000/api/users](http://localhost:5000/api/users)

Vous pouvez tester l’API avec un outil comme `curl` ou Postman :

```bash
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Amadou"}'
```

### Déploiement sur un VPS (ex : DigitalOcean, AWS, ou serveur local au Cameroun)

Supposons que vous ayez un VPS sous Ubuntu 22.04 au Maroc ou au Bénin. Voici les étapes :

1. **Connectez-vous à votre VPS via SSH**
   ```bash
   ssh root@votre-ip-vps
   ```

2. **Installez Docker et Docker Compose**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker $USER
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **Transférez votre projet**
   Depuis votre machine locale :
   ```bash
   scp -r app-fullstack-docker root@votre-ip-vps:/root/
   ```

4. **Sur le VPS, allez dans le dossier et lancez**
   ```bash
   cd /root/app-fullstack-docker
   docker-compose up -d
   ```

   L’option `-d` lance les conteneurs en arrière-plan.

5. **Ouvrez les ports dans le pare-feu**
   ```bash
   sudo ufw allow 3000
   sudo ufw allow 5000
   ```

Votre application est maintenant accessible via :
- `http://votre-ip-vps:3000` (frontend)
- `http://votre-ip-vps:5000/api/users` (backend)

### Bonnes pratiques appliquées

Ce projet met en œuvre plusieurs bonnes pratiques vues dans les chapitres précédents :
- **Isolation des environnements** : chaque composant tourne dans son propre conteneur.
- **Persistance des données** : le volume `mongodb_data` garantit que les utilisateurs ne sont pas perdus au redémarrage.
- **Réseau privé** : les services communiquent entre eux via un réseau Docker dédié.
- **Sécurité** : les ports internes ne sont exposés qu’au besoin, et l’application tourne sans privilèges excessifs.
- **Portabilité** : le même code fonctionne localement et sur un VPS, sans modification.

### Utilisation dans un portfolio ou projet personnel

Cette application est un excellent atout à montrer à un recruteur ou à un client. Vous pouvez :
- L’héberger sur un VPS à petit coût (moins de 5€/mois)
- Ajouter un nom de domaine (ex : `monapp-docker.sn`)
- Créer un dépôt GitHub public avec documentation en français
- L’utiliser comme base pour une application de gestion scolaire, de suivi de commandes, ou de réservation d’espaces de coworking à Abidjan ou à Dakar

Par exemple, un développeur à Tunis a utilisé une architecture similaire pour un système de gestion de bibliothèque universitaire, entièrement conteneurisé et facile à maintenir.

## Points clés

- Docker Compose permet d’orchestrer facilement des applications multi-conteneurs.
- Un projet full-stack (React + Node.js + MongoDB) peut être entièrement conteneurisé en quelques fichiers.
- Le même `docker-compose.yml` fonctionne en local et sur un VPS, assurant la cohérence entre les environnements.
- L’utilisation de volumes assure la persistance des données critiques comme les bases de données.
- Déployer sur un VPS est simple une fois Docker installé, et ouvre la porte à des applications accessibles en production.
- Ce projet constitue une preuve concrète de vos compétences Docker, idéale pour votre portfolio ou vos candidatures.

En maîtrisant ce projet, vous passez du statut d’apprenant à celui de praticien. Vous êtes désormais capable de concevoir, développer et déployer des applications modernes, scalables, et professionnelles — exactement ce que recherchent les entreprises et les clients dans l’écosystème tech africain.