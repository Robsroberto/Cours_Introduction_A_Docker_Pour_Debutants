## Créer vos propres images avec Dockerfile

Jusqu’ici, vous avez utilisé des images Docker préexistantes pour lancer des conteneurs. C’est un excellent point de départ, mais pour tirer tout le potentiel de Docker, vous devez apprendre à créer **vos propres images**. C’est là qu’intervient le **Dockerfile** : un fichier texte qui contient une suite d’instructions permettant de construire une image Docker personnalisée, pas à pas.

Le Dockerfile est comme une **recette de cuisine** pour votre application. Il décrit tout ce dont vous avez besoin : l’ingrédient de base (le système d’exploitation ou le runtime), les dépendances à installer, les fichiers à copier, les ports à exposer, et la commande à exécuter au démarrage du conteneur.

### Comprendre la structure d’un Dockerfile

Un Dockerfile commence toujours par une instruction `FROM`. C’est le point de départ de votre image. Toutes les instructions suivantes s’ajoutent par couches par-dessus cette base.

Prenons un exemple concret : vous développez une petite application web en **Node.js** pour un site de gestion de marchés locaux, comme un outil pour suivre les prix des légumes dans les marchés de Dakar ou de Lomé. Vous souhaitez la déployer de manière reproductible sur différents serveurs, y compris sur des machines avec peu de ressources.

Voici à quoi pourrait ressembler votre Dockerfile :

```dockerfile
# Utilisation d'une image légère de Node.js comme base
FROM node:18-alpine

# Création d'un répertoire de travail à l'intérieur du conteneur
WORKDIR /app

# Copie du fichier de dépendances
COPY package.json .

# Installation des dépendances
RUN npm install

# Copie de tous les fichiers du projet
COPY . .

# Exposition du port 3000 (port par défaut de nombreuses apps Node.js)
EXPOSE 3000

# Commande à exécuter au démarrage du conteneur
CMD ["node", "server.js"]
```

Analysons chaque instruction une par une.

### Les instructions clés du Dockerfile

#### `FROM` : Choisir la base de votre image

`FROM` spécifie l’image de base à utiliser. Il est crucial de bien la choisir. Pour une application web simple, préférez une **version alpine** de Node.js, comme `node:18-alpine`. Alpine Linux est une distribution extrêmement légère (moins de 10 Mo), ce qui réduit considérablement la taille de l’image finale.

Cela fait une différence majeure dans les régions où la bande passante est limitée ou coûteuse. Une image de 50 Mo se télécharge bien plus vite qu’une de 500 Mo, surtout sur un réseau 3G.

#### `WORKDIR` : Définir le répertoire de travail

`WORKDIR /app` crée et définit un répertoire de travail à l’intérieur du conteneur. Toutes les commandes suivantes (`COPY`, `RUN`, etc.) s’exécuteront dans ce répertoire. Cela évite les chemins absolus longs et rend le Dockerfile plus lisible.

#### `COPY` : Copier vos fichiers dans l’image

`COPY` permet de transférer des fichiers de votre machine hôte vers l’image. Ici, nous copions d’abord `package.json`, puis nous exécutons `npm install`, avant de copier le reste du code.

**Pourquoi copier `package.json` en premier ?**

Parce que Docker met en cache chaque couche. Si vous modifiez un fichier de code, Docker n’a pas besoin de réinstaller les dépendances à chaque fois, car la couche `npm install` n’est pas invalide tant que `package.json` n’a pas changé. Cela accélère grandement les reconstructions locales et sur serveur.

#### `RUN` : Exécuter des commandes pendant la construction

`RUN npm install` installe les dépendances de votre application. Cette commande s’exécute **pendant la construction de l’image**, pas au démarrage du conteneur. C’est une étape cruciale pour préparer votre environnement.

#### `EXPOSE` : Indiquer les ports utilisés

`EXPOSE 3000` informe Docker que le conteneur écoute sur le port 3000. Ce n’est **pas** une ouverture de port automatique. Pour accéder à l’application depuis l’extérieur, vous devrez toujours utiliser `-p 3000:3000` lors du lancement du conteneur.

#### `CMD` : Commande par défaut au démarrage

`CMD ["node", "server.js"]` définit la commande qui s’exécute quand le conteneur démarre. C’est celle qui lance votre application. Vous pouvez la remplacer au moment du lancement avec un argument passé à `docker run`.

> ⚠️ Attention : `CMD` ne peut être qu’une seule par Dockerfile. Si vous en mettez plusieurs, seule la dernière sera prise en compte.

### Construire et tester votre image

Avec votre Dockerfile prêt, place à la construction. Depuis le répertoire contenant le Dockerfile, exécutez :

```bash
docker build -t mon-app-marche .
```

L’option `-t` permet de **taguer** (nommer) votre image. Le point `.` indique que le contexte de construction est le répertoire courant.

Une fois l’image construite, lancez un conteneur :

```bash
docker run -p 3000:3000 mon-app-marche
```

Votre application est désormais accessible via `http://localhost:3000`.

### Optimiser la taille de vos images

La taille des images est un enjeu crucial, surtout dans les environnements à ressources limitées. Voici des pratiques simples mais efficaces :

#### Utiliser des images de base légères

Privilégiez `alpine` ou `slim` :
- `node:18-alpine` (~120 Mo)
- `node:18` (~900 Mo)

#### Éviter les fichiers inutiles

Utilisez un fichier `.dockerignore` pour exclure les fichiers non nécessaires à la construction, comme `node_modules/`, `.git/`, ou les logs.

Exemple de `.dockerignore` :

```
node_modules
.git
.gitignore
README.md
*.log
```

Cela évite de copier des fichiers inutiles dans le contexte de build, ce qui améliore les performances.

#### Fusionner les commandes RUN quand c’est pertinent

Chaque instruction `RUN` crée une nouvelle couche. Pour réduire le nombre de couches, regroupez les commandes avec `&&` :

```dockerfile
RUN apk add --no-cache curl && \
    npm install --production
```

Ici, `--no-cache` évite de stocker les paquets téléchargés, réduisant encore la taille.

### Erreurs courantes à éviter

- **Oublier de copier les fichiers** : Vérifiez que `COPY . .` est bien présente après `npm install`.
- **Utiliser `CMD` pour installer des dépendances** : Tout ce qui doit s’exécuter pendant la construction doit être dans `RUN`, pas `CMD`.
- **Exposer un port non utilisé** : Assurez-vous que votre application écoute bien sur le port indiqué dans `EXPOSE`.
- **Ne pas taguer l’image** : Sans tag, Docker génère un ID aléatoire, ce qui rend difficile son utilisation ultérieure.

### Bonnes pratiques pour un développement local efficace

Dans les régions où Internet est intermittent, chaque téléchargement compte. Voici quelques conseils :

1. **Construisez localement avant de déployer** : Testez votre Dockerfile sur votre machine avant de l’envoyer sur un serveur distant.
2. **Utilisez des images mises en cache** : Docker réutilise les couches inchangées. Structurez votre Dockerfile pour que les parties stables (comme les dépendances) soient construites en premier.
3. **Préférez les registres locaux ou régionaux** : Si vous travaillez en équipe, envisagez un registry privé dans votre région pour réduire la latence.

### Exemple complet d’application web en Node.js

Voici un exemple minimal d’application `server.js` que vous pourriez utiliser :

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('<h1>Bienvenue sur le Marché Connecté !</h1>');
});

app.listen(3000, () => {
  console.log('Serveur démarré sur le port 3000');
});
```

Avec le `package.json` suivant :

```json
{
  "name": "marche-connecte",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

Ce petit projet, empaqueté en image Docker, peut être déployé sur n’importe quel serveur avec Docker installé — que ce soit un VPS à Abidjan, un serveur local à Yaoundé, ou un cloud international.

## Points clés à retenir

- Le **Dockerfile** est un fichier de recette pour construire une image Docker personnalisée.
- Les instructions `FROM`, `RUN`, `COPY`, `CMD`, et `EXPOSE` sont fondamentales.
- `FROM` doit utiliser une image légère comme `alpine` pour réduire la taille.
- Copiez `package.json` avant les autres fichiers pour profiter du cache Docker.
- Utilisez `.dockerignore` pour exclure les fichiers inutiles.
- La taille de l’image impacte directement les temps de build, de transfert et de déploiement — optimisez-la.
- Testez toujours votre image localement avant de la déployer.
- Une bonne structure de Dockerfile rend votre application portable, rapide à construire, et facile à maintenir.

En maîtrisant le Dockerfile, vous passez du statut d’utilisateur de conteneurs à celui de **créateur d’environnements reproductibles**. C’est une compétence essentielle pour tout développeur moderne, surtout dans des contextes où la fiabilité et l’efficacité sont primordiales.