## First steps with Kubernetes

We're going to build a simple containerized application and and deploy it to a local kubernetes cluster running on your machine.

Docker is a container image format, a container runtime library, a cli tool for packaging and running containers and an API for container management.

What exactly is a container image? Think of it like a zip file. It's a single binary file with a unique ID that holds everything needed to run the contianer.

__Docker Port Forwardin__

`docker container run -p 9999:8888 myhello` - This command will open a container which will listen for connections on port 8888, but this is the container's own private port, not a port on your machine!

In order to connect to the container's port 8888, you need to forward a port on your local machine to that port on the container. It can be any port, including 8888, but in this case we're using 9999 to make it clear which is our port and which is the container's port.

To tell Docker to forward a port you use the `-p` flag.

    docker container run -p HOST_PORT:CONTAINER_PORT ...


## Hello Kubernetes

    kubectl run demo --image=YOUR_DOCKER_ID/myhello --port=9999 --labels app-demo

This will start a pod with the container image running in it, you can then setup port forwarding like so:

    kubectl port-forward pod/demo 9999:8888

Then if you connect to localhost:9999 you'll see in your terminal `Handling connection for 9999` strings.

To delete the pod:

    kubectgl delete pod demo

