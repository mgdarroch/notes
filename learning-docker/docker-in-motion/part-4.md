__How to quickly remove unused images__

`docker rmi` - previously covered
`docker image rm` - also works when passed an Image ID
`docker images -a` - will show all images including intermediate ones.

`docker images` also takes a `-q` argument which will only return the image IDs. 

`docker images -q` - we can take the output from this and pass it into the `docker rmi`, but it won't remove all images. It also won't work if we use `-a` and `-q` at the same time.

We need to make sure to remove the containers as well, as we can't remove images if they are linked to a container, even if that container isn't running.

`docker ps -q -a` - this will give us a list of all container IDs
`docker rm $(docker ps -q -a)` - this will remove the list of containers. Once you have removed your containers, you can then remove the images like so: `docker rmi $(docker images -q)`

__How to tag your images__

To tag an existing image that has been built already, use the `docker tag` command

    docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

    Create a tag TARGET_IMAGE that refers to a SOURCE_IMAGE

So for example: `docker tag ubuntu:latest webserver:1.0`, this will essentially create a clone of the ubuntu image named webserver.

If you want to tag an image that has no tag or repo, get the Image ID with `docker images` and then use it like so `docker tag <image ID> <image_name:tag>`

If you want to tag an image during the build, you can do it during the `docker build` command using the `-t` option:

    docker build -t <REPOSITORY_NAME>/<IMAGE_NAME>:<TAG> .

    docker build -t my-company/webserver:1.0 .

__How to log in to a Docker registry__

The real benefit of docker comes from pulling down images from remote registries.

So now we'll demonstrate how to log in to remote registries like Docker Hub.

`docker login --help` will show that we need to provide a server, a username and a password to login to.  The server needs to be the full host name.

`docker login` will just default to whatever is defined by the daemon and ask for user input of username and password.

To specify a server it will be: `docker login -u <username> -p <password> <full server hostname>`


__How to push a Docker Image__

We push a Docker image with the `docker push` command.

Firstly, you can check where you're logged into by using the `docker info` command and looking for the `Registry:` property.

    docker push <repository>/<image_name>:<tag>


__How to pull images from the Docker Hub__

Unsurprisingly we pull images with the `docker pull` command.  Make sure you're logged in to your repository first though.

    docker pull <repository>/<image_name>:<tag> 

__How to update the remote images__

To update a remote image, you make your changes locally, perform a `docker build` with the repo, image name and an incremented tag!

    docker build <repository>/<image_name>:<tag++>

Then do a `docker push`:

    docker push <repository>/<image_name>:<tag++>

