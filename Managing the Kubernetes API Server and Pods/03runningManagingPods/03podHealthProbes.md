# Running and managing Pods

## Pod health and Probes

- A Pod is considered `Ready` when all containers are up and running
- **Container Probes** can be used to increment an additional intelligence to Pod's state and health
    - Allows to know when the application on the Pod is up and responding properly before changing its state to `Ready`

## Container Probe types

- `livenessProbe`: continuously runs a diagnostic check on a container
    - A check is defined for each container, even on [multi-container Pods](https://github.com/l12f3r/CKAstudy/blob/main/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/01understandingPods.md#ways-how-pods-manage-containers)
    - On failure, the kubelet restarts the container according to its [`restartPolicy`](https://github.com/l12f3r/CKAstudy/blob/main/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/02podLifecycle.md#container-restart-policy)
```
spec:
  containers:
  ...
  livenessProbe:                # this is where the livenessProbe is defined
    tcpSocket:
      port: 8080
    initialDelaySeconds: 15
    periodSeconds: 20
```

- `readinessProbe`: runs a diagnostic check on the container to determine if the Pod is `Ready` to receive traffic from Services
    - Also helps deployment rollouts to ensure a set number of Pods online and running while rolling out new versions of Pods
    - A check is defined for each container, even on [multi-container Pods](https://github.com/l12f3r/CKAstudy/blob/main/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/01understandingPods.md#ways-how-pods-manage-containers)
    - On Pod startup, the application does not receive traffic from a Service until the probe reports success
    - On failure, the endpoint controller removes the Pod's IP from receiving load balanced traffic from the Service
        - Useful for applications that temporarily cannot respond to a request, and it also prevents users from seeing errors (the load balancer will send traffic somewhere else)
```
spec:
  containers:
  ...
  readinessProbe:                # this is where the readinessProbe is defined
    tcpSocket:
      port: 8080
    initialDelaySeconds: 15
    periodSeconds: 20
```

- `startupProbe`: runs a diagnostic check on containers upon Pod startup to ensure all containers are `Ready`
    - A check is defined for each container, even on [multi-container Pods](https://github.com/l12f3r/CKAstudy/blob/main/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/01understandingPods.md#ways-how-pods-manage-containers)
    - On Pod startup, all other probes are disabled until the probe reports success
    - On failure, the kubelet restarts the container according to its [`restartPolicy`](https://github.com/l12f3r/CKAstudy/blob/main/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/02podLifecycle.md#container-restart-policy)
        - Good for applications with long startup times
```
spec:
  containers:
  ...
  startupProbe:                # this is where the startupProbe is defined
    tcpSocket:
      port: 8080
    initialDelaySeconds: 15
    periodSeconds: 20
```

### Types of diagnostic checks for probes

- `exec`: measures a process exit code from an arbitrary process executed inside the application
    - A command is executed inside a container and its exit code will determine if the probe is successful or not
- `tcpSocket`: tests if a TCP socket can be successfully open on a container
- `httpGet`: runs a check against the container's URL (`GET`) and approves the probe if the return code is greater/equal than 200 and lower than 400

### Potential outcomes for probes

- **Success**: container passed the check
- **Failure**: container did not passed the check
- **Unknown**: the probe itself failed; no action is taken

## Ways to configure Container Probes

- `initialDelaySeconds`: number of seconds elapsed between the container start and before running probes (default: `0`)
- `periodSeconds`: how frequently the probe will run against a particular container (default: `10`)
- `timeoutSeconds`: how long the probe waits before declaring failure (default: `1`)
- `failureThreshold`: number of missed probes before declaring failure (default: `3`)
- `successThreshold`: number of probes to be considered successful and live (default: `1`)

###### Return to [Summary](README.md)