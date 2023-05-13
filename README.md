# Docker 101

Docker : Hands-on Workshop 
 
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
 
Vérifier l’installation : 

```$ docker –version ```

# Exécuter un programme en utilisant Docker :

```$ docker run hello-world ```
# Create a Dockerfile, Build a Docker Image : 

```
$ mkdir my-docker-project 
$ cd my-docker-project 
```

Next, create a new file in this directory named Dockerfile with the following content: 

```
# Use an official Python runtime as a parent image 
FROM python:3.8-slim-buster 
  
# Set the working directory in the container to /app 
WORKDIR /app 
  
# Add the current directory contents into the container at /app 
ADD . /app 
  
# Install any needed packages specified in requirements.txt 
RUN pip install --no-cache-dir -r requirements.txt 
  
# Make port 80 available to the world outside this container 
EXPOSE 80 
  
# Define environment variable 
ENV NAME World 
  
# Run app.py when the container launches 
CMD ["python", "app.py"] 
```
Next, create a requirements.txt file with any Python packages your application needs. For example:

```flask```
And finally, create a simple app.py Flask application : 
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

Now, you can build your Docker image with the docker build command, 
This command tells Docker to build a Docker image from the Dockerfile in the current directory and to tag (-t) the image with the name my-python-app.

```$ docker build -t my-python-app . ```

You can then run a container from your new image with the docker run command:

``` $ docker run -p 4000:5000 my-python-app```


# Run, Inspect, Stop and Remove a Docker Container:
You can run a Docker container from an image using the docker run command. Here's an example:

```
$ docker run -p 4000:5000 my-python-app
```
You can inspect the details of a running Docker container using the docker inspect command.
This command takes the name or ID of a container and returns a large JSON object with all the configuration information of the container.

Here's how you can use it:

``` $ docker inspect <container_id> ```

You can stop a running Docker container using the docker stop command:

``` $ docker stop <container_id> ```

Once a container is stopped, it's not automatically removed unless you ran it with the --rm option. To remove a stopped container, use the docker rm command:

``` $ docker stop <container_id> ```


# Create a Docker Bridge Network and Connect Two Containers

1.Create a Network: First, create a network using the docker network create command. We'll name our network "my-network":

``` 
$ docker network create my-network
```

2. Run Containers Inside the Network
Next, let's run two containers inside this network. We'll use the busybox image, which is a small image that's useful for debugging:

```
docker run -d --name container1 --network my-network busybox sleep 3600
docker run -d --name container2 --network my-network busybox sleep 3600
```

These commands tell Docker to run two containers, named "container1" and "container2", in the background. 
They will run the sleep 3600 command, which just keeps them alive for an hour (3600 seconds).

Now, you can use the docker exec command to run a command inside one of the containers. We'll run a ping command from "container1" to "container2":

```
docker exec container1 ping -c 4 container2
```

This command tells Docker to execute the ping -c 4 container2 command inside "container1". The ping command will send 4 packets to "container2".

# Create a Docker Compose File and Run a Multi-Container Application

Let's create a simple multi-container application using Docker Compose. We'll create a Python Flask app and a Redis database in separate containers.
First, create a directory for your project and navigate into it:

``` $ mkdir my-compose-app && cd my-compose-app ```

Next, create a Dockerfile with the following content:

``` 
# Use an official Python runtime as a parent image
FROM python:3.7-slim

# Set the working directory in the container to /app
WORKDIR /app

# Add the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Run app.py when the container launches
CMD ["python", "app.py"]

``` 

Create a requirements.txt file with the following content:

``` 
flask
redis
```

And a simple app.py Flask application:

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

Now, create a docker-compose.yml file:

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

Start multi-container app

```$ docker-compose up ```

if everything is set up correctly, you should be able to visit localhost:5000 in your web browser and see a message: "Hello World! I have been seen 1 times."

Refresh the page, and the number should increment, demonstrating that data is being stored in the Redis container and retrieved by the Flask application in the web container.


