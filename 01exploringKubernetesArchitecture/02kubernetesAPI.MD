# Exploring the Kubernetes Architecture

## Kubernetes API
- A RESTful API running over HTTP/S using JSON
- Sole way of interacting (either declaratively or imperatively) with the cluster
- Requests are serialized and persisted into the cluster data store

## Key API objects
- **Pod**: collection of containers deployed as a single unit
- **Controllers**: responsible for keeping the desired state
- **Services**: persistent access point to applications deployed in pods
    - Think of route tables with DNS
- **Storage**: persistent data storage