# Running and managing Pods

## Understanding Pods

- Pod is a wrapper around the container-based application - an abstraction deployed into the Kubernetes cluster
    - In a nutshell, a container is a process that thinks that is an isolated computer, running somewhere on the cluster. The Pod is a easier way to manage containers on a higher level abstraction, where resources like networking and storage are better managed
- One container per Pod commonly, but many application scenarios have multiple containers per Pod
- To run workloads on the container-based application, some resources are consumed from (or shared with) the Node running the Pod
    - Storage, networking, application configurations (like environment variables)
- Unit of scheduling - Scheduler must figure out where to allocate Pods within Nodes on the cluster, based on available resources

## Ways how Pods manage containers

- **Single container Pods**: most commom deployment pattern
    - Single process running on a container
    - Leads to easier scaling due to minimizing dependencies between applications
- **Multi-container Pod**: for scenarios with tightly-coupled applications with producer/consumer relationship
    - Processes are scheduled together onto the same node
    - Usually, there is a shared resource between those containers, that's why they're on the same Pod
        - Usually, some container process generates data, while another container processes consumes such data
    - Should not be used to influence scheduling (there's [Labels, Namespaces and Annotations](../02managingObjectsLabelsAnnotationsNamespaces/README.md) for that)
    - Web and database servers should not live on the same Pod
        - When it comes to recovery options, the ephemerity of the stateless web server wouldn't match with the stateful database - if the Pod is killed, the database state would not be kept
        - Also limits scalability - stateful and stateless applications have different scaling patterns
- **Init containers**: containers that run before the main application container is started on a Pod
    - Necessary for setting something (like tools and utilities) up the for application container before it starts
    - Main execution is started only after init containers have been run to completion
        - Execution sequence follows the specification on the controller definition (`pod.spec`)
        - If a failure occurs, the container [`restartPolicy`](https://github.com/l12f3r/CKAstudy/blob/main/Managing%20the%20Kubernetes%20API%20Server%20and%20Pods/03runningManagingPods/02podLifecycle.md#container-restart-policy) is applied. By default, it will stop the current execution and hinder the execution of remaining init containers
    - Allows separation of duties - the main application can work under minimal privilege levels, letting init containers to handle operations at higher standards
    - Can also be used to block container startup

### Networking and storage between containers inside a Pod

- Since that containers in a Pod share the same Linux network namespace (not to be related to the [Kubernetes subdividing functionality](../02managingObjectsLabelsAnnotationsNamespaces/01workingWithNamespaces.md)), communication is held over localhost using a loopback interface
    - Be mindful of application port conflicts - make sure to have unique network ports configured
- On a storage perspective, each container within a Multi-container Pod has its own file system
    - Volumes are defined at the Pod level, that is, a volume is shared amongst containers on a Pod
        - Volumes can be mounted into the container's file system

## Controllers and Pods

- Controllers keep apps in desired state
    - Think of a thermostat: the room temperature is the current state, and the ideal temperature is set by the desired state manifest. The thermostat acts to bring the current state closer to the desired
- Used for application scaling and recovery
    - Responsible for starting, increasing/reducing the amount of and stopping Pods as defined on YAML manifest
    - Also bounces back and generates new Pods when identified that the desired state is not met after a Pod crash
- Bare/naked Pods (that is, Pods generated without a Controller) are not recommended since that they're not recreated when failure occurs
- Main controllers: ReplicaSet, Deployment, Daemonset, Statefulset, Job, Cronjob

## Static Pods

- Managed by kubelet (not by the API Server) on specific Nodes
    - The kubelet creates a "mirror Pod" for each Static Pod, allowing to read/get Static Pods from the API Server
- Can be used to start any Pod (not just Control Plane 🧠 Pods )
- Created by kubeadm bootstrapping process, using Static Pod YAML manifests
    - Must be saved on a location watched by the kubelet - the `staticPodPath` (by default, is located under `/etc/kubernetes/manifests`)
        - `staticPodPath` is configurable by editing the kubelet's configuration: `/var/lib/kubelet/config.yaml`
        - If a Node is rebooted, any manifests on the `staticPodPath` will be processed when the kubelet starts up
        - If a change to the Pod is defined on a manifest, the kubelet restarts the Pod with the change
            - If the Static Pod manifest is removed from the `staticPodPath`, the Pod is terminated and deleted
- Already mentioned on [Static Pod manifests](../../Kubernetes%20Installation%20and%20Configuration%20Fundamentals/02installingConfiguringK8s/04bootstrappingClusterKubeadm.md)


###### Return to [Summary](README.md)