# Exploring the Kubernetes Architecture

## Pod Operations

![image](https://user-images.githubusercontent.com/22382891/202746615-de210559-1f86-4f1a-a9ae-c2b5e36d3646.png)

1. Using [kubectl](07k8sClusterComponents.md#kubectl), we submit code to instruct K8s to create a [Deployment](04APIObjectsControllers.MD#types-of-controllers) with 2 replicas. This request is submitted to the [API server](02kubernetesAPI.MD);
2. [API server](02kubernetesAPI.MD) stores information persistently on [etcd](07k8sClusterComponents.md#control-plane-node-components);
3. [Controller Manager](07k8sClusterComponents.md#control-plane-node-components) spins up the replicas requested on the [Deployment](04APIObjectsControllers.MD#types-of-controllers) as a [ReplicaSet](04APIObjectsControllers.MD#types-of-controllers), creating two [Pods](03APIObjectsPods.MD);
4. Request is submitted to the [Scheduler](07k8sClusterComponents.md#control-plane-node-components), which informs the [API server](02kubernetesAPI.MD) in which Nodes these [Pods](03APIObjectsPods.MD) needs to be created (criteria: available resources). Such information is persistenly stored in [etcd](07k8sClusterComponents.md#control-plane-node-components);
5. [Kubelets](07k8sClusterComponents.md#worker-node-components) on the Nodes ask the [API server](02kubernetesAPI.MD) for workloads, and start spin up the [Pods](03APIObjectsPods.MD) requested by the [ReplicaSet](04APIObjectsControllers.MD#types-of-controllers);
6. [Controller Manager](07k8sClusterComponents.md#control-plane-node-components) monitors the state of the running replicas, provisioning [Pods](03APIObjectsPods.MD) if the desired state is not met.

## Service Operations

![image](https://user-images.githubusercontent.com/22382891/202750178-a64ae99a-cfda-41b1-8a9e-eaf4be305a48.png)

In this scenario, the [Pods](03APIObjectsPods.MD) within this Cluster are web applications exposed by a [Service](05APIObjectsServices.md) (HTTP on TCP port 80). Users can access the [Pods](03APIObjectsPods.MD) via this [Service](05APIObjectsServices.md) endpoint (which is fixed and persistent IP address or DNS name).

1. Incoming requests to the Cluster are load balanced to all web application [Pods](03APIObjectsPods.MD);
2. If a [Pod](03APIObjectsPods.MD) becomes unresponsive, the [ReplicaSet](04APIObjectsControllers.MD#types-of-controllers) controller removes the [Pod](03APIObjectsPods.MD) and add a new one. The load balancer sends traffic to other healthy [Pods](03APIObjectsPods.MD).

###### Return to [Summary](01exploringKubernetesArchitecture)