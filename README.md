# Docker 101

Docker : Atelier pratique

# Installer Docker : 

Linux (Ubuntu):  
```
$ sudo apt-get update 
$ sudo apt-get install docker-ce docker-ce-cli containerd.io 
```
MacOS  
```$ brew install --cask docker (plus simple: installer docker desktop) ```
Windows 
```Installer docker desktop ```
 
Vérifiez l’installation : 

```$ docker –version ```

# Exécuter un programme en utilisant Docker :

```$ docker run hello-world ```
# Créer un Dockerfile, Construire une image Docker : 

```
$ mkdir my-docker-project 
$ cd my-docker-project 
```

Ensuite, créez un nouveau fichier dans ce répertoire nommé Dockerfile avec le contenu suivant : 

```
# Utilisez un runtime Python officiel comme image parente 
FROM python:3.8-slim-buster 
  
# Définissez le répertoire de travail dans le conteneur sur /app 
WORKDIR /app 
  
# Ajoutez le contenu du répertoire actuel dans le conteneur à /app 
ADD . /app 
  
# Installez les packages nécessaires spécifiés dans requirements.txt 
RUN pip install --no-cache-dir -r requirements.txt 
  
# Rendez le port 80 disponible pour le monde extérieur à ce conteneur 
EXPOSE 80 
  
# Définir une variable d'environnement 
ENV NAME World 
  
# Exécutez app.py lorsque le conteneur se lance 
CMD ["python", "app.py"] 
```
Ensuite, créez un fichier requirements.txt avec tous les packages Python dont votre application a besoin. Par exemple:

```flask```
Et enfin, créez une simple application Flask app.py : 
``` 
from flask import Flask 
from os import environ 
  
app = Flask(__name__) 
  
@app.route("/") 
def hello(): 
    return f"Hello {environ.get('NAME', 'World')}!" 
  
if __name__ == "__main__": 
    app.run(host='0.0.0.0') 
```

Maintenant, vous pouvez construire votre image Docker avec la commande docker build, 
Cette commande indique à Docker de construire une image Docker à partir du Dockerfile dans le répertoire actuel et de marquer (-t) l'image avec le nom my-python-app.

```$ docker build -t my-python-app . ```

Vous pouvez ensuite exécuter un conteneur à partir de votre nouvelle image avec la commande docker run:

``` $ docker run -p 4000:5000 my-python-app```


# Exécutez, Inspectez, Arrêtez et Supprimez un Conteneur Docker :
Vous pouvez exécuter un conteneur Docker à partir d'une image en utilisant la commande docker run. Voici un exemple :

```
$ docker run -p 4000:5000 my-python-app
```
Vous pouvez inspecter les détails d'un conteneur Docker en cours d'exécution en utilisant la commande docker inspect.
Cette commande prend le nom ou l'ID d'un conteneur et renvoie un grand objet JSON avec toutes les informations de configuration du conteneur.

Voici comment vous pouvez l'utiliser :

``` $ docker inspect <container_id> ```

Vous pouvez arrêter un conteneur Docker en cours d'exécution en utilisant la commande docker stop :

``` $ docker stop <container_id> ```

Une fois qu'un conteneur est arrêté, il n'est pas automatiquement supprimé à moins que vous l'ayez exécuté avec l'option --rm. Pour supprimer un conteneur arrêté, utilisez la commande docker rm :

``` $ docker rm <container_id> ```


# Créez un réseau Bridge Docker et connectez deux conteneurs

1. Créer un réseau : D'abord, créez un réseau en utilisant la commande docker network create. Nous nommerons notre réseau "my-network" :

``` 
$ docker network create my-network
```

2. Exécutez des conteneurs dans le réseau
Ensuite, lançons deux conteneurs dans ce réseau. Nous utiliserons l'image busybox, qui est une petite image utile pour le débogage :

```
docker run -d --name container1 --network my-network busybox sleep 3600
docker run -d --name container2 --network my-network busybox sleep 3600
```

Ces commandes indiquent à Docker d'exécuter deux conteneurs, nommés "container1" et "container2", en arrière-plan. 
Ils exécuteront la commande sleep 3600, qui les maintient en vie pendant une heure (3600 secondes).

Maintenant, vous pouvez utiliser la commande docker exec pour exécuter une commande à l'intérieur de l'un des conteneurs. Nous lancerons une commande ping de "container1" à "container2" :

```
docker exec container1 ping -c 4 container2
```

Cette commande indique à Docker d'exécuter la commande ping -c 4 container2 à l'intérieur de "container1". La commande ping enverra 4 paquets à "container2".

# Créez un fichier Docker Compose et exécutez une application multi-conteneurs

Créons une simple application multi-conteneurs en utilisant Docker Compose. Nous créerons une application Flask Python et une base de données Redis dans des conteneurs séparés.
D'abord, créez un répertoire pour votre projet et naviguez à l'intérieur :

``` $ mkdir my-compose-app && cd my-compose-app ```

Ensuite, créez un Dockerfile avec le contenu suivant :

``` 
# Utilisez un runtime Python officiel comme image parente
FROM python:3.7-slim

# Définissez le répertoire de travail dans le conteneur sur /app
WORKDIR /app

# Ajoutez le contenu du répertoire actuel dans le conteneur à /app
ADD . /app

# Installez les packages nécessaires spécifiés dans requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Rendez le port 80 disponible pour le monde extérieur à ce conteneur
EXPOSE 80

# Exécutez app.py lorsque le conteneur se lance
CMD ["python", "app.py"]

``` 

Créez un fichier requirements.txt avec le contenu suivant :

``` 
flask
redis
```

Et une simple application Flask app.py :

```  
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return f'Hello World! I have been seen {count} times.\n'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)

```

Maintenant, créez un fichier docker -compose.yml avec le contenu suivant :

``` 
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:80"
  redis:
    image: "redis:alpine"
```

Démarrez l'application multi-conteneurs

```$ docker-compose up ```

Si tout est correctement configuré, vous devriez pouvoir visiter localhost:5000 dans votre navigateur web et voir un message : "Hello World! I have been seen 1 times."

Rafraîchissez la page, et le nombre devrait augmenter, démontrant que les données sont stockées dans le conteneur Redis et récupérées par l'application Flask dans le conteneur web.
