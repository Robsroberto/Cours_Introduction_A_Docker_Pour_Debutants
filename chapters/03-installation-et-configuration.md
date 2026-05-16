## Installation de Docker sur Windows

### Prérequis et choix de version

Avant d’installer Docker sur Windows, il est essentiel de vérifier que votre système répond aux exigences minimales. Docker Desktop nécessite Windows 10 ou 11 Pro, Enterprise ou Education (64 bits) avec le support de la virtualisation activé dans le BIOS/UEFI. Pour les utilisateurs de Windows 10/11 Famille, WSL2 (Windows Subsystem for Linux version 2) est une solution adaptée et souvent recommandée, surtout dans les contextes africains où les machines peuvent être moins puissantes.

Si vous êtes étudiant ou développeur débutant avec un ordinateur personnel bas de gamme, WSL2 est une excellente alternative. Elle permet d’exécuter Linux directement sur Windows sans machine virtuelle lourde, ce qui réduit la consommation de ressources. Elle est particulièrement utile dans les régions où la bande passante est limitée ou où les mises à jour sont coûteuses.

### Étapes d’installation avec WSL2

1. **Activer WSL2** :  
   Ouvrez PowerShell en tant qu’administrateur et exécutez :
   ```powershell
   wsl --install
   ```
   Cette commande installe automatiquement WSL, une distribution Ubuntu par défaut, et WSL2 en tant que noyau.

2. **Redémarrer l’ordinateur**  
   Après l’installation, redémarrez votre machine pour que les modifications prennent effet.

3. **Mettre à jour WSL2**  
   Une fois redémarré, ouvrez Ubuntu depuis le menu Démarrer, créez un utilisateur Linux, puis mettez à jour le système :
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

4. **Installer Docker Desktop**  
   Téléchargez Docker Desktop depuis [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop). Lors de l’installation, cochez l’option "Use the WSL 2 based engine". Docker s’intégrera alors nativement à votre environnement WSL2.

5. **Lancer Docker Desktop**  
   Après l’installation, démarrez Docker Desktop. L’icône apparaîtra dans la barre des tâches. Attendez que le message "Docker Desktop is running" s’affiche.

## Installation de Docker sur macOS

### Téléchargement et installation

macOS offre une expérience utilisateur fluide avec Docker Desktop. Le processus est simple et convient parfaitement aux développeurs africains utilisant des MacBook, même anciens modèles.

1. **Téléchargez Docker Desktop pour Mac**  
   Rendez-vous sur le site officiel et téléchargez l’installateur `.dmg`.

2. **Installez l’application**  
   Ouvrez le fichier `.dmg`, faites glisser l’icône Docker vers le dossier Applications.

3. **Lancez Docker**  
   Allez dans Applications et cliquez sur Docker. La première fois, macOS demandera l’autorisation d’installer des composants. Acceptez et saisissez votre mot de passe administrateur.

4. **Attendez l’initialisation**  
   Docker démarre en arrière-plan. Une fois prêt, l’icône dans la barre des menus devient bleue.

### Première vérification

Ouvrez le Terminal (dans Utilitaires) et tapez :
```bash
docker --version
```
Vous devriez voir une réponse comme :
```
Docker version 24.0.7, build afdd53b
```

Cela confirme que Docker est correctement installé.

## Installation sur Linux (Ubuntu)

### Pourquoi Ubuntu ?

Ubuntu est l’une des distributions Linux les plus accessibles, largement utilisée en Afrique francophone, notamment dans les centres de formation et les startups tech. Sa communauté active et sa documentation abondante en font un choix idéal pour les débutants.

### Étapes d’installation

1. **Mettre à jour les paquets système** :
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Installer les dépendances nécessaires** :
   ```bash
   sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
   ```

3. **Ajouter la clé GPG officielle de Docker** :
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. **Ajouter le dépôt Docker à APT** :
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. **Installer Docker Engine** :
   ```bash
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io -y
   ```

6. **Vérifier le statut du service Docker** :
   ```bash
   sudo systemctl status docker
   ```
   Vous devez voir "active (running)".

## Configuration des permissions utilisateur

Par défaut, seuls les utilisateurs root ou ceux du groupe `docker` peuvent exécuter des commandes Docker. Pour éviter d’utiliser `sudo` à chaque fois, ajoutez votre utilisateur au groupe `docker` :

```bash
sudo usermod -aG docker $USER
```

**Important** : Déconnectez-vous puis reconnectez-vous pour que le changement prenne effet. Vous pouvez vérifier votre appartenance avec :
```bash
groups
```
Le mot `docker` doit apparaître dans la liste.

## Vérification de l’installation

Quelle que soit votre plateforme, testez Docker avec un conteneur simple :

```bash
docker run hello-world
```

Si tout fonctionne, vous verrez un message d’accueil expliquant que Docker est opérationnel. Ce conteneur est léger (moins de 5 Ko) et parfait pour les zones à faible bande passante.

## Solutions alternatives pour machines limitées

### Podman : Docker sans démon

Podman est un outil compatible avec Docker mais qui fonctionne sans démon centralisé. Il est plus léger, plus sécurisé, et idéal pour les machines avec peu de RAM ou de processeur.

Pour installer Podman sur Ubuntu :
```bash
sudo apt install podman -y
```

Podman peut exécuter les mêmes commandes que Docker :
```bash
podman run hello-world
```

Et grâce à un alias, vous pouvez même utiliser `docker` comme commande :
```bash
alias docker=podman
```

Ajoutez cette ligne à votre fichier `~/.bashrc` pour la rendre permanente.

### Docker sans interface graphique (mode serveur)

Pour les développeurs travaillant sur des serveurs distants ou des machines tête-de-liste (headless), Docker Engine peut être installé en mode serveur via SSH. Cela permet de gérer les conteneurs à distance sans consommer de ressources locales.

Exemple d’installation sur un serveur Ubuntu distant :
```bash
ssh user@votre-serveur-africain.com
# Puis suivre les étapes d’installation Linux ci-dessus
```

Vous pouvez ensuite exécuter vos applications web (comme une API en Node.js ou un site WordPress) sur un serveur localisé en Afrique, réduisant la latence pour vos utilisateurs.

## Premières commandes de base

Une fois Docker installé, voici quelques commandes essentielles à maîtriser :

- Lister les conteneurs en cours :
  ```bash
  docker ps
  ```

- Lister tous les conteneurs (actifs et arrêtés) :
  ```bash
  docker ps -a
  ```

- Lister les images téléchargées :
  ```bash
  docker images
  ```

- Supprimer un conteneur :
  ```bash
  docker rm <nom_ou_id>
  ```

- Arrêter un conteneur :
  ```bash
  docker stop <nom_ou_id>
  ```

Ces commandes sont universelles, que vous soyez au Sénégal, en Côte d’Ivoire ou au Maroc.

## Bonnes pratiques de sécurité

- **Évitez d’ajouter trop d’utilisateurs au groupe `docker`** : un accès Docker équivaut à un accès root.
- **Gardez Docker à jour** : les nouvelles versions corrigent souvent des failles de sécurité.
- **Utilisez des images officielles** : privilégiez `nginx:alpine` plutôt que des images tierces non vérifiées.

## Points clés

- Docker peut être installé sur Windows (avec WSL2), macOS et Linux (notamment Ubuntu), adaptés aux réalités technologiques africaines.
- WSL2 est une solution légère et efficace pour les utilisateurs Windows avec des ressources limitées.
- L’ajout de l’utilisateur au groupe `docker` évite l’usage répété de `sudo`.
- La commande `docker run hello-world` est le test universel pour confirmer une installation réussie.
- Podman est une alternative gratuite, légère et sécurisée à Docker, particulièrement utile sur les machines anciennes.
- Les commandes de base (`ps`, `images`, `stop`, `rm`) sont essentielles pour gérer vos conteneurs au quotidien.
- La sécurité commence par une configuration propre : mise à jour régulière, utilisation d’images fiables et gestion prudente des permissions.

À la fin de ce chapitre, vous disposez d’un environnement Docker fonctionnel, sécurisé et adapté à votre contexte. Vous êtes prêt à passer à l’étape suivante : lancer et gérer vos premiers conteneurs.