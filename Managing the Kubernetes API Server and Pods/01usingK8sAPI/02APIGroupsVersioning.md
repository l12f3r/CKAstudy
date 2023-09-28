# Using the Kubernetes API

## API Groups
- Enable better organization of resources
- Two high-level organization methods:
    - **Core (Legacy) API Group**: used to build the most fundamental workloads 
        - Pod, Node, Namespace, PersistentVolume, PersistentVolumeClaim
    - **Named API Group**:
        - apps (Deployment), storage.k8s.io (StorageClass), rbac.authorization.k8s.io (Role, ClusterRole)

## API Versioning
- Versioned at the API level (`apiVersion: apps/v1`)

###### Return to [Summary](README.md)