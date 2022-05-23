# Principles

Assume that the network is always hostile.

Assume external and internal threats exist on the network at all times

Network locality is not sufficiient for deciding trust in a network.

Every device, user, and workflow is authenticated and authorised.

Policies must be dynamic and calculated from as many sources of data as possible.

__Network Policy API in Kubernetes__

The NetworkPolicy API in Kubernetes allows you to define how you will allow connections between workloads within your cluster.

Calico implements this API.

The Network Policy is defined in yaml:

    apiVersion: v1
    kind: policy
    metadata:
      name: allow-tcp-6379
    spec:
      selector: role == "database"
      ingress:
        - action: allow
          source:
            selector: role == "frontend"
          destination:
            ports: ["6379"]
      egress:
        - action: allow

Here we're saying that for anything labelled with "database" we're going to allow Ingress traffic from sources with the label "frontend" on the port 6379.

It's also allowing egress traffic to anywhere.

__Istio Service Mesh__

- Control Plane + Data Plane (`istio-proxy` / Envoy)
- Traffic management - advanced load balancing, blue/green deployments
- Policies and Telemetry
- Authentication & Encryption - cryptographic identify, mTLS
  
Istio's service mesh mTLS support:
- Global on/off setting
- certificate per serviceAccount
- mTLS encryption done on the Istio Sidecar

The `serviceAccount` serves as the identity for a container inside the mesh. (?)

__Using Istio and Calico together__

With Calico, we take a Network Policy definition which is handled by Calico to figure out how the Policy will look on the host and does things like routing and modifying iptables on the kernel.

With Istio included, Calico takes the same Network Policy, and applies it inside the Istio Proxy itself.

*TLS Policy Using Service Account Names*

    apiVersion: v1
    kind: policy
    metadata:
      name: details
    spec:
      selector: app == "details"
      ingress:
        - action: allow
          source:
            serviceAccounts:
              names: ["productpage"]
      egress:
        - action: allow

Here, we're now allowing incoming connections based on `serviceAccount` names, don't just accept based on the IP address, we're also checking service accounts.  The `serviceAccount` is a cryptographically authenticated identity for that particular endpoint.

You can also attach labels to `serviceAccount`:

    apiVersion: v1
    kind: policy
    metadata:
      name: ratings
    spec:
      selector: app == "ratings"
      ingress:
        - action: allow
          source:
            serviceAccounts:
              selector: ratings == "reader"
      egress:
        - action: allow

Here we're allowing incoming connections from any `serviceAccount` labelled `ratings = reader`

We can also combine `Pod` labels with `serviceAccount` labels in our Policy definitions:

    apiVersion: v1
    kind: policy
    metadata:
      name: ratings
    spec:
      selector: app == "ratings"
      ingress:
        - action: allow
          source:
            podSelector: app == "productpage"
            serviceAccounts:
              selector: ratings == "reader"
      egress:
        - action: allow

In this case we're allowing incoming connections from any `Pod` which is labelled `app=productpage` with a `serviceAccount` labelled `reviews=reader`

You can also extend these policies to a layer 7 level of rules:

    apiVersion: v1
    kind: policy
    metadata:
      name: ratings
    spec:
      selector: app == "ratings"
      ingress:
        - action: allow
          source:
            podSelector: app == "productpage"
            serviceAccounts:
              selector: ratings == "reader"
          http:
            methods: ["GET"]
            paths: ["/ratings/*"]
      egress:
        - action: allow

Here we're saying that `reader` is allowed to talk to `ratings` but it is only allowed to do so if it is over HTTP and `reader` is doing a `GET` on the `/ratings/*` path.

