# Installing and Configuring Kubernetes

## Pod networking

Two different approaches:
- **Overlay Networking**:
    - Network packets between [Pods](03APIObjectsPods.MD) are encapsulated in additional packets before being transmitted over the underlying network infrastructure
    - Each pod is assigned a unique IP address within a virtual network
        - On pod-to-pod communication, encapsulated (usually using tunneling, techniques like VXLAN or Geneve) packets are sent to the IP address of the destination pod. These packets are transmitted over the layer 3 virtual network to the destination pod, where they are decapsulated and delivered
    - Allows pods to have IP addresses regardless of the underlying physical infrastructure. This makes it easier to migrate pods between nodes and clusters without the need for reconfiguration.
    - Available overlay networks: Calico, Flannel, Weave Net

- **Direct Routing**:
    - [Pods](03APIObjectsPods.MD) have IP addresses from the same subnet as the underlying physical cluster infrastructure, and packets are routed directly based on these addresses.
    - On pod-to-pod communication, packets are sent directly (usually using routing) to the IP address of the destination pod
    - More performance-efficient since there is no encapsulation and decapsulation of packets. Additionally, it can be simpler to configure and manage compared to overlay networking.

