__Docker Images vs Docker Containers__

Every Docker container is built from a Docker image. You cannot have a container if you do not have an image that it is built from. A Docker container is an instance of an image.

A Docker image is made from one or more layers. Each layer creates a container thatincludes everything built from that layer. Once the layer is created, the next layer is built.  Each layer results in a container, which is the difference between the previous layer and the layer being built. This process continues until all the layers have been created. The end result is the complete image, and the final container is what is used when required.

So, an image is built from many layers, and we can build as many containers as we want from our images.

We can also have images extend other images, by doing so, each child image will include the contents and configuration of the parent images.

So, a container is not an image. It is an instance of an image, and this image can be extended. Each image and container has its own unique ID. This is a SHA-256 hex, but the first 12 characters are normally shown. If you type `docker images` into the cmd line you will see a list of all images.

`docker ps` will show the running containers.

__How to pull Docker Images__

To get a Docker image, we use the `docker pull` command. There are only a few options with it, `-a` which downloadall the tags the image has; and `Skip image verification` which when enabled, allows us to enforce the use of signature checking when downloading Docker images.

Docker image tags are like source control tags. When an image is pushed to a repository,, it can be tagged with a number or a string. This allows different tags to be downloaded, or pulled, from its source repository. If no tag is supplied, the latest tag will be used.

How do we find these images and their tags? This is where __Docker Registries__ come in. A Docker registry is a stateless and scaalable server-sided application that lets you store and distribute your Docker images.

There are two types of registries you can use. The first is aprivate registry, which means that you are in total control of where the images are stored. This also means that you have total control of the distribution of those images. The second is a public registry, such as the Docker Hub. Here you can download and contribute to the Docker images which are shared publicly.

If you need to log into a registry, then use the `docker login` command. This command has two options: the `--password` and the `--username`. You will also need to supply the server address that you are loggin into.  So what happens when the `docker pull` command is given?

Well, first of all, it checks if the image currently exists on the local system. If it finds the image, then it checks if the tag also exists. If the image or its tag doesn't exist, it will attempt to access the HUb or wherever you are logged in to. 

`docker pull nginx` - this will download the latest tag of nginx

`docker pull nginx:1.13.0` - this will download a specific version.

If you have a newer version already downloaded, it will pull the common layers from the already downloaded image.

*__It's important to know that the "Latest" tag is not the same as the "latest version". The "Latest" tag is just a placeholder for an image that doesn't have a tag.__*  

You can think of it more like __*default*__

In most cases when you download the latest tag, you're not actually getting the latest version of that image.

__What is a Docker image layer__

*"Ogres are like onions..." - Shrek*

A Docker image is made up on layers. The more layers a Docker image has, the more data it needs to digest and therefore, it becomes bigger in file size.

Certain parts of the Docker layer create metadata, which hardly consumes file size at all. Meta elements, such as - but not limited to - environment variables and labels, can be added to an image.

The first layer of a Docker image is the base layer, which is given it's own hashed ID. The layers that follow on from that have their own hashes.

These hashes are used to determine which layers already exist on the system. Each layer except the very last one is read-only. And, as they build up, each layer is the difference of the one before it. This keeps the layers nice and light and simple. This is also very handy when pulling layers down a network connection.

What makes up a layer? And how do we see what layers are used?

The contents of a lyaer are defined by a step in a __Dockerfile__. These steps are instructions that tell Docker how to build each layer and, ultimately, how to build the image. So how do we see or list the previous layers used by an image?

If you type the command `docker history <image>`, you will see the information about each layer. The `history` command takes a few options that tailors the output.

We can also inspect Docker objects, including containers and images, using the `docker inspect` command. It defaults to container objects but you can specify the `--type` option to inspect things like images, e.g. `docker inspect --type image nginx:latest`. You can also use the `--format` option to pull things out of the object, for example `--format="{{ .Id }}"` this would pull out the Id of the image. `--format="{{ .Size }}"` would pull out the size of the image and `--format="{{ .Config }}"` pulls out the config and so on.

__How to remove Docker images__

`docker rmi [OPTIONS] IMAGE [IMAGE...]` will remove an image. `-f` forces it and `--no-prune` doesn't delete the untagged parent images.

The `rmi` command removes the topmost image and all underlying images that it relies on, so `--no-prune` means that any parent image not tagged will not be removed.

There is also the `docker image prune` command which has two options. `-a` will remove all unused images, even dangling ones; and the `--force` option will do it without confirmation. But what are dangling images? A dangling image is an intermediate image which is no longer used or referenced by another image.

Because an image is built from image layers, there is a possibility that these layers become unused due to upgrades, and if so, they are classed as dangling. A dangling layer is simply a layer that no other layer depends on. 


