# Maintaining applications with Deployments

## Using Deployments to change state

### Successfully controlling Deployment rollouts
1. Define an **Update Strategy** on the Deployment `spec`
```
apiVersion: apps/v1
kind: Deployment
...
spec:                               #spec for the Deployment
  replicas: 20                      #number of replicas for the example
  strategy:                         #where the Update Strategy must be set
    type: RollingUpdate
    rollingUpdate:                  #specific rollingUpdate settings
      maxUnavailable: 20%           #value can be either integer or percent
      maxSurge: 5                   #value can be either integer or percent
```

2. Define a [readinessProbe](../../Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/03podHealthProbes.md#container-probe-types) on the Pod template `spec`
    - If no readiness probe is set, the Pod is considered ready upon starting, continuing an ongoing rollout of a nonfunction application, for example
    - A rollout will be stopped if the number of Pods not passing readiness probes goes below the `maxUnavailable` value
```
template:                           #template for Pod creation
...
    spec:                           #spec for how Pods must be created (the Pod template)
      containers:
...
        readinessProbe:             #must be set amongst others on the template.spec.containers umbrella
          httpGet:
            path: /index.html
            port: 8080
          initialDelaySeconds: 10   #waits 10 seconds before checking the readiness probe
          periodSeconds: 10         #time interval on when the probe should be executed again
```

#### Update Strategy
- Controls how Pods are rolled out
    - `RollingUpdate` (the default strategy)
        - A new `ReplicaSet` starts scaling up, while the old `ReplicaSet` starts scaling down
    - `Recreate`
        - Terminates all Pods in the current `ReplicaSet` before scaling up a new `ReplicaSet`
        - Useful for applications where version diversity is not supported; everything scaled up will be a newer version, no work will be done in two different versions during rollout

##### `RollingUpdate` configuration points
- `maxUnavailable`: controls the rate of rolling out new versions by ensuring that only a certain number of Pods will be unavailable for update during a particular point in time
    - Default: 25% (in a scenario of a Deployment with 20 replicas, only 4 will be updating while the remaining set keeps working)
- `maxSurge`: controls the rate of rolling out new versions by ensuring that only a certain number of Pods are created above the desired number of Pods
    - In a scenario of a Deployment with 20 replicas being rolled out, during the process of scaling up the new `ReplicaSet` and scaling down the older, a few replicas will be scaled over the desired state limit

### Pausing a rollout
- When a rollout is paused, none of its changes are rolled out. The current state of the Deployment is kept until the rollout is resumed
    - Remember: just changing the Deployment would trigger a new version rollout, starting up new `ReplicaSets` with new changes
        - It's ideal to batch all changes together, then resume the rollout
- `kubectl rollout pause deployment my-deployment` to pause, `kubectl rollout resume my-deployment` to resume

### Rollback
- Kubernetes tracks the rollout history (using the [`CHANGE-CAUSE` Annotation Deployment](01updatingDeploymentCheckingRolloutStatus.md#updating-a-deployment))
    - Revision History (of changes made) uses `revisionHistoryLimit` to define the amount of changes saved on history (default: 10) 
- On a rollback, the newer version is scaled down and a previous version is scaled up. To rollback properly, the specific version to rollback to must be set
    - `kubectl rollout history deployment hello-world --revision=1` would display configuration information on `hello-world`'s first version saved
        - To get the list of versions, just remove `--revision=1`
    - `kubectl rollout undo deployment hello-world` rolls back to the immediate previous version of the current one
        - To rollback to a specific version, `kubectl rollout undo deployment hello-world --to-revision=1`

### Restarting a Deployment
- Effectively restarts all the Pods (in a Deployment using the same Pod template spec) - no Pod is recreated, new `ReplicaSet` with the same Pod spec is set
- Uses either `RollingUpdate` or `Recreate` Update Strategies
- `kubectl rollout restart deployment hello-world`

###### Return to [Summary](README.md)