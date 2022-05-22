`kubectl version` - get the version

`kubectl get componentstatuses` - simple cluster diagnostic

`kubectl get nodes` - output a list of nodes on the cluster.

`kubectl describe nodes node-1` - gets a description of a node named node-1

`kubectl get daemonSets --namespace=kube-system kube-proxy` - The kubernetes proxy is responsible for routing network traffic to load-balanced services in the Kubernetes cluster. To do its job, the proxy must be present on every node in the cluster. Kubernetes has an API object named DaemonSet which is used to accomplish this.

`kubectl get deployments --namespace=kube-system core-dns` - Kubernetes also runs a DNS server which providces naming and discovery for the services that are defined in the cluster.

`kubectl get services --namepsace=kube-system core-dns` - THere is also a kubernetes service that performs the load-balancing for the DNS servers.

You can also view a Kubernetes UI through the UI server:

`kubectl get deployments --namespace=kube-system kubernetes-dashboard` - The UI is run as a single replica, but still managed by a kubernetes deployment for reliability

`kubectl get services --namespace=kube-system kubernetes-dashboard` - The dashboard also has a service that performs load balancing for the dashboard

`kubectl proxy` - You can use this command to access the UI.  This will start up a server running on locahlhost:8001.  If you visit http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy

