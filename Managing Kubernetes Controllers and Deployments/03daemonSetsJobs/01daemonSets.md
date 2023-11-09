# Deploying and Maintaining Applications with DaemonSets and Jobs

## DaemonSets
- Ensures that all or some nodes run a copy of a Pod
- Effectively, an init daemon inside the cluster to run basic services, much like a traditional Linux daemon
- Example workloads:
    - `kube-proxy`, for network services within the cluster
    - Log collectors and metric servers
        - Need to be running on every node for collecting data and shipping it off to other tools
    - Resource monitoring agents
    - Storage daemons

### Controller Operations
- When a DaemonSet is declared, its controller is responsible for deploying a Pod on each worker node 👩‍🏭 in the cluster
    - Control plane node 🧠 is tainted for non-system service functions and workloads; only toleration is the `kube-proxy`
- Apart from custom labels to determine membership, each Pod will be created with `controller-revision-hash` and `pod-template-generation` labels
  - This is how the DaemonSet tracks updates - checking labels associated with versions of Pods rolled out
  - If a Pod label is changed, the DaemonSet controller pushes it outside of the selector and provisions a new Pod to replace it, according to the desired state

### Pod Scheduling
- By default, K8s ensures that one Pod will be scheduled to each worker node 👩‍🏭 by the default-scheduler
- As further nodes are added to the cluster, they get a Pod each
    - Arbitrary: one can define which nodes get Pods, using `nodeSelector` and labeling nodes properly
      - Basically, only nodes labeled with the `nodeSelector` get Pods

### Declaration
```
apiVersion: apps/v1
kind: DaemonSet                     # this is where the DaemonSet is defined
metadata:
  name: hello-world-ds              # name for DaemonSet
spec:                               # specs for this DaemonSet controller object
  selector:
    matchLabels:                    # for Pod membership determination; matchExpressions can also be used
      app: hello-world-app          # label to be associated to the selector
  template:                         # Pod template
    metadata:
      labels:
        app: hello-world-app        # must match the selector
    spec:                           # specs for the Pod template
      containers:
        - name: hello-world
          image: gcr.io/google-samples/hello-app:1.0
```

#### Declaration with a nodeSelector
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: hello-world-ds
spec:
  selector:
    matchLabels:
      app: hello-world-app
  template:
    metadata:
      labels:
        app: hello-world-app
    spec:
      nodeSelector:                 # where the nodeSelector is defined
        node: hello-world-ns        # this label will be assigned to each node in the cluster
```

### Update Strategies
- `RollingUpdate`: after updating the DaemonSet Pod template, old DaemonSet Pods are killed for new ones to be provisioned in their place, in a controlled fashion
    - Has `maxUnavailable` as setting; default is 1
- `OnDelete`: new DaemonSet Pods are created only after manually deleting another Pod in execution

###### Return to [Summary](README.md)