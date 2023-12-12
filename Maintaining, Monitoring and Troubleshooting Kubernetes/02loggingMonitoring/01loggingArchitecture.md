# Logging and Monitoring in Kubernetes Clusters

## Logging architecture

### Containers and Pods
- Eventually write logs to `stdout` and `stderr`, with container runtime and configuration defining where those standard streams go
    - Default logging driver for containerd sends the log stream files to `/var/log/containers`
    - If the container is not available, the log data can still be accessed as files on the node (i.e. `tail /var/log/containers/$CONTAINER_NAME_$CONTAINER_ID`)
- When containers are running on a node, the last two log files are retained on the node 
    - The current running log and the previous running log (if this container was restarted)
- Logs are tied to the Pod's lifecycle; when the Pod is removed from a node, so are its log files (**log aggregation** systems are recommended, to ship logs outside of the cluster and persist them for analysis)
- Upon using `kubectl logs $POD_NAME`, the kubelet on the node handles the request and it reads directly from the container's log files
    - If the `--previous` parameter is passed, logs from the previous container will be displayed
    - Use `kubectl logs $POD_NAME -c $CONTAINER_NAME` to check logs for a specific container
    - Selectors can be used to filter for specific groups of Pods (`kubectl logs --selector app=loggingdemo --all-containers`)
- Logs consume space - it's good to manage the space using **log rotation** tools
- If the API Server is down, log directly into the node and use `crictl --runtime-endpoint unix:///run/containerd/containerd/sock logs $CONTAINER_ID` to access the logs

### Nodes
- Runs two core services: 
    - **kubelet** (responsible for running containers for Pods on the node)
        - On systemd-based systems, the kubelet runs as a systemd unit/service
            - Any log data admitted from the kubelet will be stored in journald (`journalctl -u kubelet.service`)
        - Non-systemd-based systems have their logs stored in `/var/log/kubelet.log`
    - **kube-proxy** (responsible for implementing Services for Pods on the node)
        - Implemented as a Pod - logs are available via `kubectl logs` and in `/var/log/containers` on the node
        - If kube-proxy is not running inside of a Pod, log data is stored in `/var/log/kube-proxy`
- Local OS logs (like `/var/log/message` or the kernel log) can contain useful information

### Control Plane 🧠
- When the Control Plane 🧠 is running as Pods, each of the logs for its elements (such as API Server, Controller Manager, Scheduler, etcd) are available by running `kubectl logs -n kube-system $POD_NAME`
    - It is also possible to log directly into the node and access log data using `crictl logs $CONTAINER_ID` or accessing `/var/log/containers`
- If running as systemd units/services, logs can be obtained on journald
- If not running as systemd, logs will be on `/var/log/kube-apiserver.log`, `/var/log/kube-scheduler.log` and `/var/log/kube-controller-manager.log`

### Events (pod scheduling/eviction, Deployment creation/scaling, node state changes)
- Logs for cluster operations on resources defined in the cluster
- Events log changes in resource state
- Good starting point when troubleshooting
- `kubectl get events` gets cluster-wide events on any resource, regardless of its lifecycle 🪦
- `kubect describe $TYPE $NAME` to get events from a specific resource (Pod, Deployment etc)
    - Available only on existing resources
- Events are retained for one hour - log aggregators are recommended

###### Return to [Summary](README.md)