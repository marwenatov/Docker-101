# Docker-101

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

```docker build -t my-python-app . ```

You can then run a container from your new image with the docker run command:

```docker run -p 4000:5000 my-python-app```


