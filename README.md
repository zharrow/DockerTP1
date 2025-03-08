# DevOps Training - Docker
by *Florent DETRES*

---

## Étape 3 : Exécuter un serveur web dans un container Docker

### a. Récupérer l'image sur Docker Hub

Pour Apache :

```
docker pull httpd
```

[Docker Hub](https://hub.docker.com/_/httpd)

### b. Vérifier que l'image est disponible en local

Commande :

```
docker image ls
```

[Docker Image](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)

### c. Créer un fichier `index.html`

Commande :

```
mkdir html
echo "Hello World" > html/index.html
```


### d. Démarrer un nouveau container et servir le fichier HTML via un montage absolu

Commande pour Apache :

```
docker run -d -p 80:80 -v $(pwd)/html:/usr/local/apache2/htdocs httpd
```

[Documentation Volume](https://docs.docker.com/engine/storage/volumes/)<br>
[Docker Run](https://docs.docker.com/reference/cli/docker/container/run/)

### e. Supprimer le container

Liste des commandes :

1. Obtenir l’ID du container actif :
   ```
   docker ps
   ```
2. Stopper le container :
   ```
   docker stop <container_id>
   ```
3. Supprimer le container :
   ```
   docker rm <container_id>
   ```

[Docker Copy](https://docs.docker.com/reference/cli/docker/container/cp/)<br>
[Docker Remove](https://docs.docker.com/reference/cli/docker/container/rm/)

### f. Relancer le container sans option `-v` en utilisant `docker cp`

1. Lancer le container :
   ```
   docker run -d -p 80:80 httpd
   ```
2. Copier le fichier dans le container :
   ```
   docker cp html/index.html <container_id>:/usr/local/apache2/htdocs/index.html
   ```


## Étape 4 : Builder une image Docker

### a. Création d’une image avec un Dockerfile

Contenu du fichier `Dockerfile` (Apache) :

```
FROM httpd
COPY ./html/index.html /usr/local/apache2/htdocs/
```
[Ecrire un DockerFile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/)<br>
[Reference DockerFile](https://docs.docker.com/reference/dockerfile/)

Commande pour builder l’image :

```
docker build -t custom-httpd .
```

### b. Exécution de l’image

Commande :

```
docker run -d -p 80:80 custom-httpd
```

### c. Différences entre les procédures (mount volume vs copy)

**Montage de volume (`-v`)** :

- **Avantages** :
  - Permet de modifier les fichiers en local sans avoir à reconstruire l’image.
  - Pratique pour le développement.
- **Inconvénients** :
  - Dépend du système de fichiers hôte.
  - Peut poser des problèmes de permissions ou de performances.

**Copie avec `docker cp` ou via Dockerfile** :

- **Avantages** :
  - Produit une image autonome et portable.
  - Idéal pour la production.
- **Inconvénients** :
  - Toute modification des fichiers nécessite une reconstruction de l’image.

---

## Étape 5 : Utiliser une base de données dans un container Docker

### a. Récupérer les images depuis Docker Hub

Pour MySQL :

```
docker pull mysql
```

[MySQL sur Docker Hub](https://hub.docker.com/_/mysql/)

Pour phpMyAdmin :

```
docker pull phpmyadmin/phpmyadmin
```

### b. Exécuter deux containers à partir des images

1. Créer un réseau Docker :
   ```
   docker network create my-net
   ```

[Créer un réseau Docker](https://docs.docker.com/reference/cli/docker/network/create/)

2. Lancer un container MySQL :
   ```
   docker run -d --name <msqyl-container-name> --network <network-name> -e MYSQL_ROOT_PASSWORD=root mysql
   ```
3. Lancer un container phpMyAdmin :
   ```
   docker run -d --name <phpmyadmin-container> --network <network-name> -p 8080:80 -e PMA_HOST=mysql-container phpmyadmin/phpmyadmin
   ```
4. Accéder à phpMyAdmin via le navigateur à l'adresse : `http://localhost:8080`

---

## Étape 6 : Utilisation de docker-compose.yml

### a. Avantages de docker-compose.yml par rapport aux commandes Docker classiques

- **Commandes Docker classiques** :

  - Nécessitent de lancer chaque container individuellement.
  - La gestion des réseaux et des variables d'environnement peut être fastidieuse.

- **Fichier docker-compose.yml** :

  - Permet de définir les services (containers), réseaux, et volumes dans un seul fichier.
  - Simplifie le déploiement grâce à une seule commande pour démarrer ou arrêter tous les services.

### b. Commandes pour gérer les containers via docker-compose

- **Lancer les containers** :
  ```
  docker-compose up -d
  ```
- **Arrêter les containers** :
  ```
  docker-compose down
  ```

### c. Exemple de fichier docker-compose.yml pour MySQL et phpMyAdmin

```yaml
version: '3.8'
services:
  mysql:
    image: mysql
    container_name: mysql-container
    environment:
      MYSQL_ROOT_PASSWORD: root
    networks:
      - db-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin-container
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mysql-container
    networks:
      - db-network

networks:
  db-network:
    driver: bridge
```
