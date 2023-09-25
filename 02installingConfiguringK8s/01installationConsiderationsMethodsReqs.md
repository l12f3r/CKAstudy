# Installing and Configuring Kubernetes

## Where to install?
- In the cloud ☁️
    - Internet as a service (IaaS) - **Virtual machines**
        - All networking, hypervisor and infrastructure is provisioned by cloud - user provides patching the OS and installing K8s
        - More control, but more responsibility managing
    - Platform as a service (PaaS) - **Managed service**
        - Kubernetes as a Service, like Amazon's Elastic Kubernetes Service (EKS) 
        - Less responsibility, but less flexibility
- On premises ⚙️
    - Bare metal
    - Virtual machines

## Other considerations
- Cluster networking
    - [Overlay networking](05podNetworkingFundamentals.md)? Or having a network engineering team to ensure layer 2 and 3 connectivity?
    - Ensure no network IP range overlaps between cluster and rest of networking infra
- Scalability 🧗‍♀️
    - Enough nodes with enough resources (CPU, RAM) to meet workload demand, or if failure occurs
- High Availability and Disaster Recovery 💩
    - How to create replicas of Control Plane nodes, [API server](02kubernetesAPI.MD) and [etcd](07k8sClusterComponents.md#control-plane-node-components) to ensure redundancy
    - Backup and recovery of etcd data

## Installation methods
- Desktop installation 🖥️
    - Ideal for development and training environments
- [`kubeadm` (a.k.a. "kubeadmin")](04bootstrappingClusterKubeadm.md) 🛠️
    - Package that allows bootstrapping a cluster and get it running fast
- Cloud scenarios ☁️
    - Deployment of IaaS and PaaS in cloud providers

## Installation requirements
- System requirements
    - Linux 🐧 (Ubuntu, RHEL, CentOS etc)
    - 2 CPUs
    - 2GB of RAM
    - [Swap process](https://www.linux.com/news/all-about-linux-swap-space/) disabled
- Container runtime
    - Container runtime interface (CRI) compatible
        - Docker (deprecated from Kubernetes version 1.23 onwards), containerd or CRI-O
- Networking
    - Connectivity between all nodes
    - Each system must have a unique hostname and MAC address

###### Return to [Summary](https://github.com/l12f3r/CKAstudy/tree/main/02installingConfiguringK8s#readme)