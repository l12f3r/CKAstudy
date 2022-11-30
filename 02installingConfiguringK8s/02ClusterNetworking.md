# Installing and Configuring Kubernetes

## Cluster network ports
- Cluster has network ports, that are necessary when building firewalls 🔥🧱 or other security perimeters around Kubernetes resources.
- Port numbers are usually configureable, but default values are commom (and good) practice

## How resources communicate
- [Kubelet](07k8sClusterComponents.md#worker-node-components) and [Kube-proxy](07k8sClusterComponents.md#worker-node-components) on the worker node 👩‍🏭 communicate with the [API server](02kubernetesAPI.MD) on the control plane node 🧠 via [TCP/IP](https://www.techtarget.com/searchnetworking/definition/TCP-IP).
    - Some TCP/IP ports are reserved for cluster communication and should be taken into account upon developing firewall rules

| Component  | Ports (TCP) | Used by |
| ------------- | ------------- | ------------- |
| API Server 🧠  | **6443**  | All resources[^1] |
| etcd 🧠  | **2379-2380**  | API server 🧠 / etcd 🧠[^2] |
| Scheduler 🧠  | **10251**  | Self[^3] |
| Controller Manager 🧠  | **10252**  | Self[^3] |
| Kubelet 🧠  | **10250**  | All Control Plane 🧠 components |
| Kubelet 👩‍🏭  | **10250**  | All Control Plane 🧠 components |
| NodePort | **30000-32767**  | All resources[^4] |

[^1]: Every communication towards/within a cluster (that is, from either outside or internally [on either worker 👩‍🏭 or control plane 🧠 nodes]) passes by the API server.
[^2]: etcd communicates with "itself" when a redundant configuration is set: etcd's multiple replicas would need to fetch data within themselves
[^3]: Only on `localhost`: not exposed to outside communications
[^4]: The NodePort Service exposes ports on each individual node in the cluster. So, anything that needs to access the content of a specific node needs to access this node's port.