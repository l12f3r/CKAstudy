# Exploring the Kubernetes Architecture

## Controllers
- Keeps the desired state for the cluster (and apps running in it) if disaster occurs 
    - Monitors and responds to [Pod](03APIObjectsPods.MD)'s:
        - state (if [Pod](03APIObjectsPods.MD) is up and running) and 
        - health (if containers within [Pod](03APIObjectsPods.MD) are up and running)
    - Schedules the creation and management of [Pods](03APIObjectsPods.MD)

### Types of Controllers:
- **ReplicaSet**: Defines a number of replicas of [Pods](03APIObjectsPods.MD) that needs to be up and running at all times
- **Deployments**: Manages rollout of ReplicaSets; 
    - Also manage the transition of two versions of ReplicaSets (i.e. moving between two versions of an app,), rolling back to a previous version if needed

###### Return to [Summary](01exploringKubernetesArchitecture)