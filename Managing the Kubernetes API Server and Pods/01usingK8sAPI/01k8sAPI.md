# Using the Kubernetes API

## Kubernetes API
- Single surface area over resources in a cluster
- Allows to use **API objects** to model a system and build workloads to be deployed on a cluster
    - Collection of primitives to represent the system's state
    - Enables configuration of a desired state for the system to meet

## API Server
- Sole way to interact with the cluster
- Follows a client/server architecture:
    - Client and server exchange JSON objects using a RESTful API over HTTP
        - GET, POST, SET operations, for example
    - Client submits requests over HTTP/S
    - Server responds to the request based on the actions it had to take
- Stateless - data is serialized and persisted on etcd

## Control Plane Node 🧠
- API Server
- etcd
- Controller Manager
- Scheduler
(Further info in [Kubernetes Installation and Configuration Fundamentals](https://github.com/l12f3r/CKAstudy/blob/main/Kubernetes%20Installation%20and%20Configuration%20Fundamentals/01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components))

## API Objects
- Persistent entities that allows building and deploying systems
- Represent the state of the system to be built
- Types of objects:
    - **Kind**: 
        - Pod: group of containers within a node
        - Service: persistent access point and load balancing services to applications
        - Deployment: declaration of how objects should be scheduled or scaled
        - PersistentVolumes: persistent storage
    - **Group**: core, apps, storage
    - **Version**: v1, beta, alpha

## kubectl operations
- [`dry-run`](https://github.com/l12f3r/CKAstudy/blob/main/Kubernetes%20Installation%20and%20Configuration%20Fundamentals/03workingK8sCluster/02applicationPodDeployment.md#example-of-a-basic-deployment-manifest-file)
- `diff`: generates the difference between current state (resources currently running) and desired state (resources defined on a manifest)
    - differences are outputted to stdout
    - helps understanding what will be changed upon execution
    - `kubectl diff -f deployment.yaml`

###### Return to [Summary](README.md)