# Managing and Controlling the Kubernetes Scheduler

## Controlling scheduling in Kubernetes
- Tools used to control Scheduling in Kubernetes through Policy:
    - [**Node Selector**](02controllingScheduling.md#node-selector): labels are assigned to nodes, and selectors to Pods, to place Pods on certain nodes
    - [**Affinity**](02controllingScheduling.md#affinity-and-anti-affinity): controls Pod placement based on more complicated expressions and scenarios
    - [**Taints and Tolerations**](02controllingScheduling.md#taints-and-tolerations): to define which Pods should be kept off certain nodes on the cluster
    - [**Node Cordoning**](02controllingScheduling.md#node-cordoning-🛑): to disable nodes from getting new Pods scheduled to it
    - [**Manual Scheduling**](02controllingScheduling.md#manually-scheduling): how to place a Pod in a particular node manually, independent of the scheduling process

### Node Selector
- A `nodeSelector` applies labels to nodes to make a scheduling decision (placing a Pod on a subset of nodes)
- Scheduling is influenced by assigning Pods to a node with a matching label
- Simple key/value equality check based on `matchLabels`, rather than a complicated expression match
- Often used to map Pods to nodes based on...
    - Special hardware requirements (access to local high-speed disks or GPUs available)
    - Workload isolation (some nodes are dedicated to specific workloads)
- Further info on [Managing Kubernetes API Server and Pods](../../Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/02managingObjectsLabelsAnnotationsNamespaces/02workingWithLabels.md)

#### How the node selector works in practice
1. Suppose the existance of two nodes: one with 🅰️ and 🅱️ labels, and another with only 🅰️
2. Pods with an 🅰️ node selector can be scheduled on both nodes; Pods with 🅱️ node selector are scheduled only on the 🅰️🅱️ node (because that's the only place that accepts 🅱️-node selector Pods)
3. Pods without any node selector can be scheduled on any node
4. Pods with node selectors without any matching labels are unschedulable and remain on `PENDING` status until getting a node selector that matches a node

#### How to
- Interactively: `kubectl label node c1-node3 hardware=local_gpu` assigns the `hardware=local_gpu` label to the `c1-node3` node
- Declaratively:
```
spec:                                               # The Pod's spec
  containers:
  - name: hello-world
    image: gcr.io/google-samples/hello-app:1.0
    ports:
    - containerPort: 8080
  nodeSelector:                                     # where the node selector is defined
    hardware: local_gpu                             # must match to the node's label
```

### Affinity and Anti-Affinity
- Like `nodeSelector`, `nodeAffinity` also uses labels on nodes to make a scheduling decision, but with `matchExpressions` instead
    - Gives control over scheduling in terms of when and how these rules are evaluated
- `podAffinity` uses labels and selectors to schedule Pods onto the same node (or onto the same datacenter zone) as some other Pod
    - Enables to use both `matchExpressions` and `matchLabels`
- `podAntiAffinity` uses labels and selectors to repel Pods from being scheduled on the same node (Pods are never scheduled on the same node/zone)
    - Enables to use both `matchExpressions` and `matchLabels`
- Uses the following configurations:
    - `requiredDuringSchedulingIgnoredDuringExecution`: a Pod will not be scheduled unless the rules or operations herein defined evaluate the `true`; if `false`, Pod remains on `PENDING` status because no node satisfying the affinity rule could be found
    - `preferredDuringSchedulingIgnoredDuringExecution`: a Pod will still be scheduled if evaluates `false`
      - On a scaling scenario, if the number of replicas exceeds the number of available nodes, Pods will not be on `PENDING` status; they'll be scheduled on nodes with Pods already present (that is, rules are disregarded)
    - `topologyKeys`: key defined to specify which topologic features (physical/logical structure, like cloud region or availability zone) affinity must be based on
- Further info on [Managing Kubernetes API Server and Pods](../../Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/02managingObjectsLabelsAnnotationsNamespaces/02workingWithLabels.md)

#### How podAffinity and podAntiAffinity work in practice
1. On a cluster with two nodes (or zones), an anti-affinity rule is defined for a collection of Pods, so that Pods with different node selectors are never scheduled on the same node/zone
2. If an affinity rule is defined to another collection of Pods, the scheduler ensures that those are always scheduled on the same node

#### How to
```
spec:                                                       # the Pod's spec
  containers:
  - name: hello-world-cache
  ...
  affinity:                                                 # where affinity is defined
    podAffinity:                                            # affinity type
      requiredDuringSchedulingIgnoredDuringExecution:       # if the rule is false during scheduling, Pod is not scheduled
      - labelSelector:
          matchExpressions:
          - key: app                                        # Key of the label that should be considered for matching
            operator: In                                    # Operator to use for matching
            values:
            - hello-world-web                               # Value that Pods with "app" label should have to match
        topologyKey: "kubernetes.io/hostname"               # node's hostname considered when deciding where to place the Pod
```

### Taints and Tolerations
- **Taints** gives control over scheduling by repelling certain Pods to be scheduled to certain nodes (those nodes become tainted)
- **Tolerations** allows Pods to ignore tainting rules and be normally scheduled on tainted nodes
- Both are useful in scenarios where the cluster administrator needs to influence scheduling without depending on the user (like isolating Pods from the Control Plane 🧠 node)
- Syntax: `key=value:effect`
- Effects: 
  - `NoSchedule`(Pods are not scheduled to this node unless there's a toleration; Pods already running on this node will remain),
  - `PreferNoSchedule` (Pods are avoided to be scheduled on this node unless there are no other untainted nodes available; Pods already running on this node will remain), and
  - `NoExecute` (running Pods are evacuated from this node and no future Pods are scheduled on it, unless there's a toleration)

#### How Taints and Tolerations work in practice
1. Suppose the existance of two nodes: one with 🅰️ taint and `NoSchedule` effect, and another without taints
2. Pods with an 🅰️ would only be scheduled on the 🅰️-tainted node if there was a matching toleration defined on their Pod's spec; otherwise, they are repelled from this node
3. Workloads without a matching toleration (or without any toleration at all) will be scheduled elsewhere in the cluster

#### How to
- `kubectl taint nodes c1-node1 key=MyTaint:NoSchedule` imperatively adds a taint to a node  
- To add a toleration to a Pod:
```
  spec:                                                   # the Pod's spec
    containers:
    - name: hello-world
      image: gcr.io/google-samples/hello-app:1.0
      ports:
      - containerPort: 8080
    tolerations:                                          # where tolerations for the specified taint are defined
    - key: "key"
      operator: "Equal"
      value: "MyTaint"
      effect: "NoSchedule"                                # must match to the taint defined on the node
```
### Node Cordoning 🛑
- Marks a node as unschedulable, preventing new Pods from being scheduled to it
- Does not affect existing Pods on the marked node
- Nodes with static Pods demand a `--force` command to be cordoned (since that it's not managed by a controller, Kubernetes cannot re-schedule this Pod; that's why it prevents ungraceful deletions)
- Often used as a preparatory step before rebooting or maintaining a node

#### How to
- `kubectl cordon c1-node3`
- To gracefully evict Pods from a node, `kubectl drain c1-node3 --ignore-daemonsets`
  - Daemonsets (such as the `kube-proxy`) are often kept for maintenance and logging reasons

### Manually scheduling
- As normal behavior, the `nodeName` field of a Pod's object is populated upon scheduling via the API server
- If the `nodeName` is specified on the Pod definition, it'll be started on the node defined, allowing to manually select a node to schedule the Pod in
  - This bypasses the Scheduler's process of searching for a Pod without a `nodeName` defined
- The manually assigned `nodeName` must exist (that is, there must be a node with that matching label), otherwise, it remains on `PENDING` state
- Even though the manually process occurs, it is still subject to node resource constraints

#### How to
```
spec:                         # the Pod's spec
  nodeName: 'c1-node3'        # where the nodeName is defined
  containers:
  ...
```

### Configuring multiple Schedulers
- Kubernetes allows to implement custom Schedulers, and run multiple Schedulers concurrently
  - Regular workload can be handled by the default Scheduler, while custom workloads are handled by custom Schedulers, for instance
- Definition on which Scheduler should be used must be declared on the Pod's spec
- Custom Schedulers are deployed in the cluster as system Pods

###### Return to [Summary](README.md)