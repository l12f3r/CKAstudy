# Running and managing Pods

## Pod lifecycle

- Transitions through three phases: 
    - **Creation**: Pods are created in two different ways:
        - Administratively, at the CLI (using kubectl to provision), or
        - By a Controller (ReplicaSet etc)
    - **Running**: living state of a Pod after its creation
        - This is where Pods are scheduled into Nodes 
    - **Termination**: eventually, this is how the Pod ceases to exist. Can be terminated due to:
        - Process termination/crash
        - Pod deletion (by Controller or administratively)
        - Eviction by a Node that detected lack of resources
        - Node failures or maintenance

## How Pods are stopped/terminated

1. User/Controller sends command to delete a Pod
2. API Server is updated with a 30-second-default grace period timer, to ensure a graceful shutdown
    - If processes on containers are still up when the grace period is expired, those are killed with `SIGKILL`
    - To set a grace period timer imperatively: `kubectl delete pod <name> --grace-period=<seconds>`
        - To set declaratively, just define a numeric value to the `spec.terminationGracePeriodSeconds` key
3. Pod status is changed to `Terminating`
4. kubelet on the current Node sends `SIGTERM` to processes in containers, so they begin terminating
5. Services and Controllers have the Pod removed from their endpoint lists
6. API and etcd are updated with the Pod's removal

- When unable to terminate gracefully, a **force deletion** is required
    - Immediately deletes records in API Server and etcd
    - Non-terminating processes needs to be cleaned after forced deletion
    - Useful when a Pod name needs to be reused to get an application back online
    - `kubectl delete pod <name> --grace-period=0 --force`

## Persistency of Pods

- No Pod is redeployed - they can be recreated on other resources in the cluster if necessary
- If a Pod has stopped, a new one is created in its place based on its Controller
    - That is, no state is transitioned between the previous execution and the new one
- Upon recreation, Pods return to the original container image declared

- To have the ability of persisting data between stopped and recreated Pods, state and configuration are decoupled from the Pod and managed externally
    - Secrets and ConfigMaps are stored on the cluster
    - Environment variables can also be passed to containers
    - PersistentVolumes can live outside of the Pod and be mounted to Pod's file system

## Container restart policy

- Additional layer of application resiliency
- A container in a Pod can restart independent of the Pod
    - This is applied to containers inside a Pod and defined on Pod's `spec`
- During container restarts, Pods cannot be rescheduled to another Node - Container restart is initiated by the kubelet on the Node where the Pod resides
- On multiple failures, Kubernetes protects itself and applications with exponential backoff (from 10, 20, 40 seconds, capping at 5 minutes and resetting to 0 after 10 minutes running successfully) restart
- Three different configurations:
    - `Always` (default): restarts all containers on a Pod if they stop running
    - `OnFailure`: restarts containers on non-graceful termination
    - `Never`: disables container restarts

```
apiVersion: 1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
  restartPolicy: OnFailure              # Defined at Pod level (not container level)
```

###### Return to [Summary](README.md)