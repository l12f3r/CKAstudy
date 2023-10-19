# Using Controllers to deploy applications and Deployment basics

## ReplicaSet controller
- Deploys a number of Pods on a cluster and ensures that this number of Pods is online, based on desired state declared
    - If a Pod is no longer available, the ReplicaSet senses the deviation from desired state, removes the faulty Pod from the cluster and requests the API Server for replacement
    - If a Pod has its label removed, it will no longer satisfy the Selector for the ReplicaSet. This Pod will be removed from the ReplicaSet and a new one will be created with the label, to attend the desired state
- Consists of:
    - A **Selector**, determining which Pods are members of this ReplicaSet
        - Allow for more complex representations, such as `matchExpressions` (apart from common `matchLabels`)
            - Usage of operators: `In`, `NotIn`, `Exists` and `DoesNotExist` when looking at label's keys and values for a particular Pod resource
    - A defined Number of replicas to start, and
    - A Pod Template
    - Further info: [Working with labels](../../Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/02managingObjectsLabelsAnnotationsNamespaces/02workingWithLabels.md#controller-operations-deployments)
- Not created directly; first, a [Deployment](02deployment.md) must be created, which then creates a ReplicaSet

### ReplicaSets handling failures
- Pod failures
    - Pod is removed and a new Pod is started on the cluster
- Node failures
    - If it's a **transient failure** (that is, a resource being unreachable/down for a short period of time), Pods on this Node are marked with an error status on the API server (because the Controller Manager cannot reach them to get status)
        - If a Node is back online and Pods within it aren't running, the [container restart policy](../../Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/02podLifecycle.md#container-restart-policy) is applied and the kubelet restarts the containers within those Pods in that Node
    - On a **permanent failure**, the `pod-eviction-timeout` setting (living on the [kube-controller-manager](../01usingControllersDeploymentBasics/01k8sPrincipalsControllerManager.md#controller-manager)) is defaulted to 5 minutes; after the elapsed time, Pods on that Node are marked for termination on the API Server and new Pods will be created to reach the desired state

#### Legacy documentation
- On older applications, there might be a `ReplicationController` API object listed. Different from its replacement `ReplicaSet`, only a single label (key/value pair) is accepted

## ReplicaSet operations
- Create a Deployment (to create a ReplicaSet)
    - Once a Deployment is created, just run `kubectl get replicaset` to see the ReplicaSet properly provisioned
- Isolating a Pod from a ReplicaSet by changing labels (in order to keep a faulty Pod around to gather logs or whatnot)
    - `kubectl label pod image-name-[tab]-[tab] app=DEBUG --overwrite` would change the label of the `image-name-[tab]-[tab]` Pod to `app=DEBUG`, forcing the ReplicaSet to create a new Pod with labels that satisfy its Selector and separating the `app=DEBUG` Pod from the ReplicaSet
    - If this `app=DEBUG` Pod is relabeled to the pattern that satisfies the ReplicaSet's Selector, the latest Pod created is terminated so the number of replicas meet the desired state

###### Return to [Summary](README.md)