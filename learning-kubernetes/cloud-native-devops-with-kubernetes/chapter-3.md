## Getting Kubernetes

Kubernetes is the operating system of the cloud native world.

## Cluster Architecture

Kubernetes connects multiple servers into a *cluster*, but what is a cluster and how does it work?

__The Control Plane__ 

The cluster's brain is called *the control plane*, and it runs all the tasks required for Kubernetes to do its job:

- scheduling containers
- managing services
- serving API requests
- and more...
  
The control plane is made up of several components:

- `kube-apiserver` : this is the frontend server for the control plane, handling API requests
- `etcd` : this is the database where Kubernetes stores all its information; what nodes exist, what resources exist on the cluster, and so on.
- `kube-scheduler` : this decides where to run newly created Pods.
- `kube-controller-manager` : this is responsible for running resource controllers, such as Deployments.
- `cloud-controller-manager` : this interacts with the cloud provider (in cloud based clusters), managing resources such as load balancers and disk volumes.
  
The members of the cluster which run the control plane components are called __*master nodes*__.

__Node Components__

Cluster members that run user workloads are called *worker nodes* 
Each worker node in a Kubernetes cluster runs these components:

- `kubelet` - This is responsible for driving the container runtime to start workloads that are scheduled on the node, and monitoring their status.
- `kube-proxy` - This does the networking magic that routes requests between Pods on different nodes, and between Pods and the internet.
- `Container runtime` - This actually starts and stops containers and handles their communications. Usually Docker, but Kubernetes supports other container runtimes, such as `rkt` and `CRI-O`

Other than running different software components, there's no intrinsicc difference between master nodes and worker nodes. Master nodes don't usually run user workloads though, except in very small clusters (like Minikube).

__High Availability__

A correctly configured Kubernetes control plane has multiple master nodes, making it *highly available*; that is, if any individual master node fails or is shut down, or one of the control plane components on it stops running, the cluster will still work properly.

A highly available control planew ill also handle the situation where the master nodes are working properly, but some of them cannot communicate with the others, due to a network failure (known as a network partition).

The `etcd` database is replicated across multiple nodes, and can survive the failure of individual nodes, so long as a quorum of over half the original number of `etcd` replicas are still available.

__Control Plane Failure__

A damaged control plane doesn't necessarily mean that your applications will go down, although it might well cause strange and errant behavior.

For example if you were to stop all the master nodes in your cluster, thte Pods on the worker nodes would keep on running - at least for ahwile. But you would be unable to deploy any new containers or change any Kubernetes resources. Controllers such as Deployments would stop working. 

Therefore, high availability of the control plane is critical to a properly functioning cluster. You need to have enough master nodes available that the cluster can maintain a *quorum* even if one fials; for production clusters, the workableminimum is three. 

__Worker Node Failure__

By contrast, the failure of any worker node doesn't resally matter. Kubernetes will detect the failure and reschedule the node's Pods somewhere else, so long as the control lane is still working.

If a large number of nodes fail at once, this might mean that the cluster no longer has enough resources to run all the workloads you need. FOrtunately, this doesn't happen often, and even if it does, Kubernetes will keep as many of your Pods running as it can while you replace the missing nodes.

https://github.com/helseyhightower/kubernetes-the-hard-way - this is an opinionated walkthrough of the process of building a Kubernetes cluster which os good for getting a sense of how things work under the hood.

