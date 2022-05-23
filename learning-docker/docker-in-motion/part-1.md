`docker run -d -p 8080:80 nginx` 

`docker ps -a` - a list of containers running

`docker images` - a list of images

`docker rm -f <container-name>` - removes a container

`docker rmi <image-name>` - removes an image

Docker is very useful when architecting an application into many microservices.  

__What is Docker Engine__

At the heart of Docker is the Docker Engine.

The engine consists of 3 components: the server, the client and a RESTful API.

The server is a long running daemon called dockerd. It's job is to create and manage Docker objects.

The server listens to requests that are sent to the Docker socket,  this can be mapped to another port or another Unix socket.

The client is a command-line interface called docker, which sends commands to the server via the API. The clinet comands are very singular and prcise, like `docker run`, `docker create` and `restart`; but behind the scenes the Docker API is doing a lot of work.  For example `docker run` will cause the API to create a container, then create an image if it doesn't exist, start the container once the image is built and then attach the container if it's required.

The API is the glue between the server and the client.

Docker is not just about containers; Docker has a host of other tools and services, which are tailored to the day-to-day management of Docker-based applications.