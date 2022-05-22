## Working with Kubernetes Objects

Here we'll learn about fundamental Kubernetes objects: Pods, Deployments and Services.

We'll also learn about Helm.

When you ran this command:

    kubectl run demo --image=YOUR_DOCKER_ID/myhello --port=9999 --labels app-demo
     
What went on under the hood was that the `kubectl run` command created a Kubernetes resource called a `Deployment`. 

## Deployments

If we ran our demo app with Docker, the `docker container run` command will start a container, and it will run until killed with `docker stop`.

But what if the container exits for some other reason; maybe the program crashed, or there was a system error, or your machine ran out of disk space, or a cosmic ray hit your CPU at the wrong moment.

Assuming this is a production applicatoin, you now have unhappy users until someone can get to a terminal and type `docker container run` to start the container again.

What you really want is a kind of supervisor program, which continually checks that the container is running, and if it ever stops, starts it again immediately. On traditional servers, you can use a tool like `systemd` or `runit`, or `supervisord`. Docker has something similar too. 

In Kubernetes, the supervisor feature is called a `Deployment`.

## Supervising and Scheduling

For each program that Kubernetes has to supervise, it creates a corresponding `Deployment` object, which records some information about the program: the name of the container image, the number of replicas you want to run, and whatever else it needs to know to start the container.

Working together with the `Deployment` resource is a kind of Kubernetes component called a *`controller`*. `Controllers` watch the resources they're responsible for, making sure they're present and wroking.  If a given Deployment  isn't running enough replicas, the controller will create some new ones. The controller makes sure that the real state matches the desired state.

A Deployment doesn't manage replicas directly: instead, it automatically creates an associated object called a ReplicaSet, which handles that. We'll talk more about ReplicaSets in a moment. 

## Restarting Containers

The behaviour of Deployment might be a little strange: if your container finishes its worka nd exits, the Deployment will restart it. If it crashes, or if you terminate it with kubectl, the Deployment will restart it (conceptually).

We can specify restart policies for individual containers if we want, the default behaviour is to restart always.

A Deployments job is to watch its associated containers and make sure that the specified number of them is always running. If there are fewer, it will start more. If there are too many, it will terminate some.

## Querying Deployments

You can see all the Deployments active in your namespace by running the following command:

    kubectl get deployments

To get more detailed info on a specific Deployment:

    kubectl describe deployments/<name>

A Deployment contains the information Kubernetes needs to run the container, but what is a Pod Template and what is a Pod?

## Pods

A `Pod` is the Kubernetes object that represents a group of one or more containers. 

Why doesn't a Deployment just manage an individual container? The answer is that sometimes a set of containers needs to be scheduled together, running on the same node, and communicating locally, perhaps sharing storage.

For example, a blog application might have one container that syncs content with a Git repository, and an Nginx web server container that serves the blog content to users. Since they share data, the two containers need to be scheduled together in a Pod.

In practice though, many Pods only have one container.

So a Pod specification (*spec* for short) has a list of Containers. In the following example there is only one, *demo*:

    demo:
      Image:        <REPOSITORY>/<IMAGE>:<TAG>
      Port:         8888/TCP
      Host Port:    0/TCP
      Environment:  <none>
      Mounts:       <none>

The `Image` spec will be a Docker image, and together with the port number, that is all the info the Deployment needs to start the Pod and keep it running.

And that's an important point. The `kubectl run` command didn't actually create the Pod directly. Instead it created a Deployment and then the Deployment started the Pod. The Deployment is a declaration of your desired state: "A Pod should be running with the myhello container inside it."

## Replica Sets

We said that Deployments start Pods, but there's a little more to it than that. In fact Deployments don't manage Pods directly. That's the job of the ReplicaSet object.

A ReplicaSet is responsible for agroup of identical Pods, or *replicas*. If there are too few or too many Pods, compared to the specification, the ReplicaSet controller will start or stop some Pods to rectify the situation.

Deployments in turn manage ReplicaSets, and control how the replicas behave when you update them.

When you update a Deployment, a new ReplicaSet is created to manage the new Pods, and when the update is completed, the old ReplicaSet and its Pods are terminated.

## Maintaining Desired State

Kubernetes controllers continually check the desired state specified by each resource against the actual state of the cluster, and make any necessary adjustments to keep the min sync. This process is called the *reconciliation loop*, because it loops forever, trying to reconcile the actual state with the desired state.

For example, when you first create the `demo` Deployment, there is no