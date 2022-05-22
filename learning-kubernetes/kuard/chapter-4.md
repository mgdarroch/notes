## Common kubectl commands

__Namespaces__

Kubernetes uses *namespaces* to organixe objects in the cluster. You can think of each namespace as a fodler that holds a set of objects. By default, the `kubectl` command line tool interacts with the `default` namespace.

If you want to use a different namespace, you can pass `kubectl` the `--namespace` flag.

__Contexts__

If you want to change the default namespace more permanently, you can use a *context*. This gets recorded in a `kubectl` configuration file, usually located at `$HOME/.kube/config`. This configuration file also stores how to both find and authenticate to your cluster.

For example, you can create a context with a different default namespace for your `kubectl`  commands using:

    kubectl config set-context my-context --namespace=mystuff

This creates a new context, but it doesn't actually start using it yet. To use this newly created context, you can run:

    kubectl config use-context my-context

Contexts can also be used to manage different clusters or different users for authenticating to those clusters using the `--users` or `--clusters` flag.

__Viewing Kubernetes API Objects__

Everything contained in Kubernetes is represented by a RESTful resource. 