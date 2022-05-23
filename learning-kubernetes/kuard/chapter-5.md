## Pods

You'll often find that you want to colocate multiple applications into a single atomic unit, scheduled onto a single machine. 

For example, imagine a container servicing web requests and a container synchronisiing the filesystem with a remote Git repo. It might be tempting to wrap both the web-server and the Git-synchroniser into a single container.