## Créer vos propres images avec Dockerfile

Vous savez désormais lancer des conteneurs à partir d’images existantes, mais pour développer vos propres applications de manière reproductible, vous devez apprendre à créer vos propres images. C’est ici que le **Dockerfile** entre en jeu. Un Dockerfile est un fichier texte qui contient une suite d’instructions permettant de construire une image Docker étape par étape. Chaque instruction crée une couche dans l’image, ce qui permet une construction rapide et efficace grâce à la mise en cache.

### Comprendre la structure d’un Dockerfile

Un Dockerfile commence toujours par une instruction `FROM` et se termine généralement par `CMD` ou `ENTRYPOINT`. Voici les instructions essentielles que vous devez maîtriser :

#### FROM : Définir l’image de base

Toute image Docker dérive d’une image de base. L’instruction `FROM` permet de spécifier cette base. Elle est obligatoire et doit être la première ligne du Dockerfile.

```dockerfile
FROM node:18-alpine
```

Ici, on part d’une image légère basée sur Alpine Linux, contenant Node.js version 18. Le choix d’une image légère comme `alpine` réduit la taille finale de l’image, ce qui est crucial pour une livraison rapide, surtout si vous déployez depuis un pays où la bande passante est limitée.

#### RUN : Exécuter des commandes pendant la construction

L’instruction `RUN` permet d’exécuter des commandes dans le conteneur pendant la phase de construction. Elle est utilisée pour installer des dépendances, créer des répertoires, ou configurer l’environnement.

```dockerfile
RUN apk add --no-cache curl
```

Dans cet exemple, on installe `curl` sur une image Alpine. L’option `--no-cache` évite de conserver les métadonnées du gestionnaire de paquets, ce qui réduit la taille de l’image.

#### COPY : Copier des fichiers locaux dans l’image

Une fois les outils installés, vous devez copier le code de votre application dans l’image. C’est le rôle de `COPY`.

```dockerfile
COPY . /app
WORKDIR /app
```

Ici, tous les fichiers du répertoire courant (.) sont copiés dans `/app` à l’intérieur de l’image. Ensuite, `WORKDIR` définit ce répertoire comme le répertoire de travail par défaut pour toutes les commandes suivantes.

> 💡 **Astuce** : Pour améliorer les performances, copiez d’abord le fichier de dépendances (`package.json` pour Node.js) avant de copier tout le code. Cela permet de tirer parti du cache Docker si vous modifiez uniquement le code, pas les dépendances.

#### WORKDIR : Définir le répertoire de travail

Comme vu ci-dessus, `WORKDIR` change le répertoire courant dans le conteneur. C’est une bonne pratique de l’utiliser systématiquement pour éviter les chemins relatifs complexes.

```dockerfile
WORKDIR /var/www/html
```

Cela est particulièrement utile pour les développeurs PHP sur des projets locaux, comme une application de gestion scolaire ou un site de e-commerce local.

#### ENV : Définir des variables d’environnement

Les variables d’environnement permettent de configurer l’application sans modifier le code. L’instruction `ENV` les définit de manière permanente dans l’image.

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```

Ces variables peuvent ensuite être utilisées dans vos scripts ou par votre application. Par exemple, une API développée à Dakar ou à Abidjan peut utiliser `PORT` pour écouter sur un port spécifique, sans avoir à le coder en dur.

#### EXPOSE : Indiquer les ports utilisés

L’instruction `EXPOSE` indique à Docker que le conteneur écoutera sur un port spécifique à l’exécution. Elle ne publie pas le port, elle le documente seulement.

```dockerfile
EXPOSE 3000
```

Pour que le port soit accessible depuis l’hôte, il faudra toujours utiliser `-p` lors du `docker run`.

#### CMD : Définir la commande par défaut

`CMD` spécifie la commande qui s’exécutera lorsque le conteneur démarrera. Il ne peut y avoir qu’un seul `CMD` dans un Dockerfile.

```dockerfile
CMD ["node", "server.js"]
```

Cette forme, appelée *exec form*, est recommandée car elle permet à Docker de gérer correctement les signaux (comme `SIGTERM` lors d’un `docker stop`).

> ⚠️ **Attention** : Ne confondez pas `CMD` et `RUN`. `RUN` s’exécute pendant la construction, `CMD` au démarrage du conteneur.

### Exemple complet : Containeriser une application Node.js simple

Imaginons une application Node.js développée par un jeune développeur à Yaoundé. Elle affiche "Bienvenue sur mon API locale !" sur le port 3000.

Voici la structure du projet :

```
mon-api/
├── server.js
├── package.json
└── Dockerfile
```

Contenu de `server.js` :

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Bienvenue sur mon API locale !');
});

app.listen(port, () => {
  console.log(`Serveur en écoute sur le port ${port}`);
});
```

Contenu de `package.json` :

```json
{
  "name": "mon-api",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.0"
  },
  "scripts": {
    "start": "node server.js"
  }
}
```

Et maintenant, le **Dockerfile** :

```dockerfile
# Image de base légère
FROM node:18-alpine

# Créer le répertoire d'application
WORKDIR /app

# Copier les dépendances d'abord
COPY package*.json ./
RUN npm install --production

# Copier le code source
COPY . .

# Définir les variables d'environnement
ENV NODE_ENV=production
ENV PORT=3000

# Exposer le port
EXPOSE 3000

# Commande de démarrage
CMD ["npm", "start"]
```

### Construire, taguer et inspecter l’image

Une fois le Dockerfile prêt, place à la construction :

```bash
docker build -t mon-api:1.0 .
```

- `-t mon-api:1.0` : permet de **taguer** l’image avec un nom et une version.
- Le `.` indique que le contexte de construction est le répertoire courant.

> 🔄 **Astuce Empire du Web** : Utilisez des tags significatifs comme `1.0`, `latest`, ou `dev` pour mieux gérer vos versions.

Pour voir l’image créée :

```bash
docker images
```

Vous devriez voir `mon-api` dans la liste.

Pour inspecter les détails de l’image :

```bash
docker inspect mon-api:1.0
```

Cela affiche des informations techniques comme les couches, les variables d’environnement, ou la commande par défaut.

### Bonnes pratiques à retenir

1. **Utilisez des images de base légères**  
   Privilégiez `alpine`, `slim`, ou `distroless` pour réduire la taille et les vulnérabilités.

2. **Minimisez le nombre de couches**  
   Combine les commandes `RUN` avec `&&` pour éviter des couches inutiles.

   ```dockerfile
   RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
   ```

3. **N’exécutez pas en root**  
   Créez un utilisateur non privilégié pour des raisons de sécurité :

   ```dockerfile
   RUN addgroup -g 1001 -S appuser && \
       adduser -u 1001 -S appuser -G appuser
   USER appuser
   ```

4. **Utilisez un fichier .dockerignore**  
   Comme `.gitignore`, il exclut des fichiers inutiles (comme `node_modules`, `.git`, logs) du contexte de construction.

   ```
   node_modules
   .git
   .env
   ```

### Erreurs fréquentes et comment les éviter

- **Erreur : "file not found" lors du COPY**  
  Vérifiez que le fichier existe dans le contexte de construction. Docker ne voit que ce que vous lui transmettez via `.`.

- **Erreur : L’application ne démarre pas**  
  Vérifiez que la commande dans `CMD` correspond bien au binaire disponible. Utilisez la forme tableau (`["node", "app.js"]`) pour éviter les problèmes de shell.

- **Image trop lourde ?**  
  Vérifiez les couches inutiles, utilisez des images de base légères, et nettoyez les caches dans les commandes `RUN`.

- **Port non accessible ?**  
  N’oubliez pas que `EXPOSE` ne publie pas le port. Utilisez `-p 3000:3000` dans `docker run`.

### Lancer le conteneur pour tester

```bash
docker run -d -p 3000:3000 --name api-container mon-api:1.0
```

Accédez à `http://localhost:3000` – vous devriez voir le message de bienvenue.

### Points clés

- Un **Dockerfile** permet de créer une image personnalisée de manière automatisée et reproductible.
- Les instructions `FROM`, `RUN`, `COPY`, `WORKDIR`, `ENV`, `EXPOSE` et `CMD` sont fondamentales.
- Construire une image légère et sécurisée passe par le choix d’une base légère, l’éviction des privilèges root, et l’usage de `.dockerignore`.
- Le tagging (`-t`) permet de mieux organiser vos images.
- Les erreurs courantes viennent souvent d’un mauvais chemin de copie, d’un port non publié, ou d’une commande mal formulée.
- Tester chaque étape avec `docker build` et `docker run` est essentiel pour valider votre Dockerfile.

Vous êtes maintenant capable de transformer n’importe quelle application en image Docker. Dans le prochain chapitre, nous verrons comment partager des données entre conteneurs grâce aux **volumes**, et comment les faire communiquer via des **réseaux Docker**.