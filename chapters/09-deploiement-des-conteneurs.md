## Déploiement de conteneurs sur un serveur distant
Le déploiement d'applications Docker sur un serveur distant est une étape cruciale pour mettre vos applications à disposition du public. Dans ce processus, vous allez apprendre à transférer vos images, à configurer Docker sur le serveur, à lancer vos conteneurs en arrière-plan et à utiliser des outils comme Nginx comme reverse proxy.

### Choix d'un fournisseur de serveur
Avant de commencer, vous devez choisir un fournisseur de serveur qui correspond à vos besoins. Il existe de nombreux fournisseurs de serveurs qui offrent des services de qualité, tels que Digital Ocean, AWS, Google Cloud, etc. Pour les développeurs africains francophones, il est recommandé de choisir un fournisseur qui a des régions proches de l'Afrique, comme par exemple Digital Ocean qui a des serveurs à Londres ou à Francfort, ou AWS qui a des régions à Paris ou à Francfort.

### Création d'un serveur Linux
Une fois que vous avez choisi votre fournisseur de serveur, vous devez créer un serveur Linux. La plupart des fournisseurs de serveurs offrent des images de système d'exploitation prêtes à l'emploi, telles que Ubuntu, Debian, CentOS, etc. Pour cet exemple, nous allons utiliser Ubuntu.

```bash
# Création d'un serveur Ubuntu sur Digital Ocean
# Utilisation de la commande doctl pour créer un serveur
doctl compute droplet create --image ubuntu-20-04-x64 --size s-1vcpu-1gb --region fra1 --ssh-keys 12345678 my-ubuntu-server
```

### Installation de Docker sur le serveur
Une fois que votre serveur est créé, vous devez installer Docker sur le serveur. Vous pouvez utiliser la commande suivante pour installer Docker sur Ubuntu :

```bash
# Installation de Docker sur Ubuntu
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce
```

### Transfert des images Docker
Une fois que Docker est installé sur le serveur, vous devez transférer vos images Docker sur le serveur. Vous pouvez utiliser la commande `docker save` pour sauvegarder votre image Docker en local, puis la commande `docker load` pour la charger sur le serveur.

```bash
# Sauvegarde de l'image Docker en local
docker save -o my-image.tar my-image

# Transfert de l'image Docker sur le serveur
scp my-image.tar user@serveur:/home/user/

# Chargement de l'image Docker sur le serveur
docker load -i my-image.tar
```

### Lancement des conteneurs
Une fois que vos images Docker sont sur le serveur, vous pouvez lancer vos conteneurs. Vous pouvez utiliser la commande `docker run` pour lancer un conteneur.

```bash
# Lancement d'un conteneur
docker run -d -p 80:80 my-image
```

### Utilisation de Nginx comme reverse proxy
Pour mettre vos conteneurs à disposition du public, vous devez utiliser un reverse proxy. Nginx est un serveur web populaire qui peut être utilisé comme reverse proxy. Vous pouvez installer Nginx sur votre serveur et configurer le fichier de configuration pour qu'il pointe vers votre conteneur.

```bash
# Installation de Nginx sur Ubuntu
sudo apt install -y nginx

# Configuration de Nginx pour qu'il pointe vers le conteneur
sudo nano /etc/nginx/sites-available/default
```

```nginx
# Fichier de configuration de Nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:80;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Optimisation des coûts
Pour optimiser les coûts, vous pouvez utiliser des fournisseurs de serveurs qui offrent des tarifs compétitifs. Vous pouvez également utiliser des serveurs plus petits et augmenter la taille du serveur à mesure que vos besoins augmentent. Il est également important de surveiller vos coûts et de les ajuster en conséquence.

## Points clés
* Choix d'un fournisseur de serveur qui correspond à vos besoins
* Création d'un serveur Linux
* Installation de Docker sur le serveur
* Transfert des images Docker sur le serveur
* Lancement des conteneurs
* Utilisation de Nginx comme reverse proxy
* Optimisation des coûts en utilisant des fournisseurs de serveurs qui offrent des tarifs compétitifs et en surveillant vos coûts.