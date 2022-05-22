__Understanding Docker Containers and Images__

`docker container rm -f $(docker container ls -aq)` - use this command to cleanup containers after testing things out.

`docker container run` - This command tells Docker to run an application in a container. This has aready been packaged to run in Docker and has been published on a public site that anyone can access.

The container package is what Docker calls an __"image"__

__What is a container?__

A Docker container is the same idea as a physical container -- think of it like a box with an application in it. Inside the box, the application seems to have a computer all to itself: it has its own machine name and IP address, and it also has its own disk drive.

These are all virtual resources -- the hostname, IP address, and filesystem are all created by Docker. They're logical objects that are managed by Docker, and they're all joined together to create an environment where an application can run. That's the __"box"__ of the container.

