# Installing and Configuring Kubernetes

## Installation considerations

- Where to install?
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
    - Overlay networking? Or have a network engineering team to ensure layer 2 and 3 connectivity?
    - Ensure no network IP range overlaps between cluster and rest of networking infra
- Scalability 🧗‍♀️
    - Enough nodes with enough resources (CPU, RAM) to meet workload demand, or if failure occurs
- High Availability and Disaster Recovery 💩
    - How to create replicas of Control Plane nodes, API server and etcd to ensure redundancy
    - Backup and recovery of etcd data