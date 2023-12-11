# Maintaining Kubernetes Clusters

## High Availability cluster overview
- Applications must be online at all times by ensuring the availability of the following resources:
    - API Server
        - Stateless; multiple API Servers can run across many nodes, load balancing techniques can be applied to distribute workload
    - Cluster data store (etcd)
        - Several etcd nodes can be clustered together (**etcd cluster**) 
        - Own internal elections for replication and leader (quorum)
            - etcd requires at least two online nodes, to establish consensus; no consensus, no etcd
    - Control Plane 🧠 nodes
        - Multiple nodes can be executed to protect such services
- Infrastructure must also be redundant and fault-tolerant: use multiple availability zones or regions (on a cloud scenario)

### Topology - stacked etcd cluster with three Control Plane 🧠 nodes
- One node 1️⃣ works with its API Server pointing to the local etcd Pod
    - Remaining Pods (Controller Manager, Scheduler) work normally
- A second node 2️⃣ works with its API Server pointing to the local etcd Pod as well, but this etcd handles replicating its data between the multiple etcds Pods from other nodes
    - Remaining Pods (Controller Manager, Scheduler) work on a standby mode: they use a lease mechanism to ensure only one Scheduler and Controller Manager are running on the cluster and become online if something breaks
- The third node 3️⃣ works similarly to the second, backing it up
- A load balancer is set outside of the cluster to manage it, absorbing operations from kubectl and Worker 👩‍🏭 nodes' API calls
- The load balancer distributes workload to the underlying API Servers

### Topology - external etcd with three Control Plane 🧠 nodes
- etcd is separated from the Control Plane 🧠 node - it runs in an external etcd cluster, with at least three nodes (for quorum and consensus rules)
- The API Server on one 1️⃣ of the Control Plane 🧠 nodes point to the etcd cluster; remaining Pods work as usual
- The next nodes (2️⃣ and 3️⃣) will also have an API Server pointing to the etcd cluster, with standby Scheduler and Controller Manager
- To access the API services, a load balancer is used

###### Return to [Summary](README.md)