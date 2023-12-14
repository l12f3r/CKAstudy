# Troubleshooting Kubernetes Clusters

## Troubleshooting tools
- `kubectl logs`: returns logs from applications to investigate issues inside of a container or Pod (like an application crash)
    - Helps troubleshooting both user and Control Plane 🧠 Pods
    - Container runtime's log tools (like `crictl logs`) and looking at the log files in `/var/log/containers` can also be used
- `kubectl events`: used when troubleshooting workloads deployed in the cluster, or the cluster state itself (like node readiness)
- `systemctl`: used to control and manage systemd units/services - node-level issues
    - Commonly used when working with the kubelet or container runtime to start services, stop them, get critical information about, or ensure readiness upon starting on system boot
    - `systemctl status kubelet.service` returns the status of the systemd unit/service
    - `systemctl enable kubelet.service` sets the kubelet to start on system boot
    - `systemctl start kubelet.service` starts a stopped kubelet; `systemctl stop kubelet.service` stops it
    - The systemd unit configuration file is located at `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
        - Determines the location of the kubelet's binary, its kubeconfig files for authentication and the location of the kubelet `ComponentConfig` manifest (used to define the core configuration of the kubelet itself): `/var/lib/kubelet/config.yaml`
- `journalctl`: access logging information from systemd-managed units/services (like the kubelet) - node-level issues
- System logs (like the kernel log, dmesg or `/var/log/messages`)

## Troubleshooting nodes
1. Confirm that the node is online (is the server up and running?)
2. Check network reachability (is the node reachable over the network? Is there any firewall rules blocking communication from/to server?)
3. Check systemd (is it starting up the container runtime and the kubelet? Are both set to start up on system boot? Are they properly configured?)
4. Look for the kube-proxy Pod (is there one running on the node, as DaemonSet on the kube-system namespace?)
- **Key concept: is the kubelet up, running and able to reach the API Server?**

## Troubleshooting Control Plane 🧠
1. Confirm that the Pod is online (is the API Server up and running?)
2. Check network reachability (are the `--namespace kube-system` Pods reachable over the network? Is there any firewall rules blocking communication from/to server?)
3. Check systemd (is it starting up the container runtime and the kubelet? Are both set to start up on system boot? Are they properly configured?)
    - `crictl --runtime-endpoint unix:///run/containerd/-containerd.sock ps`
4. Are static Pod manifests on their directory (`/etc/kubernetes/manifests` by default; configurable on the `/var/lib/kubelet/config.yaml` file at `staticPodPath`) on the Control Plane 🧠 node?
    - There is a static Pod manifest for each Control Plane 🧠 Pod resource (API Server, etcd, Scheduler, Controller Manager)
4. Look for the kube-proxy Pod (is there one running on the node, as DaemonSet on the kube-system namespace?)
- **Key concept: are the manifests for Control Plane's 🧠 static Pods accessible to the kubelet, in the defined location from its configuration? Are those manifests correctly configured, not containing any errors (like pointing to a wrong image)?**

###### Return to [Summary](README.md)