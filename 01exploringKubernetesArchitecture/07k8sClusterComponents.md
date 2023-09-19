# Exploring the Kubernetes Architecture

## Cluster components

### Control Plane Node: 🧠
- Coordinates cluster operations, monitoring, [Pod](03APIObjectsPods.MD) scheduling
- Primary access point to cluster administration (probably where Prometheus gets info from)
- Previously known as "Master"

#### Control Plane Node components:
- **[API server](02kubernetesAPI.MD)**: primary access point for cluster and administrative operations; communication hub
    - All configurations changes pass through it
    - Validates API operations and persists it on etcd
- **etcd**: stores state (since that the [API server](02kubernetesAPI.MD) is stateless)
    - Persists state of API objects as key/value pairs
- **Scheduler**: watches the [API server](02kubernetesAPI.MD) for unscheduled [Pods](03APIObjectsPods.MD), scheduling it on Nodes
    - Evaluates [Pod](03APIObjectsPods.MD)'s necessities (CPU, RAM, storage) to ensure availability upon scheduling
    - Respects previously defined contraints (handles [Pod](03APIObjectsPods.MD) affinity and anti-affinity)
- **Controller Manager**: responsible for looping (implementing Pod lifecycle functions to) [Controllers](04APIObjectsControllers.MD)
    - Watches and updates the [API server](02kubernetesAPI.MD) to ensure the desired state
- **All (Worker) Node components**

### (Worker) Node: 👩‍🏭
- Where application [Pods](03APIObjectsPods.MD) run; contributes to cluster's compute capacity
- Usually multiple on clusters, depending on scalability requirements
- Starts [Pods](03APIObjectsPods.MD) and containers within;
- Implement [networking](09NetworkingFundamentals.md);
- Either virtual or physical machines

#### (Worker) Node components:
- **Kubelet** 👩‍🏭: Starts [Pods](03APIObjectsPods.MD) on a Node
    - Communicates directly with [API server](02kubernetesAPI.MD), monitoring it for changes 
        - Applies [Pod](03APIObjectsPods.MD) lifecycle in reaction to state changes
    - Reports Node and [Pod](03APIObjectsPods.MD) state
    - Executes probes
- **Kube-proxy** 📰: Pod networking, implementing [Services](05APIObjectsServices.md) abstraction on Node using iptables (and other modes)
    - Communicates directly with [API server](02kubernetesAPI.MD), monitoring it for changes (such as network topology changes)
    - Routes traffic to [Pods](03APIObjectsPods.MD)
    - Provides load balancing
- **Container runtime**: Downloads container images (from container registry) and runs/starts them
    - Provides execution environment for the container image and [Pod](03APIObjectsPods.MD) abstraction
    - containerd, Docker etc

![image](https://user-images.githubusercontent.com/22382891/201743268-46178043-b5c5-4828-bcc1-06d75c35ce8f.png)

### Add-on Pods ➕𝟭: Provide special services to the cluster
- **DNS**: Uses CoreDNS (the Domain Name Server) to provide DNS services to the cluster
    - DNS Pods have their IP addresses and search suffixes placed on the networking configuration of every [Pod](03APIObjectsPods.MD), Node or [Service](05APIObjectsServices.md) created by the [API server](02kubernetesAPI.MD)
    - Used for service discovery for applications deployed on a cluster
    - Occur in nearly every k8s cluster
- **Ingress Controller**: advanced HTTP/Layer 7 load balancer/content router (optional)
- **Dashboard**: web-based administration of the cluster (optional)
- **Network Overlays**: TBD

### kubectl:
- CLI used to interact with the [API server](02kubernetesAPI.MD)

###### Return to [Summary](https://github.com/l12f3r/CKAstudy/tree/main/01exploringKubernetesArchitecture#readme)