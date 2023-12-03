# Kubernetes networking fundamentals

### Introducing the Kubernetes networking model
- Basic communication rules:
    - **All Pods can communicate with each other on all nodes**
        - Anywhere in the cluster, regardless of underlying topology
    - **Agents on a node can communicate with all Pods on that node**
        - Agents = software (like the kubelet or kube-proxy, that need to contact the Pods that they need to orchestrate)
    - **No Network Address Translation (NAT)** 🚫🛜
        - Pods need to communicate with each other (regardless of the node) using its real IP address

## Kubernetes network topology
- **Node network**: 🟠
    - Cloud/database network, where the cluster's nodes are connected to (AWS VPC, for instance)
    - Each node has an IP address assigned from this network, and nodes can reach each other (apart from reaching other resources within this network)
- **Pod network**: 🔵
    - Each Pod has a single IP address assigned, either from the node network or (more commonly) from a pool of IPs called `PodCIDR` range
- **Cluster network** 🟢
    - Used by services using the `clusterIP` Service type
    - IP addresses for Services on this network are allocated from the range specified on its range parameter

## Pod networking and communication
1. Inside a multi-containered Pod: 🔵
    - Containers within it share the same network namespace (implemented by the **Pause/Infrastructure container**), a single IP address and port range for applications to run on
1. Inside a multi-containered Pod:
    - Containers within it share the same network namespace (implemented by the **Pause/Infrastructure container**), a single IP address and port range for applications to run on
        - The Pause/Infrastructure container starts before the creation/start of a Pod, setting up the network namespace for the containers to share
            - It also enables application containers to be restarted without interrupting the network namespace
            - Has the same lifecycle as the Pod (if Pod is deleted, so is the Pause/Infrastructure container)
    - Communication is held over `localhost`
2. Communication between Pods within the same node: 🟠
    - Held by an interface inside each Pod (`eth0`, `veth0`), that is attached to a local software bridge/tunnel interface, to communicate using their actual Pod's IP addresses
3. Communication between Pods from different nodes: 🟢
    - Held by using Pod's real IP addresses
    - Network between those nodes must use a Layer 2/3 (link/network) or overlay (virtual network over network) connectivity
4. kube-proxy implements access to Services by exposing them to both internal and external cluster users (depending upon the [Service type](../02services/01understandingServices.md#service-types))

## Container Network Interface (CNI)
- Implements container and Pod networking in a cluster on an abstracted manner
- Defines a **standardized specification** for managing container networking, used across other container orchestration platforms as well
    - It interacts with the container runtime interface, taking responsibility over operations such as setting up namespaces, interfaces, bridge/tunnel configurations and IP addressing
- Kubernetes uses **CNI plugins** to implement its networking model (like Calico or Cilium, for instance)
    - Such plugins are usually deployed as Pods controlled by Daemonsets running on each node
    - Some features brought by plugins could be management, application of network policies (like QoS), encryption, traffic segmentation etc
- kubelets on nodes are also responsible for controlling local network configuration, which in turn defines a **network plugin** (either CNI or Kubenet)
    - When configured for CNI, the kubelet loads its plugin and configuration
    - If set for Kubenet, the node is configured with routes and bridges inside the BaseOS

## Cluster DNS
- DNS is provided as a Service in a cluster, using CoreDNS as default
- Pods are configured to use this internal cluster DNS server as default
- In a DNS server, records are created for resources deployed on the cluster
    - Services - A(IPv4)/AAAA(IPv6)
    - Namespaces get Subdomains records
- The DNS Service is core to Service discovery inside of K8s (as resources and Services are created, Pods can reference resources by their DNS names)
- Both the DNS Service and Pods configuration can be customized

### Configuring a forwarder
- Allows controlling where the DNS Service is forwarding requests to
```
apiVersion: v1
kind: ConfigMap                                                 # configuration is stored as a ConfigMap
metadata:
  name: coredns                                                 # specific name and location on namespace
  namespace: kube-system
data:                                                               
  Corefile: |                                                   # mapped as volume under this name on /etc/coredns
    .:53 {
        ...
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-add.arpa ip6.arpa
            ttl 30
        }
        forward . /etc/resolv.conf                              # forwards to all DNS zones to /etc/resolv.conf, or to IP address
        ...
    }
```

#### Specifying DNS servers
```
...
    spec:                                               # the Pod's spec
      containers:
      - name: hello-world
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
      dnsPolicy: "None"                                 # definition of specific DNS configuration to be applied
      dnsConfig:                                        # where DNS specific configurations are defined
        nameservers:
          - 9.9.9.9
        searches:
          - db1.ns1.svc.cluster.local
```

###### Return to [Summary](README.md)
