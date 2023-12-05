# Configuring and Managing Application Access with Ingress

## Ingress architecture
1. Deploy an **Ingress Controller**
2. Create an **Ingress Class** for the Ingress Controller
3. Define **Ingress Resources** referencing the Ingress Class to be used and routing rules for traffic

### Ingress Resource 
- Defines how **HTTP-based** services in a cluster can be accessed externally
    - Rules for how requests are passed into the cluster

### Ingress Controller 💻
- Actual implementation of the Ingress Resource rules
- Provides access to cluster-based services using a HTTP reverse proxy
    - Receives the external connection, finds a matching rule, then passes traffic directly to Service's Pod and points to respond to the request
- Available in many different types:
    - Running as Pods in a cluster (NGINX Ingress Controller)
    - External hardware to the cluster (Citrix, F5)
    - Cloud Ingress Controllers (Azure's AppGW, Google Load Balancer, AWS Load Balancer Ingress)
- Defined, interface spec

### Ingress Class 📋
- Enables to associate an Ingress Resource with a specific Ingress Controller (in scenarios where more than one Ingress Controller exists)
- Defines a default Ingress Class by defining `isDefault` annotation

## Ingress Overview
- Provides **load balancing** directly to endpoints in the process, bypassing the `ClusterIP` Service
- Allows [name-based virtual hosting](01ingressArchitectureOverview.md#name-based-virtual-hosts-with-ingress), enabling access provision to many different services inside of the cluster, based on the `Host` header in the HTTP request
    - Reads the `Host` header and sends request to the correct backend cluster service
- Allows [path-based routing](01ingressArchitectureOverview.md#exposing-multiple-services-with-ingress), allowing to direct requests to unique services based on the path in the URL
    - Granular control on where incoming traffic is routed to
- Allows implementing [TLS termination](01ingressArchitectureOverview.md#using-tls-certificates-for-https-ingress)
    - A certificate configuration can be defined on the Ingress Resource and used to provide encryption from browser to Ingress Controller

### Why Ingress rather than Load Balancers?
- Ingresses functions at **Layer 7**, which allows path-based routing and name-based virtual hosting
- Ingresses also enables to implement higher level capabilities such as URL rerouting, session persistency and dynamic waiting
- Load balancers lack the Ingresses' capabilities and can load balance traffic only on IP and port
- Ingress Controllers are a single resource that provide access to multiple internal services
    - Load balancers expose a single service on a single IP and port
    - Saves cloud money using less resources
- By using an Ingress Controller, latency is reduced since that requests are sent directly to Pod endpoints (rather than kube-proxy then Pod endpoints, across nodes depending on the situation)
- Load balancers are more suited for TCP/UDP-based services (function on **Layer 4**)

#### Exposing a single service with Ingress
- From a [Cluster Network](../01k8sNetworkingFundamentals/01introducingK8sNetwork.md#kubernetes-network-topology) 🟢 standpoint; a cluster running some Pods is front-ended with a `ClusterIP`-based Service 🟢 (`10.1.22.10:80` for instance), access to the Pod-based applications 🔵 is completed with Ingress by implementing the following steps:
1. Defining an Ingress Controller 🔵, then an Ingress Resource defining access rules for outsiders
2. The Ingress Controller exposes a `NodePort` 🟠 (`172.16.94.XY:32235` for example) or `LoadBalancer` (a public IP address on port 80) Service
3. Requests come through the `NodePort` IP 🟠 (or from the `LoadBalancer` Service), and get routed to the Ingress Controller Pod 🔵
4. Ingress Controller routes traffic to the correct Pod endpoint based on what is defined on the Ingress Resource
```
apiVersion: networking.k8s.io/v1
kind: Ingress                                   # where the Ingress is defined
metadata:
  name: ingress-single
spec:
  ingressClassName: nginx                       # Ingress Controller that will implement Ingress Resources
  defaultBackend:                               # accepts all requests when there's no matching rule defined 
    service:                                    # traffic is sent to this single service
      name: hello-world-service-single
      port:
        number: 80
```

#### Exposing multiple services with Ingress
- `ClusterIP` Services 🟢 named Blue and Red are exposed on a cluster with running Pods 🔵
- Ingress Rules must be defined for path-based routing, then an Ingress Controller must be deployed to expose resources on a public IP address
- As HTTP-based traffic comes into the Ingress Controller, path-based routing is used on Ingress Rules to look at the URL request and route traffic to the correct applications based on host header or DNS name
    - If a request comes in on `path.example.com/red`, traffic can be routed to the Red Service, while `path.example.com/blue` routes to the Blue Service
        - If a request comes for anything other than `/red` or `/blue`, clients get a 404 error code unless a `defaultBackend` is defined
- Three different `pathTypes`:
    - `Prefix` is a partial prefix match based on the path (an incoming URL request path beginning with `/red` will match)
    - `Exact` is an exact case-sensitive match to the request
    - `ImplementationSpecific` pushes this configuration's responsibility to the Ingress Class
- Service ports defined for the paths (`4242` and `4343` in the examples) are valid only for the backing `ClusterIP` Services; the Ingress listens on port 80 for incoming HTTP-based requests and routes traffic based on the rules here defined
```
spec:
  ingressClassName: nginx
  rules:                                        # defines how traffic is handled when received by the Ingress Controller
    - host: path.example.com                    # request must contain this hostname for the Ingress to listen on
      http:                                     # rule type
        paths:                                  # collection of paths to route on
        - path: /red
          pathType: Prefix                      # way to define how to match incoming request to a defined path
          backend:                              # where the request will be forwarded to
            service:
              name: hello-world-service-red
              port:
                number: 4242
        - path: /blue
          pathType: Exact                       # Could be either Prefix or Implementation
          backend:                              # where the request will be forwarded to
            service:
              name: hello-world-service-blue
              port:
                number: 4343
  defaultBackend:
    service:
      name: hello-world-service-single
      port:
        number: 80
```

### Name-based virtual hosts with Ingress
- Hosts are defined on Ingress Rules, enabling to route incoming HTTP-based requests based on the host name coming into the Ingress Controller (which is exposed as a `LoadBalancer` service on a public IP address)
- `red.example.com` and `blue.example.com` domains are the name-based virtual hosts
    - DNS `A` records must be defined for those host names
- As requests come in, the Ingress Controller looks at their host headers and distributes traffic to Blue Service (if from `blue.example.com`) or to Red (if from `red.example.com`)
```
spec:
  ingressClassName: nginx
  rules:
  - host: red.example.com
    http:
      paths:
      - pathType: Prefix
        path: "/"                               # all incoming requests are sent to the backend below
        backend:
          service:
            name: hello-world-service-red
            port:
              number: 4242
  - host: blue.example.com
    http:
      paths:
      - pathType: Prefix
        path: "/"                               # all incoming requests are sent to this backend
        backend:
          service:
            name: hello-world-service-blue
            port:
              number: 4343
```

### Using TLS certificates for HTTPS Ingress
- A [certificate](01ingressArchitectureOverview.md#generating-a-tls-certificate) must be stored as a [Secret](/Configuring%20and%20Managing%20Kubernetes%20Storage%20and%20Scheduling/02configurationAsDataEnvironmentVariablesSecretsConfigMaps/02secrets.md) in the cluster
- The TLS certificate will be on the Ingress Controller listening on port 443 by default, encrypting traffic from the HTTP client
    - From the Ingress Controller to the Pod, traffic goes unencrypted, running on port 80
```
spec:
  ingressClassName: nginx
  tls:                                          # where one or many host names are defined in the certificate
  - hosts:
      - tls.example.com
    secretName: tls-secret                      # secret containing the certificate and its private key
  rules:
  - host: tls.example.com                       # must match to the host defined in spec.tls.hosts[]
    http:
      paths:
      - path: "/"
        pathType: Prefix
        backend:
          service:
            name: hello-world-service-single
            port:
              number: 80
```
#### Generating a TLS certificate
1. Generate a certificate: `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=US/ST=ILLINOIS/L=CHICAGO/O=IT/OU=IT/CN=tls.example.com"`
2. Create a secret with the key and certificate: `kubectl create secret tls tls-secret --key tls.key --cert tls.crt`

###### Return to [Summary](README.md)