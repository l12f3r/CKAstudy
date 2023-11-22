# Managing and Controlling the Kubernetes Scheduler

## Controlling scheduling in Kubernetes
- Tools used to control Scheduling in Kubernetes through Policy:
    - **Node Selector**: labels are assigned to nodes, and selectors to Pods, to place Pods on certain nodes
    - **Affinity**: controls Pod placement based on more complicated expressions and scenarios
    - **Taints and Tolerations**: to define which Pods should be kept off certain nodes on the cluster
    - **Node Cordoning**: to disable nodes from getting new Pods scheduled to it
    - **Manual Scheduling**: how to place a Pod in a particular node manually, independent of the scheduling process

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

###### Return to [Summary](README.md)