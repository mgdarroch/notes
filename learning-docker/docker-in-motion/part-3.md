__What is a Dockerfile?__

A Docker image is built from a Dockerfile.

A Dockerffile is a text file with a set of instructions that tells Docker how to build the Docker image.

The sequence of the instructions is very important, as they are performed in order. You cannot install a web servers packages until you've built the foundations for them to be installed upon.

__Dockerfile Structure__

Dockerfiles can be quite complex. Here is an example:

    FROM ubuntu:16.10
    
    LABEL maintainer "Inter Service API Squad"

    RUN apt-get update && apt-get upgrade -y

    RUN apt-get install -y vim

`FROM` - This statement needs to be the first non-comment statement in the Dockerfile. After the `FROM` keyword, we can supply a mixture of values, such as `FROM <image>`, `FROM <image:tag>`, or `FROM <image@digest>`.  This tells the `docker build` command what base image to use; our example is using `ubuntu` tagged at `16.10`:

Essentially the `FROM` statement means that we are extending our custom image, which this Dockerfile is creating from the `ubuntu` image, and therefore the `ubuntu` image becomes our own base image.

After the `FROM` statement, we have a `LABEL`. You can have as many labels as you wish, they are very handy for identifying your Docker objects. A `LABEL` simply adds metadata to the image, and it works like a key-value pair. If Docker encounters a `LABEL` from the base image which is the same as the `LABEL` in this Docckerfile, then the base image's `LABEL` will be overwritten. It's important to remember that these `LABEL`'s can go anywhere inthe Dockerfile.

The `RUN` statement is where can start customising our Docker image. Each `RUN` command will execute its statement against our Docker image. The `&&` symbol means that the command on the right will be executed after the command on the left.

Because these commands are being executed when the Dockerfile is built, we should always pass the `-y` option to commands like `apt-get upgrade` which will say yes to any prompt the command may return.

So, our example has 4 instructions. Some of these instructions will create intermediate layers. The first intermediate layer will pull down `ubuntu:16.10`. `LABEL` does not create a layer but it adds metadata to the overall image. The Dockerfile then create another intermediate layer that updates and upgrades the existing Ubuntu packages. Finally, the last instruction installs the Vim software, so in total, it will create three layers.

Each layer builds a temporary container which runs the instruction. When the instruction is finished and that layer is built, the next temporary container is created.

__Building a Dockerfile__

To build a Dockerfile, we use the `docker build` command.  For example, from the folder that your Dockerfile is located in, you just run: `docker build .` or `docker build ./Dockerfile`


__Dockerfile good practices__

Merge your `RUN` statements together!

    RUN apt-get update \
        && apt-get upgrade -y \
        && apt-get install -y vim

By combining our `RUN` statements, we remove a layer, lowering the size of our image and also avoiding issues with caching that can occur with Docker images. We will ensure that installed packages are updated, this technique is called `cache busting`.

We should also properly define the purpose of our images.  Each container should only have one concern. The image the container is built from should only consist of the packages necessary to perform the functions of a web server. Meaning we can probably remove Vim.

    # A basic Dockerfile with vim
    FROM ubuntu

    LABEL maintainer "Mark Darroch"

    RUN apt-get update \
        && apt-get upgrade -y \
        && apt-get install -y \
        vim \
        nginx \
        php7.0 \
        && rm -rf /var/lib/apt/lists/*


__Dockerfile copying and adding__

Two very useful Dockerfile instructions are `COPY` and `ADD`. These handle file and directory management within a Docker image. Let's take a look at the `COPY` command first.

The `COPY` instruction will copy new files and directories into the Docker image's filesystem. The instruction can be written in two ways, but both require a source and a destination. The source is the location of the __local file or directory__. The destination is the location in the Docker image that you want the file or directory to be added to. __For paths with whitespace, you must put the source and destination in quotes.__

    COPY /code/my* /var/www # anything starting with my
    COPY /code/mysite/page?.html var/www/mysite # page.html pages.html

The `ADD` instruction can do everything that the `COPY` command can do and more. The source of the `ADD` instruction can also be a remote URL. In the following example, the Dockerfile will download the main.css from `some-website.com` and add it to the `mysite` folder with the filename `temp.css`

    ADD http://some-website.com/css/main.css /var/www/mysite/temp.css

The `ADD` instruction can also extract compressed files on the fly. If we used the `COPY` instruction instead of `ADD`, then we would need to add more layers to our Dockerfile to extract the contents and remove the archive. So by using the `ADD` instruction of the `COPY` instruction, we have made our image smaller and more efficient.

There is lots of rules that the `COPY` and `ADD` instructions both follow, so I recommend reading the Docker documentation to get more information. But one rule of thumb that I will mention is something that often catches people out, and that is the build context. 

Both the `ADD` and `COPY` instructions require the source path to be within the context of the build. So, every local file or directory that you wish to include with your Docker image must be within the current working directory. You cannot use a local source directory outside of the scope of the build.

__Dockerfile Environmental Variables__

Another helpful instruction is `ENV`, which stands for environmental variable. This can be handy for manipulating and changing the behaviour of the image when the Dockerfile is building.

`ENV` is a key value pair, where the key is the variable name and the value is the variable's value. For example:

    ENV DOC_ROOT /var/www/mysite-dev

Here the variable name is DOC_ROOT and the value is a document root.

You can also use variables within the Dockerfile, for example:

    ENV DOC_ROOT /var/www/mysite-dev
    
    ADD code/sites/mysite ${DOC_ROOT}

Here we're using `DOC_ROOT` as the destination for this `ADD` instruction.

We can also inject values of environmental variables using Docker Compose and Kubernetes(?)

It's typically good practice to declare all your `ENV` variables at the start of your Dockerfile.

    FROM ubuntu

    LABEL maintainer "Mark Darroch"

    ENV DOC_ROOT /var/www/mysite-dev

    RUN apt-get update \
        && apt-get upgrade -y \
        && apt-get install -y \
        vim \
        nginx \
        php7.0 \
        && rm -rf /var/lib/apt/lists/

    ADD code/sites/mysites ${DOC_ROOT}

We're starting to see the flow of Dockerfiles:

1. First define your `FROM` instruction.
2. Then we add our `LABEL` instructions.
3. Then we add our `ENV` variables.
4. Then we install our required packages with `RUN`.
5. Finally, we add the bits and pieces to build up the image.

<font size=5>__How to pass variables into the build__</font>

We can pass custom variables to the `docker build` command whilst the Dockerfile is being built. This allows us to customise the Docker image as it is being built. 

We do this using the `ARG` instruction.  We can use the argument instruction with the `ENV` instruction but the order is very important.

    ARG VAR_NAME # an argument named VAR_NAME

    ARG VAR_NAME=VAR_VALUE # an argument named VAR_NAME with a default value.

After defining the variable name in the Dockerfile, we then supply the argument during the `docker build` command usiong the `--build-arg` argument.

    docker build --build-arg VAR_NAME=VAR_VALUE

    docker build --build-arg VAR_NAME1=VAR_VALUE1 VAR_NAME2=VAR_VALUE2

Assume you wanted an `ENV` variable, but you wanted to make it so that you could pass in a new value as an argument at build time to override this. The way to do this is to declare the `ARG` first __WITH A DEFAULT VALUE BEING THE ONE YOU WANT YOUR `ENV` VARIABLE TO HAVE__, and then pass that as the value of the `ENV`.  Now, whenever the `ARG` is passed in it will update the ENV.

    FROM ubuntu

    LABEL maintainer "Mark Darroch"

    ENV DOC_ROOT /var/www/mysite-dev
    ARG JQUERY_VERSION=3.2.1
    ENV JQUERY_VERSION ${JQUERY_VERSION}

    RUN apt-get update \
        && apt-get upgrade -y \
        && apt-get install -y \
        vim \
        nginx \
        php7.0 \
        && rm -rf /var/lib/apt/lists/

    COPY code/sites/mysite ${DOC_ROOT}
    ADD https://code.jquery.com/jquery-${JQUERY_VERSION}.min.js ${DOC_ROOT}/js/

Now we can run `docker build` either with or without the arg to pass in, it will default or be overridden. We have total control.

    docker build --build-arg JQUERY_VERSION=3.2.0

This will now update the `ENV` variable to 3.2.0.

There are far more Dockerfile instructions that we can add, but these are easier to demonstrate when we have running containers. We'll see them soon.

