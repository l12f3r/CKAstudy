# Exploring the Kubernetes Architecture

## What is Kubernetes (technically speaking)?
- A container orchestrator 
    - Creates groups of interconnected containers ([Pods](03APIObjectsPods.MD)) that, together, can be the base of an application
- Workload placement
    - Defines how the application is deployed (in which server it should phisically live on, and if it's necessary to keep everything inside the same cluster or not)
- Infrastructure abstraction
    - Handles how infrastructure should be dealt (no need to raise concern over provisioning load balancers to define traffic flow, or defining in which cloud server the cluster should live on etc)
- Desired state
    - Ensures that the system attends a specific state defined in code (how many pods should be available for traffic, how to bounce back if a pod is down etc)


## Why Kubernetes?
- Speed of deployment: get code from dev's computer to production faster
- Ability to absorb change quickly: iterate and get new versions of code quickly
- Ability to recover quickly: if not on the desired state, K8s ensure to recover to it i.e. by recovering a crashed/failed server
- Hide complexity on the cluster: infrastructure (network, storage etc) is not exposed, allowing developers to focus on other stuff

## K8s principles
- Desired state / declarative configuration 🧾✔️
    - Where application is defined (what we want to deploy)
    - Person writes code, Kubernetes follows code to meet the desired state
- [Controllers](04APIObjectsControllers.MD) / control loops 🔄
    - Monitor the running state of system to ensure it's within the desired state, making changes when desired state is not met (pulling and starting the amount of container images specified on code, allocating load balancers and public IPs etc)
- [Kubernetes API / The API server](02kubernetesAPI.MD) ↔
    - Central communication hub for information in a cluster
    - Where interaction with the cluster occurs

###### Return to [Summary](01exploringKubernetesArchitecture)