# Configuring and Managing Application Access with Services

## Services
- Provide **persistent endpoint** access for clients
    - Adds persistency to the ephemerality of Pods: ensures that Pods keep receiving its desired workload after its lifecycle is completed (that is, when new Pods are scheduled to replace faulty ones)
- Networking abstraction providing **persistent virtual IP addresses and DNS names**
- Load balances traffic to backend (non-public) Pods
    - Are automatically updated during Pod controller operations (Deployments, etc) to reflect load balancing changes upon scaling

### How Services work
- Uses [labels and selectors](/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/02managingObjectsLabelsAnnotationsNamespaces/02workingWithLabels.md#Services) to determine which Pods are member to which Services in a cluster
- A controller monitors which Pods satisfy the selector, and creates and registers endpoints objects on the Service for each matching Pod
    - Objects: Pod IP and Port pair 📮
    - Each matching endpoint is added to a list 📋, which is used to load balance traffic across all Pods supporting that Service
- Service implementation is responsibility of the kube-proxy, that exposes such Services onto the network using iptables rules (that's firewalling) 🔥🧱
    - kube-proxy is executed as a Pod controlled by a DaemonSet, which means that every node has one
    - kube-proxy watches the API Server and Services/endpoints as they come and go, updating the local configuration to match the desired state

### Service types

#### `ClusterIP`
- A `ClusterIP` Service gets an IP address assigned from the [Cluster Network](../01k8sNetworkingFundamentals/01introducingK8sNetwork.md#kubernetes-network-topology) 🟢, who assigns it from a range of addresses specified on configuration
    - Usually, it gets an IP address and Port range (such as `10.1.22.10:80` for example) 📮
- kube-proxy configures the necessary iptables rules for servers in all nodes, defining Service's cluster IP address and port, and also implementing load balancing rules to backend endpoints
    - If a traffic is load balanced to a Pod endpoint not present on the current node, that traffic is routed across the Pod network to the correct node to reach the desired endpoint
- To access the service, traffic must be sent to the `ClusterIP` address and port 📮 from inside a cluster
    - For traffic destined to the `ClusterIP`, the iptables rules 🔥🧱 on that node will intercept the traffic before it leaves the node
    - An endpoint IP address is selected from the iptables load balancing rules list 📋, and iptables will send traffic to that selected Pod IP address over the [Pod Network](../01k8sNetworkingFundamentals/01introducingK8sNetwork.md#kubernetes-network-topology) 🔵
- The `ClusterIP` address is completely virtual and lives only inside iptables on the node
- **Service is exposed on a cluster internal IP address and is only available inside of the cluster**
- Used mostly in scenarios of Services that don't need to be accessed outside of the cluster

#### `NodePort`
- In this scenario, kube-proxy exposes the Service on the real IP addresses of each node in the cluster (that means that node01 could have `172.16.94.11:32235` 🟠 and node02 would have `172.16.94.12:32235` 🟠)
    - By default, port range is 30000 to 32767
- When a `NodePort` Service is created, a `ClusterIP` Service and port is also allocated, and kube-proxy configures it on each node (as it normally does)
- Traffic can come from outside of the cluster and can be sent to any nodes' real IP address (`172.16.94.XY:32235` 🟠) 
    - kube-proxy routes the traffic request to the internal `ClusterIP` service, who routes it to Pod endpoints (as it normally does)
- Used mostly on integrating load balancing from external traffic, development scenarios and desktop-based clusters

#### `LoadBalancer`
- In this scenario, the cluster interacts with the cloud provider to get a load balancing service from them
    - This load balancer gets a real public IP address assigned from the cloud provider and will point users at that public IP address to access the services defined in the cluster
- Upon creation, a `NodePort` and a `ClusterIP` Services are also created
    - Cloud provider's load balancer accesses the `NodePort` real node IP addresses 🟠, who sends the request to the `ClusterIP` Service, that sends to underlying Pod endpoints

#### Other types
- `ExternalName`: service discovery for external services
    - A DNS name is defined for a Service outside of the cluster and Kubernetes creates a `CNAME` record for it
- **Headless** Services: Service with DNS, but no `ClusterIP` defined
    - If defined with selectors, a DNS record is defined for each matching endpoint IP address
    - Querying the DNS for the Headless Service returns all Pod's IP addresses
    - Used commonly in databases and stateful applications
- Services **Without Selectors**: enable to map services to specific endpoints
    - Requires to manually create the endpoint objects, defining whatever IP address from inside or outside the cluster upon creating
    - Used commonly in scenarios where full traffic control must be in place

### Definition
- Imperatively: `kubectl expose deployment hello-world --port=80 --target-port=8080 --type NodePort` on an existing `hello-world` Deployment

#### Deployment
```
kind: Deployment
...
  template:
    metadata:
      labels:                   # all Pods from this Deployment will have such labels
        run: hello-world        # must match to the Service's selector
    spec:
        containers:
...
```

#### Service (declaratively)
```
kind: Service                   # where the object type is defined
...
spec:
  type: ClusterIP               # Service type definition
  selector:
    run: hello-world            # must match to the Deployment's label
  ports:
  - port: 80                    # users will be pointed here to access the service; listening here
    protocol: TCP               # protocol used for this Service (TCP, UDP, SCTP)
    targetPort: 8080            # container-based application port; sending traffic here
```

###### Return to [Summary](README.md)