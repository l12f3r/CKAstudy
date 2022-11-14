# Exploring the Kubernetes Architecture

## Cluster components

### Control Plane Node (previously known as "Master"): 🧠
    - Coordinates cluster operations, monitoring, Pod scheduling
    - Primary access point to cluster administration; probably where Prometheus gets info from

#### Control Plane Node components
    - **API server**: primary access point for cluster and administrative operations; communication hub
        - All configurations changes pass through it
        - Validates API operations and persist it on etcd
    - **etcd**: stores state (since that the API server is stateless)
        - Persists state of API objects as key/value pairs
    - **Scheduler**: watches the API server for unscheduled Pods, scheduling it on Nodes
        - Evaluates Pod's necessities (CPU, RAM, storage) to ensure availability upon scheduling
        - Respects previously defined contraints (handles Pod affinity and anti-affinity)
    - **Controller Manager**: responsible for looping (implementing Pod lifecycle functions to) Controllers
        - Watches and updates the API Server to ensure the desired state

![image](https://user-images.githubusercontent.com/22382891/201721955-1e431571-ad52-4f54-bcdc-535909d55916.png)

### (Worker) Node: 👩‍🏭
    - Where application Pods run; contributes to cluster's compute capacity
    - Usually multiple on clusters, depending on scalability requirements
    - Starts Pods and containers within;
    - Implement networking;
    - Either virtual or physical machines

#### (Worker) Node components:

### kubectl:
    - CLI used to interact with the API server