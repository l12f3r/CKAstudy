# Managing Objects with Labels, Annotations and Namespaces

## Labels

- Non-hierarchical, key/value pair
    - Keys can be 63 characters or less, Values can be 253 characters or less
- Organizes resources - Pods, Nodes and More
- Uses Label Selectors to select / query objects
    - Returns collection of objects that satisfy the search conditions
- Enables performing operations on collections of resources
- Influence internal operation of Kubernetes by allowing to schedule specific Pods to specific Nodes (using NodeSelector)
- Objects can have more than one label per resource

## Using Labels

- Resources can be created with Labels interactively (`kubectl`) or declaratively (in a Manifest in YAML/JSON)
- Resources' Labels can also be deleted or edited by assiging (adding) a new Label, or overwriting an existing one
    - `kubectl label pod nginx tier=PROD app=v1` to assign some new labels
    - `kubectl label pod nginx tier=DEBUG app=v1 --overwrite` to edit the existing one(s)
    - `kubectl label pod nginx app-` would remove the `app=v1` label 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
    labels:                   # This is where Labels are declared
      app: v1                 # Key/value pair #1
      tier: PROD              # Key/value pair #2
spec:
...

```

## Querying using Labels and Selectors

| Command | Verb | Resource | Parameter | Effect
| --- | --- | --- | --- | --- |
| `kubectl` | `get` | `pods` | `--show-labels` | Lists all Labels associated with the selected Pods |
| `kubectl` | `get` | `pods` | `--selector tier=prod` | Queries for all Pods matching the `tier=PROD` Selector |
| `kubectl` | `get` | `pods` | `-l 'tier in (prod,qa)'` | Outputs Pods where the Selector key matches `tier` and value matches `prod` or `qa` |
| `kubectl` | `get` | `pods` | `-l 'tier notin (prod,qa)'` | Outputs all Pods, but those where the Selector key matches `tier` and value matches `prod` or `qa` |

## How Kubernetes uses Labels and Selectors

- Controllers and Services matches to Pods using Selectors
    - How Kubernetes knows if a subset of Pods belongs to a specific Deployment, or Service
    - Services can route workload to a Pod with a matching Label
- Influences Pods scheduling into specific Nodes (Pods that need SSD can be scheduled to Nodes with a SSD `nodeSelector`, for instance)

### Services

- A service usually acts as frontend / load balancer for Pods within a cluster
  - Pods that will receive workload must have matching labels with the Service's Selector
    - If a Pod has its label removed, it'll be decommissioned as endpoint and will no longer receive workloads

### Controller Operations (Deployments)

- Deployments can instantiate objects (like a ReplicaSet) with a specific label
  - Pods scheduled by this ReplicaSet will have this label associated to it
  - The ReplicaSet's has a pattern label, `pod-template-hash`, included as suffix of its name. This pattern is passed to Pods scheduled by the ReplicaSet as label 
    - For instance, a ReplicaSet imperatively created as `hello-world` will actually be called something like `hello-world-5646fcc96b` and will carry and pass the `pod-template-hash=5646fcc96b` label to the Pods scheduled by it
  - If this label is changed on the Deployment level (`version=1` to `version=2`, for instance), a new ReplicaSet with this new label will be instantiated and new Pods will be scheduled on the new ReplicaSet, with the new label
    - If a Pod loses the matching label, it'll be removed from the ReplicaSet but will still be running. A new Pod will be scheduled by the ReplicaSet to replace the edited, un-matching label Pod
- Further info: [Maintaining applications with Deployments](../../Managing%20Kubernetes%20Controllers%20and%20Deployments/02maintainingApplicationsDeployments/01updatingDeploymentCheckingRolloutStatus.md)

## Declaring Deployments and Services with Labels 

#### Deployment
```
kind: Deployment
...
spec:
  selector:
      matchLabels:
        run: hello-world            # Declaration of key/value pair that defines the members of this Deployment
...
  template:                         # Declaration of Pod template
    metadata:
      labels:                       # Declaration of Labels for the Pods created for this template
        run: hello-world            # Must match with selector.matchLabels to associate this Pod template to the Deployment
    spec:
        containers:
...
```

#### Service
```
kind: Service
...
spec:
  selector:                         # Declaration of key/value pair that defines the members of the Service
    run: hello-world                # Must match to Pod template's key/value pair to associate with it
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
...
```

###### Return to [Summary](README.md)