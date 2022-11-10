# Exploring the Kubernetes Architecture

## Controllers
- Keeps the desired state for the cluster (and apps running in it) if disaster occurs 
    - Monitors and responds to Pod's:
        - state (if Pod is up and running) and 
        - health (if containers within Pod are up and running)
    - Schedules the creation and management of Pods

### Types of Controllers:
    - **ReplicaSet**: Defines a number of replicas of Pods that needs to be up and running at all times
    - **Deployments**: Manages rollout of ReplicaSets; 
        - Also manage the transition of two versions of ReplicaSets (i.e. moving between two versions of an app,), rolling back to a previous version if needed