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
- Provides stability for existing implementations
- Enables backward compatibility and forward change
- As an API version develops, it moves between different phases of the Kubernetes development process
    - Alpha -> Beta -> Stable

###### Return to [Summary](README.md)