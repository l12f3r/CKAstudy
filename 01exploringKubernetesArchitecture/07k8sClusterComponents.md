# Exploring the Kubernetes Architecture

## Cluster components

### Control Plane Node: 🧠
- Coordinates cluster operations, monitoring, Pod scheduling
- Primary access point to cluster administration (probably where Prometheus gets info from)
- Previously known as "Master"

#### Control Plane Node components:
- **API server**: primary access point for cluster and administrative operations; communication hub
    - All configurations changes pass through it
    - Validates API operations and persist it on etcd
- **etcd**: stores state (since that the API server is stateless)
    - Persists state of API objects as key/value pairs
- **Scheduler**: watches the API server for unscheduled Pods, scheduling it on Nodes
    - Evaluates Pod's necessities (CPU, RAM, storage) to ensure availability upon scheduling
    - Respects previously defined contraints (handles Pod affinity and anti-affinity)
- **Controller Manager**: responsible for looping (implementing Pod lifecycle functions to) Controllers
    - Watches and updates the API Server to ensure the desired state
- **All (Worker) Node components**

### (Worker) Node: 👩‍🏭
- Where application Pods run; contributes to cluster's compute capacity
- Usually multiple on clusters, depending on scalability requirements
- Starts Pods and containers within;
- Implement networking;
- Either virtual or physical machines

#### (Worker) Node components:
- **Kubelet**: Starts Pod on a Node
    - Communicates directly with API server, monitoring it for changes 
        - Applies Pod lifecycle in reaction to state changes
    - Reports Node and Pod state
    - Executes probes
- **Kube-proxy**: Pod networking, implementing Services abstraction on Node using iptables (and other modes)
    - Communicates directly with API server, monitoring it for changes (such as network topology changes)
    - Routes traffic to Pods
    - Provides load balancing
- **Container runtime**: Downloads container images (from container registry) and runs/starts them
    - Provides execution environment for the container image and Pod abstraction
    - containerd, Docker etc

![image](https://user-images.githubusercontent.com/22382891/201743268-46178043-b5c5-4828-bcc1-06d75c35ce8f.png)

### Add-on Pods ➕𝟭: Provide special services to the cluster
- **DNS**: Uses CoreDNS (the Domain Name Server) to provide DNS services to the cluster
    - DNS Pods have their IP addresses and search suffixes placed on the networking configuration of every Pod, Node or Service created by the API server
    - Used for service discovery for applications deployed on a cluster
    - Occur in nearly every k8s cluster
- **Ingress Controller**: advanced HTTP/Layer 7 load balancer/content router (optional)
- **Dashboard**: web-based administration of the cluster (optional)
- **Network Overlays**: TBD

### kubectl:
- CLI used to interact with the API server