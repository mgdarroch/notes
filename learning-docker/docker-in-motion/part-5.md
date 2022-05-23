__How to create a container from an image__

Creating containers from a Docker image is usually simple, but the number of options that can be supplied can make things quite complex.

To create a container, we need to specify a command that will run within it. This could be aterminal shell, a MySQL prompt, or whatever the application that you have installed within theimage. To do this, yhou need to run the Docker command `docker run`. 

The command is split into four chunks: First, we supply the options to the container. Then we pass the image we want the container to be built from. After that, we supply the comand that we want the container to run. Finally, we supply the arguments that are passed to the container. 

The number of options we can supply to the container is HUGE. The simplest thing we can do is to type `docker run <repository>/<image_name>:<tag>`.

Use `docker ps -a` and check the `COMMAND` field to see what command your container is executing to start.

Important options are `-i` which keeps STDIN open even if the container is not attached.

`-d` will detach or run the contianer in the background. Very useful when the Docker container is running its own command using a command instruction.

`-t` emulates a terminal shell, so when used with `-i` we can start typing inside the container.  

So if we supply `-t` and `-i`, we will get an interactive shell. If we just supply `-d` the container would be a background process.


__How to name a container__

Docker picks mental randomly generated names for containers. You can rename containers with `docker rename`

    docker rename existing_name new_name

We can also pass a name in with the `docker run` command with the `--name` option.

__How to stop a container__

To stop a container we use the `docker stop` command. We can add a delay to the stop with the `-t` option or `--time`.

If the time option isn't provided the contianer is stopped immediately.

    docker stop <container-name>

Simple.


__How to start a container__

We start a container with `docker start`.  There are a few options we can apply, like attaching standard output, standard error and standard input.

    docker start <container-name>

You can start many containers at once using `docker ps -a` and passing that to the `docker start`.

__How to restart a container__

We can restart a container with `docker restart`

__How to run a command against a container__

You can append commands to the end of the `docker run` command because `bash` is on the path.

__How to get inside a container__ 

The previous method would require us to start a new container everytime we wanted to run the command. What if we want to run a command inside a container that is already running?

We use the `docker exec` command. So, we can do `docker exec -i -t <container-name> bash` and this will let us into the container inside a bash shell where we can execute things.

We can also use the `-u` property to specify the user who will be running the command of the `docker exec` command. `docker exec -it -u <username> <container-name> bash`

You can also pass in environment variables with the `-e` property.

