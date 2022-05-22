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

Everything contained in Kubernetes is represented by a RESTful resource. We refer to these things as *Kubernetes objects*.  Each Kubernetes object exists at a unique HTTP path; for example. https://your-k8s.com/api/v1/spaces/default/pods/my-pod leads to the representation of a Pod in the `default` namespace named `my-pod`

The `kubectl` command makes HTTP requests to these URLs to access the Kubernetes objects that reside at these paths.

The most basic command for viewing Kubernetes objects via `kubectl` is `kubectl get <resource-name> <obj-name>`, you can leave out the object name if yhou want to get a listing of all resources.

By default, `kubectl` uses a human-readable printer for viewing the responses from the API server which removes many of the details of objects.

If you want to see more details, use the `-o wide` flag, which gives more details.You can also use the `-o json` or `-o yaml` to get json and yaml responses respectively.

A common option for manipulating the output of a `kubectl` is to remove the headers, which is often usefull when combining `kubectl` with Unix pipes (e.g. `kubectl ... | awk ...`).  If you specify `--no-headers`, `kubectl` will  skip the headers at the top of the human-readable table.

Another common task is extracting specific fields from the object. `kubectl` uses the JSONPath query lanaguage to select fields in the return object.

    kubectl get pods my-pod -o jsonpath --template={.status.podIP}

This will extract and print the IP address of the specified Pod.

If you want detailed information about a particular object, use the `describe` command:

    kubectl describe <resource-name> <obj-name>

__Creating, Updating and Destroying Kubernetes Objects__

Objects in the Kubernetes API are represented as JSON or YAML fiels. These files are either returned by the server in response to a query or posted to the server as part of an API request. You can use these YAML or JSON files to create, update, or delete objects on the Kubernetes server.

If you have an object stored in `obj.yaml`, you can use `kubectl` to create this object in Kubernetes by running:

    kubectl apply -f obj.yaml

You don't need to specify the resource type of the object, this is obtianed from the file itself.

Similarly, you can make changes to the `obj.yaml` and use the exact same command to apply these updates to the object.

The apply tool will only modify objects that are different from the current objects in the cluster. If the objects you  are creating already exist in the cluster, it will simply exit successfully without making any changes. 

This makes it useful for loops where you want to ensure the state of the cluster matches the state of the filesystem. You can repeatedly use `apply` to reconcile state. 

You can also use the `--dry-run` flag to print the objects to the terminal without actually sending them over. A way to see what is going to happen without causing any changes.

If you want to make interactive edits instead of editing a local file, you can instead use the `edit` command which will download the latest object state and then launch an editor that contains the definition. After you save the file, it will be automatically uploaded back to the Kubernetes cluster.

The apply command also recrods the history of previous configurations in an annotation within the object. You can manipulate these records with the `edit-last-applied`, `set-last-applied` and `view-last-applied` commands.

    kubectl apply -f myobj.yaml view-last-applied

This will show you the last state that was applied to the object.

When you want to delete an object, you can simply run: 

    kubectl delete -f obj.yaml

Likewise, you can delete an object using the resource type and name:

    kubectl delete <resource-name> <obj-name>

__Labelling and Annotating Objects__

Labels and annotations are tags for your objects. We'll discuss the differences in Chapter 6, but for now, you can update the labels and annotations on any Kubernetes object using the `annotate` and `label` commands. For example, to add the `color=red` label to a Pod named `bar`, you can run:

    kubectil label pods bar color=red

The syntax for annotation is identical. By default, `label` and `annotate` will not let you overwrite an existing label. TO do this you need to add the `--overwrite` flag.

If you want to remove a label, you can use the `<label-name>-` syntax, e.g `color-`:

    kubectl label pods bar color-

__Debugging Commands__ 

`kubectl` also makes a number of commands available for debugging, eg logs:

    kubectl logs <pod-name>

If you have multiple containers in a Pod you can chooose the container to view using the -c flag.

By default `kubectl` lists the current logs and exits, if you want the logs to be streamed you can add the `-f` command-line flag.

You can also use the `exec` command to execute a command in a running container:

    kubectl exec -it <pod-name> -- bash

This will provide you with an interactive shell inside the running container so that you can perform more debugging.

If you don't have bash or some other terminal available within your container, you can always attach it to the running process 

    kubectl attach -it <pod-name>

This will attach to the running process. It's ismilar to `kubectl logs` but will allow you to send input to the running process, assuming that the process is setup to read form standard input.

You can also copy files to and from a container using the `cp` command:

    kubectl cp <pod-name>:</path/to/remote/file> </ath/to/local/file>

This will copy a file from a running container to your local machine.  You can specify directories or reverse the syntax to copy a file from your local machine back out to the container.