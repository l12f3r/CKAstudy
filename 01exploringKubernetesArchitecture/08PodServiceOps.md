# Exploring the Kubernetes Architecture

## Pod Operations

![image](https://user-images.githubusercontent.com/22382891/202746615-de210559-1f86-4f1a-a9ae-c2b5e36d3646.png)

1. Using kubectl, we submit code to instruct K8s to create a Deployment with 2 replicas. This request is submitted to the API server;
2. API server stores information persistently on etcd;
3. Controller Manager spins up the replicas requested on the Deployment as a ReplicaSet, creating two Pods;
4. Request is submitted to the Scheduler, which informs the API server in which Nodes these Pods needs to be created (criteria: available resources). Such information is persistenly stored in etcd;
5. Kubelets on the Nodes ask the API server for workloads, and start spin up the Pods requested by the ReplicaSet;
6. Controller Manager monitors the state of the running replicas, provisioning Pods if the desired state is not met.

## Service Operations

![image](https://user-images.githubusercontent.com/22382891/202750178-a64ae99a-cfda-41b1-8a9e-eaf4be305a48.png)

In this scenario, the Pods within this Cluster are web applications exposed by a Service (HTTP on TCP port 80). Users can access the Pods via this Service endpoint (which is fixed and persistent IP address or DNS name).

1. Incoming requests to the Cluster are load balanced to all web application Pods;
2. If a Pod becomes unresponsive, the ReplicaSet controller removes the Pod and add a new one. The load balancer sends traffic to other healthy Pods.