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
- **Init containers**: containers that run before the main application container is started on a Pod, necessary for setting something up the for application container before it starts

## Controllers and Pods

- Controllers keep apps in desired state
    - Think of a thermostat: the room temperature is the current state, and the ideal temperature is set by the desired state manifest. The termostat acts to bring the current state closer to the desired
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