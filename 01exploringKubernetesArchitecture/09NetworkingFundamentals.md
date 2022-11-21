# Exploring the Kubernetes Architecture

## Networking Fundamentals
- Pods can reach all Pods in all Nodes without NAT
    - Each Pod has its own IP address
- Agents (Kubelet, system daemons etc) within a Node can only communicate to all Pods within this same Node

## Communication Patterns
- Containers within a Pod communicate via localhost using namespaces
- Pods within a Node communicate over a layer-2 (software) bridge, via IP addresses
- Pods on different Nodes communicate over a layer-2 (software) or layer-3 (network) bridge, via IP addresses
    - Network reachability must be ensured
    - Overlay networks can be used to simulate that Pods on different Nodes share the same layer
- To expose the cluster to the Internet, the kube-proxy provides HTTP external access via a Service, which organizes and distributes traffic to the front-end Pods

![image](https://user-images.githubusercontent.com/22382891/203161110-33887c0c-b5a8-43a7-b269-9dfb6b4e29c9.png)


