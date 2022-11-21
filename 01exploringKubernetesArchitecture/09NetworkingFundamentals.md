# Exploring the Kubernetes Architecture

## Networking Fundamentals
- [Pods](03APIObjectsPods.MD) can reach all [Pods](03APIObjectsPods.MD) in all Nodes without NAT
    - Each [Pod](03APIObjectsPods.MD) has its own IP address
- Agents ([Kubelet](07k8sClusterComponents.md#worker-node-components), system daemons etc) within a Node can only communicate to all [Pods](03APIObjectsPods.MD) within this same Node

## Communication Patterns
- Containers within a [Pod](03APIObjectsPods.MD) communicate via localhost using namespaces
- [Pods](03APIObjectsPods.MD) within a Node communicate over a layer-2 (software) bridge, via IP addresses
- [Pods](03APIObjectsPods.MD) on different Nodes communicate over a layer-2 (software) or layer-3 (network) bridge, via IP addresses
    - Network reachability must be ensured
    - Overlay networks can be used to simulate that [Pods](03APIObjectsPods.MD) on different Nodes share the same layer
- To expose the cluster to the Internet, the [kube-proxy](07k8sClusterComponents.md#worker-node-components) provides HTTP external access via a [Service](05APIObjectsServices.md), which organizes and distributes traffic to the front-end [Pods](03APIObjectsPods.MD) (more on [Service Operations](08PodServiceOps.md#service-operations).)

![image](https://user-images.githubusercontent.com/22382891/203161110-33887c0c-b5a8-43a7-b269-9dfb6b4e29c9.png)

###### Return to [Summary](01exploringKubernetesArchitecture)