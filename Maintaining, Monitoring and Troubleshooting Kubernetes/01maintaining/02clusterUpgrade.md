# Maintaining Kubernetes Clusters

## Cluster upgrade 

### Process overview 
- To upgrade an existing kubeadm-based cluster:
    1. Update the Control Plane 🧠 node
    2. If there are more than one Control Plane 🧠 node, all the others must be upgraded 
    3. Upgrade each Worker 👩‍🏭 node in the cluster
- kubeadm-based clusters running Control Plane 🧠 Pods as static Pods:
    - Can only upgrade between minor versions (from 1.17 to 1.18, for instance; version by version, no skips)
    - Read the release notes for the version being installed, checking the change log

### Control Plane 🧠
1. Log into the Control Plane 🧠 node and **update kubeadm package** to the desired version using the package manager (apt, yum etc)
    - `sudo apt-mark unhold kubeadm` to unlock the kubeadm package for updates
    - `sudo apt-get update` to update the package's metadata
    - `sudo apt-cache policy kubeadm` to get the available package versions of kubeadm
    - `sudo apt-get install -y kubeadm=$TARGET_VERSION`
    - `sudo apt-mark hold kubeadm` to lock (hold) the package once again
2. **[Drain](/Configuring%20and%20Managing%20Kubernetes%20Storage%20and%20Scheduling/03managingControllingScheduler/02controllingScheduling.md#node-cordoning) the node** of any non-Control Plane 🧠 Pods
    - Those Pods must stay online on the node
    - `kubectl drain c1-cp1 --ignore-daemonsets`
3. **Run `kubeadm upgrade plan`** for some pre-flight checks, ensuring the proper upgrade
4. **Run `kubeadm upgrade apply v$TARGET_VERSION`** to apply the planned upgrade:
    - Pre-flight checks are once again executed;
    - Pre-pulls container images to reduce Control Plane 🧠  Pods downtime during upgrade;
    - Updates authentication certificates for each Pod;
    - Creates new static Pod manifests in `/etc/kubernetes/manifests`, saving the previous ones to `/etc/kubernetes/temp`;
    - New manifests are read by the kubelet, that restarts Control Plane 🧠 Pods on the upgraded container images
5. **[Uncordon](/Configuring%20and%20Managing%20Kubernetes%20Storage%20and%20Scheduling/03managingControllingScheduler/02controllingScheduling.md#node-cordoning) the node**
    - `kubectl uncordon c1-cp1`
6. **Upgrade kubelet and kubectl** to the desired version using the package manager (apt, yum etc)
    - `sudo apt-mark unhold kubelet kubectl`
    - `sudo apt-get update`
    - `sudo apt-cache policy kubelet kubectl`
    - `sudo apt-get install -y kubelet=$TARGET_VERSION kubectl=$TARGET_VERSION`
    - `sudo apt-mark hold kubelet kubectl`
7. Repeat the process on any additional Control Plane 🧠 node using `kubeadm upgrade node`

### Worker 👩‍🏭 nodes
1. **Drain the node**, gracefully terminating all Pods running on it and marking it unschedulable
    - Make sure to have other nodes available, to support the workload
    - `kubectl drain c1-wnode1 --ignore-daemonsets`
2. Log into the Worker 👩‍🏭 node and **update kubeadm** to the same version as the upgraded Control Plane 🧠 node using the package manager
    - `sudo apt-mark unhold kubeadm` to unlock the kubeadm package for updates
    - `sudo apt-get update` to update the package's metadata
    - `sudo apt-get install -y kubeadm=$TARGET_VERSION`
    - `sudo apt-mark hold kubeadm` to lock (hold) the package once again
3. **Run `kubeadm upgrade node`** to perform the upgrade
    - Updates the kubelet's configuration for the node
    - Retrieves the updated cluster configuration
4. **Update kubelet and kubectl** to the same version as the upgraded Control Plane 🧠 node using the package manager
    - `sudo apt-mark unhold kubelet kubectl`
    - `sudo apt-get update`
    - `sudo apt-get install -y kubelet=$TARGET_VERSION kubectl=$TARGET_VERSION`
    - `sudo apt-mark hold kubelet kubectl`
5. **Uncordon the node**
    - `kubectl uncordon c1-wnode1`

###### Return to [Summary](README.md)