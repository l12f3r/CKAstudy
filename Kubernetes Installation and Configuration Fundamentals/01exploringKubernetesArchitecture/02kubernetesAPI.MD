# Exploring the Kubernetes Architecture

## Kubernetes API
- A RESTful API running over HTTP/S using JSON
- Sole way of interacting (either declaratively or imperatively) with the cluster
- Requests are serialized and persisted into the cluster data store
- All configurations changes pass through it
- Validates API operations and persists it on etcd

## Key API objects
- **[Pods](03APIObjectsPods.MD)**: collection of containers deployed as a single unit
- **[Controllers](04APIObjectsControllers.MD)**: responsible for keeping the desired state
- **[Services](05APIObjectsServices.md)**: persistent access point to applications deployed in pods
    - Think of route tables with DNS
- **[Storage](06APIObjectsStorage.md)**: persistent data storage

###### Return to [Summary](https://github.com/l12f3r/CKAstudy/tree/main/01exploringKubernetesArchitecture#readme)