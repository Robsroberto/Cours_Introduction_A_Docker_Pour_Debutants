## Installation de Docker sur Windows

Installer Docker sur Windows dépend fortement du type de machine que vous possédez, notamment en ce qui concerne le processeur et la version de Windows. Pour les développeurs en Afrique, où certains utilisent encore des ordinateurs plus anciens, il est essentiel de bien comprendre les prérequis.

### Vérifiez les prérequis

Docker Desktop pour Windows nécessite :
- Windows 10 ou 11 Pro, Enterprise ou Education (64 bits)
- Au moins 4 Go de RAM
- Virtualisation activée dans le BIOS

Beaucoup de machines vendues en Afrique n’ont pas Windows Pro, mais la version Home. Dans ce cas, Docker Desktop **ne fonctionnera pas directement**. Heureusement, il existe une alternative : **WSL2 (Windows Subsystem for Linux)**.

### Installation via WSL2 (solution recommandée pour les machines limitées)

WSL2 permet d’exécuter Linux directement sur Windows, même sur des PC avec peu de ressources. C’est une excellente option pour les développeurs africains utilisant des ordinateurs abordables.

Ouvrez PowerShell en tant qu’administrateur et exécutez :

```powershell
wsl --install
```

Cette commande installe automatiquement WSL2 et Ubuntu. Redémarrez votre machine si nécessaire.

Une fois WSL2 installé, rendez-vous sur le [site officiel de Docker](https://www.docker.com/products/docker-desktop) et téléchargez **Docker Desktop pour Windows**. Lors de l’installation, cochez l’option "Use WSL 2 based engine".

Après installation, ouvrez Docker Desktop. Il détectera automatiquement WSL2 et démarrera.

> **Astuce pour connexion lente** : Si vous avez une connexion Internet instable, téléchargez le fichier d’installation sur un autre réseau (cybercafé, lieu public) via clé USB, ou utilisez un gestionnaire de téléchargement comme **Free Download Manager**.

### Tester l’installation

Ouvrez un terminal WSL (tapez `wsl` dans PowerShell) et lancez :

```bash
docker --version
```

Vous devriez voir quelque chose comme :
```
Docker version 24.0.7, build afdd53b
```

Ensuite, testez le fonctionnement avec :

```bash
docker run hello-world
```

Si vous voyez un message de bienvenue, Docker fonctionne correctement.

---

## Installation de Docker sur macOS

macOS offre une installation plus simple, mais attention aux anciens modèles de Mac, fréquents dans certaines régions.

### Prérequis

- Mac avec processeur Apple Silicon (M1, M2) ou Intel
- macOS 12 ou supérieur
- Au moins 4 Go de RAM

Les Mac plus anciens (avant 2012) ne sont plus supportés. Si vous avez un Mac ancien, envisagez une machine virtuelle (voir solution alternative plus bas).

### Téléchargement et installation

Allez sur [docker.com](https://www.docker.com/products/docker-desktop) et téléchargez **Docker Desktop pour Mac**. Ouvrez le fichier `.dmg`, puis faites glisser l’icône Docker dans le dossier Applications.

Lancez Docker depuis Applications. Au premier démarrage, il peut prendre quelques minutes, surtout sur des machines avec peu de RAM.

### Test du fonctionnement

Ouvrez le terminal et tapez :

```bash
docker --version
docker run hello-world
```

Si le conteneur s’exécute et affiche un message de bienvenue, l’installation est réussie.

> **Conseil pour les connexions limitées** : Sur certains réseaux africains, le téléchargement peut être lent. Téléchargez Docker sur un réseau plus rapide (université, espace coworking) et transférez-le via clé USB.

---

## Installation de Docker sur Linux

Linux est le système le plus adapté à Docker, surtout sur des serveurs ou machines anciennes. Cette méthode est idéale pour les développeurs utilisant des distributions légères comme **Ubuntu**, **Linux Mint**, ou **Xubuntu**.

### Sur Ubuntu (ou dérivées)

Ouvrez un terminal et mettez à jour les paquets :

```bash
sudo apt update && sudo apt upgrade -y
```

Installez les dépendances nécessaires :

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

Ajoutez la clé GPG officielle de Docker :

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Ajoutez le dépôt Docker :

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Mettez à jour à nouveau :

```bash
sudo apt update
```

Installez Docker Engine :

```bash
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

### Gestion des permissions utilisateur

Par défaut, Docker nécessite `sudo` pour chaque commande. Pour éviter cela, ajoutez votre utilisateur au groupe `docker` :

```bash
sudo usermod -aG docker $USER
```

Déconnectez-vous et reconnectez-vous pour que le changement prenne effet.

### Testez l’installation

```bash
docker run hello-world
```

Si le message s’affiche sans `sudo`, tout fonctionne.

> **Attention aux anciens matériels** : Si vous utilisez un PC avec 2 Go de RAM ou moins, privilégiez une distribution légère comme **Lubuntu** ou **Puppy Linux**, et évitez d’exécuter trop de conteneurs en même temps.

---

## Solutions alternatives pour machines anciennes ou contraintes

Dans plusieurs pays africains, les développeurs utilisent des ordinateurs de bureau limités. Voici des solutions adaptées.

### Utiliser une machine virtuelle

Si Docker ne fonctionne pas nativement, installez une **machine virtuelle** avec **VirtualBox** et une version légère de Linux (ex : Ubuntu Server).

1. Téléchargez [VirtualBox](https://www.virtualbox.org)
2. Installez une machine virtuelle Ubuntu (2 Go de RAM suffisent)
3. Installez Docker à l’intérieur (comme vu plus haut)

> **Avantage** : Vous isolez Docker du système principal, ce qui évite les ralentissements.

### Utiliser un serveur distant low-cost

Certaines startups africaines utilisent des serveurs VPS à bas prix (ex : **DigitalOcean**, **Hetzner**, **VPS Congo**, **Afrihost**). Vous pouvez installer Docker sur un serveur distant et y développer via SSH.

Exemple de VPS abordable :
- 1 vCPU, 1 Go RAM, 25 Go SSD
- Coût : ~3 000 FCFA / mois

Commande pour se connecter :

```bash
ssh utilisateur@ip_du_serveur
```

Puis installez Docker comme sur Linux.

> **Avantage** : Vous utilisez une machine performante, même avec un vieux PC local.

---

## Configuration avancée : Optimisation pour les environnements limités

### Réduire les ressources utilisées par Docker Desktop

Sur Windows ou macOS, Docker Desktop consomme beaucoup de RAM. Pour les machines avec 4 Go ou moins :

1. Ouvrez Docker Desktop
2. Allez dans **Settings** > **Resources**
3. Réduisez :
   - **Memory** : 2 Go maximum
   - **CPUs** : 2 cœurs
   - **Disk image size** : 16 Go

Cela évite que Docker ralentisse tout le système.

### Nettoyer régulièrement les conteneurs inutilisés

Les conteneurs et images non supprimés occupent de l’espace. Utilisez ces commandes régulièrement :

```bash
# Supprimer les conteneurs arrêtés
docker container prune

# Supprimer les images non utilisées
docker image prune -a

# Nettoyage complet (attention : supprime tout sauf les conteneurs en cours)
docker system prune -a
```

### Utiliser des images légères

Privilégiez les images basées sur **Alpine Linux** (très légères) plutôt que sur Ubuntu.

Exemple :
```bash
# Image lourde (100 Mo+)
docker run ubuntu:22.04 echo "Bonjour"

# Image légère (~5 Mo)
docker run alpine:latest echo "Bonjour"
```

Dans un projet africain de gestion scolaire, par exemple, utiliser une image Alpine pour le backend réduit le temps de téléchargement et l’occupation mémoire.

---

## Résolution des problèmes courants

### Problème : « Cannot connect to the Docker daemon »

Cela signifie que le service Docker n’est pas lancé.

**Solution Linux** :
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

**Solution Windows/macOS** : Redémarrez Docker Desktop.

### Problème : « Permission denied » sans sudo

Votre utilisateur n’est pas dans le groupe `docker`.

**Solution** :
```bash
sudo usermod -aG docker $USER
```
Puis reconnectez-vous.

### Problème : WSL2 ne démarre pas

Dans PowerShell :
```powershell
wsl --update
wsl --shutdown
wsl
```

---

## Points clés

- Docker peut être installé sur **Windows, macOS et Linux**, mais les prérequis varient.
- Sur **Windows Home**, utilisez **WSL2** pour exécuter Docker efficacement.
- Sur **Linux**, l’installation via le dépôt officiel est la plus fiable.
- Pour les **machines anciennes ou connexion lente**, privilégiez :
  - Les images légères (ex: Alpine)
  - Les machines virtuelles
  - Les serveurs VPS à bas coût
- Ajoutez votre utilisateur au groupe `docker` pour éviter d’utiliser `sudo`.
- Testez toujours avec `docker run hello-world` après installation.
- Nettoyez régulièrement les conteneurs et images inutilisés pour gagner de l’espace.
- Ajustez les ressources allouées à Docker Desktop pour éviter les ralentissements.

À la fin de ce chapitre, vous avez Docker opérationnel, quelle que soit votre configuration. Vous êtes prêt à passer à la pratique : **lancer vos premiers conteneurs**.