# Managing and Controlling the Kubernetes Scheduler

## Scheduling in Kubernetes
- Scheduling (that is, distributing workload among Worker 👩‍🏭 nodes on the cluster) is done by the **Scheduler**, which is part of the Control Plane 🧠 node
    - Scheduling can also be defined as "selecting a node to start a Pod on"
    - Kubernetes' default scheduler is `kube-scheduler`
- Takes some criteria in consideration to distribute workload properly, such as **resources** and **policies**
    - Resources: hardware (CPU, RAM, storage)
        - This criteria is taken in consideration not only upon scheduling a Pod, but also for workload already running on the cluster and remaining resources available on the nodes
    - Policies: administrative influence on where workload runs in the cluster
        - Two Pods that must run on the same (or in a different) node, for example

### Process
1. The Scheduler watches the API Server looking for unscheduled (that is, without a node name assigned on object definition) Pods
2. Scheduler selects a node
3. `nodeName` is updated in the Pod object via the API Server
4. kubelets constantly watch the API Server for workload to be scheduled for them
5. Once an workload is found by the kubelet, it signals the container runtime to start a container associated with the Pod being scheduled

#### The node selection process looked closely 🔎
| **Filtering** -> | **Scoring** -> | **Binding** |
| --- | --- | --- |
| Removes nodes that cannot run the Pod to be scheduled | Yields a list of eligible nodes to run the Pod | Updating `nodeName` on Pod's object |
| From all nodes with a `READY` status | Evaluates a collection of scoring functions to get priority weights for finding the proper place to run a Pod | Highest priority nodes are added to a **Selected Nodes** list |
| Applies filters (or predicates) to reduce the amount of eligible nodes to a **Filtered Nodes** list | List of eligble nodes: **Feasible Nodes** list | Ties are broken - a random node is selected upon race conditions |
| Hard constraints (can this Pod run on this node?), `nodeSelector` | Policy constraints (is the container image already on the node?), taints and tolerations | API server is notified and `nodeName` update takes place |

- Filtering finds if a node can run a Pod - Scoring tries to find the most appropriate place to run a Pod

### Resource requests
- Setting `requests` on the Pod's spec will cause the scheduler to find a node to fit the workload/Pod
    - `requests` are guarantees - based on CPU and RAM
- To know how much resources are available, the node tracks an allocatable amount of resources available, and as Pods make requests, this amount is reduced (based on type and amount requested)
- If scheduling occurs while there are not enough resources available, the Pod goes to `PENDING` status
```
...
    spec:                                                   # Pod's template spec
      containers:
      - name: hello-world
        image: gcr.io/google-samples/hello-app:1.0
        resources:                                          # Where resources are defined
          requests:                                         # A request definition
            cpu: "1"                                        # One CPU is requested to start the Pod
        ...
```

###### Return to [Summary](README.md)