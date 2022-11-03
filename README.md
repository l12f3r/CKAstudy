# CKAstudy

# Exploring the Kubernetes Architecture

## What is Kubernetes (technically speaking)?
- A container orchestrator 
    - Creates groups of interconnected containers (pods) that, together, can be the base of an application
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