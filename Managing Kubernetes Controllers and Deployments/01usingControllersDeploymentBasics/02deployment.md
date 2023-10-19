# Using Controllers to deploy applications and Deployment basics

## Deployment controller
- Provides declarative updates to [ReplicaSets](03replicaSet.md) (and Pods it creates)
- Provides the orchestration to Pod creation/deletion
- Used to **manage the state of the application** over time (by rolling out/back versions or scaling up/down the application)
    - Desired state is defined, Deployment controller follows it

### Managing application state with Deployments
- Primary operations:
    - Creating
    - Updating
    - Scaling
#deve ser linkado a Maintaining Applications with Deployments

### Creating Deployments
- Declaratively: YAML code, desired state
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:                               # specs for the Deployment
  replicas: 5                       # number of replicas of the Pod that must be provisioned
  selector:                         # determines which Pods belong to this deployment
    matchLabels:                    # matchExpressions can also be used
      app: hello-world              # must be matched to spec.template.metadata.labels
  template:                         # technical details
    metadata:
      labels:
        app: hello-world            # must match spec.selector.matchLabels
    spec:                           # specs for the content created by the Deployment
      containers:
...
```
- Imperatively: CLI
    - `kubectl create deployment hello-world --image=gcr.io/google-samples/hello-app:1.0`: creates a Deployment with exactly one replica, running the hello-app container image
    - `kubectl scale deployment hello-world --replicas=5`: scales the application be adding four more replicas to the Deployment

### From a cluster level... (operations)
- When a Deployment is executed, Kubernetes creates a ReplicaSet, which is responsible for instantiating the number of replicas defined on code 
- A Service is used to access the content provisioned by the Deployment

###### Return to [Summary](README.md)