# Deploying and Maintaining Applications with DaemonSets and Jobs

## StatefulSets
- Enable stateful applications to be managed by a controller
- Example workloads:
    - Databases
    - Caching servers
    - Application state for web farms

### Capabilities
- **Persistent naming**
    - Unique, persistent naming identifications for Pods running on the StatefulSet
    - Valuable for databases: they need to know precisely where data is located
- **Persistent storage**
    - Data must be stored on a known location that can be accessed when necessary
- **Headless Service**
    - Services without load balancers or even clusterIPs, that use clusterDNS to locate each other by name

###### Return to [Summary](README.md)